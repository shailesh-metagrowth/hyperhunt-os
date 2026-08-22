# Hyperhunt OS — Build Status

Last updated: Sprint 1 spec approved and committed → first build session not yet started

## Current sprint

**Sprint 1 — Identity, Spine & Magic Upload.** Spec is **approved and committed**.
Implementation **not started**. The next working session is the first build
session: brief the Builder in Plan Mode on the approved spec.

## Done

**Environment (Sprint 0 setup — complete)**
- Repo `hyperhunt-os` created (private, HSES-governed), baseline committed and pushed
- Committed: `CLAUDE.md`, `docs/product-vision.md`, `docs/cheatcodes.md`,
  `docs/backlog.md`, `docs/status.md`, `docs/prompt-library.md`;
  `docs/specs/` and `docs/adr/` scaffolded
- Claude Code Builder installed, briefed, passed its briefing interview
- Plugins active: ui-ux-pro-max, ponytail, commit-commands, playwright,
  context7, code review, TypeScript LSP
- Supabase project `hyperhunt-os-dev` created; MCP connected and scoped to
  dev only (production off-limits — standing rule in CLAUDE.md)
- Vercel account connected via GitHub
- Claude Project "Hyperhunt OS — Build" created; knowledge loaded; standing
  Project Instructions set

**Product definition**
- Vision doc complete: 9 modules, 3 actors, event-log foundation,
  elimination philosophy, 4-sprint phasing, exit criteria
- Founder briefing, prompt library, cheatcodes all written and committed

**Sprint 1 planning (complete)**
- Sprint 1 spec written, revised, and **approved for build**; committed to
  `docs/specs/sprint-1-identity-spine-upload.md`
- All six opening questions resolved (stages, dedup, auth, CV formats,
  required fields, clean start)
- Backlog reconciled to approved decisions (Google sign-in in Sprint 1;
  open-internal access noted)

## Next objective

**Start the first build session.** Open Claude Code, Plan Mode, brief it:
"Read CLAUDE.md and docs/specs/sprint-1-identity-spine-upload.md. Propose your
implementation plan before writing any code." Review the plan against build
order (spec §9), approve, then build slice 1 (schema + auth + org-boundary RLS).

## Locked model decisions (Sprint 1 spec, approved)

- **Pipeline stages:** New → CV Shared → Round 1 → Round 2 → Round 3 →
  Round 4 → Round 5 → Offer → Joined / Rejected. Fixed canonical set, **no
  Screening stage, no custom stages.** Each role sets `interview_rounds`
  (1–5, default 3) to choose how many round columns show. "CV Shared"
  replaces "Submitted".
- **Stage and outcome are separate fields.** Participation outcomes (five):
  `joined` / `rejected_by_client` / `withdrawn_by_candidate` /
  `offer_declined` / `role_cancelled` — stored with `stage_reached` and
  `ended_at`. Never mixed with **role status** (`active` / `paused` /
  `closed`, with its own close reasons).
- **Rejection reasons:** fixed one-tap picklist, no free text (ADR-012).
  Context-aware suggestions deferred to Sprint 3+.
- **Access (Phase 1: open internally):** Google sign-in, three roles.
  RLS enforces the **org boundary** and signed-out = no access; all internal
  users see all org data — focus comes from filters (My roles / by consultant
  / by client), not hard walls. Per-consultant walls deferred to the
  multi-tenant phase.
- **Two source fields:** `candidates.source` (first-ever touch, immutable)
  vs `candidate_roles.source` (per-role origination). Origination path
  (`magic_upload` / `bank_add`) recorded on the linking event.
- **Add-from-Bank:** schema designed in Sprint 1; search UI ships Sprint 2
  (attribute-based ranking only; AI relevance stays with the relevance engine).
- **Assignment (ADR-014):** clients assigned to a consultant, roles inherit;
  consultant / lead / CEO can reassign (emits event). Role title editable;
  multiple openings = sibling roles.
- **Calibration:** reserved, nullable, untouched in Phase 1.
- **No global candidate profile score** — rejected on principle.
- **No manual task lists anywhere** (ADR-011).
- ADRs 001–014 to be written to `docs/adr/` during the build.

## Schema notes for Sprint 1

The event taxonomy accommodates, from day one:
- Reserved events used from Sprint 3: `commitment_made`, `interaction_logged`,
  `feedback_recorded`, `flag_raised`
- A nullable `severity` dimension on flag events (nudge / warn / escalate)
- A visibility flag on every event (internal / client_safe)
- `org_id` on every table; RLS (org boundary) on every table with `org_id`
- Events append-only; Role carries a nullable `calibration` field
  (reserved for the future relevance engine — unused in Phase 1)

## Open decisions (product-level)

1. Product repo — RESOLVED: `hyperhunt-os`, private, HSES-governed
2. Pipeline stage set — RESOLVED in the approved Sprint 1 spec
3. Nudge thresholds — confirm the seven v1 SLA values with consultants
   before Sprint 3
4. Dedup policy edge cases — keyless CV (no email/phone) resolved: fuzzy +
   always-confirm. Changed email/phone still open, revisit if it bites
5. WhatsApp OBA verification — **not yet started; start soon.** Weeks-long
   clock only begins on submission
6. One-tap interaction log — RESOLVED: approved

## On the horizon (logged, not scheduled)

- **Hyperhunt OS MCP server** — read-only conversational access to live data
  from Claude, for strategic questions. Post-Sprint-4, once the event log has
  real history. No Sprint 1 change required — the schema is already
  MCP-ready. Full reasoning in backlog.md.

## Blockers

- None blocking the first build session — planning is complete and the spec
  is approved.
- WhatsApp OBA not started (does not block internal phase; does delay the
  external phase).

## Notes for the next chat

- Build has **not** started yet. The next session is the first build session
  (planning with the Builder in Plan Mode).
- Founder is non-technical. Plain language for every decision; teach while
  building.
- Elimination philosophy governs all scope: features must delete manual
  work, not add capability.
- Event log is the foundation — every derived view reads from it; never
  build separately maintained state.
- New ideas go to `backlog.md` with one line, not into the current sprint.
- The approved spec is law for Sprint 1; CLAUDE.md is law for build questions.
  If they ever disagree, stop and reconcile before building.