# Hyperhunt OS

## Product Vision v1.0

**Status:** Draft for review
**Owner:** Founder
**Governance:** Built under HSES (constitution and operating manual apply)

---

# One-Liner

**The recruiter's operating system: an AI-in-the-background system of record that eliminates clerical work from the hiring pipeline and turns operations into shared intelligence.**

What Notion did for organizing knowledge, Hyperhunt OS does for the hiring pipeline and recruiting operations — a small set of opinionated primitives, composable views, and automation that makes the software disappear behind the workflow.

---

# The Problem

Recruiting agencies run on Google Sheets and WhatsApp groups. This creates three compounding costs:

1. **Duplicate clerical work.** Every candidate, stage change, and client update is typed multiple times across sheets, messages, and memory. Consultants spend a large share of their day on data entry that produces no placement.
2. **Blackbox operations.** Pipeline status, bottlenecks, and team load live in people's heads and scattered cells. Every operational question is answered by interrupting someone. Intelligence exists only as tribal knowledge.
3. **Client-side latency.** Roles stall on the client's side — delayed feedback, unconfirmed interviews, slow approvals — and the agency has no instrument to surface this with evidence. Chasing is manual, awkward, and ignored.

Incumbent ATS/CRM products solve these problems by addition: dozens of modules, most of which are bloat. They are candidate-record systems with AI bolted on top of CRUD data, built around email and portals rather than how Indian agencies actually operate.

---

# Vision

Hyperhunt OS is **New Age Software for recruiters**, defined by three commitments:

1. **Elimination over addition.** Every capability must delete a category of manual work. If a feature adds capability without removing a chore, it is bloat and does not ship.
2. **AI in the background.** No chat windows, no copilots demanding attention, no autonomous personas. AI works invisibly — parsing, summarizing, composing, flagging — as a silent consumer of the system's event stream. The user experiences outcomes, not AI.
3. **Operations as shared intelligence.** The same system of record that eliminates data entry produces the decision layer — for consultants, for leadership, and, in curated form, for clients. Intelligence is never a blackbox because every metric is traceable to the events beneath it.

The long arc: Hyperhunt OS begins as Hyperhunt's internal operating system, matures through daily use and consultant feedback, and becomes the first face of the Hyperhunt AI SaaS — sold to agencies and hiring teams who live the same pain. It is also the structural skeleton that the future relevance engine (Idea 1) attaches to.

---

# Strategic Positioning

## Phase strategy

| Phase | Audience | Purpose |
|---|---|---|
| **Internal** (now) | Hyperhunt consultants and leadership | Replace sheet-based operations; refine through daily dogfooding; accumulate the event history that powers intelligence |
| **External** (later) | Agencies and hiring teams (the ICP) | Productize as the Hyperhunt AI SaaS once internally undeniable |

Internal-first is a deliberate advantage, not a delay: Hyperhunt is customer zero, feedback cycles are measured in hours, and the product is proven on a live desk before a single sales conversation.

## Why build instead of buy

Off-the-shelf ATS/CRMs could replace the sheets. Building is justified only because this is a **product, not a tool**:

- The data model, event log, and outcome data must live in Hyperhunt's own schema — they are the substrate for the relevance engine and future intelligence.
- The differentiation is architectural (role-centric model, event-log foundation, elimination UX, India-native channels), which cannot be configured into a rented system.
- Dogfooding an owned product compounds; configuring a vendor's product does not.

## Differentiation (architectural, not feature-based)

1. **Role-centric, not candidate-centric.** The Hiring Pipeline belongs to the Role. Consultants open roles, not candidates; candidates participate in hiring processes. Incumbents cannot adopt this without rebuilding their schema.
2. **Event log as foundation.** Every action is a structured event; everything else (timelines, metrics, dashboards, nudges, AI) is a view or consumer of the stream. Incumbents bolt AI onto records; here the AI is born inside a complete, structured history.
3. **Opinionated data, composable views.** The Notion lesson applied correctly: flexibility at the view layer (filter, group, save), opinionation at the data layer (fixed spine, enforced pipeline semantics, automatic capture). This is precisely why "just use Notion" fails for hiring ops — Notion is neutral and manual where this system is opinionated and automatic.
4. **India-native.** WhatsApp is the first-class client channel. Workflows assume Indian agency mechanics. Western incumbents structurally underserve both.

---

# Objectives

## Internal phase objectives

1. Eliminate duplicate data entry from candidate intake to role closure.
2. Retire the pipeline-tracking spreadsheet on every active mandate.
3. Give every consultant a single workspace covering 90% of their day.
4. Replace "asking" with the decision layer: pipeline status, bottlenecks, ageing, and load visible without interrupting anyone.
5. Reduce role-closure latency by surfacing client-side pendency with evidence and nudges.
6. Accumulate a complete, structured event history of Hyperhunt's operations.

## Long-term objectives

1. Become the system of record and intelligence layer sold to the ICP as the Hyperhunt AI SaaS.
2. Serve as the attachment skeleton for the relevance engine (calibration, profiles, outcomes as event-stream citizens).
3. Build the compounding assets: the Candidate Bank (one candidate, one profile, forever) and the outcome-labeled event history that no competitor can replicate.

---

# Users & Access

Hyperhunt OS serves three actors. Every module exists for the overall business; each actor meets it at a different altitude over the same event stream:

| Actor | Uses the OS to… | Scope |
|---|---|---|
| **Consultant** | Operate: run roles, move candidates, compose briefs and packs, work the Today queue | Own book of work |
| **Delivery Lead** | Orchestrate: balance load, unblock bottlenecks, review desks, uphold SLAs | All desks |
| **CEO** | Decide: pipeline health, client health, placements, where to intervene | Whole business |

Access is tiered by these three roles — scope of aggregation differs; the numbers do not. A consultant's status figure and the CEO's rollup are the same events viewed at different altitudes, which is what makes reviews run on shared truth instead of prepared reporting. Client access remains excluded from Phase 1 (briefs travel via WhatsApp); a Senior Consultant tier is added only if a real permission difference from Consultant ever emerges.

## Identity & Access Model (Sprint 1 essentials)

Every user is an account. Every account carries an `org_id` (single-tenant now, tenancy-ready by design) and one of three `role` values: `consultant`, `delivery_lead`, `ceo_admin`.

The database enforces scope, not application code. Every table with an `org_id` also carries **Supabase row-level security policies** that filter by role and ownership: consultants see roles assigned to them and their linked candidates; delivery leads see all roles in the org; CEO/admins see everything. The same query executed by different users returns different rows — no `if role == X` gymnastics in the codebase, no accidental leakage.

**Minimum viable admin, Phase 1:** the CEO/admin creates the two other user accounts once, from the Supabase dashboard. Three accounts, one-time work. No user-management UI, no invitation flow, no password-reset screens — Supabase Auth handles the primitives; anything else is a backlog item until a real user-management chore exists to eliminate.

---

# Core Primitives

The entire system is built from five objects and one stream. Nothing else is primitive; everything else is derived.

| Primitive | Definition |
|---|---|
| **Client** | An organization Hyperhunt serves. Owns roles, team contacts, preferences. |
| **Role** | A mandate. The primary unit of work. Owns exactly one Hiring Pipeline, a JD, priority, consultant assignment, and (future) a calibration slot for the relevance engine. |
| **Hiring Pipeline** | The staged process belonging to a Role: New → Screening → Submitted → Interview rounds → Offer → Joined / Rejected. The consultant's primary workspace. |
| **Candidate** | A person, stored once in the Candidate Bank, deduplicated, linked to any number of roles as a participant. |
| **Event** | An immutable, timestamped, structured record of any action: upload, stage move, feedback, submission, note, nudge. Carries actor, object references, and a **visibility dimension** (internal / client-safe). |

**The Event Log** is the system's foundation. Timelines, stage history, metrics, dashboards, client briefs, nudges, and all background AI are views over or consumers of this stream. If a capability cannot be expressed as a producer or consumer of events, it does not belong in the architecture.

---

# Modules & Capabilities

Each module is defined by what it eliminates.

## 1. The Spine (Client → Role → Pipeline → Candidate Bank)

**Eliminates:** scattered records; "which sheet is this on?"

- Client list and profile (status, team, notes, communication preferences)
- Role workspace with JD, hiring manager, priority, consultant, open/close dates
- Global search via database full-text (a capability, not a module)

## 2. Candidate Bank

**Eliminates:** re-sourcing people Hyperhunt already knows; candidate knowledge dying when a role closes.

The complementary lens to the role-centric pipeline — the system's two views of the same events:

- **Role-centric (Hiring Pipeline)** — daily execution: open the role, manage its process
- **Candidate-centric (Candidate Bank)** — long-term talent intelligence: open the person, see everything

Capabilities:

- One candidate, one profile, forever — dedup on entry (exact match on email/phone; AI fuzzy-match fallback)
- Complete cross-role history per candidate: every role participated in, every stage reached, every feedback received, source and consultant — all derived from the event log, never entered
- Bank-wide search across skill, company, experience, and role history
- Closed roles feed the bank instead of ending the relationship: rejected-but-strong candidates become the first pool searched on the next mandate
- **Conflict warnings, derived free from the event log:** linking a candidate to a role surfaces "active in [role] with [consultant]" or "previously submitted to this client on [date]" — preventing duplicate submissions and internal collisions before they embarrass anyone
- The compounding asset: every mandate enriches the bank, and the bank is the retrieval pool the future relevance engine ranks against

## 3. Magic Upload

**Eliminates:** duplicate data entry — the founding chore.

One action: consultant uploads a CV to a role. The system automatically:
parses the resume → creates or matches the candidate → links candidate to role → places them in the pipeline → records source and timestamp → emits events → generates the AI candidate summary → makes everything searchable.

One upload. Eight side effects. Zero re-typing. This flow is the product's thesis in miniature; if it works, the philosophy is validated.

## 4. Pipeline & Movement

**Eliminates:** status bookkeeping.

- Drag-and-drop stage movement as the only manual act
- Automatic stage history, timestamps, time-in-stage, consultant attribution
- Candidate cards: name, company, experience, stage, source, resume, last activity, last feedback — minimal and actionable
- Candidate timeline and role activity timeline as derived views (never entered)
- Simple feedback capture: interview → feedback → decision → notes; nothing more

## 5. Consultant Workspace (My Desk)

**Eliminates:** assembling your day — the morning scan across sheets, WhatsApp groups, and memory to figure out what needs doing, for whom, in what order.

The consultant's home screen and the product's primary surface. One space holding a consultant's entire book of work:

- **My roles** — the portfolio across all clients: priority, pipeline summary, ageing flags, next action per role
- **My candidates in motion** — everyone the consultant is currently moving, across all roles
- **My clients** — owned accounts with latest activity and pending briefs
- **Today** — an automatically assembled work queue: interviews scheduled, feedback due, nudges triggered, briefs to send
- **Prioritization is computed, not typed.** The queue orders itself from role priority, SLA nudges, and ageing. Consultants can pin items; they are never obliged to maintain a task list — a manual todo layer would be data entry by another name, and it would rot exactly like the sheets did
- Every item deep-links into its Role's Hiring Pipeline: My Desk is where the day starts; the pipeline is where the work happens

Architecturally this is still a view — the consultant-scoped lens over the same primitives and event log — which is why it stays cheap to build and always consistent with what leadership sees.

## 6. Composers (Client Briefs · Submission Packs · Candidate Messages)

**Eliminates:** update-typing to clients; CV reformatting before submission; repetitive candidate messaging.

One composition capability, three outputs, all delivered copy-to-paste (zero behavior change for anyone):

**Client Brief** — updates in Hyperhunt's voice from client-safe events: pipeline snapshot, movement since last update, what's pending on whose side, one nudge. Curated, not raw: shows pendency, never blame; the framing is always "here's what unblocks your role."

**Submission Pack** — the largest single clerical chore in agency work, eliminated: one click at the Submitted stage generates the branded Hyperhunt submission CV from the already-parsed profile (standard template, candidate contact details stripped), plus the AI candidate summary and the consultant's note. What took 20–40 minutes per candidate becomes a review-and-send.

**Candidate Messages** — interview confirmation with details and prep pointers, offer congratulations, joining-date reminders. Composed from events, in Hyperhunt's voice, one tap to copy into the consultant's existing WhatsApp thread.

Direct WhatsApp Cloud API delivery remains a deliberate deferral to the external phase (requires Official Business Account and group migration).

## 7. The Decision Layer

**Eliminates:** asking. Turns the blackbox into shared intelligence.

Not dashboards as separately-built artifacts — **one view system over the event log** shipping with three named internal dashboards plus the client projection. Each is a configuration of the same primitives, which keeps cost low and numbers consistent across levels:

| Dashboard | Audience | Basic contents (Phase 1) |
|---|---|---|
| **Consultant** | Daily execution | Delivered as the Consultant Workspace (Module 5) — My Desk is the consultant lens of the decision layer |
| **Delivery Lead** | Team review | Team workload by consultant, bottlenecks, ageing roles, pending feedback across desks, stage conversion |
| **CEO** | Decision making | Active clients, active roles, pipeline health, submissions and placements, feedback latency trend |
| **Client (curated)** | Shared intel | Client-safe projection: their pipeline, their pendencies, benchmark-backed nudges |

All three internal dashboards read the same six metrics — a status number the CEO sees is the same number the consultant sees, aggregated differently. Review meetings run from these screens, not from prepared updates.

**Shared-truth rule:** measurement is a property of the OS, not a watchtower attached to it. Every actor sees the metrics for their scope, and consultants always see everything the OS computes about their own book — same numbers, different altitudes. Reviews and decisions happen through the system rather than through interrupted reporting, which is precisely what "OS" means.

**Command centre:** the decision layer is where operational risk surfaces itself. Breached SLAs, missed commitments, stalled roles, and ageing pendencies appear as tiered flags — nudge, warn, escalate — routed to the altitude that can act on them. Nobody has to go looking; the OS raises its own hand.

**Self-correction loop:** measurement's first customer is the person being measured. The OS doesn't just show consultants their numbers — it evaluates them against the consultant's own baselines and team norms, flags deviations, and suggests course corrections: *"Feedback latency on your Snapmint roles is 2× your usual — three pendencies are with the client; the brief composer has a nudge ready."* The consultant gets the first look and the first chance to fix the pattern; Delivery Lead intervention is the second line, not the first. Distinct from nudges: nudges say *do this now* (operational SLAs); course-correction insights say *this pattern needs attention* (performance vs. own baseline). Both are consumers of the same event stream. This is not a performance-management module — no goals, no OKRs, no ratings — it is the OS doing for the consultant what the CEO dashboard does for the founder: turning events into judgment support.

Six canonical metrics, computed from events, never entered:
**time to first submission · feedback latency · stage ageing · stage conversion · submissions per role · desk load.**

Every number is traceable: click any metric to see the events beneath it. Intelligence is auditable by construction.

## 8. Nudges

**Eliminates:** roles silently stalling.

- v1: seven honest SLA rules — feedback pending > 48h; candidate ageing in stage > threshold; interview unscheduled > N days; role without submission > N days; offer pending > N days; **offer accepted with no candidate contact in 14 days during notice period → warm-touch nudge; joining date within 7 days → confirmation check** (the offer-to-join limbo is where Indian placements die to counteroffers — these two rules guard the revenue moment)
- **Commitments** — Hyperhunt's own promises to clients ("batch of 3 CVs by Thursday 6pm"), captured in seconds when made, attached to the Role as an event. The nudge engine watches them like any other rule. This closes the gap generic ageing rules can't see: missed *delivery commitments* damage agency credibility far more than slow drift, and they are the side Hyperhunt actually controls
- **Tiered escalation** — flags carry severity, not a flat alarm: **nudge** (consultant's Today queue) → **warn** (consultant + flagged on Delivery Lead's dashboard) → **escalate** (surfaced to Delivery Lead as an action item, and to the CEO dashboard if it involves a client relationship at risk). A 25-day-old pendency is qualitatively different from a 3-day one, and the system treats it that way
- **One-tap interaction log** — calls and WhatsApp messages happen outside the system, so nudges can be wrong ("chase feedback" an hour after the consultant chased it). A single tap — "called candidate," "chased feedback" — logs an interaction event. One tap, never typing. **No manual task lists anywhere in the product**: the system tracks what happened, never asks consultants to maintain a to-do
- Nudges surface in the decision layer and flow (client-safe ones) into client briefs
- As the event log accumulates closed roles, rules graduate into benchmark-backed claims from Hyperhunt's own data
- External phase: cross-client benchmarks — every agency on the platform makes nudges smarter for all — a genuine data network effect that exists only because of the event-log foundation

## 9. Background AI

**Eliminates:** reading, summarizing, and composing labor. Invisible by design.

One LLM integration, expressed as event-triggered prompts:

| Trigger event | AI output |
|---|---|
| CV uploaded | Parsed profile + candidate summary |
| Role created | JD summary |
| Feedback recorded | Feedback summary |
| Client update due | Client brief draft |
| Week closes | Weekly summary per consultant — done, pending, and one or two course-correction insights vs. own baseline — plus the leadership rollup |
| Decision-layer anomaly | Contextual annotation (e.g., "submission rate on this role is half your norm — calibration issue?") |

No agent frameworks, no orchestration layers, no chat interface, no autonomous action. Agents, where they exist, are background workers on the event stream that the user never sees.

---

# Build vs. Integrate Policy

**Rule: build only what encodes Hyperhunt's opinion; integrate everything that doesn't.**

| Capability | Decision | Rationale |
|---|---|---|
| Spine, event log, upload flow, views, brief composer | **Build** | This is the product's opinion |
| Database, auth, storage | **Integrate** (Supabase) | Designated backend platform per HSES operating manual |
| Resume parsing & all summaries | **Integrate** (single LLM API) | Solved problem; one call serves parsing and summary |
| Dedup machinery | **Thin build on integrated parts** | The policy is opinion; the matching isn't |
| Search | **Integrate** (Postgres full-text) | A search "module" is bloat |
| WhatsApp delivery | **Defer** | Copy-to-paste now; Cloud API groups in external phase |
| Email, calendar, enrichment, job boards | **Neither, yet** | No internal chore dies by adding them — elimination test fails |

---

# What We Will Not Build (Phase 1)

Explicit non-goals, each with the reason recorded so the decision survives:

- **AI sourcing / relevance engine** — Idea 1; attaches later to the calibration slot and outcome events
- **Client logins & portals** — internal phase has no external users; briefs travel via WhatsApp
- **Permissions beyond the three internal roles** — Consultant, Delivery Lead, CEO/Admin cover the internal team; Senior Consultant and Client tiers wait for a demonstrated need
- **Insights module / BI / metric builder** — the three basic dashboards ship as fixed views over the six canonical metrics; a configurable analytics platform is external-phase territory at best
- **Notifications infrastructure** — in-app badges only; the team sits in one office and one WhatsApp
- **Custom fields, metric builder, automation builder** — flexibility at the data layer is the incumbents' bloat disease
- **Calendar sync / self-scheduling links** — the swamp; an interview date entered once as an event feeds the Today queue and briefs, which is enough for Phase 1
- **Email/WhatsApp inbox inside the product, in-app team chat, auto-dialers** — comms stay where they live; the composers meet them there
- **A task manager** — the computed Today queue is the task manager
- **Gamified leaderboards** — leadership reviews consultant-level metrics through the dashboards; a gamification layer adds no decision and eliminates no chore
- **Candidate portal, community, marketplace, mobile app, billing, AI chat, AI interviewing** — later phases or never. The web app must be fully responsive (consultants live on phones between calls); a native app is external-phase at best

---

# Phasing

Each sprint has one objective (per HSES constitution) and is defined by the manual work it kills.

| Sprint | Kills | Delivers | Done when |
|---|---|---|---|
| **1** | Duplicate data entry | Identity (Supabase Auth + RLS, three roles) + Spine + Magic Upload + event log foundation (taxonomy must accommodate commitment and interaction events, and a severity dimension on flags — designed in now, used in Sprint 3) | A real CV uploaded to a real role by a real logged-in consultant produces the full automatic cascade, with row-level security enforcing per-user scope |
| **2** | Status bookkeeping | Drag-and-drop movement + auto history/metrics + Candidate Bank profile view (cross-role history) + My Desk v1 (role portfolio + candidates in motion) + search | Stage moves generate history and metrics with zero manual entry; a consultant's whole book of work is visible from one screen |
| **3** | Update-typing, CV reformatting, candidate messaging | Composers: Client Briefs + Submission Packs + Candidate Messages (all copy-to-paste) + seven v1 nudge rules | A consultant sends a real client update and a real submission pack without typing or reformatting anything |
| **4** | Asking | Decision Layer: Consultant Workspace matured (Today queue + computed prioritization) + Delivery Lead and CEO dashboards + six metrics with drill-down | Consultants run 90% of their day from My Desk; reviews and leadership decisions run from the dashboards |
| **Deferred** | — | WhatsApp Cloud API groups; external-phase features | OBA verification (start early, in parallel) and internal-phase exit |

Sequencing discipline: the decision layer cannot jump ahead of Sprints 1–2. A dashboard over sparse events doesn't eliminate asking — it causes asking, plus distrust. Event coverage precedes intelligence.

**Architecture check built into the plan:** if Sprint 4 turns out expensive, the event log was designed wrong in Sprint 1. The decision layer arriving almost for free is the proof the foundation is right.

---

# Success Metrics

## Sprint-level

- Sprint 1: time from CV-in-hand to candidate-in-pipeline drops to under one minute with zero duplicate entry
- Sprint 2: pipeline-tracking sheet retired on every active mandate
- Sprint 3: client updates composed in one tap; consultant typing time on updates approaches zero
- Sprint 4: consultants start their day in My Desk instead of sheets and groups; status questions measurably collapse; leadership reviews run from the dashboards

## Internal phase exit criteria

1. All active mandates run end-to-end in Hyperhunt OS; the pipeline sheets are gone.
2. Consultants complete 90% of their daily workflow from the Consultant Workspace (My Desk) and the role pipelines it opens into.
3. The six canonical metrics are trusted — used in leadership decisions without verification against any sheet.
4. At least one full quarter of complete event history is accumulated.
5. Client briefs with nudges are the default update mechanism, and feedback latency shows measurable improvement.

Deliberately absent: MAUs, revenue, AI feature counts. The internal phase optimizes for elimination and trust.

---

# Future Leverage

Three assets compound quietly while the internal phase runs:

1. **The Idea 1 attachment points.** The Role carries a nullable calibration field; terminal pipeline stages (joined/rejected) are first-class outcome events. The relevance engine later plugs in as another background consumer of the same stream — reading calibrations, learning from outcomes — with no remodel.
2. **Tenancy-readiness.** Every table carries an organization ID from day one, containing only "Hyperhunt" until the external phase. One column now; a migration nightmare avoided later. No Hyperhunt-specific logic is hardcoded.
3. **The event history itself.** Every week of internal use deepens the dataset that powers benchmarks, nudges, and eventually cross-client intelligence — the moat that cannot be bought, only accumulated.

---

# Open Decisions

Tracked here until resolved; each resolution becomes an ADR in the repo.

1. ~~Product repo~~ **Resolved (Jul 2026):** new clean repository `hyperhunt-os`, private, under HSES governance.
2. Pipeline stage set: confirm the canonical stages and whether interview rounds are fixed or per-role configurable (recommendation: fixed set with optional rounds — opinionation over flexibility).
3. Nudge thresholds: confirm the seven v1 SLA values (including notice-period touch cadence) with consultants before Sprint 3.
4. Dedup policy edge cases: candidate with changed email/phone; same person via two sources.
5. WhatsApp OBA: confirm when to begin Meta business verification (recommendation: immediately, in parallel — it costs nothing but waiting).
6. ~~One-tap interaction log~~ **Resolved (Jul 2026):** approved. One tap, never typing — a heartbeat, not data entry. Corollary rule adopted: **no manual task lists anywhere in the product.** Commitments and tiered escalation approved in the same decision.

---

*This document is the product's source of truth. Per the HSES constitution: documentation before implementation; GitHub is the source of truth; chats are temporary. Amendments to this vision are deliberate and versioned.*