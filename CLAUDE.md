# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**rtk (Rust Token Killer)** is a high-performance CLI proxy that minimizes LLM token consumption by filtering and compressing command outputs. It achieves 60-90% token savings on common development operations through smart filtering, grouping, truncation, and deduplication.

This is a fork with critical fixes for git argument parsing and modern JavaScript stack support (pnpm, vitest, Next.js, TypeScript, Playwright, Prisma).

### Name Collision Warning

**Two different "rtk" projects exist:**
- This project: Rust Token Killer (rtk-ai/rtk)
- reachingforthejack/rtk: Rust Type Kit (DIFFERENT - generates Rust types)

**Verify correct installation:**
```bash
rtk --version  # Should show "rtk 0.28.2" (or newer)
rtk gain       # Should show token savings stats (NOT "command not found")
```

If `rtk gain` fails, you have the wrong package installed.

## Development Commands

> **Note**: If rtk is installed, prefer `rtk <cmd>` over raw commands for token-optimized output.
> All commands work with passthrough support even for subcommands rtk doesn't specifically handle.

### Build & Run
```bash
cargo build                   # raw
rtk cargo build               # preferred (token-optimized)
cargo build --release         # release build (optimized)
cargo run -- <command>        # run directly
cargo install --path .        # install locally
```

### Testing
```bash
cargo test                    # all tests
rtk cargo test                # preferred (token-optimized)
cargo test <test_name>        # specific test
cargo test <module_name>::    # module tests
cargo test -- --nocapture     # with stdout
bash scripts/test-all.sh      # smoke tests (installed binary required)
```

### Linting & Quality
```bash
cargo check                   # check without building
cargo fmt                     # format code
cargo clippy --all-targets    # all clippy lints
rtk cargo clippy --all-targets # preferred
```

### Pre-commit Gate
```bash
cargo fmt --all && cargo clippy --all-targets && cargo test --all
```

### Package Building
```bash
cargo deb                     # DEB package (needs cargo-deb)
cargo generate-rpm            # RPM package (needs cargo-generate-rpm, after release build)
```

## Architecture

rtk uses a **command proxy architecture**: `main.rs` routes CLI commands via a Clap `Commands` enum to specialized filter modules in `src/cmds/*/`, each of which executes the underlying command and compresses its output. Token savings are tracked in SQLite via `src/core/tracking.rs`.

For the full architecture, component details, and module development patterns, see:
- [ARCHITECTURE.md](docs/contributing/ARCHITECTURE.md) — System design, module organization, filtering strategies, error handling
- [docs/contributing/TECHNICAL.md](docs/contributing/TECHNICAL.md) — End-to-end flow, folder map, hook system, filter pipeline

Module responsibilities are documented in each folder's `README.md` and each file's `//!` doc header. Browse `src/cmds/*/` to discover available filters.

Supported ecosystems: git/gh/gt, cargo, go/golangci-lint, npm/pnpm/npx, ruff/pytest/pip/mypy, rspec/rubocop/rake, dotnet, playwright/vitest/jest, docker/kubectl/aws.

### Proxy Mode

**Purpose**: Execute commands without filtering but track usage for metrics.

**Usage**: `rtk proxy <command> [args...]`

**Benefits**:
- **Bypass RTK filtering**: Workaround bugs or get full unfiltered output
- **Track usage metrics**: Measure which commands Claude uses most (visible in `rtk gain --history`)
- **Guaranteed compatibility**: Always works even if RTK doesn't implement the command

**Examples**:
```bash
rtk proxy git log --oneline -20    # Full git log output (no truncation)
rtk proxy npm install express      # Raw npm output (no filtering)
rtk proxy curl https://api.example.com/data  # Any command works
```

All proxy commands appear in `rtk gain --history` with 0% savings (input = output).

## Coding Rules

Rust patterns, error handling, and anti-patterns are defined in `.claude/rules/rust-patterns.md` (auto-loaded into context). Key points:

- **anyhow::Result** everywhere, always `.context("description")?`
- **No unwrap()** in production code
- **lazy_static!** for all regex (never compile inside a function)
- **Fallback pattern**: if filter fails, execute raw command unchanged
- **No async**: single-threaded by design (startup <10ms)
- **Exit code propagation**: `std::process::exit(code)` on child failure

Testing strategy and performance targets are defined in `.claude/rules/cli-testing.md` (auto-loaded). Key targets: <10ms startup, <5MB memory, 60-90% token savings.

For contribution workflow and design philosophy, see [CONTRIBUTING.md](CONTRIBUTING.md). For the step-by-step filter implementation checklist, see [src/cmds/README.md](src/cmds/README.md#adding-a-new-command-filter).

## Build Verification (Mandatory)

**CRITICAL**: After ANY Rust file edits, ALWAYS run the full quality check pipeline before committing:

```bash
cargo fmt --all && cargo clippy --all-targets && cargo test --all
```

**Rules**:
- Never commit code that hasn't passed all 3 checks
- Fix ALL clippy warnings before moving on (zero tolerance)
- If build fails, fix it immediately before continuing to next task

**Performance verification** (for filter changes):
```bash
hyperfine 'rtk git log -10' --warmup 3          # before
cargo build --release
hyperfine 'target/release/rtk git log -10' --warmup 3  # after (should be <10ms)
```

## Working Directory Confirmation

**ALWAYS confirm working directory before starting any work**:

```bash
pwd  # Verify you're in the rtk project root
git branch  # Verify correct branch (main, feature/*, etc.)
```

**Never assume** which project to work in. Always verify before file operations.

## Avoiding Rabbit Holes

**Stay focused on the task**. Do not make excessive operations to verify external APIs, documentation, or edge cases unless explicitly asked.

**Rule**: If verification requires more than 3-4 exploratory commands, STOP and ask the user whether to continue or trust available info.

**Examples of rabbit holes to avoid**:
- Excessive regex pattern testing (trust snapshot tests, don't manually verify 20 edge cases)
- Deep diving into external command documentation (use fixtures, don't research git/cargo internals)
- Over-testing cross-platform behavior (test macOS + Linux, trust CI for Windows)
- Verifying API signatures across multiple crate versions (use docs.rs if needed, don't clone repos)

**When to stop and ask**:
- "Should I research X external API behavior?" → ASK if it requires >3 commands
- "Should I test Y edge case?" → ASK if not mentioned in requirements
- "Should I verify Z across N platforms?" → ASK if N > 2

## Plan Execution Protocol

When user provides a numbered plan (QW1-QW4, Phase 1-5, sprint tasks, etc.):

1. **Execute sequentially**: Follow plan order unless explicitly told otherwise
2. **Commit after each logical step**: One commit per completed phase/task
3. **Never skip or reorder**: If a step is blocked, report it and ask before proceeding
4. **Track progress**: Use task list (TaskCreate/TaskUpdate) for plans with 3+ steps
5. **Validate assumptions**: Before starting, verify all referenced file paths exist and working directory is correct

---

## Developer Context

> Auto-updated after each session. Last updated: June 08, 2026.
> **Edit source files** at `skill/context/` (or `brain/context/` once vault repo is set up) — not directly here.
> At end of each session: update `skill/context/PROJECTS.md` with any status changes before ending.

### About Shahab

---

## Who I Am
- Based in **Mumbai, India** (Mahim area)
- Day job: **Export Customer Service & Documentation** at Sarang Maritime Logistics Pvt Ltd
- Domain expertise: NVOCC operations, carrier bookings, vessel scheduling, B/L documentation, Delivery Orders, port operations, Exim India data
- Actively building skills in **AI and data science** — positioning at intersection of logistics expertise + AI tooling
- Schedule: 9AM–6PM job → Mahim social service → home by 9:30PM (~45–60 mins free at night)

---

## My Setup
- **Laptop:** ASUS ExpertBook P1503CVA, Windows
- **Python env:** Miniforge3 conda
- **AI tools:** Claude Code (terminal), claude.ai (mobile app)
- **Security:** Windows Defender + Malwarebytes Free
- **Investing:** Groww (mutual funds + demat), SIP ₹10K/month

### Memory & AI Stack (Active)
- **claude-mem** ✓ working — auto-captures sessions, web viewer at localhost:37777
- **MemPalace** ✓ connected
- **obsidian-mcp** ✓ connected — vault at `C:\Users\shaha\OneDrive\Documents\Obsidian Vault`
- **RTK** ✓ hooked globally into Claude Code
- **Odysseus stack** ✓ running at localhost:7000 (ChromaDB + SearXNG + ntfy, 10 Anthropic models)
- **Local models:** Ollama (llama3.2)
- **context-sync** — connected but set_project has 401 auth bug (pending fix)
- **plugin:mempalace** — python3 path error (pending fix: change `python` → `python3`)

### Obsidian
- **Version:** 1.12.7
- **Vault path:** `C:\Users\shaha\OneDrive\Documents\Obsidian Vault`
- **MCP command:** `claude mcp add obsidian-mcp -- npx -y obsidian-mcp "C:\Users\shaha\OneDrive\Documents\Obsidian Vault"`
- **Obsidian Git:** pending setup (brain repo not yet created)

---

## Current Financial Goal
- **₹5 lakh** — need to keep aside as debt repayment (owner can ask anytime)
- Currently have no savings toward this
- Strategy: freelance logistics documentation consulting via LinkedIn visibility

---

## Work Context (Sarang Maritime)
- Role: Export dept — customer service + documentation only
- Company acts as **agent** for principals (does not own containers)
- Principals' containers tracked until empty return to depot
- Data sources: Excel, Email, WhatsApp, Botim, Exim India subscription
- Pain: Exim India data sometimes mismatches VO (vessel operator) — VO word preferred
- Branch in Gandhidham, Gujarat (separate management — Director: Mr. Jaymin Thakkar)

---

## GitHub Repos
- github.com/shahabshaikh1990/rtk
- github.com/shahabshaikh1990/caveman
- github.com/shahabshaikh1990/skill
- github.com/shahabshaikh1990/Mastermind-Assignment

---

## LinkedIn
- Profile: linkedin.com/in/shahab-shaikh-562522b1
- Headline + About section: optimized and ready to paste
- Status: First post not yet published

---

## How I Work Best with Claude
- Direct, no fluff
- Practical over theoretical
- Indian market context always applies
- Mobile app user (claude.ai)
- Honest about limitations (lazy with outreach, busy schedule)
- Responds well to step-by-step actionable plans

---

### Active Projects

---

## 1. LinkedIn Visibility & Freelance Documentation Consulting
**Status:** In Progress — profile optimized, first post pending
**Goal:** Make exporters come to me, build toward ₹5 lakh side income

**Positioning:**
> "Export documentation specialist | Helping small Mumbai exporters avoid costly B/L mistakes and shipment delays"

**Target Clients:**
- Small Mumbai exporters (garments, handicrafts, spices, chemicals)
- Can't afford full-time documentation person
- Struggle with B/L errors, shipping instructions, freight coordination

**Content Strategy:**
- Claude drafts posts, Shahab reviews + publishes
- Post ideas:
  - "3 mistakes small exporters make in shipping instructions"
  - "What to check when Exim India data doesn't match VO details"
  - "How to read a B/L — common errors that cost money"
  - "COD vs prepaid freight — what exporters often get wrong"

**Revenue Model:**
- 2–3 clients × ₹8K–15K/month = ₹25K–45K/month side income
- Timeline to ₹5 lakh: 12–18 months realistically

**Done:**
- [x] LinkedIn headline + About section optimized (ready to paste)

**Next Steps:**
- [ ] Paste optimized headline + About into LinkedIn profile
- [ ] Draft and publish first post
- [ ] Set up simple client inquiry template

---

## 2. Work Automation (Sarang Maritime — Internal)
**Status:** Planned — build after LinkedIn momentum starts
**Goal:** Reduce daily repetitive work, free up energy

**Pain Points Identified:**
- Replying to repetitive customer emails (shipment status, vessel position)
- Delayed inventory reports to principals
- Releasing Delivery Orders (DO) to shippers
- Container tracking until empty return to depot
- Reconciling empty returns with principal inventory
- Exim India data vs VO mismatch — manual cross-checking

**3 Tools to Build:**
1. **Email Reply Assistant** — paste incoming email → get drafted reply with correct shipment details
2. **Inventory + Container Tracker Dashboard** — all active containers, vessel position, empty return status, auto-generates principal inventory reports
3. **DO Release Workflow** — structured checklist + auto-generated DO letter

**Data Sources Available:**
- Excel (primary)
- Email / WhatsApp / Botim
- Exim India subscription
- VO emails for cut-off, ETD, ETA

**Stack:** Python + Claude Code + Excel automation

---

## 3. Instagram Saves Engine
**Status:** On hold until LinkedIn + freelance income stream is stable
**Goal:** Anonymous AI content brand — grow through saves/shares

**Target Audience:** Indian small business owners, freelancers, solopreneurs

**Content Pillars:**
- AI tool releases and reviews
- Motivation / mindset for builders
- "Build in public" content
- Practical AI workflows for non-technical people

**Tech Stack:**
- Notion MCP for content database
- Python automation for scheduling
- Claude Code for content generation
- Windows-specific setup

**Key Constraint:** Fully anonymous — no personal branding

---

## 4. Meta Ads Freelance
**Status:** Built system, no client yet
**Goal:** Manage Meta ad campaigns for Indian D2C brands

**Built:**
- Meta Ads Analysis System (Windsor.ai + Claude integration)
- India-specific KPIs: INR ROAS, COD order rate, RTO %

**Target Clients:** Small D2C brands spending ₹50K–5L/month on ads

**Note:** Secondary priority — LinkedIn consulting is faster path to ₹5 lakh

---

## 5. MerchantAntiques (Shopify)
**Status:** Active store
**URL:** merchantantiques.in
**Product:** Brass kitchenware, antique-style home goods
**Platform:** Shopify
**Next:** Meta ads strategy implementation

---

## 6. Dropshipping
**Status:** On hold
**Niche:** Pet wellness (India)
**Blocker:** Time + focus needed elsewhere first

---

## 7. AI & Data Science Upskilling
**Status:** Ongoing (slow)
**Platform:** UpGrad
**Course:** Python for AI & Data Science + Gen AI course

**Honest pattern:** Tendency to enroll and not complete
**Fix:** Build practical projects alongside learning

**Tools actively using:**
- Claude Code + RTK (global hook) ✓
- claude-mem ✓ working
- MemPalace ✓ connected
- obsidian-mcp ✓ connected
- Odysseus stack ✓ running (localhost:7000)
- Ollama / llama3.2
- ScrapeGraphAI

---

## 8. Obsidian Second Brain
**Status:** In Progress
**Vault:** `C:\Users\shaha\OneDrive\Documents\Obsidian Vault`
**MCP:** obsidian-mcp ✓ connected

**Done:**
- [x] obsidian-mcp connected via Claude Code
- [x] 815 WhatsApp links extracted and categorized into 11 notes (ready to copy)

**Pending:**
- [ ] Create `shahabshaikh1990/brain` GitHub repo (private)
- [ ] Install Obsidian Git plugin → point to brain repo
- [ ] Copy 11 WhatsApp .md files into vault
- [ ] Review and organize 260 YouTube Shorts + 23 videos

---

## Priority Order (as of June 2026)
1. **LinkedIn first post** — this week (profile already optimized)
2. **Create brain repo + Obsidian Git setup** — enables auto-sync
3. **Fix plugin:mempalace** — change `python` → `python3`
4. **Copy WhatsApp notes into vault**
5. **Email Reply Assistant (Tool 1)** — first automation win
6. **MerchantAntiques Meta ads**
7. **Instagram Saves Engine**

---

## Pending Setup Checklist
- [x] Install claude-mem
- [x] Install MemPalace
- [x] Connect obsidian-mcp
- [x] Hook RTK globally into Claude Code
- [x] LinkedIn headline + About section optimized
- [ ] Paste LinkedIn headline + About into profile
- [ ] Draft and publish first LinkedIn post
- [ ] Create `shahabshaikh1990/brain` GitHub repo (private)
- [ ] Install Obsidian Git plugin → point to brain repo
- [ ] Copy 11 WhatsApp .md files into vault (`C:\Users\shaha\Downloads\*.md` → vault)
- [ ] Fix plugin:mempalace: change `python` → `python3` in `.claude.json`
- [ ] Fix context-sync 401 auth bug
- [ ] Install ECC plugin in Claude Code
