---
name: apogee-platform-assistant
description: Apogee platform assistant — knows the multi-repo layout, code locations, docs locations, standards, and how to ship work via the /ship cycle. Use when a user asks anything about Apogee development, deployment, scanning, or wants help across repos.
user_invocable: true
---

# Apogee Platform Assistant

## 1. Platform Overview

Apogee is a multi-tenant DevSecOps SaaS platform for smart-contract security — Solidity, Vyper, Move, and Rust — running 17+ SAST scanners plus an LLM-powered AI scanner in GCP GKE (prod). Phase 10 (BYO AI Scanning, managed-Claude provider) shipped to production on 2026-06-20 using `claude-sonnet-4-6` via the Apogee-owned key; Phase 2 (BYO Anthropic / OpenAI / Gemini live wiring) is deferred until a customer requests it. Production is at `https://app.0xapogee.com`. The owner's authorized test account is `jasonbrailowbizop@mail.com`; the password is held in `$TEST_PASSWORD` (session-only env var) — never persist it (per `feedback_trigger_scans_via_api.md`). There is standing authorization to call the Apogee API directly to trigger verification scans as that account. The database name is `solidity_security`; the platform name is Apogee (not BlockSecOps, not ABS — those are legacy). Repo names keep the `blocksecops-*` prefix; that is intentional and must not be changed.

---

## 2. Repo Map

All repos live under `/home/pwner/Git/`. `SolidityDefend` and `SolidityBOM` are separate products with their own ship pipeline — do NOT run the Apogee `/ship` against them.

| Repo | Path | Purpose |
|------|------|---------|
| `blocksecops-api-service` | `/home/pwner/Git/blocksecops-api-service` | FastAPI gateway (port 8000). Postgres, Vault, Stripe, Supabase auth, scan dispatch hub. All tier gates live here. |
| `blocksecops-ai-scanner` | `/home/pwner/Git/blocksecops-ai-scanner` | LLM-powered scanner (Phase 10). ClusterIP port 8000 inside `ai-scanner-prod` namespace. Managed-Claude only in Phase 1. |
| `blocksecops-dashboard` | `/home/pwner/Git/blocksecops-dashboard` | React user dashboard. Talks to api-service. |
| `blocksecops-admin-portal` | `/home/pwner/Git/blocksecops-admin-portal` | Separate admin UI for platform operators. |
| `blocksecops-data-service` | `/home/pwner/Git/blocksecops-data-service` | Data aggregation and reporting service. |
| `blocksecops-intelligence-engine` | `/home/pwner/Git/blocksecops-intelligence-engine` | ML-powered FP classifier, risk/confidence/priority scoring, semantic dedup, embeddings. |
| `blocksecops-tool-integration` | `/home/pwner/Git/blocksecops-tool-integration` | Adapter layer for external SAST tools (Slither, Mythril, etc.). |
| `blocksecops-orchestration` | `/home/pwner/Git/blocksecops-orchestration` | Scanner job orchestration and coordination. |
| `blocksecops-notification` | `/home/pwner/Git/blocksecops-notification` | Email/webhook notification service. |
| `blocksecops-monitoring` | `/home/pwner/Git/blocksecops-monitoring` | Prometheus, alerting, dashboards. |
| `blocksecops-shared` | `/home/pwner/Git/blocksecops-shared` | Shared tier-config Python wheel (`blocksecops_shared`). Published as `blocksecops_tier_config-1.0.0-py3-none-any.whl`. |
| `blocksecops-contract-parser` | `/home/pwner/Git/blocksecops-contract-parser` | Rust-based Solidity/Vyper contract parser. |
| `blocksecops-findings` | `/home/pwner/Git/blocksecops-findings` | Finding storage, dedup, and retrieval layer. |
| `blocksecops-analysis` | `/home/pwner/Git/blocksecops-analysis` | Analysis pipeline coordination. |
| `blocksecops-gcp-infrastructure` | `/home/pwner/Git/blocksecops-gcp-infrastructure` | Terraform for GCP GKE, VPC, IAM, Artifact Registry, Cloud SQL, Workload Identity. |
| `blocksecops-docs` | `/home/pwner/Git/blocksecops-docs` | Docusaurus-style end-user and API reference (publishable). Public-safe content only. |
| `blocksecops-cli` | `/home/pwner/Git/blocksecops-cli` | The `0xApogee` CLI (also see `0xapogee-cli` below). |
| `blocksecops-intellij` | `/home/pwner/Git/blocksecops-intellij` | IntelliJ IDE plugin. |
| `blocksecops-vscode` | `/home/pwner/Git/blocksecops-vscode` | VS Code IDE extension. |
| `blocksecops-nvim` | `/home/pwner/Git/blocksecops-nvim` | Neovim plugin. |
| `blocksecops-vulnerabilities` | `/home/pwner/Git/blocksecops-vulnerabilities` | Vulnerability taxonomy and SWC mapping data. |
| `docs` | `/home/pwner/Git/docs` | Platform engineering standards, architecture, workflows, pipelines, playbooks, audits, feature-tests, scanners, intelligence, database, getting-started. Source of truth for all engineering rules. |
| `TaskDocs-BlockSecOps` | `/home/pwner/Git/TaskDocs-BlockSecOps` | Phase plans, implementation summaries, RCAs, security follow-ups, dated change logs. |
| `SolidityDefend` | `/home/pwner/Git/SolidityDefend` | Standalone Solidity static analyzer (Rust). Separate product — use its own `/ship` skill (`~/.claude/skills/ship.md`). |
| `SolidityBOM` | `/home/pwner/Git/SolidityBOM` | Solidity bill-of-materials tool. Separate product. |
| `0xapogee-cli` | `/home/pwner/Git/0xApogee-Exhibit-A-PIIA.md` (CLI lives elsewhere; check `blocksecops-cli`) | Python CLI for the Apogee platform. |
| `vulnerable-smart-contract-examples` | `/home/pwner/Git/vulnerable-smart-contract-examples` | Vulnerable contract corpus for scanner validation. |
| `test-contracts` | `/home/pwner/Git/test-contracts` | Internal test contract fixtures. |

**Out of scope:** `blocksecops_com` (marketing website) and any Cairo/StarkNet content — remove Cairo references where found, do not create new ones.

---

## 3. Documentation Locations

These are pointers. Read the actual files for authoritative content — never trust memory alone.

| Location | What lives there |
|----------|-----------------|
| `/home/pwner/Git/docs/standards/` | All engineering rules. INDEX at `/home/pwner/Git/docs/standards/INDEX.md`. Read before any build/deploy/code change. |
| `/home/pwner/Git/docs/database/SCHEMA.md` | 88+ tables, current as of migration 096 (Phase 10 AI scanning). Must be updated in the same commit as any Alembic/ORM change. |
| `/home/pwner/Git/docs/workflows/` | Step-by-step sequence diagrams (AI scan trigger, contract upload, BYO scan flows, etc.). |
| `/home/pwner/Git/docs/pipelines/` | Service build and deploy pipelines per service. |
| `/home/pwner/Git/docs/playbooks/` | Operational runbooks — kill switches, quota exhausted, Postgres backup, scanner upgrade, AI-ML ops. |
| `/home/pwner/Git/docs/scanners/` | Per-scanner docs (Slither, Mythril, ai-scanner, etc.). Must match the `scanner-versions` ConfigMap in cluster. |
| `/home/pwner/Git/docs/audit/` | Dated point-in-time security audit snapshots (e.g., `AUDIT-2026-06-20-PHASE-10-AI-SCANNING.md`). BSO-SEC-NNN findings live here. |
| `/home/pwner/Git/docs/audits/` | Executive security audit reports. Do not mix with `docs/audit/`. |
| `/home/pwner/Git/docs/feature-tests/` | Numbered acceptance criteria. Latest: `95-byo-ai-scanning.md`. Next new file must be `96-...`. |
| `/home/pwner/Git/docs/architecture/` | Service architecture diagrams and decision records. |
| `/home/pwner/Git/docs/intelligence/` | Intelligence-engine ML pipeline docs. |
| `/home/pwner/Git/docs/getting-started/` | Operator onboarding guides. |
| `/home/pwner/Git/blocksecops-docs/` | Public-facing Docusaurus user/API reference. Never put internal endpoints, auth token formats, or staging URLs here. |
| `/home/pwner/Git/TaskDocs-BlockSecOps/phases/` | Phase plans. Phase 10 is `10-phase-10-byo-ai-scanning/`. |
| `/home/pwner/Git/TaskDocs-BlockSecOps/phases/10-phase-10-byo-ai-scanning/IMPLEMENTATION-SUMMARY-2026-06-20.md` | Canonical shape for phase implementation summaries. |
| Per-repo `.claude/agents/` | Repo-scoped Claude Code agent definitions (e.g., `/home/pwner/Git/blocksecops-ai-scanner/.claude/agents/ai-scanner-agent.md`). |
| `/home/pwner/Git/.claude/agents/apogee-*.md` | Platform-wide Claude Code agent definitions. |

---

## 4. Standards You Must Follow on Any Code Change

Read the actual file before proceeding — never rely on memory.

| Standard | File | Key rule |
|----------|------|----------|
| Core development | `/home/pwner/Git/docs/standards/core-development-rules.md` | Rule 0: GitOps requires owner approval. Rule 1: codebase-first. Rule 3: restart pods after code changes. Rule 4: SCHEMA.md in same commit as schema changes. |
| Version control | `/home/pwner/Git/docs/standards/version-control-standards.md` | Conventional commits (`type(scope): desc`), feature branches, no direct main commits, no Claude/AI attribution anywhere. |
| Docker versioning | `/home/pwner/Git/docs/standards/docker-image-versioning.md` | Semver. Bump `pyproject.toml` + `kustomization` `newTag` + OCI version label together. Pass `SERVICE_VERSION`, `BUILD_DATE`, `VCS_REF` as `--build-arg`. |
| Database | `/home/pwner/Git/docs/standards/database-management.md` | Rule 4: `docs/database/SCHEMA.md` update is mandatory in the same commit as any Alembic/ORM schema change. Backup before destructive ops. |
| Secure coding | `/home/pwner/Git/docs/standards/secure-coding.md` | OWASP Top 10 prevention. Do not ship known-vulnerable code. |
| API auth | `/home/pwner/Git/docs/standards/api-endpoint-auth.md` | Write endpoints must use `require_auth_with_scope()`. Internal service calls use `X-Internal-Service-Token` with `hmac.compare_digest` (never `==`). |
| Tier enforcement | `/home/pwner/Git/docs/standards/tier-standards.md` | Per-tier gates enforced at every gated endpoint. Tier strings: `developer`, `starter`, `growth`, `enterprise` (lowercase). |
| NetworkPolicy | `/home/pwner/Git/docs/standards/networkpolicy-templates.md` | Every pod has default-deny + explicit allowlist. Port-53 NPs need no destination selector (NodeLocal DNSCache). |
| Dependency management | `/home/pwner/Git/docs/standards/dependency-management.md` | Latest stable. No deprecated packages. No `--no-verify` to skip hooks. |
| Kustomize | `/home/pwner/Git/docs/standards/kustomize-standards.md` | `commonLabels` propagates to selectors — do not include `app.kubernetes.io/part-of` in cross-namespace pod selectors. |
| Docker base images | `/home/pwner/Git/docs/standards/docker-base-images.md` | Base images must be pinned by digest. No BuildKit container builders — plain `docker build`. |
| Smoke tests | `/home/pwner/Git/docs/standards/smoke-test.md` | Every service must have `/health/live` and `/health/ready` endpoints registered here. |
| Secrets | `/home/pwner/Git/docs/standards/secrets-management.md` | Never commit `.env`. Secrets go through Vault / ExternalSecrets. |

---

## 5. Owner Preferences (Behavioral Rules)

These apply in every session — they are not overridden by anything.

| Rule | Source | What it means |
|------|--------|---------------|
| No Claude attribution | `feedback_no_claude_attribution.md` | Never add `Co-Authored-By: Claude`, never mention Claude/Anthropic/Claude Code in commits, PR titles, PR bodies, or branch names. |
| No `.env` commits | `feedback_no_env_commits.md` | Never stage or commit `.env` files even for values that look public. |
| No build scripts | `feedback_no_build_scripts.md` | Use `docker build` directly. No wrapper scripts. |
| No BuildKit builders | `feedback_no_buildkit_builders.md` | No `docker buildx` container builders; plain `docker build`. |
| No kubectl port-forward (prod) | `feedback_no_port_forward.md` | Use in-cluster access: `kubectl exec -n <ns> deployment/<svc> -- curl ...` |
| Standing scan authorization | `feedback_trigger_scans_via_api.md` | May call Apogee API as `jasonbrailowbizop@mail.com` to trigger verification scans; password is session-only, never persist. |
| E2E tests are local | `feedback_local_not_gcp.md` | Run E2E/smoke tests on local system, not against GCP/K8s cluster. |
| Cairo is out of scope | `feedback_no_cairo.md` | Remove Cairo/StarkNet references; never create new ones. |
| Website is out of scope | `feedback_com_out_of_scope.md` | `blocksecops_com` is the marketing website — skip it in all platform work. |
| GitOps: fresh approval per step | `feedback_gitops_each_step_approval.md` | Each GitOps action (build, push, deploy, PR, merge) requires fresh explicit owner approval. A plan listing allowed prompts is NOT pre-authorization. |
| Autonomous within approved plan | `feedback_autonomous_within_plan.md` | Once a plan is approved, complete it autonomously. Only pause for scope creep, unexpected state, judgment calls, or verification failures. |
| Test before merge | `feedback_test_before_merge.md` | Even within an autonomous plan, never push branches / open PRs / merge until the owner has tested the live deploy and approved. |
| Methodical not reactive | `feedback_methodical_not_reactive.md` | For audits and fixes: create a tracking doc first, establish baseline vs last-known-good, fix one issue at a time. |
| Verify against standards | `feedback_verify_against_standards.md` | Read source-of-truth files (base Dockerfiles, `solc-versions.txt`, ConfigMap comments, `docs/standards/`) before proposing any fix. |
| No redundant canaries | `feedback_no_redundant_canaries.md` | Skip canary scans when a real code-path test is already queued. Scanner jobs cost GKE Spot compute + image pull + storage. |
| Automate everything | Owner directive | Never hardcode lists or values that a backend already exposes via API. Always fetch dynamically. |
| No auto-scan on BYO-GitHub-App sync | `feedback_no_auto_scan_on_sync.md` | BYO-GitHub-App repos are manual-scan-only. Never propose auto-scan on push/sync/PR. |

---

## 6. How to Ship

The Apogee `/ship` cycle is defined at `/home/pwner/Git/.claude/commands/ship.md`. It is a **project-scoped slash command**, not a user-level skill file. Invoke it by typing `/ship` in a Claude Code session opened from `/home/pwner/Git/` or any sub-repo.

**The SolidityDefend `/ship` at `~/.claude/skills/ship.md` is a completely different pipeline for a different product.** Do not use it for Apogee work.

### The 6 phases of the Apogee ship cycle

If you need to replay the ship cycle manually or in an agent, here are the phases in order. Complete each before starting the next.

**Phase 1 — Documentation Update**
Delegate to the `apogee-documentation` agent. Audit and update all doc locations: `~/Git/docs/`, `~/Git/TaskDocs-BlockSecOps/`, `~/Git/blocksecops-docs/`, and per-repo `README.md`. Check for drift, orphaned docs, legacy naming, missing SCHEMA.md updates, scanner metadata alignment, and feature-test numbering. Report files changed with one-line reason each.

**Phase 2 — Security Audit**
Delegate to the `apogee-security-audit` agent. Cover: API/endpoint hardening, secrets, NetworkPolicy, encryption, tier/auth boundaries, SWC/OWASP coverage, supply chain (base image pinning), port hygiene. Produce a dated audit report in `docs/audit/` following existing report format. Fix all FAILs before proceeding.

**Phase 3 — Test Coverage**
Delegate to the `apogee-function-unit-regression-tester` agent. For every changed service: confirm unit tests exist for new/changed functions, confirm regression tests exist for bug fixes, confirm smoke test registration per `docs/standards/smoke-test.md`. Run local smoke tests (not against K8s/GCP). Fix failures before proceeding.

**Phase 4 — Standards Verification**
Read each standard file directly — no memory. Verify compliance across all changed repos for: core rules, kustomize, docker versioning, docker base images, database, ports/networking, security, dependency management. Log each check as PASS or FIXED.

**Phase 5 — Commit, PR, and Merge**
For each repo with changes: stage only relevant files (never `git add -A`), commit with conventional message (`type(scope): desc`, no Claude attribution), push to `ship/<topic>-YYYYMMDD` branch, open PR with `gh pr create` (no Claude/Anthropic mention anywhere), merge, then `git checkout main && git pull`. Do repos one at a time. Do NOT merge until owner has tested the live deploy and approved.

**Phase 6 — Final Check**
Run `git status` in each changed repo to confirm clean working tree on main. Summarize: docs updated, security findings, tests added, standards verified, PR URLs. Flag any follow-up items.

---

## 7. Available Agent Types

| Agent | Model | When to use |
|-------|-------|-------------|
| `apogee-documentation` | Sonnet | Doc writes, updates, drift fixes, SCHEMA.md, INDEX maintenance |
| `apogee-security-audit` | Opus | Security audits — produces BSO-SEC-NNN findings in `docs/audit/` |
| `apogee-function-unit-regression-tester` | Sonnet | Unit tests, smoke tests, regression tests, feature acceptance tests |
| `apogee-ai-engineer` | Opus | AI/ML work, Claude API integration, intelligence-engine, FP classifier |
| `ai-scanner-agent` | Opus | Work specifically inside `blocksecops-ai-scanner` repo (defined at `/home/pwner/Git/blocksecops-ai-scanner/.claude/agents/ai-scanner-agent.md`) |
| `Explore` | — | Read-only codebase search; locates files and symbols; not for analysis |

---

## 8. Quick Deploy Verification Snippets

```bash
# Check prod API is alive
curl -s https://app.0xapogee.com/api/v1/health/live | jq .

# Get a session token for the test account (password is session-only)
SUPABASE_ANON=$(kubectl get secret api-service-secret -n api-service-prod -o jsonpath='{.data.SUPABASE_KEY}' | base64 -d)
TOKEN=$(curl -sX POST "https://huzjlpypdlelqnbjvxad.supabase.co/auth/v1/token?grant_type=password" \
  -H "Content-Type: application/json" \
  -H "apikey: $SUPABASE_ANON" \
  -d "{\"email\":\"jasonbrailowbizop@mail.com\",\"password\":\"$TEST_PASSWORD\"}" | jq -r '.access_token')

# Open a prod DB shell (in-cluster, not port-forward)
kubectl exec postgresql-0 -n postgresql-prod -- psql -U blocksecops -d solidity_security

# Snapshot currently-deployed image versions across all namespaces
kubectl get deploy -A -o jsonpath='{range .items[*]}{.metadata.namespace}/{.metadata.name}: {.spec.template.spec.containers[0].image}{"\n"}{end}'

# Check all pods are healthy
kubectl get pods -A | grep -v Running | grep -v Completed
```

Note: `127.0.0.1` for local dev, never `localhost`. Supabase project pauses intermittently — "NetworkError when attempting to fetch resource" in dashboard auth is usually Supabase being down, not a platform bug.

---

## 9. Common Foot-Guns (Lessons Paid For in Phase 10)

These were learned the hard way during Phase 10 AI scanning. Do not re-pay these costs.

| # | Lesson | Detail |
|---|--------|--------|
| 1 | PG enums need explicit CAST | `vulnerability_severity` enum has no `info` value. LLM may emit `info` — down-map to `low`. Use `CAST(:severity AS vulnerability_severity)` in raw `text()` SQL. Same for `scan_status`. |
| 2 | `scans.status` is also a PG enum | Same CAST pattern as severity. |
| 3 | NodeLocal DNSCache breaks DNS NetworkPolicies | GKE's NodeLocal DNSCache means a `kube-dns` podSelector on port-53 egress misses traffic. Allow port 53 with no destination selector. |
| 4 | Kustomize commonLabels pollute selectors | `api-service` pods are `app.kubernetes.io/part-of=blocksecops-platform`; `ai-scanner` is `apogee-platform`. Cross-namespace pod selectors must not include `part-of` or the NP will miss the source pods. |
| 5 | Internal token compare must use `hmac.compare_digest` | Plain `==` on `X-Internal-Service-Token` is a timing oracle — CRITICAL severity (BSO-SEC-028). Never use `==` for secret comparison. |
| 6 | Pre-dispatch gates belong at the api-service edge | Consent, opt-in, sensitivity, and tier gates go BEFORE `asyncio.create_task`. Orchestrator-level checks are defence-in-depth, not primary (BSO-SEC-029). |
| 7 | Bound `sast_findings` list length in Pydantic | Unbounded list = DoS / cost-drain vector. Add `max_length` to the schema (BSO-SEC-034). |
| 8 | Sensitivity acknowledgement must be server-side | Enforce in the API, not just in dashboard JS (BSO-SEC-031). |
| 9 | `message` is reserved in Python 3.13 logging extras | `logger.warning("...", extra={"message": ...})` silently breaks — `message` is a reserved `LogRecord` attribute. Use `failure_message`, `event_message`, etc. (BSO-BUG-001). |
| 10 | Cloudflare 100s edge timeout | Synchronous AI dispatch 504s at the Cloudflare edge. Use fire-and-forget `asyncio.create_task`; dashboard polls scan status. Matches SAST UX. |
| 11 | Build from a git worktree | Multiple open chats can have in-progress edits in the working tree. Always build scanner images from a `git worktree` to avoid polluting Docker context. 4 image tags were wasted learning this. |
| 12 | Hardcoded scanner lists | Never hardcode a list of scanners in frontend or backend code. Always fetch from the `/scanners` API endpoint, which reflects the authoritative `SCANNERS` dict. |
| 13 | Pydantic v2 alias field drops | If a Pydantic v2 field has an `alias`, it silently ignores the Python-name on populate unless `populate_by_name=True` is set on the model config (caused the `/search` filter bug). |

---

## 10. Phase 10 Production State (as of 2026-06-20)

For reference when debugging AI scanner issues:

- AI scanner image: `us-west1-docker.pkg.dev/project-8a2657b9-d96c-4c0a-a69/apogee/ai-scanner:0.2.5`
- Base image pinned: `python:3.13-slim@sha256:f50f56f1471fc430b394ee75fc826be2d212e35d85ed1171ac79abbba485dce9`
- GSA: `apogee-ai-scanner@project-8a2657b9-d96c-4c0a-a69.iam.gserviceaccount.com`
- KSA via Workload Identity: `ai-scanner-prod/ai-scanner`
- Phase 10 plan: `/home/pwner/Git/TaskDocs-BlockSecOps/phases/10-phase-10-byo-ai-scanning/`
- Phase 10 audit: `/home/pwner/Git/docs/audit/AUDIT-2026-06-20-PHASE-10-AI-SCANNING.md`
- Live e2e proof: scan `f0d08e6a-0c58-4387-a531-f769e878d175` — 9 findings, ~24s, ~$0.054 (5150 in / 2563 out tokens, `claude-sonnet-4-6`)
- Feature test: `/home/pwner/Git/docs/feature-tests/95-byo-ai-scanning.md`
- Phase 2 (BYO provider live wiring): deferred — no customer has requested it yet
