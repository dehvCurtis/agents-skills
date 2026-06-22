---
name: audit-scanners-apogee
description: Full scanner-matrix reaudit for the Apogee platform — API-driven end-user scans as the standing test account against the last-known-good baseline contracts, with detector-firing verification, double-check before declaring drift, Linear ticket-filing, and complete documentation update. Use when the owner asks to test/reaudit/verify all scanners (SAST + AI).
metadata:
  type: skill
  scope: apogee-platform
---

# Apogee scanner matrix reaudit

You are running a comprehensive verification of every scanner shipping on the Apogee platform (17 SAST + 1 AI). The audit is **API-driven** (real end-user scans as the owner's authorized test account `jasonbrailowbizop@mail.com`), **baseline-compared** (no new fixtures, only the exact `contract_id`s from the last passing audit per `feedback_baseline_fixtures.md`), and **methodical** (run matrix first, only triage what's verified-drifted per `feedback_test_before_fixing.md`).

## Phase 0 — Preconditions

Before running anything:

1. **Session-only password** — the test account password is held in `$TEST_PASSWORD` env var (per `feedback_trigger_scans_via_api.md`). If not set, ask the owner to run `! export TEST_PASSWORD='...'` in their terminal so the literal stays in shell state, not the chat transcript.
2. **Read the last-known-good baseline** — most recent `TaskDocs-BlockSecOps/audit-YYYY-MM-DD-scanner-matrix-reaudit.md` and current `docs/scanners-audit/scanner-status.md`. These give you the `contract_id` → scanner → expected-counts mapping.
3. **Map prefix → full UUID** — baseline docs use 8-char prefixes (`eafe2b12`); the API needs full UUIDs. Query `GET /api/v1/contracts?limit=200` once and filter by prefix to build the mapping.
4. **Verify cluster health up-front** — `kubectl get pods -n tool-integration-prod -n ai-scanner-prod -n api-service-prod` — flag any non-Running pods before scanning.

## Phase 1 — Baseline matrix (17 SAST scans)

One scan per SAST scanner against the exact baseline contract from the last-known-good audit:

| Lang | Baseline contract prefix | Scanners |
|---|---|---|
| Solidity multi-file | `eafe2b12` (foundry-oz) | slither, aderyn, semgrep, solhint, mythril, wake, soliditydefend, halmos, medusa, echidna |
| Vyper single | `8d5bea5f` (E2E-Vyper-Ownable) | vyper |
| Vyper multi | `a355b720` (E2E-Vyper-tokens-tree) | moccasin |
| Rust/Anchor | `86096252` (E2E-Rust-anchor-basic1) | sol-azy, rustdefend, sec3-xray, trident, cargo-fuzz-solana |

Dispatch all 17 in sequence (the system enforces a 20-concurrent-scan cap; queueing is automatic). Poll until all complete or fail.

## Phase 2 — Detector-firing on known-buggy fixtures (5 scans)

Proves detectors actually fire — catches silent-crash scanners that pass baseline-match but have a broken parser:

| Scanner | Fixture prefix | Expected |
|---|---|---|
| halmos | `86205bd5` halmos-buggybank-v6-pure | c=1 |
| echidna | `0d0c1935` hardhat-echidna | h=1 |
| mythril | `70d006e1` TestA-VulnGood-0.8.20 | h=1 m=2 l=2 |
| vyper | `075b2643` audit-vy-single-reentrancy | h=1 l=1 |
| rustdefend | `09db9052` vulnerable_solana_vault | c=1 |

## Phase 3 — AI scanner baseline (1 scan)

`ai-anthropic` against `eafe2b12` (the Solidity project baseline). Establish or re-verify the AI baseline. If dispatch fails, capture the exact error text, log api-service + ai-scanner pod state, and file a ticket — do not retry indefinitely (Supabase auth rate-limit + scanner dispatch rate-limit apply).

## Phase 4 — Tabulate (CRITICAL: use the RIGHT source of truth)

For each scan, `GET /api/v1/scans/{id}` returns the scan record with aggregate counts at the top level: `critical_count`, `high_count`, `medium_count`, `low_count`, `info_count`. **These aggregates are the source of truth for this audit.**

Do NOT rely on `/scans/{id}/vulnerabilities` for the count check — that endpoint is currently broken (returns empty even when counts > 0; tracked as **ADV-17**). Always cross-check both, but tabulate against the aggregates.

Duration: `completed_at - created_at` (the scan record `duration_seconds` may be null; compute it).

## Phase 5 — Triage (the double-check step per `feedback_test_before_fixing.md`)

For every drift case (observed ≠ expected):

1. Re-pull `GET /api/v1/scans/{id}` raw — confirm aggregates vs vulnerabilities endpoint disagree, or vice versa
2. Check the scan's `failure_type` and `error_message` fields
3. Pull tool-integration + ai-scanner logs scoped to the scan's `completed_at` window
4. Re-run the same scan once to see if it reproduces (transient vs persistent)
5. Only declare a scanner broken if the drift is reproducible AND the aggregates disagree with the baseline

If the apparent drift turns out to be a downstream endpoint bug (like ADV-17), **file the downstream bug as its own ticket** — do not blame the scanner.

## Phase 6 — File Linear tickets per verified issue

For every verified broken scanner or downstream bug:

```
mcp__plugin_linear_linear__save_issue({
  team: "83f0063d-fc9c-4e23-a2f1-1a3ade496494",
  project: "Apogee Core Platform",
  title: "<scanner> regression vs 2026-MM-DD baseline" OR "<endpoint> bug surfaced by audit",
  priority: 2,  // P2 High by default; P1 if customer-facing failure mode
  labels: ["bug", "repo:<owning-repo>", "source:audit"],
  state: "Backlog",
  description: full evidence: baseline counts, observed counts, scan_id, contract_id,
                api-service log excerpt, suspected root cause, acceptance criteria
})
```

For owner-surfaced UX gaps (e.g. misleading error messages): same format, `type:improvement`, `source:user-report`.

## Phase 7 — Update documentation

In ONE commit (per `feedback_methodical_not_reactive.md`):

1. **New TaskDocs audit file**: `TaskDocs-BlockSecOps/audit-YYYY-MM-DD-scanner-matrix-reaudit.md` — full evidence per scanner with scan IDs, durations, observed vs baseline
2. **CHANGELOG entry**: prepend to `docs/scanners-audit/CHANGELOG.md` — date header + what changed + why + verification summary + linked Linear tickets
3. **scanner-status.md note**: prepend a `> YYYY-MM-DD verification update:` block at the top, KEEPING the previous update blocks for historical traceability
4. **Per-row updates**: optional — only touch the per-row "Last verified" column if the audit ran a deeper-than-baseline check on that row (otherwise leave the previous "Last verified" cell alone and let the top-of-file update note serve as the audit-cycle reference)

## Phase 8 — Final report to owner

One concise summary message with:
- `X/Y SAST scanners match baseline — Z drift(s)` headline
- AI scanner status (baselined / failed-to-dispatch / blocked-on-tier)
- Number of new Linear tickets filed + their IDs
- Cross-link to the new TaskDocs audit doc

## Tools you will use

- `mcp__plugin_linear_linear__save_issue` for ticket-filing
- `Bash` for `curl` against the API + `kubectl` for cluster health
- `Write` / `Edit` for the documentation updates
- DO NOT use `Agent` for the scanning itself — the matrix is small enough to drive directly and you need the raw responses

## Standards you MUST follow

- `feedback_scanner_audit_via_api.md` — end-user API as `jasonbrailowbizop@mail.com`, NOT the local E2E regression matrix
- `feedback_baseline_fixtures.md` — reuse exact baseline `contract_id`s, never upload new fixtures
- `feedback_test_before_fixing.md` — run matrix FIRST, only fix what's verified-broken
- `feedback_methodical_not_reactive.md` — track everything in `docs/scanners-audit/CHANGELOG.md` + a TaskDocs audit doc
- `feedback_trigger_scans_via_api.md` — password is session-only, use `$TEST_PASSWORD` env var, never persist
- `feedback_no_redundant_canaries.md` — skip canary scans when a real fixture is already queued
- `feedback_no_claude_attribution.md` — no Claude/Anthropic mentions in any commit, PR, ticket, or doc

## Anti-patterns to refuse

- Uploading new throwaway fixtures instead of reusing the baseline `contract_id`s
- Pre-emptively "fixing" a scanner before the matrix has actually run
- Filing a Linear ticket against a scanner when the actual bug is downstream (endpoint, aggregator, parser, etc.) — file against the right owning repo
- Closing a verification cycle without filing tickets for every verified issue
- Skipping the CHANGELOG update because "the audit doc has the info" — both are required for traceability
- Running the matrix without first checking cluster health (the scanner Jobs depend on tool-integration + ai-scanner being up)
- Tabulating against `/scans/{id}/vulnerabilities` instead of the aggregate counts on the scan record (until ADV-17 is closed)
