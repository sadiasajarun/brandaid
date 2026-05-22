# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

BrandAid is an AI-powered **pre-launch marketing campaign simulation platform** (hackathon MVP). A marketer submits a domain URL → the system crawls it and builds a BusinessProfile → the marketer fills out a conversational campaign brief → BrandAid produces an explainable pre-launch prediction dashboard with ranked recommendations. The signature demo moment ("the ROAS flip"): applying two AI recommendations visibly raises the predicted ROAS in a before/after comparison.

This repo is **not yet an application.** It currently holds only pipeline tooling and design-phase artifacts — there is no `backend/`, `frontend/`, `package.json`, build, or test suite. Do not look for application source code.

## Repository layout — two distinct trees

| Tree | Purpose |
|---|---|
| `.claude/` | The **claude-fullstack pipeline** machinery — slash commands, skills, quality gates, phase definitions, rule files. Generic tooling reused across projects; not BrandAid-specific. |
| `.claude-project/` | **BrandAid's actual artifacts** — spec, PRD, design prototypes, pipeline status. This is where project work lives. |

`.claude/` is the engine; `.claude-project/` is its output.

## Current pipeline state

BrandAid is mid-pipeline on the **PM track** (`/fullstack-pm`). The live status is `.claude-project/status/brandaid/PIPELINE_STATUS.md`.

| Phase | State | Output |
|---|---|---|
| P1-spec | done | `.claude-project/status/brandaid/seed-brandaid-mvp-v1.yaml` |
| P2-prd | done | `.claude-project/docs/PRD.md` (archived under `prd/`) |
| P3-design | in progress | HTML prototypes in `.claude-project/generated-screens/` |

After P3 design approval, the **Dev track** (`/fullstack-dev`, phases D1–D10) builds the actual app. It has not started.

## Source-of-truth documents (read these first)

- `.claude-project/status/brandaid/seed-brandaid-mvp-v1.yaml` — the seed: goal, hard/soft constraints, acceptance criteria, and the **ontology** (authoritative definitions of BusinessProfile, Campaign, CampaignBrief, SimulationRun, Recommendation, LaunchPlan, Benchmark, etc.).
- `.claude-project/docs/PRD.md` — the canonical PRD: full functional spec, MVP scope, and explicit exclusions.
- `.claude-project/status/brandaid/PIPELINE_STATUS.md` — current phase, scores, deferred decisions.

## Defining product constraints (from the seed — easy to get wrong)

- **The stack is deliberately undecided.** Frontend + backend stack choice is deferred until after design approval. `.claude/rules/stacks/` ships NestJS and React rules, but **no stack is committed** — do not assume them.
- **The AI provider is OpenAI, not Anthropic/Claude** (`gpt-4o-mini` for vision/extraction, `gpt-4o` for reasoning narratives).
- **The forecasting engine (Layer 4) is fully deterministic** — a static benchmark JSON lookup, not ML. Identical inputs must always yield identical numbers, so the ROAS-flip is reproducible. Only creative vision scoring, recommendations, and reasoning text use real OpenAI calls.
- **Mock auth only** — a single hardcoded demo user. No JWT, no organizations, no multi-tenancy.
- MVP funnel stages are Awareness / Consideration / Conversion; Retention + Advocacy are "coming soon" placeholders.

## Design prototypes

`.claude-project/generated-screens/` holds standalone HTML prototypes — each fully self-contained (inline `<style>` and `<script>`, no build step): `landing.html`, `dashboard.html`, `new-campaign.html`, `simulation-results.html`.

**Palette migration is in progress.** The seed specifies a "Growth Emerald" palette (`#059669`), but the prototypes are being re-themed to **indigo `#4338CA` + cyan `#06B6D4`**. `landing.html` and `new-campaign.html` are migrated; `dashboard.html` and `simulation-results.html` are still emerald. When editing a screen, match the palette already present in that file. Inter-page navigation uses real `*.html` filenames.

### Previewing a prototype

`file:` URLs are blocked in the browser tooling — serve the folder over HTTP first, then drive it with `playwright-cli`. Node is available (Python is not):

```bash
# from .claude-project/generated-screens/ — any static HTTP server works
npx http-server -p 8765
playwright-cli goto http://127.0.0.1:8765/<page>.html
playwright-cli screenshot
```

`playwright-cli` is on PATH; `.playwright-cli/` collects its screenshots and logs.

## Quality gates

Each pipeline phase is checked by a gate script in `.claude/gates/`. Run one directly:

```bash
bash .claude/gates/<gate-name>.sh <target_dir>
```

Named gates referenced throughout the rule files (e.g. `swagger-all-controllers`, `no-dead-buttons`, `has-stable-testids`, `no-bad-waits`) are enforced when their phase runs.

## Phase & stack rules

`.claude/rules/` holds mandatory rules surfaced to Claude as project instructions: `common.rules.md`, `phases/*.rules.md` (one per pipeline phase), and `stacks/{base,nestjs,react}.rules.md`. They become authoritative once their corresponding phase begins. Note the rule files use a generic placeholder project name ("tirebank") — they still apply to BrandAid.
