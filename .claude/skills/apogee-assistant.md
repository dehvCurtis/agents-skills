---
name: apogee-assistant
description: >
  Complete business, sales, product, and funding context for Apogee (formerly BlockSecOps,
  formerly SolidityOps), the unified Web3 DevSecOps platform built by Dehvon Curtis under
  Advanced Blockchain Security LLC. Use this skill whenever Dehvon asks about: Apogee product
  features, sales pitches, customer outreach, investor applications, pricing, competitive
  positioning, design partners, partnerships, content strategy, grant applications, cold
  email templates, ICP targeting, demo scripts, objection handling, or anything related to
  running or growing the Apogee business. Also trigger for questions about SolidityDefend,
  RustDefend, SolidityBOM, the Blockchain Vulnerability Database, or the four platform
  pillars (Scanner Coverage, AI Scanner, Intelligence Engine, Enterprise Operations).
  Always consult this skill before any Apogee-adjacent response -- do not rely on memory alone.
---

# Apogee — Business & Sales Intelligence Skill

## Company Identity

- **Brand name:** 0xApogee
- **Legal entity:** Advanced Blockchain Security LLC
- **Website:** 0xapogee.com
- **Previous names:** BlockSecOps (2025 rebrand from SolidityOps)
- **Tagline / positioning line:** "Every scanner, one dashboard -- pay only for what you scan."
- **Mission:** Unified Web3 DevSecOps platform -- enterprise-grade smart contract security at developer speed
- **Status:** Live, accepting sign-ups, early revenue stage
- **Founder:** Dehvon Curtis (sole founder, sole developer)
- **EIN:** Exists (Advanced Blockchain Security LLC)
- **IP protection:** 0xApogee, SolidityDefend, SolidityBOM, and Blockchain Vulnerability Database are all carved out of Counterpart PIIA via Exhibit A
---

## Products

### 0xApogee Platform -- Four Pillars

---

#### Pillar 1: Scanner Coverage
- **19 active scanners, 4 languages, 509 total detectors** -- 3-4x the coverage of any single competitor
- **Languages:** Solidity, Vyper, Rust/Solana, Move
- **Scan presets:** Quick / Standard / Deep (UX innovation -- removes the "which tool do I run?" decision)
- **Integrated scanners (Solidity):** Slither, Aderyn, SolidityDefend, Semgrep, Solhint, Halmos, Echidna, Wake, Medusa, Certora, Manticore, and others
- **Integrated scanners (Rust/Solana):** RustDefend, Sec3 X-ray, Sol-azy, and others (2 of 4 active)
- **Fuzzers:** Echidna, Medusa, Halmos, and others (included in the 19 active scanners)
- **Chain coverage:** 37+ blockchain ecosystems including Ethereum, Arbitrum, Polygon, BSC, Solana, Optimism, Base, Aptos, and others
- **Competitive moat:** No competitor aggregates across all 4 languages with fuzzing built in
---

#### Pillar 2: AI Scanner
- **What it does:** Semantic reasoning over smart contract code -- catches vulnerability classes that SAST cannot detect through pattern matching alone
- **Why it matters:** SAST finds what it has rules for. AI finds what it can reason about.
- **Vulnerability categories it covers:**
  - Oracle staleness and price feed manipulation
  - ERC-4626 share manipulation
  - MEV (maximal extractable value) exposure
  - Spec drift (contract behavior diverging from documentation/intent)
- **Cost:** $0.05 per scan
- **Speed:** 37 seconds wall time
- **Model:** Managed Claude (`claude-sonnet-4-6`) is live natively on paid tiers. BYO model support (OpenAI, Gemini, other providers) is planned for Starter+ tiers but not yet live -- Phase 2 is deferred until a customer requests it.
- **UX:** Similar to Cursor -- interrogate contracts with any AI model of your choosing (managed model today; BYO coming)
---

#### Pillar 3: Intelligence Engine
- **False-positive reduction:** Trained AI dedup model (trained on DeFiHackLabs exploit corpus and HackenProof audit history -- not a general-purpose model)
- **Semantic deduplication:** Normalizes findings across 19 scanners with incompatible schemas, severity ratings, and code location references
- **Risk scoring:** Unified severity scoring across all scanner outputs
- **Blockchain Vulnerability Database (BVD):** Proprietary vulnerability registry that gets smarter with every scan
  - BVC ID format: `BVC-[PLATFORM]-[YEAR]-[ID]` (e.g., BVC-ETH-2024-001)
  - IPFS-based storage architecture
  - Long-term moat -- compounds with usage
- **CI/CD integration:** GitHub Actions native; runs on every commit
---

#### Pillar 4: Enterprise Operations
- **IDE integration:** Scan from inside the developer's editor
- **CI/CD pipeline:** Shift-left security -- catch issues before they ship
- **Collaboration:** Team workflows, shared dashboards, finding assignment
- **Compliance reporting:** DORA Article 26, MiCA, CRA, EO 14028, Immunefi compliance requirements
- **White-label report export:** For audit firms reselling the platform
- **Plugin SDK:** Third-party tool integration
- **SBOM generation:** Via SolidityBOM (see below)
- **Positioning:** Security tooling that doesn't fit workflows doesn't get used
---

### Advanced Blockchain Security LLC -- Proprietary Tools
*Built by Dehvon Curtis, shipped as part of 0xApogee. All three are open-source (MIT/Apache 2.0, decision made Jun 21, 2026).*

#### SolidityDefend
- **Type:** Rust-based Solidity static analysis security tool (SAST)
- **Built by:** Advanced Blockchain Security LLC
- **OSS status:** Open-source -- MIT or Apache 2.0
- **Repo:** github.com/AdvancedBlockchainSecurity/SolidityDefend
- **Detectors:** 90 detectors
- **Speed:** 60ms average analysis time
- **Coverage:** Reentrancy (cross-function, cross-contract variants), flash loan attack patterns, oracle manipulation, governance attack vectors (proposal spam, griefing, frontrunning), access control gaps, signature replay, emergency pause centralization, DeFi-specific vulnerability classes
- **Validated against:** CrossCurve, YO Protocol, Balancer V2, HyperVault real-world exploits
- **Distribution:** GitHub Action, cargo install, GitHub releases
- **GTM role:** OSS wedge driving platform adoption (Semgrep CE model)
#### SolidityBOM
- **Type:** Solidity-specific Software Bill of Materials (SBOM) generator
- **Built by:** Advanced Blockchain Security LLC
- **OSS status:** Open-source -- MIT or Apache 2.0 (decision made Jun 21, 2026)
- **Repo:** https://github.com/AdvancedBlockchainSecurity/SolidityBOM
- **Distribution:** cargo install + GitHub releases (standalone binary); also integrated inside 0xApogee platform
- **Output formats:** CycloneDX and SPDX
- **Capability:** Understands Solidity's import system -- remappings, relative paths, npm packages, git submodules, Foundry/Hardhat project detection, proxy contract patterns (UUPS, Transparent, Diamond), storage collision detection, complete PURL generation
- **Speed:** Under 2 seconds for typical 50-contract projects
- **Why unique:** No other tool (Syft, Trivy, or any SBOM tool) understands Solidity dependency trees. First and only Solidity-native SBOM generator in existence.
- **Regulatory angle:** EU Cyber Resilience Act (CRA), EO 14028, DORA Article 26, MiCA compliance, Immunefi compliance requirements
- **Flagship demo:** Curve Finance / Vyper hack ($70M, 2023) -- compiler version vulnerability; a correct SBOM would have flagged Vyper 0.2.15 before deployment
- **Grant eligibility:** Qualifies for independent grant applications (ESP, Horizon Europe, Spanish government grants) -- standalone distribution + unique prior art
- **Version history:** Built from Oct 6, 2025; v0.6.0+
#### RustDefend
- **Type:** Rust-based smart contract static analysis security tool (Solana/Rust ecosystem)
- **Built by:** Advanced Blockchain Security LLC
- **OSS status:** Open-source -- MIT or Apache 2.0
- **Role:** Covers the Rust/Solana side of the scanner coverage pillar alongside third-party tools (Sec3 X-ray, Sol-azy)
- **IP protection:** Carved out of Counterpart PIIA via Exhibit A alongside SolidityDefend and SolidityBOM
---

## Pricing Tiers

*Source of truth: tier-config v1.4.0 / tiers.json v4.1, competitive pricing v4.0 amounts set 2026-03-15. Exact dollar amounts for Starter/Growth/Enterprise are set via Stripe and tiers.json -- confirm before quoting in sales or applications. All prices below are from Stripe TEST MODE -- switch to live mode before first real customer.*

| Tier | Price (test mode) | AI Scanning | Notes |
|---|---|---|---|
| **Developer** | $0 (free) | Disabled (0 tokens, no managed Claude, no BYO) | Entry tier, no AI features |
| **Starter** | $199/mo \| $169/mo annual (Stripe: price_1TB6NR3ZtjkVcNXVY9S4MCcC) | 500K input / 50K output tokens/month, 30K per-scan cap, managed Claude only | BYO model planned, not yet live |
| **Growth** | $499/mo \| $419/mo annual (Stripe: price_1TB6NS3ZtjkVcNXVagseBeGL) | 2.5M input / 250K output tokens/month, 100K per-scan cap, managed Claude only | BYO model planned, not yet live |
| **Enterprise** | $1,499/mo \| no annual pricing yet (Stripe: price_1TB6NT3ZtjkVcNXVnbgex2YD) | 10M input / 1M output tokens/month, 200K per-scan cap, managed Claude only | Annual pricing needed before YC; BYO planned |

**Pay-per-scan (x402, Stripe-only -- USDC/crypto rail removed as of dashboard v0.47.0):**
- Simple (1-5 files): $0.50
- Standard (6-25 files): $1.00
- Larger complexity tiers exist in config
**AI scanning details:**
- Default managed model: claude-sonnet-4-6 (structured mode)
- BYO (Bring Your Own) model: planned for Starter, Growth, Enterprise tiers -- NOT yet live; Phase 2 deferred until first customer requests it
- AI overage: $0.015 per 1K input tokens, $0.075 per 1K output tokens
- Developer tier has zero AI access -- no managed Claude, no BYO
**Critical notes for sales and applications:**
- USDC/crypto x402 rail has been REMOVED -- do not pitch crypto-native payments as a current live feature
- x402 is now Stripe-only pay-per-scan at $0.50/$1.00
- ALL prices above are Stripe TEST MODE -- confirm live mode prices before quoting to prospects or in grant/accelerator applications
- Set Enterprise annual pricing before YC submission (currently missing)
- BYO AI model integrations are NOT live -- do not promise OpenAI/Gemini BYO to prospects
---

## x402 Pay-Per-Scan Model (Current)
- Protocol: x402 via Stripe (USDC/crypto rail removed as of dashboard v0.47.0)
- Pricing: $0.50 (Simple, 1-5 files), $1.00 (Standard, 6-25 files), larger tiers in config
- No account required for pay-per-scan
- Historical note: Platform previously accepted USDC on Base; this has been removed. Do not reference crypto-native payments in current sales or marketing materials until/unless reinstated.
---

## Community & Distribution Assets

| Asset | Status | Notes |
|---|---|---|
| LinkedIn (personal) | ~15,000 followers | Dehvon's personal account; DevSecOps/MLSecOps audience; highest-leverage channel |
| LinkedIn (Apogee company page) | ~255 followers | Growing; cross-post build-in-public content here |
| Reddit r/web3dev | Owned (moderator) | Active Web3 dev community |
| Reddit r/CryptoPeople | Owned (moderator) | Crypto-general audience |
| Reddit r/smartcontracts | Owned (moderator) | Most targeted; Solidity devs |
| Medium | ~45 followers | Technical articles; repurpose content here |
| Telegram | ~100 members, low engagement | Low priority; repurpose content |
| HackenProof | Verified profile | Real disclosures; credibility proof for outreach |
| Snyk West Coast Chapter | Dec 2023–Oct 2025 | ~12 enterprise companies every 60–90 days; warm enterprise access |

---

## Ideal Customer Profile (ICP)

### Tier 1 — DeFi Protocol Teams (Primary)
- **Profile:** 5–50 engineers, TVL $1M–$500M, shipping new contracts monthly, Solidity-first
- **Pain:** One exploit ends the protocol; audits take 6+ weeks and don't cover every deploy; tool sprawl (running Slither + Aderyn + MythX manually)
- **Product fit:** SolidityDefend in CI/CD + full platform. Every commit scanned.
- **ACV:** $5,900–$23,500/year
- **Entry motion:** Free scan of their public contracts → report → design partner ask
### Tier 1 — Web3 Infrastructure / Bridges
- **Profile:** Bridge teams, rollup infrastructure, L2s handling cross-chain assets; Rust+Solidity stacks common
- **Pain:** Bridges are #1 hack target ($2B+ lost); missing validation (Wormhole, Nomad) is the root cause
- **Product fit:** Full platform — Solidity + Rust scanning, SolidityBOM for dependency tracking
- **ACV:** $23,500–$50,000/year
- **Why prioritize:** Apogee is the only platform covering both Solidity AND Rust — unique positioning
### Tier 2 — Web3 Startups / Launchpads
- **Profile:** Pre-launch teams, NFT protocols, gaming protocols, <50 employees
- **Pain:** Can't afford $50K audit pre-launch; need credibility signal for investors
- **Product fit:** x402 micropayment scans ($0.50–$1.00) → Startup tier ($199/mo); "Scanned by Apogee" badge
- **ACV:** $0–$5,900/year
### Tier 2 — Smart Contract Auditors (Channel Partner)
- **Profile:** Independent auditors, boutique firms (Cantina, Sherlock, Code4rena competitors)
- **Pain:** Manual review doesn't scale; need automated pre-screening
- **Product fit:** Platform subscription + white-label report export
- **ACV:** Variable; also a referral channel
### Tier 3 — Enterprise / TradFi entering Web3
- **Profile:** Banks, fintech, institutional players building Web3 products
- **Pain:** DORA/MiCA regulatory compliance requirements; need auditability
- **Product fit:** Enterprise custom + SolidityBOM compliance reporting
- **ACV:** $100,000+/year
- **Note:** Longer sales cycle; pursue after 3–5 referenceable customers
---

## Sales Methodology

### The Free Scan → Design Partner → Close Funnel
1. **Identify target** — Protocol with public contracts, recent deploy, or recent fundraise
2. **Run SolidityDefend** — Auto-generate "Apogee Security Report" on their public contracts
3. **Cold outreach** — Lead with a specific finding from the report (<80 words, single CTA)
4. **Design partner ask** — "Free 90-day Enterprise access in exchange for feedback and a case study"
5. **Discovery call** — Understand their stack, CI/CD setup, audit history
6. **Live demo on their code** — Run the platform on their actual contracts during the call
7. **Insurance math close** — Frame cost vs. TVL: "Enterprise at $1,499/mo = $17,988/yr. Your TVL is $50M. That's 0.036% of assets at risk."
### Cold Email Template (<80 words, signal-based)
```
Subject: [Protocol Name] — [specific vulnerability class] finding

Saw [specific signal — new deployment/raise/hack in their ecosystem].
Ran SolidityDefend (open-source) on your public contracts — flagged
[specific finding] in 60ms.

Happy to share the full report. No ask.

[Link to report or offer]

— Dehvon Curtis
Snyk WC Chapter Lead | HackenProof-verified | 0xapogee.com

Want the report?
```

### Demo Arsenal — Hacks to Use in Sales Demos
Always open with: *"Pick a number — $25M, $80M, $190M, $326M."*

**Reentrancy demos (SolidityDefend catches these):**
- Cream Finance ($130M, Oct 2021) — flash loan + reentrancy
- Rari Capital / Fei Protocol ($80M, Apr 2022) — cross-contract reentrancy
- Grim Finance ($30M, Dec 2021) — ERC777 callback reentrancy
- Curve Finance / Vyper ($70M, Jul 2023) — compiler reentrancy guard bug (SolidityBOM demo)
**Access control / missing validation (SolidityDefend catches):**
- Wormhole ($326M, Feb 2022) — missing guardian signature validation (Rust/Solana)
- Nomad Bridge ($190M, Aug 2022) — initialization bug
- Cashio ($52M, Mar 2022) — missing account validation (Rust/Solana)
**Oracle/price manipulation:**
- Mango Markets ($117M, Oct 2022) — oracle price manipulation
- PancakeBunny ($45M, May 2021) — flash loan oracle manipulation
**Governance/flash loan:**
- Beanstalk ($182M, Apr 2022) — governance flash loan attack
- Euler Finance ($197M, Mar 2023) — donation attack / logic error
**SolidityBOM flagship demo:**
- Curve/Vyper ($70M) — toolchain vulnerability, not a code bug; SolidityBOM would have tracked the vulnerable compiler version
**Source databases for demo contracts:**
- DeFiHackLabs (GitHub) — primary source
- DeFiVulnLabs (GitHub)
- pcaversaccio/reentrancy-attacks (GitHub)
- Rekt News (rekt.news)
- DeFiLlama Hacks
---

## Competitive Positioning

### Positioning Statement
"Apogee is the only Web3 security platform that aggregates 19 scanners into one CI dashboard, includes a proprietary 60ms Rust SAST engine (SolidityDefend), and lets you pay per scan via Stripe — no subscription required. We cover Solidity, Vyper, AND Rust/Solana across 37+ chains."

### Competitive Landscape

| Competitor | Type | Funding | Apogee Edge |
|---|---|---|---|
| **Olympix** | Solidity SAST + mutation testing | ~$13.1M (Boldstart; Snyk founder Guy Podjarny angel) | Apogee aggregates 19 scanners vs. single engine; pay-per-scan pricing |
| **MetaTrust (MetaScan)** | Multi-engine SAST + AI (GPTScan) | M Ventures seed | Closest to Apogee's aggregation — but no flexible pay-per-scan pricing |
| **Cyfrin Aderyn** | Free/OSS Rust Solidity SAST | Monetizes via audits/CodeHawks | Apogee is the platform layer on top; SolidityDefend competes feature-for-feature |
| **Octane** | AI-agent continuous SAST | $6.75M (Archetype/Winklevoss) | Apogee has breadth + proprietary scanner + flexible pricing |
| **Slither (Trail of Bits)** | Free OSS | Open source | One of 19 Apogee scanners |
| **MythX** | CI SAST (**SHUT DOWN Mar 31 2026**) | ConsenSys | Gap in market — former MythX users are active prospects |
| **Code4rena** | Audit contest platform (**shut down May 2026**) | — | Customer base absorbed by Immunefi — pitch directly |
| **CertiK** | Audit firm + Skynet monitoring | $400M+ raised | Audits $15K–$150K/engagement; Apogee is continuous, not point-in-time |
| **SolidityScan** | SAST platform | Draper Associates | 450+ detectors but less proprietary; 200K+ monthly scans (traction benchmark) |

### Pricing Comparison (mid-2026)
```
Free OSS ←————————————————————————————→ Enterprise Audits
    |              |              |              |
 Slither       Apogee        MetaTrust       CertiK
 Mythril       $0–$1,499/mo  $599/mo+       $15K–$150K/engagement
 Aderyn        + pay-per-scan
 (Free)          $0.50–$1.00
```

### Key Differentiators vs. Objections

**"Slither is free, why pay?"**
→ Slither is one of 19 scanners in Apogee. Running Slither alone means manually managing output, no deduplication, no false-positive reduction, no unified dashboard, no SolidityBOM, no SolidityDefend detectors. Tool sprawl is the real cost.

**"We already have an audit scheduled."**
→ Audits are point-in-time and don't cover every deploy. 90% of exploited contracts were already audited. Apogee runs on every commit — it's what you do *between* audits and *after* shipping.

**"We'll use the free tier of [Competitor]."**
→ pay-per-scan: "Try Apogee right now for $0.50, no account, no subscription. You'll have results in 60 seconds."

**"Why not build this ourselves?"**
→ 19 scanner integrations, 509 detectors, AI deduplication, SolidityBOM, and a proprietary Rust SAST took years to build. Your engineers' time is worth more than $199/month.

**"We're not sure about the ROI."**
→ Insurance math: Enterprise at $1,499/mo = $17,988/year. If your TVL is $10M, that's 0.18% of assets protected. Every DeFi protocol should see this as table stakes, not optional.

---

## Warm Contacts & Target Relationships

### Kevin Ma — CTO, Omen (agentomen.com)
- **Relationship:** Went through full interview process; Omen withdrew offer after difficult negotiation; Dehvon closed graciously; referenced Apogee in closing email; sent soft pitch email for design partnership
- **Status:** Warm; has been offered free Apogee subscription; soft pitch sent
- **Next action:** Follow up if no response to pitch; offer free 90-day Enterprise in exchange for design partner status and case study
- **Value:** Omen is MegaETH-based (Solidity); perpetual futures platform = high-value DeFi target; Susa Ventures/Chad Byers backing = potential VC intro path
### Snyk West Coast Chapter Companies
- ~12 enterprise companies seen every 60–90 days
- Not for hard selling; for thought-leadership talks and warm referrals
- Talk idea: "Continuous Smart Contract Security in CI — What Web2 DevSecOps Gets Right That Web3 Hasn't Adopted Yet"
### HackenProof Protocols
- Any protocol where Dehvon has submitted disclosures = warmest possible intro
- "I found a bug in your code — want to ensure it never happens again in CI?"
---

## Partnership Targets

| Partner Type | Companies | Pitch |
|---|---|---|
| Bug bounty platforms | Immunefi, HackenProof, Cantina | "Pre-bounty continuous scan reduces noise and catches what human hunters miss" |
| Audit firms | Sherlock, Cyfrin, Halborn, OpenZeppelin | "Continuous monitoring between your audits — referral revenue both directions" |
| Ecosystem foundations | Arbitrum, Solana, BNB, Optimism, Base | Grant funding + embedded in developer pipelines |
| Immunefi (Code4rena absorbed) | Immunefi | Code4rena shut down May 2026; their customer base is now Immunefi's — pitch directly |

---

## Funding Pipeline

### Active / Urgent (deadline order)
1. **Startup Wise Guys Flagship** — Jul 12, 2026 | ~€100K convertible | B2B SaaS + Cybersecurity + Spain alignment
2. **YC Fall 2026** — Jul 27, 2026 | $500K for ~7% | Fintech 3.0 / Base / crypto-native angle; Quantstamp (YC) is precedent
3. **NSF SBIR Phase I** — Jul 27, 2026 | Up to $305K non-dilutive | Cybersecurity & Authentication topic; SolidityDefend = novel Rust SAST R&D; SolidityBOM = novel SBOM R&D; employment letter drafted June 20, 2026
4. **Alliance DAO (ALL18)** — Rolling (Sep 7 start) | $450K–$500K SAFE at $5M post | Most crypto-native accelerator; 5% acceptance rate
### Ecosystem Grants (Rolling)
- **Arbitrum Stylus Sprint** — 5M ARB for Rust/WASM tooling (SolidityDefend Rust core = direct fit)
- **Arbitrum Foundation** — $20K–$150K; security tooling explicitly funded
- **Arbitrum DDA 3.0 (Questbook)** — arbitrum.questbook.app; developer tooling category
- **Base Builder Grants** — retroactive; x402 on Base = already eligible
- **Ethereum Foundation ESP** — $5K–$500K+; open-source developer tooling; OSS SolidityDefend required
- **Solana Foundation** — non-dilutive; Rust/Solana coverage relevant
- **BNB Chain** — up to $200K; developer tools category
### Future / Post-Spain
- Horizon Europe — "Software Supply Chain Security" call; consortium grant; SolidityBOM = SBOM for smart contracts
- Spanish government startup grants — via Startup Wise Guys Spain; up to €125K non-refundable
- Optimism RetroPGF — retroactive; build on Superchain first, apply after
### Key Application Note
Open-sourcing SolidityDefend unlocks nearly every grant on this list (ESP, Solana, Arbitrum, BNB, Optimism all prefer/require OSS). The OSS decision is both a marketing and a funding strategy.

---

## Content Strategy

### Build-in-Public Framework
- Post 3–4x/week on Dehvon's personal LinkedIn (15K followers) + X
- Content mix: 95% pure value / 5% product mention
- Formats: vulnerability breakdowns, SolidityDefend speed demos, recent-hack post-mortems, honest build updates, concrete metrics ("SolidityDefend found X in 60ms")
### Content Series Ideas
- **"Hall of Shame"** — Weekly exploit breakdown: what happened, what contract code caused it, whether SolidityDefend/Apogee would have caught it
- **Same-day hack commentary** — Post within hours of a new DeFi exploit going viral; tie to a specific Apogee detector
- **"What 90 Detectors Found in [Protocol]'s Contracts"** — Scan a top DeFi protocol's public contracts; publish results on Medium + LinkedIn
- **"How to Add Smart Contract Scanning to GitHub Actions in 5 Minutes"** — Cornerstone CI integration guide
- **State of Web3 Security** — Quarterly data report using Apogee scan aggregate data
### LinkedIn Post Template
> [Hook — specific number or finding]
> [2–3 sentence story or lesson]
> [Soft product tie-in]
> [Question to drive comments]

---

## Current Metrics (as of Jun 21, 2026)

- GitHub stars: 1 (SolidityDefend + SolidityBOM combined)
- Paying customers: 0
- Sign-ups: 0
- Revenue: $0 (Stripe confirmed $0 processed)
- Stripe transactions: 0
- Total scans run: 1,561 (all internal/test)
- Platform live date: December 15, 2025 (first scan in DB -- honest YC answer)
- GCP project created: January 9, 2026
- GKE production cluster: March 9, 2026
**Pricing (Stripe TEST MODE -- confirm live mode before using in materials):**
- Developer: $0/mo
- Starter: $199/mo | $2,028/yr ($169/mo)
- Growth: $499/mo | $5,028/yr ($419/mo)
- Enterprise: $1,499/mo | no annual pricing yet
- Pay-per-scan: $0.50 (Simple 1-5 files) | $1.00 (Standard 6-25 files)
**Action needed before YC/NSF submissions:**
- Set Enterprise annual pricing (currently missing)
- Switch Stripe to live mode before first real customer

**YC framing:** "Platform has been live since December 2025. 1,561 scans run. No external customers or revenue yet -- all internal testing. Actively recruiting first design partners."

**YC application:** Answer No to both "Are people using your product?" and "Do you have revenue?"

---

## 90-Day Targets (By Sep 2026)
- $5K–$10K MRR
- ≥5 paying customers across tiers
- ≥200 GitHub stars (SolidityDefend community edition)
- ≥3 named design partners (logos for deck)
- ≥1 published exploit breakdown with 500+ LinkedIn impressions
- ≥1 partnership conversation started (Immunefi or audit firm)
- All 3 accelerator applications submitted
---

## Key Terminology (Domain Glossary)
- **SAST** — Static Application Security Testing (code analysis without running the code)
- **SBOM** — Software Bill of Materials (inventory of software components and dependencies)
- **SolidityDefend** — Apogee's proprietary Rust-based Solidity SAST
- **SolidityBOM** — Apogee's proprietary Solidity SBOM generator
- **x402** — HTTP payment protocol (Coinbase/Base origin); Apogee uses Stripe-based pay-per-scan; USDC/crypto rail removed
- **CI/CD** — Continuous Integration / Continuous Deployment
- **TVL** — Total Value Locked (DeFi metric for assets in a protocol)
- **ACV** — Annual Contract Value
- **MRR** — Monthly Recurring Revenue
- **ICP** — Ideal Customer Profile
- **DeFi** — Decentralized Finance
- **Reentrancy** — #1 smart contract vulnerability class (recursive call exploit)
- **Oracle manipulation** — Price feed exploitation in DeFi
- **CEI** — Checks-Effects-Interactions pattern (reentrancy prevention)
- **DeFiHackLabs** — GitHub repo of real-world hack PoC contracts used for demos
- **HackenProof** — Bug bounty platform where Dehvon has verified auditor status
- **DORA** — EU Digital Operational Resilience Act (financial sector ICT security regulation)
- **MiCA** — Markets in Crypto-Assets regulation (EU crypto licensing framework, final deadline Jul 1 2026)
- **CRA** — EU Cyber Resilience Act (software supply chain security requirements)
- **PLG** — Product-Led Growth (self-serve, usage-driven conversion model)
- **PQL** — Product-Qualified Lead (user who has hit a usage threshold indicating purchase intent)
---

## NSF SBIR Phase I — Project Pitch (Filed June 20, 2026)

Application submitted via NSF SBIR-STTR Project Pitch portal. All 7 eligibility criteria answered Yes. Employment letter signed June 20, 2026. EIN confirmed. Fast-Track: No. Questions 10/11/12: all No (first-time applicant). Backup deadline: November 4, 2026.

**Key framing:** Apogee is the commercial product. SolidityDefend is open-source and feeds the platform. SolidityBOM is open-source. The R&D being funded is the platform intelligence layer — cross-scanner correlation, cross-contract detection, and provably complete SBOM — none of which are solved at production scale.

**Critical note on PI employment:** NSF requires PI to be primarily employed (51%+) by the small business at time of award, not submission. Dehvon is currently full-time at Counterpart. This is not a submission blocker but must be resolved before any Phase I award is accepted.

---

### Section 13 — Technology Innovation (Field 13, up to 3500 chars)

I built Apogee because I kept running into the same problem during smart contract audits: every security tool produces different output, uses different severity ratings, and flags the same vulnerability five different ways. Teams end up with spreadsheets, not security programs. Apogee (0xapogee.com) is a unified security platform that runs 19 smart contract scanners — covering 509 vulnerability detectors across Solidity, Vyper, and Rust/Solana — and collapses the output into a single, deduplicated, actionable dashboard integrated directly into CI/CD pipelines.

The platform has three technically novel components that don't exist elsewhere. The first is a cross-scanner vulnerability correlation engine that normalizes findings across tools with incompatible schemas and taxonomies. Slither calls it "reentrancy-eth," Aderyn calls it "reentrancy-vulnerabilities," Semgrep flags it at the AST node level — these are the same vulnerability and no existing system can reliably say so at scale. The second is SolidityDefend, a static analysis engine I wrote in Rust that builds an intermediate representation of Solidity's execution semantics rather than parsing source text. This lets it model cross-contract call graphs and catch flash loan attack patterns, oracle price manipulation, and governance attack vectors that Python-based tools miss entirely — in under 100 milliseconds. The third is SolidityBOM, the only SBOM generator that understands how Solidity projects actually organize dependencies: path remappings, git submodules, UUPS/Transparent/Diamond proxy patterns, and npm-linked libraries. Syft and Trivy don't parse any of this. The Curve Finance exploit in 2023 — $70 million lost — came from a vulnerable Vyper compiler version that a correct SBOM would have flagged before deployment.

None of these components are finished research. The correlation engine hasn't been validated at enterprise scale. SolidityDefend's inter-procedural analysis doesn't yet model cross-contract call graphs with formal completeness bounds. SolidityBOM can't yet resolve proxy targets that require on-chain state queries. Phase I is the R&D program to prove these three things work at production quality — not to build the prototype, which already exists.

---

### Section 14 — Technical Objectives and Challenges (Field 14, up to 3500 chars)

Three research problems need to be solved before Apogee can operate reliably at enterprise scale. Each one has a specific technical risk that could cause the approach to fail even with a competent team working on it.

The first is cross-scanner vulnerability correlation. Right now I can deduplicate findings across a small set of contracts with good results. The open question is whether the same approach holds when a protocol has 500 contracts, multiple inheritance hierarchies, and upgradeable proxy architecture — environments where the same underlying code surface gets flagged by different scanners under different names depending on how the contract tree is traversed. The challenge isn't deduplication in the simple case; it's building a vulnerability ontology specific enough to blockchain security that the classifier doesn't collapse precision when the codebase complexity goes up by an order of magnitude. My plan for Phase I is to construct a labeled training dataset from the DeFiHackLabs exploit corpus and my own HackenProof audit history, build the ontology from first principles against that dataset, and measure precision/recall degradation as contract set size increases.

The second problem is cross-contract dataflow analysis in SolidityDefend. Intra-contract detection works. The hard part is what happens when a reentrancy vulnerability or a flash loan attack spans two or three contracts — where the attacker calls contract A, which calls contract B, which calls back into contract A. Reconstructing that call graph statically is fundamentally incomplete when contracts use delegatecall or interface-based dispatch, because the target address isn't known at compile time. My approach is to build a bounded approximation: use ABI definitions and on-chain bytecode to recover likely call targets, accept that the graph will have gaps, and establish formal bounds on what classes of vulnerability the bounded graph can and cannot detect. The goal isn't perfect completeness — it's a model with known and documented failure modes.

The third problem is SolidityBOM dependency completeness under proxy non-determinism. Foundry and Hardhat projects resolve cleanly. The hard cases are protocols using proxy patterns where the implementation address is stored in a storage slot and only retrievable via an on-chain RPC call at a specific block height. That introduces non-determinism into SBOM generation: two runs at different block heights could produce different dependency graphs if an upgrade happened in between. Phase I will develop a hybrid resolution model that separates the static dependency surface (always deterministic) from the dynamic proxy surface (block-height-dependent) and documents the uncertainty bounds explicitly in the SBOM output — which is actually what regulators under DORA Article 26 need anyway.

These three outcomes together produce a platform whose intelligence layer can't be replicated by stringing together existing open-source tools. The individual scanners are commodities. The correlation, the cross-contract detection, and the complete SBOM are not.

---

### Section 15 — Market Opportunity (Field 15, up to 1750 chars)

Smart contracts control somewhere between $80 billion and $160 billion in on-chain assets right now, and the dominant security practice is still a point-in-time manual audit that costs $30,000 to $300,000, takes six weeks to schedule, and covers a snapshot of the codebase that's often stale by the time the report ships. In the first five months of 2026, over $840 million was lost to exploits — a 70% increase over the same period in 2025. The majority of those protocols had been audited.

The immediate customers are protocol engineering teams — the five to fifty developers shipping Solidity and Rust contracts on Ethereum, Arbitrum, Solana, and a dozen other chains — who need security running in their CI pipeline on every commit, not six weeks before launch. There are more than 5,000 active DeFi protocols globally fitting this profile. Beyond that, MiCA and DORA are forcing European crypto firms to demonstrate continuous security testing for regulatory compliance, which creates a second buyer category that didn't exist two years ago.

MythX, which was the only serious CI-integrated scanner before Apogee, shut down in March 2026. Those teams are actively looking for a replacement. The remaining tools — Slither, Olympix, MetaTrust — either require manual operation, cover a single language, or don't offer the cross-scanner intelligence layer that makes continuous scanning actually useful rather than just noisy.

---

### Section 16 — Company and Team (Field 16, up to 1750 chars)

Advanced Blockchain Security LLC is a single-founder company. I'm Dehvon Curtis — I designed and built everything on the Apogee platform: SolidityDefend in Rust, SolidityBOM, the scanner aggregation layer, the AI deduplication engine, and the CI/CD integration. The company has been operating since 2022, starting with independent smart contract auditing on HackenProof and expanding into platform development as I ran into the tool-fragmentation problem repeatedly across engagements.

My background is split between enterprise security infrastructure and blockchain security research. On the enterprise side, I spent years at Adobe, Broadcom/Symantec, Berkadia, and Kaseya building DevSecOps programs — CI/CD security pipelines, Kubernetes security infrastructure, SAST/DAST/SCA toolchain integration at scale across 35+ products. I hold a CKA certification and a GIAC Cloud Security Automation certification. I teach ethical hacking at Salt Lake Community College. I ran the Snyk West Coast chapter for two years, which meant regular exposure to how enterprise security teams actually buy and integrate tools.

On the blockchain side, I'm a verified auditor on HackenProof with disclosed findings across Ethereum, Polygon, Arbitrum, and BSC protocols. That auditing work is directly where SolidityDefend's detector library came from — I was writing manual detection rules and eventually decided to automate them.

Phase I funding would go toward two things: a Rust systems engineer to work on the inter-procedural dataflow analysis in Objective 1, and a formal methods consultant to build the completeness proof framework for Objective 2. Both are specialized enough that I can't execute them alone at the speed Phase I requires.

---

### NSF SBIR Key Framing Notes
- Primary subject is Apogee the platform, not SolidityDefend the tool
- SolidityDefend is open-source — the commercial moat is the platform intelligence layer
- Three R&D risks: cross-scanner correlation at scale, cross-contract call graph completeness, SolidityBOM proxy non-determinism
- Working prototype exists and is live — this is a strength ("preliminary work"), not disqualifying
- PI employment conflict (Counterpart) must be resolved before award acceptance, not before submission
- Backup deadline: November 4, 2026
---

## How to Use This Skill

### Sales / outreach tasks
- Use ICP table to qualify prospect segment and ACV target
- Use cold email template as starting point; customize with actual finding from free scan
- Use demo arsenal to select the right hack story for the prospect's stack (Solidity vs. Rust)
- Use competitive positioning section for objection handling
### Funding / accelerator tasks
- Use funding pipeline section for deadlines, amounts, and fit notes
- Frame applications around: novel Rust SAST (SolidityDefend), novel SBOM (SolidityBOM), $840M+ in 2026 DeFi losses, MythX shutdown gap, MiCA/DORA tailwinds, x402 on Base
- Use accurate numbers: 19 active scanners, 509 total detectors, 90 SolidityDefend detectors, 4 languages
### Content / marketing tasks
- Use content strategy section for post templates and series ideas
- Always tie content to a specific hack, detector, or metric — concrete > abstract
### Product / technical tasks
- Check product specs above before describing features to customers or in applications
- SolidityDefend: 90 detectors, 60ms, Rust, Solidity only
- Platform: 19 active scanners, 509 total detectors, Solidity + Vyper + Rust/Solana + Move
- SolidityBOM: only Solidity-native SBOM generator in existence
- BYO AI model: planned, NOT live — do not promise to prospects
### Codebase context
- GitHub: github.com/AdvancedBlockchainSecurity/blocksecops-*
- Dehvon may prompt codebase details when needed for technical accuracy
- Always ask if codebase context would improve the output for technical tasks
