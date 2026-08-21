# Hyperhunt OS — Founder & Product Briefing
### For anyone building, evaluating, or joining this product

**Read this first.** It's the single entry point to Hyperhunt OS. Every claim
here has a deeper source in the six-file kit; sections end with pointers to
where the detail lives.

**Author:** Shailesh, Founder & Product Lead, Hyperhunt
**Status:** Ready for build. Sprint 1 in progress.
**Audience:** engineers, contractors, advisors, future hires, and future-me.

---

## 1. The one-paragraph pitch

Hyperhunt OS is an **AI-in-the-background operating system for recruiting
operations** — an internal tool for Hyperhunt today, the first product of the
Hyperhunt AI SaaS tomorrow. It replaces the Google Sheets and WhatsApp groups
that agencies actually run on, with one system where every action produces a
structured event and every derived view (timelines, dashboards, briefs,
nudges) is a consumer of that event stream. What Notion did for organizing
knowledge, Hyperhunt OS does for the hiring pipeline: opinionated primitives,
composable views, and automation that makes the software disappear behind
the workflow.

## 2. Why this exists

Agency recruiting runs on three compounding costs. Consultants **type the
same information across sheets, messages, and memory** — every candidate,
every stage change, every client update entered multiple times. **Operations
are blackbox** — pipeline status, bottlenecks, and team load live in people's
heads; every operational question is answered by interrupting someone.
**Client-side latency is invisible** — roles stall on feedback, unconfirmed
interviews, slow approvals, and the agency has no instrument to surface it
with evidence.

Incumbents (Bullhorn, Loxo, Recruit CRM, Manatal, Crelate, PCRecruiter,
Zoho Recruit) solve these by *addition* — dozens of modules, most bloat.
They are candidate-record systems with AI bolted onto CRUD data, built for
email and portals, and blind to how Indian agencies actually operate.

## 3. What makes it different — architecturally, not featurally

Four bets that incumbents structurally can't copy:

**Role-centric, not candidate-centric.** The Hiring Pipeline belongs to
the Role. Consultants open roles, not candidates; candidates are participants
in hiring processes. Incumbents' schemas are candidate-centric; retrofitting
this takes a decade.

**Event log is the foundation.** Every meaningful action is an immutable,
timestamped event with an actor, object references, and a visibility flag
(internal / client_safe). Timelines, metrics, dashboards, and every AI
feature are views over or consumers of this stream — never separately
maintained state. If a capability can't be expressed as producer or consumer
of events, it doesn't belong in the architecture.

**Opinionated data, composable views.** The Notion lesson applied
correctly: flexibility at the view layer (filter, group, save),
opinionation at the data layer (fixed spine, enforced pipeline semantics,
automatic capture). Precisely why "just use Notion" fails for hiring ops —
Notion is neutral and manual where this system is opinionated and automatic.

**India-native.** WhatsApp is the first-class client channel. Workflows
assume Indian agency mechanics — Naukri, notice-period limbo, group-based
client comms. Not a localization; a wedge Western platforms structurally
underserve.

## 4. Product philosophy — three commitments

**Elimination over addition.** Every capability must delete manual work. If
a feature adds capability without removing a chore, it is bloat and does not
ship. This principle governs every scope decision.

**AI in the background.** No chat windows, no copilots demanding attention,
no autonomous personas. AI works invisibly — parsing, summarizing,
composing, flagging — as a silent consumer of the event stream. Users
experience outcomes, not AI.

**Operations as shared intelligence.** The same system of record that
eliminates data entry produces the decision layer — for consultants, for
leadership, and (curated) for clients. Intelligence is never a blackbox
because every metric is traceable to the events beneath it. Measurement is
a property of the OS, not a watchtower attached to it.

**Deep detail:** `docs/product-vision.md` sections *Vision* and
*Strategic Positioning*.

## 5. Who uses it

Three actors, three altitudes over the same event stream.

| Actor | Uses the OS to… | Scope |
|---|---|---|
| **Consultant** | Operate: run roles, move candidates, compose briefs and packs, work the Today queue | Own book of work |
| **Delivery Lead** | Orchestrate: balance load, unblock bottlenecks, review desks, uphold SLAs | All desks |
| **CEO/Admin** | Decide: pipeline health, client health, placements, where to intervene | Whole business |

Scope of aggregation differs; the numbers do not. A consultant's status
figure and the CEO's rollup are the same events viewed at different
altitudes.

**Deep detail:** `docs/product-vision.md` section *Users & Access* and
*Identity & Access Model*.

## 6. The primitives — everything is built from these

| Primitive | Definition |
|---|---|
| **Client** | An organization Hyperhunt serves. Owns roles, team contacts, preferences. |
| **Role** | A mandate. The primary unit of work. Owns exactly one Hiring Pipeline, JD, priority, consultant assignment, and a nullable `calibration` field (reserved for the future relevance engine — do not use yet). |
| **Hiring Pipeline** | The staged process belonging to a Role: New → CV Shared → Round 1 → Round 2 → Round 3 → Offer → Joined. Canonical set by default; a custom stage can be added case-by-case per mandate. The consultant's primary workspace. |
| **Participation outcome** | How a candidate's involvement in one role ended: `joined` / `rejected_by_client` / `withdrawn_by_candidate` / `offer_declined` / `role_cancelled`, stored with the stage reached and a one-tap rejection reason. Candidate outcomes are separate from Role status (`active` / `paused` / `closed`) and never mixed. |
| **Candidate** | A person stored once in the Candidate Bank, deduplicated (exact email/phone, LLM fuzzy fallback), linked to any number of roles as a participant. |
| **Event** | Immutable, timestamped, structured record of any action: upload, stage move, feedback, submission, note, nudge. Carries actor, object references, visibility flag (internal / client_safe). Append-only, forever. |

## 7. The nine modules — each defined by what it kills

1. **Spine** (Client → Role → Pipeline → Candidate Bank) — scattered records
2. **Candidate Bank** — re-sourcing people we already know; candidate knowledge dying when a role closes; conflict warnings surface duplicate submissions and internal collisions
3. **Magic Upload** — duplicate data entry: one CV upload triggers parse → dedup match → link to role → pipeline slot → source and timestamp → events → AI summary → searchable
4. **Pipeline & Movement** — status bookkeeping: drag-and-drop stage moves auto-generate history, timestamps, and time-in-stage
5. **Consultant Workspace (My Desk)** — assembling the day: role portfolio, candidates in motion, own clients, and a computed Today queue (not a manual task list)
6. **Composers** — three outputs from one capability: **Client Briefs** (formatted updates from client-safe events), **Submission Packs** (branded, contact-stripped CV + summary — kills 20–40 min per candidate), **Candidate Messages** (interview confirmations, offer congratulations, joining reminders). All copy-to-paste in Phase 1
7. **Decision Layer** — asking: three named dashboards (Consultant / Delivery Lead / CEO), six canonical metrics, drill-down to source events; plus the **self-correction loop** where the OS evaluates each consultant against their own baseline and suggests course corrections
8. **Nudges** — silent stalls: seven SLA rules including offer-to-join limbo (feedback pending, ageing candidates, notice-period touch cadence, etc.)
9. **Background AI** — reading, summarizing, composing labor: event-triggered LLM calls for parsing, summaries, briefs, packs, insights. No chat surface, no agents, no autonomous action

**Explicit non-goals** (each entry has its "why deferred" in the backlog):
AI sourcing (Idea 1, later attaches to the calibration slot); client
portals/logins; calendar sync and self-scheduling links; email/WhatsApp
inbox in-product; in-app team chat; manual task manager; leaderboards;
configurable BI / metric builder / custom fields; notifications
infrastructure beyond in-app badges; candidate portal, community,
marketplace, mobile app, billing, AI chat, AI interviewing.

**Deep detail:** `docs/product-vision.md` sections *Modules & Capabilities*
and *What We Will Not Build*.

## 8. The technical stack — decided, not open

Deliberately boring. Everything a solo non-technical founder can maintain
today and a small team can inherit tomorrow.

- **Frontend & backend:** Next.js (App Router, TypeScript) on Vercel
- **Database & auth & storage:** Supabase — Postgres, Auth, Storage, RLS,
  full-text search
- **AI:** one Anthropic API integration; prompts triggered by events
- **No additional frameworks, ORMs, state libraries, or services without an
  ADR in `docs/adr/`**

**Non-negotiable schema rules:**
- Every table carries `org_id` from day one (tenancy-ready, single-tenant now)
- Every table with `org_id` has row-level security policies enforcing per-role
  scope — the database enforces scope, never application code
- Events table is append-only; never update or delete event rows
- Stage and outcome are separate fields; participation outcomes are first-class events carrying the stage reached and a structured (never free-text) rejection reason
- Role status (`active` / `paused` / `closed`) is never mixed with candidate participation outcomes
- The Role object includes a nullable `calibration` field, reserved for the
  future relevance engine
- All schema changes go through migration files, never dashboard edits

**Identity model:** every user has an account with `org_id` and one of exactly
three role values — `consultant`, `delivery_lead`, `ceo_admin`. RLS policies
scope queries automatically. For Phase 1, the admin creates the two other
accounts once via the Supabase dashboard; no user-management UI until a
fourth account becomes a recurring chore.

**Toolkit (`docs/cheatcodes.md`):** shadcn/ui + Tailwind, dnd-kit
(pipeline drag-and-drop), TanStack Table (Bank, My Desk), Recharts
(dashboards), react-dropzone (upload), Vercel AI SDK (LLM calls),
@react-pdf/renderer (submission packs), Motion (animations, sparingly),
Lucide (icons).

**Build-vs-integrate policy:** build only what encodes Hyperhunt's opinion
(spine, event log, upload flow, views, composers). Integrate everything
that doesn't (database, auth, storage, parsing, summaries, search, PDF).

**Deep detail:** `CLAUDE.md` (the Builder's rulebook) and
`docs/cheatcodes.md`.

## 9. How it gets built — governance and workflow

The whole build runs under **HSES** (Hyperhunt AI Software Engineering
System) — a separate governance repo containing the constitution, operating
manual, and improvement register. Every product decision honors HSES rules:

1. **Documentation before implementation.** No feature work without an
   approved spec in `docs/specs/`. Ideas without specs get written up before
   building.
2. **Architecture is frozen during implementation.** Mid-build discoveries
   about wrong architecture pause the sprint and produce an ADR, never a
   silent redesign.
3. **One session, one objective.** New ideas go to `docs/backlog.md` with
   one line of rationale — never expanded into the current sprint.
4. **Baseline before edits.** Clean git state confirmed; small logical
   commits with clear messages.
5. **Definition of done:** implementation complete, tests pass, docs
   updated, committed. Explicitly stated at session close.
6. **Verify, don't assume.** Actual library versions, actual schema state,
   actual tool behavior. Uncertainty said out loud.

**Decision records (ADRs)** for every consequential technical choice —
context, decision, alternatives, consequences — in plain language a
non-technical founder can understand. Numbered sequentially in `docs/adr/`.

## 10. How work actually gets done — the roles in the room

- **Founder + Product Lead** (Shailesh): plans in the Claude Project called
  "Hyperhunt OS — Build," writes and approves specs, tests every feature by
  hand as the acceptance layer, holds scope.
- **PM Assistant** (Claude, in the Project): sprint tracking, spec drafting,
  scope defense, plain-language decisions. Sees the vision, playbook,
  cheatcodes, setup guide, and status doc as project knowledge.
- **Builder** (Claude Code, in VS Code): implementation. Reads `CLAUDE.md`
  and the current spec every session. Never redirected mid-build.
- **Governance** (HSES repo): constitution, operating manual, improvement
  register — the rulebook the Builder inherits via `CLAUDE.md` and the PM
  inherits via project knowledge.

The founder is not a copy-paste layer between PM and Builder. Ideas
**travel only as committed documents** — spec written and committed to the
repo, then Builder builds from the committed spec. No chat summaries
pasted into code sessions.

## 11. The build order — four sprints, kill list not feature list

Each sprint has one objective and is defined by the manual work it
eliminates.

| Sprint | Kills | Delivers | Done when |
|---|---|---|---|
| **1** | Duplicate data entry | Identity (Supabase Auth + RLS, three roles) + Spine + Magic Upload + event log foundation | A real CV uploaded to a real role by a real logged-in consultant produces the full automatic cascade, with RLS enforcing per-user scope |
| **2** | Status bookkeeping | Drag-and-drop movement + auto history/metrics + Candidate Bank profile view + My Desk v1 + search | Stage moves auto-generate history and metrics; a consultant's whole book of work is visible from one screen |
| **3** | Update-typing, CV reformatting, candidate messaging | Composers (Briefs + Packs + Messages, copy-to-paste) + seven v1 nudge rules | A consultant sends a real client update and a real submission pack without typing or reformatting anything |
| **4** | Asking | Decision Layer: My Desk matured with Today queue + Delivery Lead and CEO dashboards + drill-down | Consultants run 90% of their day from My Desk; reviews and leadership decisions run from the dashboards |
| **Rollout** | Sheets | Consultant onboarding, live-mandate migration, hardening | Every active mandate runs in Hyperhunt OS end-to-end; the pipeline sheets are retired |

**Realistic timeline** for a solo non-technical founder building with
Claude Code, ~15 hours/week: **consultants living in it daily by
late November 2026**, with one full quarter of trusted event history
completing the internal-phase exit around Feb–Mar 2027.

**Sprint sequencing discipline:** the Decision Layer cannot jump ahead of
Sprints 1–2. A dashboard over sparse events causes asking plus distrust.
Event coverage precedes intelligence.

**Deep detail:** `docs/product-vision.md` section *Phasing*.

## 12. How success is measured

**Internal phase exit criteria:**
1. All active mandates run end-to-end in Hyperhunt OS; pipeline sheets retired
2. Consultants complete 90% of their day from My Desk and role pipelines
3. The six canonical metrics are trusted — used in leadership decisions
   without verification against any sheet
4. At least one full quarter of complete event history accumulated
5. Client briefs with nudges are the default update mechanism, and feedback
   latency shows measurable improvement

Deliberately absent from success metrics: MAU counts, revenue, AI feature
counts. Internal phase optimizes for elimination and trust.

## 13. What's already built (as of writing)

- `hyperhunt-os` private repo created under HSES governance
- Governance and scaffolding committed: `CLAUDE.md`, `docs/product-vision.md`,
  `docs/cheatcodes.md`, `docs/backlog.md`, `docs/status.md`, plus empty
  `docs/specs/` and `docs/adr/`
- Claude Code (Builder) installed, briefed, verified — passed the
  briefing interview
- Plugins active: ui-ux-pro-max, ponytail, commit-commands, playwright,
  context7, code review, TypeScript LSP
- Supabase project `hyperhunt-os-dev` created; MCP connected and scoped
  to dev-only (production off-limits, saved as standing rule)
- Claude Project "Hyperhunt OS — Build" created with vision, playbook,
  cheatcodes, setup guide, status doc in knowledge
- Meta Business verification pending (background task; needed for WhatsApp
  Cloud API in the external phase)

## 14. What's next

Writing the Sprint 1 feature specification: Identity + Spine + Magic Upload
+ Event Log. Once approved and committed to `docs/specs/`, the Builder
plans and implements from it.

## 15. The long arc — where this becomes a product

Internal use hardens the OS through daily dogfooding. In parallel:
tenancy-readiness is engineered in from day one (every table has `org_id`),
no Hyperhunt-specific logic is hardcoded, the event log accumulates
outcome data no competitor can replicate. When the OS is undeniable
internally, it becomes the first face of the Hyperhunt AI SaaS — sold to
agencies and hiring teams who live the same pain. The relevance engine
(the future "Idea 1" — AI-native candidate matching against calibrated
role definitions) plugs in later as another consumer of the same event
stream, using the Role's reserved `calibration` field and the outcome
events already accumulating.

A further capability unlocks once the event log has depth: a **read-only MCP
server** exposing Hyperhunt OS data as query tools, so leadership can
interrogate live operations conversationally from Claude — stalled roles,
rejection patterns, conversion by stage — without any chat surface entering
the product itself. Deferred until after Sprint 4 because it needs
accumulated history to be worth anything; the Sprint 1 schema already
supports it without change.

The OS's compounding assets are visible even in Phase 1:
- **The Candidate Bank** — one candidate, one profile, forever
- **The outcome-labeled event history** — every submission → interview →
  placement chain, structured
- **The tenancy-ready schema** — external phase requires zero remodel

## 16. The document map — where to go next

| To understand… | Read |
|---|---|
| The product's full vision, modules, non-goals, exit criteria | `docs/product-vision.md` |
| The Builder's rulebook (stack, schema, workflow, toolkit rituals) | `CLAUDE.md` |
| The approved toolkit and reviewed/declined tools list | `docs/cheatcodes.md` |
| Current build state, what's done, what's in progress | `docs/status.md` |
| Everything deferred and why | `docs/backlog.md` |
| Historical architectural decisions | `docs/adr/` (numbered) |
| Approved specifications for what's being built | `docs/specs/` (per sprint) |
| Daily working rhythms and context-management discipline | `founder-build-playbook.md` |
| The engineering governance layer (constitution, manual, improvement register) | separate `HSES` repo |

**When these documents disagree, `docs/product-vision.md` wins for product
questions and `CLAUDE.md` wins for build questions.** Everything else
derives from those two.

---

*This document exists to make anyone joining Hyperhunt OS productive in
one sitting. Read it, then follow section 16 for the depth you need.
Everything is decided; the only unknowns are the ones building will
reveal.*