# BrandAid — Product Requirements Document

**Version:** 1.0
**Date:** 2026-05-20
**Status:** Draft
**Source:** Generated from `seed-brandaid-mvp-v1.yaml` (P1-spec) + reference doc `brandaid_complete_knowledge.md`

---

## 0. Project Overview

### Product

**Name:** BrandAid
**Type:** Web App (SaaS Platform — MVP)
**Deadline:** Hackathon demo readiness
**Status:** Draft

### Description

BrandAid is an AI-powered marketing campaign intelligence platform that simulates campaign performance **before launch**. A marketer submits a domain URL; BrandAid auto-builds a BusinessProfile via a real crawl + OpenAI extraction, then guides the marketer through a conversational campaign brief flow (Awareness, Consideration, or Conversion goals). The output is a pre-launch prediction dashboard with reasoning, ranked recommendations, and a launch plan — letting the marketer iterate before spending a dollar of ad budget. The headline demo moment: applying 2 AI recommendations visibly jumps the predicted ROAS in a before/after comparison.

> **Core promise:** "Run your campaign before you run your campaign."

### Goals

1. Let an SME marketer go from a domain URL to a defensible pre-launch ROAS prediction in under 10 minutes.
2. Make the prediction explainable — every metric has reasoning, every recommendation has a predicted impact delta.
3. Demonstrate the "ROAS flip" hero moment: applied recommendations cause a visible upswing in the before/after view.
4. Get the post-P3 build to a state where it can be demoed end-to-end on any reasonable SME website without scripted content.

### Target Audience

| Audience | Description |
|---|---|
| **Primary** | Mid-market SME marketing managers (Mr. X persona — doc §5). Runs 3–5 campaigns/quarter, BDT 50K–500K budgets, channels Facebook/Instagram/WhatsApp/Google. Frustration: launches and waits 3 weeks to know if it worked. |
| **Secondary** | Startup founders wearing the marketing hat (BDT 8K–400K budgets, no marketing intuition). |
| **Tertiary** | Agency strategists pitching to clients (need predictive evidence). Hackathon judges as proxy users for the demo flow. |

### User Types

| Type | DB Value | Description | Key Actions |
|---|---|---|---|
| **Demo User** | `0` | The single hardcoded demo user (mock auth — no JWT, no orgs) | Submit URL, edit profile, run campaigns, apply recommendations |
| **Admin** | n/a | OUT of MVP scope (no admin dashboard) | — |

### User Status

| Status | DB Value | Behavior |
|---|---|---|
| **Active** | `0` | Default; the single demo user is always active |

> Mock auth means no real user accounts, suspension, or withdrawal. All persisted data is owned by the demo user.

### MVP Scope

**Included:**
- Stage 0 — Knowledge Base (real domain crawl + OpenAI extraction, editable profile, gap enrichment)
- Stage 1 — Campaign Brief (conversational; Awareness + Consideration + Conversion goals live)
- Stage 2 — Simulation (deterministic forecasting + real OpenAI creative vision + OpenAI reasoning)
- Stage 3 — Iteration (apply recommendations → rerun → before/after comparison; the ROAS flip)
- Stage 4 — Launch Plan (checklist + creative brief + tracking + optimization calendar)
- Currency toggle: BDT ↔ USD
- Multi-campaign persistence per the single demo user

**Excluded (deferred):**
- Stage 5 — Post-Launch Analysis (variance, learning loop)
- Retention + Advocacy funnel stages (cards visible but "coming soon")
- Real authentication / multi-user / organizations / agency tier
- Billing, pricing tiers
- Persona simulation (Layer 3 of the doc's pipeline)
- ML-based gradient-boosted forecasting (Layer 4 is replaced by deterministic benchmark lookup)
- Real-time platform data pulls (Meta / Google live integration)
- Mobile-native app
- Audience warmth probe, Conversion budget-floor / offer-benchmarking / attribution-check intelligence
- Dark mode (aspirational only)
- Multilingual UI (English-only)

---

## 1. Terminology

### Core Concepts

| Term | Definition |
|---|---|
| **BrandAid** | The product — pre-launch marketing campaign simulation platform |
| **BusinessProfile** | Persistent business knowledge per user, built from domain crawl + gap enrichment. Contains business basics, products/services, audience signals, brand voice, detected channels, confidence score, crawl metadata. |
| **Campaign** | One marketing campaign instance owned by a BusinessProfile. Contains a CampaignBrief, one or more SimulationRuns, and a LaunchPlan once accepted. |
| **CampaignBrief** | Structured input to a simulation: goal, sub-goal, selected campaign types, creative asset reference, audience calibration signal, budget, timing window, destination URL (Consideration only). |
| **SimulationRun** | One execution of the prediction pipeline against a brief snapshot. Holds predicted metrics with confidence intervals, evaluation scores, green/red flags, ranked recommendations. |
| **Recommendation** | Specific actionable change with predicted impact (delta on a key metric) and reasoning text. Applied recommendations modify brief inputs for the next SimulationRun. |
| **LaunchPlan** | Post-acceptance deliverable: setup checklist, creative brief, audience targeting summary, tracking setup, optimization calendar, risk flags. |
| **CreativeScore** | Structured score of an uploaded creative from OpenAI gpt-4o-mini vision. Components: attention, clarity, emotional resonance, CTA strength. Cached by image SHA256. |
| **ConfidenceScore** | 0–100 simulation-output score derived from BusinessProfile richness + calibration presence + benchmark coverage. Higher = tighter intervals on predicted metrics. |
| **Benchmark** | Static reference value (CPM, CTR, CVR, etc.) keyed on `(industry, channel, geography, business_model, season)` used by the deterministic forecasting layer. |

### User Roles

| Role | Description |
|---|---|
| **Guest** | Sees the landing page with the single URL input. After submitting a URL → becomes Demo User (mock auth implicit). |
| **Demo User** | Has access to everything: Profile, Campaigns list, New Campaign chat, all simulation views, Launch Plan. |

### Status Values

| Enum | Values | Description |
|---|---|---|
| **CampaignGoal** | `AWARENESS`, `CONSIDERATION`, `CONVERSION`, `RETENTION` (coming soon), `ADVOCACY` (coming soon) | The funnel stage of a campaign |
| **AwarenessSubGoal** | `NEW_BRAND`, `REPOSITIONING`, `PRODUCT_LAUNCH`, `MARKET_EXPANSION` | 4 sub-goals under Awareness |
| **ConsiderationSubGoal** | `LEAD_GENERATION`, `ENGAGEMENT_EDUCATION`, `TRAFFIC_INTENT` | 3 sub-goals under Consideration |
| **ConversionSubGoal** | `DIRECT_PURCHASE`, `SIGNUP_TRIAL`, `PROMOTIONAL_FLASH_SALE` | 3 sub-goals under Conversion |
| **CampaignType** | `SOCIAL_PAID`, `SOCIAL_ORGANIC`, `VIDEO`, `GOOGLE_SEARCH`, `GOOGLE_DISPLAY`, `INFLUENCER`, `CONTENT_SEO`, `EMAIL`, `WHATSAPP_SMS`, `RETARGETING` | Channel-specific tactics. Available types filtered by selected CampaignGoal. |
| **CampaignStatus** | `DRAFT`, `SIMULATED`, `LAUNCH_PLAN_READY` | Lifecycle of a campaign in MVP (no `LAUNCHED` / `POST_LAUNCH` states) |
| **CalibrationSignal** | `BELOW_AVERAGE`, `AROUND_AVERAGE`, `ABOVE_AVERAGE` | One-question past-performance calibration for all selected channels |
| **Currency** | `BDT`, `USD` | Display + benchmark currency |

### Technical Terms

| Term | Definition |
|---|---|
| **Crawl** | Backend visits up to 15 pages of a domain in priority order, extracts HTML, runs through OpenAI to populate BusinessProfile fields. |
| **Layer 1: Creative Scoring** | OpenAI gpt-4o-mini vision call scoring uploaded creative on attention / clarity / emotional resonance / CTA strength. Cached by SHA256. |
| **Layer 3: Persona Simulation** | OUT of MVP. (Reference doc describes synthetic audience reactions; deferred.) |
| **Layer 4: Forecasting** | Fully deterministic. Benchmark JSON lookup + adjustment formulas. No LLM. |
| **Layer 5: Recommendations** | OpenAI gpt-4o consumes numeric forecast outputs + green/red flag scoring → returns ranked recommendation list with impact and rationale. |
| **Synergy Multiplier** | 1.2–1.5x boost on combined recall lift when ≥2 campaign types are selected (doc §16). |
| **De-duplication Factor** | 0.65–0.85 multiplier on summed reach across channels to account for 15–35% audience overlap. |
| **Confidence Interval** | Low–high range on each predicted metric, widened or tightened by the ConfidenceScore. |
| **Hero Moment** | The before/after view after applying 2 recommendations and re-running the simulation; predicted ROAS visibly jumps. |

---

## 2. System Modules

### Module 1 — Knowledge Base (Stage 0)

The user submits a domain URL and BrandAid auto-builds a persistent BusinessProfile via a real crawl + OpenAI extraction. The user reviews and edits the profile, then optionally answers 5 gap-enrichment questions.

#### Main Features

1. **Single URL input** — landing page presents one input field, no wizard.
2. **Real domain crawl** — fetches up to 15 pages in priority order (homepage → about → products → pricing → blog → footer → testimonials → FAQ). Login-gated pages skipped.
3. **Animated crawl narration** — 7-step progress display while crawl runs (per doc §7).
4. **Profile card** — 5 editable sections: what you sell / who you sell to / your brand / your channels / business basics. Inline edits flag fields as `user_verified=true`.
5. **Confidence score** — 0–100 with bands: 85–100% rich, 65–84% good, 40–64% thin, <40% very limited.
6. **Gap enrichment** — 5 optional questions: monthly revenue range, past paid marketing experience + channels, biggest current marketing challenge, primary 90-day business goal, anything your website doesn't show. All skippable.
7. **Profile persistence** — survives across sessions; user can revisit and re-edit.

#### Technical Flow

##### Crawl + Extract Flow

1. User submits domain URL.
2. Frontend posts to `POST /crawl` and immediately opens a streaming endpoint (SSE or polling) to receive narration events.
3. Backend resolves robots.txt, fetches homepage, parses links, prioritizes paths matching the 8 priority categories.
4. For each page (max 15): fetch HTML, strip boilerplate, extract content. Login walls → skip. JS-heavy → fall back to inner text only.
5. Aggregate text bundles per category → call OpenAI gpt-4o-mini with structured output schema: `{business_basics, products_services, audience, brand, channels, content_signals, confidence_components}`.
6. Compute overall confidence score from coverage of each field.
7. Persist `BusinessProfile` row. Return profile + confidence.
8. On success: navigate to Profile Review page. On crawl timeout (>30s) or extraction error → fall back to manual 4-question onboarding (free-text inputs).

##### Profile Edit Flow

1. User opens Profile Review page → sees 5 section cards.
2. User clicks any field → inline editable input (text/select per field).
3. On blur or Enter → `PATCH /profile` updates field + sets `user_verified=true` on that field.
4. Re-computed confidence reflects user-verified weighting.

##### Gap Enrichment Flow

1. After profile review, user sees "Tell us a bit more" card with 5 questions.
2. Each question can be skipped. Submitted answers stored on the BusinessProfile.
3. Done CTA → "Start your first campaign" → navigate to Campaign Goal selection.

---

### Module 2 — Campaign Brief (Stage 1)

Conversational chat-style UI where the user defines their campaign step by step. A right-hand side panel displays the brief building up in real time. Each goal selection branches into sub-goals + filtered campaign types.

#### Main Features

1. **Goal selection grid** — 5 stage cards. Awareness / Consideration / Conversion clickable. Retention + Advocacy show "Coming soon" badges.
2. **Sub-goal branch** per goal: 4 for Awareness, 3 for Consideration, 3 for Conversion.
3. **Campaign type selection (chips)** — filtered to types valid for the chosen goal (per doc §13).
4. **Multi-platform calibration** — ONE past-performance question covers all selected channels. Optional anchor data point.
5. **Landing page scoring (Consideration only)** — backend fetches destination URL, scores above-fold clarity, CTA, form friction, load signal, message match. Score < 6/10 flagged.
6. **Creative upload** — JPG/PNG. Stored locally. Real OpenAI gpt-4o-mini vision scores attention / clarity / emotional resonance / CTA strength. SHA256-cached.
7. **Budget slider + timing picker.**
8. **Right-side brief panel** — updates live as user answers each prompt.
9. **Submit** — creates a Campaign + initial SimulationRun.

#### Technical Flow

##### Conversational Brief Flow

1. User clicks "New Campaign" from Profile or Campaigns list.
2. AI message: "What's the main goal of this campaign?" → goal grid renders.
3. User selects goal → AI follows up with sub-goal cards filtered to that goal.
4. User selects sub-goal → AI presents campaign type chips (filtered to that goal/sub-goal per doc §13).
5. User picks ≥1 campaign type.
6. AI asks the multi-platform calibration question (one signal for all channels).
7. AI prompts for creative upload (drag-drop / file picker).
8. Frontend POSTs creative to `POST /creative/score` → backend stores file, hashes bytes, checks cache, otherwise calls OpenAI vision, returns structured score.
9. AI displays brief commentary based on creative score.
10. AI asks budget (slider) + timing (date range picker).
11. **If goal === CONSIDERATION**: AI also asks for destination URL → backend `POST /lp/score` returns landing-page score.
12. AI summarizes brief in side panel; user confirms.
13. `POST /campaigns` creates Campaign + triggers first SimulationRun. Navigate to Simulation Output page.

---

### Module 3 — Simulation Output + Iteration (Stages 2 + 3)

Renders the prediction dashboard. Shows 4 headline metric cards, per-metric reasoning, green/red flags, per-channel breakdown, confidence, and a ranked recommendation list. User can apply recommendations and rerun → before/after view (the hero moment).

#### Main Features

1. **4 headline metric cards** — value + low–high confidence interval. Per goal: Awareness shows (Reach, CPM, Frequency, Brand Recall Lift); Consideration shows (CTR, CPC, CPL, Lead Volume); Conversion shows (CVR, CPA, ROAS, Estimated Revenue).
2. **Per-metric "why" tooltip** — 1–2 sentence reasoning generated by OpenAI gpt-4o from numeric outputs.
3. **Green flags panel** — top 3 working signals (creative voice, audience fit, etc.) with short rationale.
4. **Red flags panel** — top 3 suppressing signals with short rationale.
5. **Per-channel breakdown table** — appears when ≥2 campaign types are selected; shows per-channel metrics + audience de-duplication note + synergy multiplier applied.
6. **Overall confidence score** — visible with band label.
7. **Ranked recommendation list** (3–5 items) — each shows predicted impact delta (e.g. "+0.6x ROAS", "-15% CPL") + 1–2 sentence reasoning + select-checkbox.
8. **"Apply & Rerun" button** — submits selected recommendations, creates a new SimulationRun with adjusted inputs.
9. **Before/After comparison view** — side-by-side metric cards from previous vs new SimulationRun; deltas highlighted in emerald (positive) or red (negative). The ROAS flip lives here.
10. **Accept & Generate Launch Plan** — promotes the current SimulationRun to LaunchPlan generation.

#### Technical Flow

##### Initial Simulation Flow

1. On `POST /campaigns` creation, backend triggers `runSimulation(campaign_id)` synchronously (target: <5s).
2. Layer 4 (forecasting) — deterministic. Inputs: industry, channel(s), geography, budget, calibration signal, creative score (Layer 1 result), business model. Output: predicted metrics + confidence intervals via benchmark JSON lookup + adjustment formula.
3. Multi-type math (when ≥2 types):
   - Compute independent per-type forecasts.
   - Combined reach = sum_of_reaches × dedup_factor (0.65–0.85).
   - Combined recall lift = base × synergy_multiplier (1.2–1.5).
   - Per-channel breakdown stored on SimulationRun.
4. Evaluation scoring — score campaign against the 10 weighted criteria for the goal (doc §15). Surface top 3 green flags, top 3 red flags.
5. Recommendations — Layer 5. Backend builds a structured prompt containing the brief, numeric forecast, flags. Calls OpenAI gpt-4o with JSON schema for recommendations. Each recommendation includes a structured "adjustment" payload (e.g., `{type: "budget_reallocation", from: "Meta", to: "Google", amount_pct: 20}`) used in rerun.
6. Persist SimulationRun with `runIndex: 0`.
7. Return predictions to frontend.

##### Apply & Rerun Flow

1. User toggles recommendation checkboxes + clicks "Apply & Rerun".
2. `POST /campaigns/:id/simulate` with `applied_recommendation_ids: [...]`.
3. Backend reads the previous SimulationRun, applies the recommendations' adjustment payloads to the brief snapshot (mutating budget split, creative substitution, etc.).
4. Re-run Layers 1 → 4 → 5 (Layer 1 only re-runs if a recommendation swapped the creative; otherwise reuse cached score).
5. Forecasting is fully deterministic so same inputs always produce same outputs (reproducibility requirement).
6. Persist new SimulationRun with `runIndex: previous + 1` + `applied_from_run_index: previous`.
7. Frontend navigates to Before/After view, animates metric value transitions.

##### Launch Plan Generation Flow

1. User clicks "Accept & Generate Launch Plan" on the current SimulationRun.
2. `POST /campaigns/:id/launch-plan` triggers generation.
3. Backend assembles a structured plan from the SimulationRun + brief: setup checklist (campaign goals, audiences, creatives to ship), creative brief summary (re-using Layer 1 score reasoning), audience targeting summary (segments + channels), tracking setup (pixels / UTMs / conversion events), 2–4 week optimization calendar (per-day actions), risk flags (the red flags from the simulation).
4. The narrative sections (creative brief, optimization calendar text) are OpenAI gpt-4o-generated; structured sections (checklist, targeting summary, risk flags) are deterministic from data.
5. Persist LaunchPlan. Navigate to Launch Plan page.

---

### Module 4 — Launch Plan (Stage 4)

Final deliverable: an actionable, copyable, downloadable launch plan derived from the accepted SimulationRun.

#### Main Features

1. **Setup checklist** — itemized tasks the user must complete (set up Meta Pixel, create creative assets, define UTMs, etc.).
2. **Creative brief summary** — what the creative should communicate; tied to creative score insights.
3. **Audience targeting summary** — per channel + per platform setup notes.
4. **Tracking setup** — pixels, conversion events, UTM scheme.
5. **Optimization calendar** — 2–4 week per-day actions (e.g., "Day 3: Pause underperforming creative, scale top 2 ad sets").
6. **Risk flags** — carried over from simulation red flags.
7. **Copy + Download** — copy-to-clipboard or download as Markdown.

#### Technical Flow

1. User opens `/campaigns/:id/launch-plan` page.
2. If LaunchPlan exists → render. Otherwise → show "Generate" CTA → triggers generation flow above.
3. Each section is collapsible.
4. Copy button copies full Markdown. Download button serves `.md` file.

---

## 3. User Application

### 3.1 Page Architecture

**Stack:** TBD — frontend stack decision deferred until after P3-design approval. Routes, page names, and feature lists below are stack-agnostic.

#### Route Groups

| Group | Access |
|---|---|
| Public | Anyone (landing + URL submission) |
| Onboarding | Post-crawl profile flow |
| Protected | Demo user (mock auth) |

#### Page Map

**Public**

| Route | Page |
|---|---|
| `/` | Landing — single URL input |

**Onboarding**

| Route | Page |
|---|---|
| `/crawl/:profileId` | Crawl progress (live narration) |
| `/profile/review` | BusinessProfile review (5 editable sections) |
| `/profile/enrich` | Gap enrichment (5 optional questions) |

**Protected** (post-onboarding)

| Route | Page |
|---|---|
| `/dashboard` | Demo user home — profile snapshot + campaigns list + "New Campaign" CTA |
| `/profile` | View / edit current BusinessProfile (revisit anytime) |
| `/campaigns` | Campaigns list (cards: name, goal, status, last simulation snapshot) |
| `/campaigns/new` | New Campaign — conversational brief flow |
| `/campaigns/:id` | Simulation output dashboard + ranked recommendations + Apply & Rerun |
| `/campaigns/:id/compare` | Before/After comparison view (after rerun) |
| `/campaigns/:id/launch-plan` | Launch Plan |
| `/settings` | Currency toggle (BDT/USD) + reset profile |

> P3-design target: ~12–16 distinct HTML pages. Above lists 12. Optional designs: Recommendation detail drawer, Creative score detail modal, Profile section edit drawer, Empty states for campaigns / profile — these can be modals within the listed pages.

---

### 3.2 Feature List by Page

#### `/` — Landing

- BrandAid logo + emerald-toned hero
- One-line value prop: "Predict your campaign's ROI before you spend a taka."
- Single input: domain URL field + "Build my profile" primary CTA
- Subtle social proof / brand line at bottom
- Currency toggle (BDT/USD) appears subtle in footer for global visitors
- No nav menu, no signup link (mock auth)

---

#### `/crawl/:profileId` — Crawl Progress

- Full-screen narration view
- 7 animated progress lines (per doc §7): "Reading your homepage… Discovering your products and services… Understanding who you sell to… Mapping your brand positioning… Finding your marketing channels… Analysing your brand voice and tone… Building your profile…"
- Each line emerald-colored when complete, gray when pending
- Progress bar at top
- On completion → auto-navigate to `/profile/review`
- On failure → friendly fallback message + "Enter manually" CTA

---

#### `/profile/review` — BusinessProfile Review

- Header: business name + tagline + confidence score badge (Rich/Good/Thin/Limited)
- 5 sections as collapsible/editable cards:
  1. **What you sell** — products/services, prices, business model
  2. **Who you sell to** — detected audience, pain points, triggers
  3. **Your brand** — tone keywords, differentiators, visual style
  4. **Your channels** — detected platforms, blog/email signals
  5. **Business basics** — name, industry, geography
- Each field: click-to-edit inline (text input / dropdown depending on type)
- Edited fields highlighted with small "verified by you" tag
- Bottom CTA: "Continue → Tell us a bit more"

---

#### `/profile/enrich` — Gap Enrichment

- 5 optional questions, each skippable:
  1. Monthly revenue range (selector)
  2. Past paid marketing experience + channels used (multi-select)
  3. Biggest current marketing challenge (selector)
  4. Primary 90-day business goal (selector)
  5. Anything your website doesn't show (free text, optional)
- Progress dots at top
- "Skip all" link + "Continue to your first campaign" primary CTA

---

#### `/dashboard` — Demo User Home

- Greeting line ("Welcome back, [Business Name]")
- Profile snapshot card: confidence band, key fields, "Edit profile" link → `/profile`
- Campaigns list (cards): name (or auto-generated), goal, status, last predicted ROAS / Reach
- Empty state: "No campaigns yet — let's run your first one." with "New Campaign" CTA
- Primary CTA: "New Campaign" → `/campaigns/new`
- Currency toggle visible in header

---

#### `/profile` — Profile (Re-visit / Edit)

- Same 5 editable sections as `/profile/review`
- Confidence score visible
- "Re-crawl" action (optional stretch — out of MVP unless trivial)
- "View enrichment answers" link

---

#### `/campaigns` — Campaigns List

- Search (by name)
- Filter by goal (Awareness / Consideration / Conversion / All)
- Sort by last updated (default)
- Each card: name, goal badge, sub-goal, last predicted key metric (ROAS or CPL or Recall Lift depending on goal), date
- Card actions: Open → `/campaigns/:id`, Delete (with confirm)
- Empty state with "New Campaign" CTA
- Top-right "New Campaign" button

---

#### `/campaigns/new` — New Campaign (Conversational Brief)

- Layout: left = chat thread, right = live brief panel
- Initial AI message + goal selection grid (5 cards — only 3 active, 2 "coming soon")
- After goal: sub-goal selection grid
- After sub-goal: campaign type chips (multi-select), filtered to goal
- Calibration question (one signal for all channels)
- Creative upload prompt (drag-drop)
- Creative score commentary message
- Budget slider + timing date-range picker
- If goal === Consideration: destination URL input + score response
- Final AI confirmation: "Ready to simulate?" → "Run simulation" CTA
- Right panel mirrors brief: goal, sub-goal, channels, calibration, creative thumbnail + score, budget, timing, destination URL (if applicable)
- Back / Edit on any field
- On submit: navigate to `/campaigns/:id`

---

#### `/campaigns/:id` — Simulation Output

- Header: campaign name + goal + sub-goal badges + "Edit brief" / "Apply & rerun" actions
- **Top section — 4 headline metric cards**: large numeric value + low–high confidence interval + "why" tooltip icon
- **Confidence score band** (somewhere in the header strip)
- **Two panels side by side**: Green flags (top 3) + Red flags (top 3) — each flag has 1–2 sentence rationale
- **Per-channel breakdown table** (if multiple types): channel name, predicted CPM/CTR/CVR/spend share/share of reach. Footer row: combined + dedup factor + synergy multiplier
- **Ranked recommendations** (3–5): each card has rec title, predicted delta badge (e.g. "+0.6x ROAS"), 1–2 sentence rationale, select-checkbox
- **Bottom action bar**: Apply & Rerun (primary, disabled until ≥1 selected), Accept & Generate Launch Plan (secondary)

---

#### `/campaigns/:id/compare` — Before/After Comparison

- Header: "Run 1 vs Run 2" (or N vs N+1)
- 4 metric cards × 2 columns (before, after) with delta in middle
- Animated number transition on entry
- ROAS card visually emphasized for the hero moment
- Below: list of applied recommendations + their actual delta contribution
- CTA: "Run again" (back to `/campaigns/:id`) or "Accept & Generate Launch Plan"

---

#### `/campaigns/:id/launch-plan` — Launch Plan

- Header: campaign name + accepted simulation reference
- 6 collapsible sections:
  1. Setup checklist (checkbox list)
  2. Creative brief summary
  3. Audience targeting summary (per channel)
  4. Tracking setup (pixels, UTMs, conversion events)
  5. Optimization calendar (week-by-week timeline)
  6. Risk flags (carried from simulation red flags)
- Top-right actions: Copy to clipboard, Download Markdown

---

#### `/settings` — Settings

- Currency toggle: BDT ↔ USD (affects all benchmarks + budget displays)
- "Reset demo profile" — wipes BusinessProfile + Campaigns (for fresh demos)
- About / version info

---

## 4. Admin Dashboard

> **Not included in MVP.** Mock auth, single demo user, no organizations, no platform admin. This section intentionally omitted.

---

## 5. Tech Stack

### Architecture

**Frontend + Backend stack: DEFERRED.** The stack choice is deliberately deferred until after P3-design (HTML approval). PM artifacts must remain stack-agnostic on functional content. After P3, the developer will choose between the project's default NestJS + React Router 7 (per `.claude/rules/stacks/`), Next.js (per the BrandAid reference doc §18), or a hybrid. A reminder will surface before `/fullstack-dev` is invoked.

```
brandaid/
├── backend/        ← API server (stack TBD post-P3)
└── frontend/       ← User-facing web app (stack TBD post-P3)
```

### Technologies — Stack-Agnostic Requirements

| Layer | Requirement | Notes |
|---|---|---|
| Language (backend) | TypeScript | Strict typing required |
| Language (frontend) | TypeScript | Strict typing required |
| API style | REST/JSON | OpenAPI/Swagger doc auto-generated (if NestJS chosen, per project rules) |
| Database | PostgreSQL | Per seed constraint; via TypeORM if NestJS is chosen |
| File storage | Local filesystem (`uploads/`) | NOT S3 — explicit downgrade for MVP |
| AI provider | OpenAI API | gpt-4o-mini for vision + crawl extraction; gpt-4o for reasoning narratives |
| Auth | None (mock) | Single hardcoded demo user. No JWT, no cookies, no orgs |
| Styling | Tailwind CSS | Growth Emerald palette per doc §17 |
| Component library | shadcn/ui (or equivalent) | If React-based |
| Animation | Lightweight (Framer Motion or CSS) | Crawl narration + metric transitions |
| Charts (Launch Plan calendar) | Recharts or equivalent | |

### Third-Party Integrations

| Service | Purpose |
|---|---|
| OpenAI API (gpt-4o-mini) | BusinessProfile field extraction from crawled HTML; creative vision scoring |
| OpenAI API (gpt-4o) | Per-metric reasoning text; recommendation generation; launch plan narrative sections |

### Key Decisions

| Decision | Rationale |
|---|---|
| OpenAI over Claude (overrides doc §18) | Explicit user choice in P1 interview |
| Layer 3 (persona simulation) OUT | Cuts LLM cost + variance; doc's persona narrative reframed as deterministic flag scoring |
| Layer 4 deterministic | Reproducible hero moment — same inputs always yield same ROAS. No ML model needed for MVP |
| Mock auth | Demo simplicity; conflicts with project's nestjs.rules.md but acceptable for hackathon |
| Local storage over S3 | One fewer service dependency for the demo machine |
| PostgreSQL kept | Aligns with project conventions and is realistic for the SaaS positioning |
| BDT+USD toggle | Honors doc's Bangladesh-first persona while staying global-ready |

### Environment Variables

| Variable | Description |
|---|---|
| `DATABASE_URL` | PostgreSQL connection string |
| `OPENAI_API_KEY` | OpenAI API authentication |
| `OPENAI_VISION_MODEL` | Default `gpt-4o-mini` |
| `OPENAI_REASONING_MODEL` | Default `gpt-4o` |
| `UPLOAD_DIR` | Local filesystem path for creative uploads (default `./uploads`) |
| `CRAWL_TIMEOUT_MS` | Max wall-clock time per crawl (default `30000`) |
| `CRAWL_MAX_PAGES` | Default `15` |
| `BENCHMARKS_PATH` | Path to static benchmark JSON dataset (default `./data/benchmarks.json`) |
| `FRONTEND_URL` | Frontend base URL (CORS) |
| `BACKEND_URL` | Backend base URL |
| `DEFAULT_CURRENCY` | `BDT` or `USD`; default `USD` |
| `DEFAULT_GEOGRAPHY` | Default `BD` for benchmark lookup when unknown |

### Benchmark Dataset Requirements

The static `benchmarks.json` is the heart of the deterministic forecasting layer. Minimum coverage:

- **Industries (10+)**: Modest fashion, D2C apparel, F&B / restaurant, SaaS, E-learning, Healthtech, Real estate, Local services, Beauty / cosmetics, B2B services, etc.
- **Channels (8+)**: Meta (FB+IG), Google Search, Google Display, TikTok, LinkedIn, YouTube, Email, WhatsApp, Influencer.
- **Geographies (5+)**: BD, IN, US, UK, Generic.
- **Business models (4)**: D2C, SaaS, Services, Local.
- **Seasonal indices**: Eid, Ramadan, Christmas, Fiscal year-end.
- Each `(industry, channel, geo, model)` tuple stores typical ranges for CPM, CTR, CVR, CPL, ROAS, frequency.

Exact numeric values are deferred to the D-phase (database/backend implementation).

---

## 6. Open Questions

| # | Question | Context / Impact | Owner | Status |
|:-:|---|---|---|---|
| 1 | Frontend + backend stack? | Choose between NestJS+React Router 7 (project rules), Next.js (reference doc), or hybrid. Affects all D-phase scaffolding. | User | ⏳ Deferred until after P3-design approval |
| 2 | Exact numeric values in `benchmarks.json` | Forecasting fidelity depends on realistic benchmark coverage. Shape is locked; numbers are dev-time. | Dev | ⏳ Deferred to D-phase |
| 3 | Recommendation adjustment payload schema | When a recommendation is applied, what's the exact JSON shape that mutates the brief? (budget shift / channel swap / creative substitution / copy tweak). Needs concretization at D-phase. | Dev | ⏳ Open |
| 4 | Creative substitution UX | If a recommendation says "Try a different creative", does the user re-upload mid-rerun, pick from a library, or is the recommendation purely advisory? | User | ⏳ Open |
| 5 | Crawl fallback details | When the crawl fails (JS-heavy site, login wall, timeout): exact UI copy + which manual questions cover the gap? | User | ⏳ Open |
| 6 | Launch plan download format | Markdown only, or also PDF? | User | ⏳ Open (Markdown default) |
| 7 | Demo reset behavior | "Reset demo profile" — does it wipe everything or just the BusinessProfile (keeping Campaign history)? | User | ⏳ Open (default: full wipe) |
| 8 | Confidence interval math | Doc gives bands (80–100%, etc.) but not the formula for the per-metric interval width. Need to define: e.g., `width = base × (1 – confidence/100) × volatility_factor`. | Dev | ⏳ Open |

---

## Appendix A — Reference Doc Departures

This PRD differs from the original `brandaid_complete_knowledge.md` reference doc in four important ways. These were explicit user decisions in the P1 interview:

| # | Topic | Reference Doc Said | PRD Says | Why |
|---|---|---|---|---|
| 1 | AI provider | Claude (Anthropic) | OpenAI | User preference |
| 2 | Simulation realism | All 6 layers active; personas + ML forecasting | Layers 1+5 real OpenAI; Layer 3 OUT; Layer 4 deterministic | Reproducible demo + lower complexity |
| 3 | Stage scope | Awareness + Conversion (doc §20 MVP) | + Consideration | More complete demo |
| 4 | Tech stack | Next.js + TypeScript + Tailwind + shadcn | DEFERRED until after P3 | Decision delegated to developer post-design |

Other reference doc items that are intentionally **out of MVP scope**: Retention stage, Advocacy stage, Stage 5 (Post-Launch), persona simulation, audience warmth probe, Conversion budget-floor warning, offer benchmarking, attribution check, dark mode, multilingual UI, mobile-native app, real platform integrations, billing/tiers, agency multi-brand.

Items that are kept verbatim: Growth Emerald color palette, Arial typography, 6-layer pipeline shape (just with Layers 3+4 modified), all metrics master reference (doc §14), all 10 evaluation criteria per stage (doc §15), multi-type forecasting math (doc §16).
