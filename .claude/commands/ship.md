---
description: Full ship cycle with Linear integration — discover/create Linear tickets per work item, comment at each phase (changelog → security → tests → docs → standards → commit/PR/merge → deploy → verify), close on success. Tickets are tagged with type/repo/severity/source/phase labels and a Priority is REQUIRED on every issue.
---

You are running the Apogee full ship cycle. Execute each phase in order. Do not skip phases. Do not run phases in parallel — complete each one before starting the next.

## Context

Working directory: `/home/pwner/Git`

This skill operates across the multi-repo monorepo layout under `~/Git/`. Each sub-directory that is its own git repo may need its own commit + PR.

**Linear workspace** (single source of truth for ship lifecycle):
- Team: `Advanced Blockchain Security` (id `83f0063d-fc9c-4e23-a2f1-1a3ade496494`, key `ADV`)
- Projects:
  - `Apogee Core Platform` — api-service, ai-scanner, dashboard, intelligence-engine, data-service, notification, orchestration, tool-integration, contract-parser, analysis, findings
  - `Apogee Scanner Library` — SolidityDefend, SolidityBOM, RustDefend, and third-party scanner adapters
  - `Apogee Infrastructure` — gcp-infrastructure, k8s overlays, ExternalSecrets, NetworkPolicies, monitoring
  - `Apogee Documentation` — docs/, TaskDocs-BlockSecOps/, blocksecops-docs/
  - `Apogee Websites` — 0xapogee.com, advancedblockchainsecurity.com
- Label groups (use the lowercase grouped labels, NOT the workspace-default `Bug`/`Feature`/`Improvement`):
  - **type**: `bug` `feature` `improvement` `chore` `refactor` `security` `docs` `test` `infra`
  - **repo**: `repo:api-service` `repo:ai-scanner` `repo:dashboard` `repo:docs` `repo:taskdocs` `repo:shared` `repo:tier-config` `repo:soliditydefend` `repo:soliditybom` `repo:rustdefend` `repo:gcp-infrastructure` `repo:websites` `repo:blocksecops-docs` `repo:intelligence-engine` `repo:tool-integration`
  - **severity** (security only): `sev:critical` `sev:high` `sev:medium` `sev:low` `sev:info`
  - **source**: `source:ship-cycle` `source:user-report` `source:audit` `source:regression` `source:scan-failure`
  - **phase**: `phase:in-progress` `phase:in-review` `phase:merged` `phase:deployed` `phase:verified`

**GitOps rules that apply for the entire run:**
- Never add `Co-Authored-By` trailers or any Claude/Claude Code attribution to commits, PRs, or Linear comments.
- Push and open PRs as the repo owner account — do not inject Claude identity anywhere.
- PR descriptions and Linear comments must be substantive: what changed, why, which standards were verified. No filler.
- Do not mention Claude Code, Claude, or Anthropic in any commit message, PR title, PR body, or Linear ticket.
- After each PR is merged, run `git checkout main && git pull` in that repo before moving on.

---

## Phase 0 — Linear Ticket Discovery / Creation (NEW)

**Every distinct work item shipped in this cycle needs a Linear ticket.** Before doing any other phase work, enumerate the work items and ensure each has a ticket.

### What counts as a work item
- One bug fix = one ticket
- One feature = one ticket (sub-tickets for slices if it's multi-PR)
- One audit finding closure = one ticket
- One pre-existing test failure cluster = one ticket
- One doc cleanup batch = one ticket (multiple files acceptable on one ticket if they're the same intent)

If three files changed to fix one bug, that's ONE ticket. If three independent bugs got fixed in the same PR, that's THREE tickets that all link to the same PR URL.

### Discovery — search before creating
For each work item, FIRST search Linear for an existing matching ticket:

```
mcp__plugin_linear_linear__list_issues({
  query: "<keyword from the work item>",
  team: "83f0063d-fc9c-4e23-a2f1-1a3ade496494",
  limit: 10,
  state: "Todo,In Progress,Backlog"
})
```

If a matching ticket exists, REUSE it. If not, create one.

### Required fields on every ticket

| Field | Source / Rule |
|-------|---------------|
| `title` | One-line imperative — "Fix X" / "Add Y" / "Close BSO-SEC-NNN" |
| `team` | Always `83f0063d-fc9c-4e23-a2f1-1a3ade496494` |
| `project` | Pick from the 5 projects above based on the repo |
| **`priority`** | **REQUIRED on every ticket.** Pick `1` (Urgent — prod down, data loss, CRITICAL audit), `2` (High — HIGH audit, user-reported bug, pre-launch blocker), `3` (Medium — MEDIUM audit, test gap, doc drift), or `4` (Low — LOW/INFO audit, refactor, polish). Default for unsupervised auto-created tickets: `3`. |
| `labels` | At minimum: ONE `type:` label + ONE `repo:` label + `source:ship-cycle` (or other source if applicable). For security work: add the matching `sev:*` label. |
| `description` | Markdown: what + why + acceptance criteria. Cite file:line for code refs, BSO-SEC-NNN for audit findings, the user message that surfaced it for user-reports. |
| `state` | `Todo` if not yet started, `In Progress` once the ship cycle picks it up. |

### Priority decision tree (use, don't guess)
- Audit found CRITICAL or production-down bug → P1 Urgent
- Audit found HIGH, user-reported bug, MVP-launch blocker → P2 High
- Audit found MEDIUM, test gap that blocks /ship, doc drift that confuses customers → P3 Medium
- Audit found LOW/INFO, internal refactor, polish, post-launch follow-up → P4 Low
- Pure docs PR with no functional impact → P4 Low

### Create the ticket

```
mcp__plugin_linear_linear__save_issue({
  title: "...",
  team: "83f0063d-fc9c-4e23-a2f1-1a3ade496494",
  project: "Apogee Core Platform",
  priority: 2,
  labels: ["bug", "repo:api-service", "source:ship-cycle"],
  description: "...",
  state: "In Progress"
})
```

After creation, ADD the `phase:in-progress` label immediately so the ticket reflects its current lifecycle state.

### Emit the ticket map
Before any further phase, print the table of `work item → ADV-NNN` mappings to the operator so they can confirm coverage. Format:

```
| Work item | Linear ticket | Priority | Project |
|---|---|---|---|
| Fix multi-file source loading | ADV-12 | P2 High | Apogee Core Platform |
| ... | ... | ... | ... |
```

---

## Phase 1 — Changelog

Append an entry to `~/Git/CHANGELOG.md`. If the file does not exist, create it with a standard Keep a Changelog header first.

Entry format:
```
## [Unreleased] — YYYY-MM-DD

### Changed / Added / Fixed / Removed
- <one-line summary per meaningful change> (ADV-NNN)
```

Rules:
- Use today's date.
- Group changes under `Changed`, `Added`, `Fixed`, or `Removed` as appropriate. Omit empty groups.
- One bullet per logical change — not per file. Match each bullet to the Linear ticket(s) created in Phase 0.
- **Every bullet ends with the Linear ticket ID** in parens, e.g. `(ADV-12)`. This is the bidirectional link.
- Reference the affected repo(s) in brackets when ambiguous: `[blocksecops_com]`, `[api]`, etc.
- Do not write prose. No "we" or "I". Imperative tense.
- Keep each bullet under 120 characters.

After writing the entry, output it so the user can confirm it looks right before proceeding.

**Linear**: for each ticket, post a comment to the issue with the changelog bullet for traceability:

```
mcp__plugin_linear_linear__save_comment({
  issueId: "ADV-12",
  body: "**Phase 1 — Changelog entry**\n\n`Fixed: AI orchestrator handles multi-file Hardhat/Foundry projects [ai-scanner]`\n\nAppended to ~/Git/CHANGELOG.md under 2026-06-21."
})
```

---

## Phase 2 — Security Audit

Audit all recent changes for security issues. Cover:

1. **Secrets** — grep for hardcoded secrets, tokens, passwords, API keys (including in `~/.claude/skills/` + `**/.claude/agents/*.md` per BSO-SEC-048 precedent). Verify `.env` files are gitignored.
2. **Input validation** — any new user-facing input (forms, API params, request bodies) must be validated before use.
3. **Auth boundaries** — new endpoints/routes must have appropriate auth checks (`require_auth_with_scope()` for writes per `docs/standards/api-endpoint-auth.md`). Gated features must verify tier/role.
4. **XSS / injection** — `dangerouslySetInnerHTML`, template interpolation into SQL or shell, unescaped output, raw SQL without bind params.
5. **Supply chain** — new `npm`/`pip`/`cargo` dependencies should be from known publishers; flag anything unfamiliar.
6. **External URLs** — any new `fetch()` calls must go to known, intentional endpoints. Flag unexpected domains.
7. **Implicit/explicit consent gates** — sub-processor consent server-side enforced (BSO-SEC-031); GDPR audit-log requirements (BSO-SEC-041).
8. **Inter-service auth** — `hmac.compare_digest` on every token compare (BSO-SEC-028 precedent).
9. **Dynamic registry anti-pattern** — flag any hardcoded scanner/model/provider list literal in dashboard or client code.

For each FAIL or WARN: record the file, line, issue, and required fix. Fix all FAILs before proceeding to Phase 3. WARNs may proceed with a note in the changelog and the Linear ticket.

If any NEW finding is discovered (not already on a Phase 0 ticket), CREATE A NEW LINEAR TICKET for it with the appropriate `severity` label + `source:audit` + Priority per the decision tree.

**Linear**: post the audit findings to each affected ticket:

```
**Phase 2 — Security audit**

- PASS: secrets / input validation / auth boundaries / XSS / supply chain / external URLs / consent gates / inter-service auth / no hardcoded lists
- (or) FAIL: <file:line> — <issue> — fixed in <commit-sha>
- (or) WARN: <file:line> — <issue> — tracked as ADV-NN follow-up
```

Also save the full audit report to `~/Git/docs/audit/AUDIT-YYYY-MM-DD-<scope>.md` continuing the BSO-SEC-NNN sequence.

---

## Phase 3 — Test Coverage

For each service or module that changed:

1. Confirm unit tests exist for all new/changed functions. Write missing unit tests following the existing test patterns in that repo.
2. Confirm regression tests exist for any bug fix or behavior change. When a UI control or behavior is REMOVED, convert the existing test to `not.toContain` rather than deleting it.
3. Run whatever local test suite exists (`pnpm test`, `cargo test`, `pytest`, etc.) and report PASS/FAIL.
4. If no test suite exists for a repo, note it explicitly — do not silently skip.
5. Fix any failing tests before proceeding. If a test was wrong (not the code), fix the test and note why.

Do not run tests against production infrastructure — local only (per `feedback_local_not_gcp.md`).

If any new test gap is discovered that is OUT of scope for the current ship, file a Linear follow-up ticket with `type:test` + `repo:<...>` + P3 Medium.

**Linear**: post the test report to each affected ticket:

```
**Phase 3 — Test coverage**

- New tests: <count> in <file>
- Suite result: <X passed / Y failed / Z skipped>
- Regression guard: <what's covered + why>
```

---

## Phase 4 — Documentation Update

**Run AFTER tests pass and BEFORE standards verification.** Documentation must reflect the now-stable, test-validated state of the change.

For every Linear ticket touched in this cycle, walk the documentation surface and update what drifted. Cover (in this order — most-changed-by-this-work first):

1. **Standards (`~/Git/docs/standards/`)** — read the relevant standard files for this change to confirm compliance; if the change ADDS a new pattern that should become a rule, add or amend the standard. Update `docs/standards/INDEX.md` if new standard files are introduced.

2. **Database schema (`~/Git/docs/database/SCHEMA.md`)** — if any Alembic migration ran or any ORM model changed in this cycle, update SCHEMA.md in the SAME PR (Rule 4 of `core-development-rules.md`). Include: new tables, altered columns, new/dropped indexes, FK changes, table count, and the Verified date. Add a row to the migration delta table.

3. **Workflows (`~/Git/docs/workflows/`)** — step-by-step sequence diagrams. Update when the user-visible flow of an existing feature changes (e.g. fire-and-forget dispatch was added, a UI gate was removed/added). New features get a new workflow doc.

4. **Pipelines (`~/Git/docs/pipelines/`)** — service build/deploy pipelines. Update when build args, image tags, Workload Identity bindings, ExternalSecrets, or NetworkPolicy archetypes change. Bump version references to the new deployed version.

5. **Playbooks (`~/Git/docs/playbooks/`)** — operational runbooks. Update when an incident-response command, kill switch, recovery procedure, or admin endpoint changes. Add new playbooks for novel operational scenarios (e.g. quota exhaustion, BYO key rotation, scanner upgrade).

6. **Scanners (`~/Git/docs/scanners/`)** — keep aligned with the `scanner-versions` ConfigMap. Update on any scanner version bump, new scanner addition, scanner removal.

7. **Feature tests / end-user acceptance (`~/Git/docs/feature-tests/`)** — numbered files (latest is `95-byo-ai-scanning.md`; next new file → `96-`). Update existing files when acceptance criteria change. Add a new numbered file for net-new user-facing features. Sync with `TaskDocs-BlockSecOps/phases/` acceptance criteria.

8. **Audit (`~/Git/docs/audit/`)** — Phase 2 already produced the dated audit report. Verify it's present and that any FAIL items have closure references.

9. **General docs (`~/Git/docs/*`)** — architecture, intelligence, getting-started, audits. Fix any drift between code and docs. Verify no orphaned docs (no parent link), no placeholders (TODO/FIXME/TBD), no legacy "BlockSecOps"/"ABS" naming in user-facing copy.

10. **Task documentation (`~/Git/TaskDocs-BlockSecOps/`)** — append a dated entry to `phases/NN-<phase>/IMPLEMENTATION-SUMMARY-YYYY-MM-DD.md` for any user- or operator-visible change. **Append to the existing summary file rather than creating new files per iteration** (canonical pattern: `phases/10-phase-10-byo-ai-scanning/IMPLEMENTATION-SUMMARY-2026-06-20.md` has multiple dated sections). Track PR URLs + live evidence (scan IDs, version numbers).

11. **Public-facing docs (`~/Git/blocksecops-docs/`)** — Docusaurus-style end-user and API reference. **Public-safe content only** — no internal architecture, no GCP project IDs, no test-account credentials.

12. **Per-repo READMEs and `<repo>/docs/`** — bump version numbers, add a dated section for the change.

**Style rules** (enforce on every doc edit):
- Match neighboring-doc structure exactly — do not invent new layouts.
- Mermaid for sequence diagrams + flowcharts (the existing platform uses mermaid).
- No emojis (per owner's explicit feedback).
- No Claude / Claude Code / Anthropic attribution anywhere.
- Use `ai-anthropic` (lowercase, hyphenated) for the scanner ID, "AI (Claude Sonnet)" for display.
- Cross-link related docs.
- Substitute env-var placeholders (`$TEST_PASSWORD`) for any session-secret in example commands; never the literal value (BSO-SEC-048 precedent).

**Delegation** — for any change spanning more than 3 doc files, delegate the sweep to the `apogee-documentation` agent rather than doing it inline. The agent has the full standards index + Phase 10 lessons + dated-summary pattern baked in.

**Trigger condition** — skip this phase ONLY when the change is purely internal (e.g. a refactor with no user / operator / dev-facing surface change). The default is to run this phase. When skipping, post a Linear comment explaining why.

**Linear**: post the doc-update report to each affected ticket:

```
**Phase 4 — Documentation update**

Files modified (one-line reason each):
- docs/scanners/ai-scanner.md — version 0.2.7 → 0.2.8, source-hash dedup section added
- docs/database/SCHEMA.md — migration 097 row in delta table; new columns reflected
- TaskDocs-BlockSecOps/phases/10-.../IMPLEMENTATION-SUMMARY-2026-06-20.md — 2026-06-22 iteration appended
- blocksecops-docs/platform/scanning/ai-scanning-beta.md — cache-hit note in user-facing copy
- blocksecops-ai-scanner/README.md — version 0.2.8

Skipped (with reason):
- docs/playbooks/ai-cost-kill-switch.md — no operational change

Any new doc gaps surfaced and tracked separately as ADV-NN.
```

---

## Phase 5 — Standards Verification

Before committing, verify the following across all changed repos. Fix any non-compliance, then log each check as PASS or FIXED. **Read each standard file directly — do not rely on memory.**

| Check | Standard |
|-------|----------|
| Codebase / core rules (Rule 0/1/3/4) | `docs/standards/core-development-rules.md` |
| Conventional commits + feature branches + no Claude attribution | `docs/standards/version-control-standards.md` |
| Semver bumps (pyproject + kustomization newTag + version label synced) | `docs/standards/docker-image-versioning.md` |
| Base image pinning, OCI labels, --provenance=false | `docs/standards/docker-base-images.md` |
| SCHEMA.md updated in same commit as schema changes | `docs/standards/database-management.md` |
| Always HTTPS for prod, 127.0.0.1 not localhost for dev | `docs/standards/core-development-rules.md` Rule 2 |
| `require_auth_with_scope()` on every write endpoint | `docs/standards/api-endpoint-auth.md` |
| AES-256-GCM for encryption at rest, TLS 1.2+ in transit | `docs/standards/encryption-standards.md` |
| Default-deny + explicit egress allowlist per workload | `docs/standards/networkpolicy-templates.md` |
| Per-tier enforcement on every gated endpoint | `docs/standards/tier-standards.md` |
| Latest stable deps, no prohibited packages | `docs/standards/dependency-management.md` |
| No build scripts, no BuildKit container builders | `feedback_no_build_scripts.md` + `feedback_no_buildkit_builders.md` |
| No hardcoded platform stats / scanner counts | Read `platform-scanners.ts` or live `/scanners` |
| Brand naming: "0xApogee" / "Apogee" / "Advanced Blockchain Security" (no "BlockSecOps" user-facing) | (style) |
| No `.env` files staged | `feedback_no_env_commits.md` |

**Linear**: post the standards report to each affected ticket:

```
**Phase 5 — Standards verification**

| Standard | Status |
|---|---|
| version-control-standards | PASS |
| docker-image-versioning | FIXED (was missing version label) |
| ... | ... |
```

---

## Phase 6 — Commit, PR, and Merge

For **each repo that has uncommitted changes** (check with `git status` in each sub-repo):

1. **Stage** only the files that belong to this repo's changes. Never `git add -A` blindly — check `git status` for accidental `.env` or secret files first.
2. **Commit** with a message in conventional commit format:
   - `type(scope): short description`
   - Body: what changed and why (2–5 sentences). **End the body with the Linear ticket ID** on its own line: `Refs: ADV-12` (or `Closes: ADV-12` if the ticket fully closes with this commit).
   - No Claude/AI attribution. No `Co-Authored-By` trailer.
3. **Push** to a feature branch named `ship/<short-topic>-$(date +%Y%m%d)`.
4. **Open a PR** using `gh pr create` with:
   - Title: concise, under 70 characters
   - Body: what changed, why, which phases passed (changelog ✓, security ✓, tests ✓, docs ✓, standards ✓), Linear ticket links (`ADV-12`, `ADV-13`), any notable decisions
   - No mention of Claude Code, Claude, or Anthropic anywhere in the PR
5. **Linear update** on PR open — REPLACE `phase:in-progress` with `phase:in-review` on each ticket, post the PR URL as a comment:
   ```
   mcp__plugin_linear_linear__save_issue({
     id: "ADV-12",
     labels: [<keep existing type/repo/source/severity>, "phase:in-review"]
   })
   mcp__plugin_linear_linear__save_comment({
     issueId: "ADV-12",
     body: "**Phase 6 — PR opened**\n\n<PR URL>\n\nCommits: <sha-short> on branch `ship/...`. Awaiting verification before merge per feedback_test_before_merge.md."
   })
   ```
6. **Merge** the PR once opened (`gh pr merge --merge`). **Per `feedback_test_before_merge.md`, only merge after the owner confirms the live deploy works** — surface this gate explicitly in chat before calling merge.
7. **After merge**: REPLACE `phase:in-review` with `phase:merged` on each ticket and comment the merge SHA.
8. **After merge**: `git checkout main && git pull` in that repo.

Repeat for every repo with changes. Do repos one at a time — do not pipeline them.

---

## Phase 7 — Deploy + Verify + Close

For services that were built and pushed (api-service, ai-scanner, dashboard, etc.):

1. **Deploy** — `kubectl apply -k k8s/overlays/gcp/` + `kubectl rollout status deployment/<svc> -n <ns>-prod --timeout=180s`.
2. **Verify live**:
   - Service health: `curl -s https://app.0xapogee.com/api/v1/health/live` should return the new version
   - If user-facing: trigger an end-to-end action as `jasonbrailowbizop@mail.com` (password is **session-only** — use `$TEST_PASSWORD` env-var, never hardcode per `feedback_trigger_scans_via_api.md`)
   - If bug fix: re-run the original failure scenario, confirm it no longer reproduces
   - If new endpoint: hit it with the test account, confirm 200 + expected response shape
3. **Update Linear** on successful verify — for each ticket, set state `Done`, REPLACE `phase:merged` with `phase:verified`, and post a closing comment:
   ```
   mcp__plugin_linear_linear__save_issue({
     id: "ADV-12",
     state: "Done",
     labels: [<keep type+repo+source+severity>, "phase:verified"]
   })
   mcp__plugin_linear_linear__save_comment({
     issueId: "ADV-12",
     body: "**Phase 7 — Verified live**\n\nDeployed: api-service v0.46.5 (commit <sha>)\nLive evidence: <scan_id / curl output / DB query result>\nClosing."
   })
   ```
4. **If verify FAILS** — leave the Linear ticket in `phase:merged` state (NOT closed), comment with the failure details, and either fix-forward (loop back to Phase 5 with a new commit) or roll back via `kubectl rollout undo`. Never close a ticket with broken verification.
5. After all repos verified, **run a final `git status` in each changed repo** to confirm clean working tree on main.
6. **Summary report** — print to chat:
   - Linear tickets touched (with status transitions)
   - PRs merged (with URLs)
   - Deployed versions per service
   - Any follow-up Linear tickets created during the cycle (`source:audit` discoveries, `type:test` gaps, etc.)
   - Anything that requires manual operator action

---

## Reference Tools

- `mcp__plugin_linear_linear__list_issues` — search before creating
- `mcp__plugin_linear_linear__save_issue` — create or update (omit `id` to create)
- `mcp__plugin_linear_linear__save_comment` — comment on issue / project / milestone
- `mcp__plugin_linear_linear__list_issue_labels` — list available labels
- `mcp__plugin_linear_linear__list_projects` — list projects
- `mcp__plugin_linear_linear__list_issue_statuses` — list workflow states (Backlog / Todo / In Progress / Done / Canceled / Duplicate)

## Anti-patterns to refuse

- Creating a ticket without a Priority (P1/P2/P3/P4) — **the owner explicitly requires this on every ticket**.
- Creating a ticket without at least one `type:*` + `repo:*` label.
- Closing a ticket before live verification passes.
- Skipping Linear ticket creation because "it's a small change" — every meaningful change gets a ticket.
- Reusing a `Done` ticket for new work — file a new linked ticket via `relatedTo`.
- Hardcoding `TestPass123` or any password in any commit, comment, ticket body, audit report, skill, or agent file (BSO-SEC-048 precedent).
- Mentioning Claude / Claude Code / Anthropic in any commit, PR, ticket, or comment.
