# Hyperhunt OS — Build Status

Last updated: Sprint 0 (setup complete) → Sprint 1 spec in progress

## Current sprint

**Sprint 1 — Identity, Spine & Magic Upload.** Spec drafting underway;
implementation not started.

## Done

**Environment**

- Repo `hyperhunt-os` created (private, HSES-governed), baseline committed and pushed
- Committed: `CLAUDE.md`, `docs/product-vision.md`, `docs/cheatcodes.md`,
  `docs/backlog.md`, `docs/status.md`, `docs/prompt-library.md`;
  `docs/specs/` and `docs/adr/` scaffolded
- Claude Code Builder installed, briefed, passed its briefing interview
- Plugins active: ui-ux-pro-max, ponytail, commit-commands, playwright,
  context7, code review, TypeScript LSP
- Supabase project `hyperhunt-os-dev` created; MCP connected and scoped to
  dev only (production off-limits — standing rule saved to Builder memory
  and written into CLAUDE.md)
- Vercel account connected via GitHub
- Claude Project "Hyperhunt OS — Build" created; knowledge loaded; standing
  Project Instructions set from prompt-library section 2

**Product definition**

- Vision doc complete: 9 modules, 3 actors, event-log foundation,
  elimination philosophy, 4-sprint phasing, exit criteria
- Founder briefing written (single-doc onboarding for anyone joining)
- Prompt library written (project instructions, sprint templates, builder
  handoff, session close, sprint review)

## In progress

**Sprint 1 feature specification** — blocked pending answers to six open
questions raised by the PM chat:

1. Canonical pipeline stages (fixed set New to Joined/Rejected; open
   question on "On Hold")
2. Dedup rules (auto-merge on exact email/phone; fuzzy flagged for
   confirmation)
3. Auth roles and sign-in method (all three roles live from Sprint 1;
   Google sign-in recommended)
4. Supported CV formats (PDF and DOCX only recommended)
5. Minimum required fields for Client and Role records
6. Data import vs. clean start (clean start recommended)

Once answered, spec commits to `docs/specs/sprint-1-identity-spine-upload.md`.

## Next objective

Approved Sprint 1 spec committed, then Builder plans implementation in Plan
Mode, then build begins.

## Recent decisions

- **Identity model added to Sprint 1 scope:** Supabase Auth, three roles
  (`consultant`, `delivery_lead`, `ceo_admin`), row-level security
  enforced by the database rather than application code. Admin creates
  the three accounts manually in Sprint 1; no user-management UI.
- **One-tap interaction log: approved.** One tap, never typing.
  Corollary rule adopted: **no manual task lists anywhere in the product.**
- **Commitments: approved.** Hyperhunt's own delivery promises ("batch of
  3 CVs by Thursday") captured as role-attached events and watched by the
  nudge engine — closing the gap generic ageing rules cannot see.
- **Tiered escalation: approved.** Flags carry severity — nudge, warn,
  escalate — routed to the altitude that can act, rather than flat alerts.
- **Consultant Workspace (My Desk)** added as Module 5: the consultant's
  home screen with computed Today queue (prioritization is derived, never
  typed).

## Sprint 1 spec revisions (agreed, pending PM update)

The PM's spec draft needs these before any migration is written:

1. **Stages** — canonical set default on every role; rename "Submitted" to
   "CV Shared"; **"Screening" removed — it is not a stage**; interview stages
   are named Round 1 / Round 2 / Round 3; no rounds 4-5; custom stage
   creatable case-by-case per
   mandate; ADR must state how metrics treat custom stages.
2. **Outcomes** — candidate participation outcomes expand to five:
   `joined` / `rejected_by_client` / `withdrawn_by_candidate` /
   `offer_declined` / `role_cancelled`. These are CANDIDATE outcomes and are
   never mixed with Role status (`active` / `paused` / `closed`). Store the
   stage the candidate reached alongside the outcome.
3. **Rejection reasons** — one-tap picklist in Sprint 1, never free text
   (the draft's "optional short text" contradicts ADR-011). Context-aware
   suggestions are a Sprint 3+ enhancement.
4. **Add-from-Bank** — committed Sprint 2 scope (open role → search Bank →
   add candidate, ranked on stored attributes). Sprint 1 schema designed so
   it needs no migration. AI relevance scoring stays with the relevance
   engine.
5. **Calibration** — unchanged: reserved, nullable, untouched in Phase 1.
6. **Candidate profile score** — confirmed out of scope. No global score.

Spec status moves to "Revised — pending approval"; reissue ADR-001 and
ADR-003; add an ADR for rejection reasons; extend the acceptance test to
cover rejecting a candidate at Round 2 and confirming both outcome and
stage-reached are recorded.

## Schema notes for Sprint 1

The event taxonomy must accommodate, from day one:

- Commitment events and interaction events (used in Sprint 3)
- A severity dimension on flags (nudge / warn / escalate)
- A visibility flag on every event (internal / client_safe)
- `org_id` on every table; RLS on every table with `org_id`
- Events append-only; Role carries a nullable `calibration` field
  (reserved for the future relevance engine — unused in Phase 1)

Designing these in now avoids a painful retrofit in Sprint 3.

## Open decisions (product-level)

1. Product repo — RESOLVED: `hyperhunt-os`, private, HSES-governed
2. Pipeline stage set — being resolved in the Sprint 1 spec questions
3. Nudge thresholds — confirm the seven v1 SLA values with consultants
   before Sprint 3
4. Dedup policy edge cases — changed email/phone; same person via two
   sources
5. WhatsApp OBA verification — **not yet started; start soon.** Weeks-long
   clock only begins on submission
6. One-tap interaction log — RESOLVED: approved

## On the horizon (logged, not scheduled)

- **Hyperhunt OS MCP server** — read-only conversational access to live data
  from Claude, for strategic questions. Post-Sprint-4, once the event log has
  real history. No Sprint 1 change required — the schema is already
  MCP-ready. Full reasoning in backlog.md.

## Blockers

- Sprint 1 spec blocked on the six answers above (founder decision, not
  technical)
- WhatsApp OBA not started (does not block internal phase; does delay the
  external phase)

## Notes for the next chat

- Founder is non-technical. Plain language for every decision; teach while
  building.
- Elimination philosophy governs all scope: features must delete manual
  work, not add capability.
- Event log is the foundation — every derived view reads from it; never
  build separately maintained state.
- New ideas go to `backlog.md` with one line, not into the current sprint.
- A fractional technical person may join in an advisory/review capacity.
  Test for any technical recommendation: after they are gone, can the
  founder plus Claude Code maintain this?
