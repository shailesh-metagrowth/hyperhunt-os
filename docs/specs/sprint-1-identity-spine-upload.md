# Sprint 1 Spec — Identity + The Spine + Magic Upload + Event Log Foundation

**Status:** Approved for build
**Sprint objective (one line):** A real CV uploaded to a real role **by a real logged-in consultant** produces the full automatic cascade — parsed, deduplicated, in the pipeline, summarized, searchable — with every step recorded as an event, the org-boundary secured at the database, in under one minute, with zero duplicate typing.
**Kills:** Duplicate data entry.
**Timeline:** 3–4 weeks at ~20h/week.
**Governance:** Built under CLAUDE.md rules. Stack is frozen: Next.js (App Router, TypeScript) on Vercel, Supabase (Postgres, Auth, Storage), Anthropic API via Vercel AI SDK. No other frameworks without an ADR.

---

## 1. Decisions resolved in this spec (each becomes a short ADR)

| # | Decision | Resolution |
|---|---|---|
| ADR-001 | Pipeline stages | Fixed canonical set, no custom stages. "Submitted" renamed **CV Shared** (consultant language). Interview stages are Round 1–5; each role sets `interview_rounds` (1–5, default 3) to choose how many round columns it shows. The variation between mandates is *how many rounds*, never *different kinds of stage* — so every stage means the same thing on every role and cross-role metrics stay comparable by construction. |
| ADR-002 | Role lifecycle | Role status: `active` / `paused` / `closed`. **Paused freezes the pipeline** (no uploads, no stage moves) but the role stays visible in the working view under a Paused group, with reason and date, so follow-up is never missed. **Closed removes the role from the working view** into a separate Closed roles view, with a recorded outcome and reason ("client put it on hold indefinitely" is one of them). Closed roles can be reopened. Not a pipeline stage. |
| ADR-003 | Candidate participation outcomes (distinct from role status) | A participation (`candidate_roles`) ends with one outcome: `joined` / `rejected_by_client` / `withdrawn_by_candidate` / `offer_declined` / `role_cancelled`. Stored **with the stage the candidate reached** and an `ended_at` timestamp, so stage-wise conversion is queryable directly, without replaying event history. This is about the *candidate on this role* and is never mixed with **role status** (ADR-002: `active` / `paused` / `closed` with its own close outcomes — filled through us / filled elsewhere / cancelled / on hold indefinitely). Two separate concepts, two separate columns. |
| ADR-004 | Deduplication | Nothing merges automatically. All suspected duplicates (exact and fuzzy) are flagged for one-tap consultant confirmation. |
| ADR-005 | Auth & access (Phase 1: open internally) | Google sign-in via Supabase Auth. Three roles from day one: `consultant`, `delivery_lead`, `ceo_admin`. **Phase 1 decision: all internal users can see all org data** — the team is co-located and trust is total, so hard per-consultant walls are deliberately deferred. Scope is handled by **filters/views** (My roles · by consultant · by client), not by hiding data. Row-level security (RLS) still enforces the **org boundary** (a user only ever sees their own org's rows — the tenancy wall that matters when external agencies arrive) and blocks all access for signed-out users; it just doesn't wall consultants off from each other yet. Per-consultant walls become an ADR when multi-tenancy ships. *(This is intentionally looser than the vision doc's identity section, which describes the walled end-state; recorded here as a conscious Phase-1 simplification, not a drift.)* |
| ADR-006 | CV formats | PDF and DOCX only. Images/others rejected with a clear message. |
| ADR-007 | Data migration | Clean start. No import of existing sheets. |
| ADR-008 | Candidate tags | Free-form tags field exists on candidates from day one (searchable later). No automation or AI reads tags in Phase 1. |
| ADR-009 | AI-assisted role creation | Creating a role accepts a JD and/or an onboarding-call transcript — pasted text or a document file. AI pre-fills role fields from whatever is provided; consultant reviews and confirms before save. Audio files are not accepted (speech-to-text is a new service — fails the stack test); audio transcription goes to the backlog. |
| ADR-010 | Event taxonomy future-proofing | The event schema accommodates, from day one: **commitment events** (Hyperhunt's delivery promises — what, to whom, by when), **interaction events** (one-tap "called candidate" / "chased feedback"), and a **severity dimension** on flag events (`nudge` / `warn` / `escalate`). Designed into the schema in Sprint 1, first emitted/used in Sprint 3. No Sprint 1 UI produces them. |
| ADR-011 | No manual task lists — product rule, permanent | Nowhere in the product does a consultant maintain a to-do. The system tracks what happened (events), computes what needs doing (later: the Today queue), and accepts at most one tap to log an interaction. Any feature request implying a manually maintained list is rejected against this ADR. |
| ADR-012 | Rejection reasons are one-tap, not free text | Ending a participation as `rejected_by_client` requires picking one reason from a fixed list — no typing (this is what ADR-011 demands; an "optional text reason" would smuggle data entry back in). Sprint 1 base list: not enough depth · comp mismatch · location · notice period · communication · culture fit · overqualified · client changed brief · better candidate available · other. Stored structured (an enum) on the participation record and the outcome event. `other` carries **no** free-text box — a recurring reason earns a place on the list instead. Context-aware reason suggestions (tailored to the role/client/candidate) are a **Sprint 3+** enhancement, once enough rejection history exists to make suggestions meaningful. |
| ADR-013 | Add-from-Bank — designed now, ships Sprint 2 | A candidate reaches a role by two paths: **Magic Upload** (creates candidate + participation) and **Add-from-Bank** (open role → search the Candidate Bank → add an existing candidate → creates a participation only). `candidate_roles` is designed in Sprint 1 to support both with **no later schema change**; the origination path is recorded on the linking event. The Bank-search UI ships in **Sprint 2** and ranks using stored attributes only — skills from the parsed JD, experience range, location, prior participation with this client. Attribute-based filtering only; AI relevance scoring stays deferred to the future relevance engine. |
| ADR-014 | Client & role assignment | A client is assigned to a consultant; roles under that client inherit that consultant as their owner (drives the "My roles" / "by consultant" filters). The **assigned consultant, the Delivery Lead, or the CEO can reassign** a client's consultant; reassignment emits an event. Role title is freely editable at any time (multiple openings for the same job are modelled as sibling roles — "Role 1", "Role 2" — not a count inside one role). |

---

## 2. The canonical pipeline

Fixed stage set, identical semantics on every role:

```
New → CV Shared → Round 1 → Round 2 → Round 3 → Round 4 → Round 5 → Offer → Joined / Rejected
```

- **New = just landed.** Magic Upload places the candidate here. The consultant assesses them in this stage (reads the CV and AI summary) and either shares the CV with the client (→ CV Shared) or rejects. There is no separate "Screening" stage — assessment happens in New. (Earlier drafts had New → Screening as two steps; consolidated to one on the founder's call: on this desk a consultant reads a CV when it lands, so the "inbox wait" stage was bookkeeping, not reality.)
- **Rounds are not a separate concept from stages.** Round 1–5 *are* stages in the fixed list above. Each role's `interview_rounds` setting (1–5, default 3, editable while active) only controls how many of those round columns the role shows. A role that runs three interviews shows three round columns; a role that runs five shows five — the stages themselves, and what they mean, are identical on every role, which is what keeps metrics comparable across roles and clients. There are **no custom stages**: the difference between mandates is quantity of rounds, not new kinds of stage.
- "Submitted" is called **CV Shared** — the consultant's own word for sharing a candidate's CV with the client.
- `Joined` and `Rejected` are terminal. Ending a participation records a structured **outcome** and the **stage reached** (see §3 `candidate_roles` and ADR-012/003). A rejection reason is a one-tap picklist, never typed. **When a candidate is marked `joined` on one role, the system flags their other live participations** (reusing the conflict-warning mechanism) so the consultant can close them with one tap — it never closes them automatically, but it makes sure they're not silently left open. Rare, but never missed.
- **Sprint 1 movement rule:** Magic Upload places candidates in `New`. Manual stage movement (drag-and-drop, history, time-in-stage) is Sprint 2. In Sprint 1 a candidate can only be moved via a simple stage dropdown on the candidate detail panel — minimal, no drag-and-drop, no metrics — so the pipeline isn't a dead end during testing. Every such move still emits a proper stage-change event (the event log must be complete from day one even if the UI is minimal).

### Role status (not a stage)

- `active` — normal operation. Lives in the main role lists.
- `paused` — **frozen but in sight.** No CVs flow, no stage moves — uploads and movement are blocked with a friendly "this role is paused" message. Pausing requires a one-line reason, captured in the event. The role remains in the working view, grouped under a **Paused** section with its reason and pause date, so consultants see it every day and can't forget to follow up. (Sprint 3 hook: "role paused for N+ days — check status with client" becomes a nudge rule over this event, for free.)
- `closed` — the ending. The role **leaves the working view** and moves to a separate **Closed roles** view (the archive). Closing asks one question: filled through us (which candidate joined) / filled elsewhere / cancelled / put on hold indefinitely — with an optional reason. Captured as an outcome event. A closed role can be reopened (status back to active, event recorded) if the client revives it.

---

## 3. Data model (plain language)

Every table carries `org_id` from day one (single org "Hyperhunt" for now). All schema changes via migration files only — never dashboard edits.

**users** — name, email, role (`consultant` / `delivery_lead` / `ceo_admin`). Created via Google sign-in; role assigned by admin (from the Supabase dashboard — three accounts, one-time work; no user-management UI in Phase 1).

### Identity & access (Phase 1: open internally)

**What RLS is, in plain terms:** Row-Level Security is a rule that lives inside the database table itself, deciding which rows each signed-in user is allowed to see — so the database refuses to hand over forbidden rows no matter what a screen asks. The wall is in the vault, not the door; no forgotten check in the code can leak data.

**Phase 1 decision — everyone internal sees everything.** The team is small and co-located, so we do **not** wall consultants off from each other's clients, roles, or candidates. What RLS enforces in Sprint 1 is narrower but essential:

- **The org boundary** — a user only ever sees rows belonging to their own `org_id`. Single-org today, but this is the tenancy wall that will matter the moment a second agency exists; building it now costs nothing and avoids a terrifying retrofit later.
- **Signed-out = nothing** — no access without a valid session.

Within the org, all three roles (`consultant` / `delivery_lead` / `ceo_admin`) see the same data. Focus comes from **filters/views, not walls**: every client and role list offers **My roles**, **filter by consultant**, and **filter by client** so a consultant can narrow to their own book by choice, while a lead or CEO can look across everyone. Per-consultant hard walls become an ADR when multi-tenancy ships.

*(This is deliberately looser than the vision doc's identity section, which describes the eventual walled state. It's a conscious Phase-1 simplification — recorded here rather than by amending the vision — so the two documents don't silently disagree.)*

**clients** — name, status (`active` / `inactive`), free-text notes, **assigned consultant** (the owner; roles under this client inherit it; reassignable per ADR-014).

**client_contacts** — belongs to a client: name, title, email, phone. The role's hiring manager is picked from these.

**roles** — belongs to a client: title, JD (text + optional original file), onboarding transcript (text + optional original file, if provided), hiring manager (contact reference), priority (`high` / `medium` / `low`), assigned consultant, status (`active` / `paused` / `closed`), interview_rounds (1–5, default 3), open date, closed date, close outcome. Includes the **nullable `calibration` field — reserved, unused, do not touch** (future relevance engine).

**candidates** — the Candidate Bank. One person, one row, forever: full name, email, phone, current company, total experience, location, skills (list), tags (free-form list), **source** (how this person *first* entered Hyperhunt's world — `referral` / `linkedin` / `job_board` / `direct` / `other` — set once, on creation), original CV file reference, parsed profile (structured JSON from the AI), AI candidate summary (text). Full-text search indexes are created on name, company, skills, and summary in this sprint, even though the search UI ships in Sprint 2.

**candidate_roles** — the participation link: candidate ↔ role, current stage, who added them, when, **how they were added** (`origination`: `magic_upload` / `bank_add` — the second path's UI ships Sprint 2, but the column exists now so no migration is needed then), and **role-source** (how this candidate came to *this specific role* — the same picklist as candidate source; on Magic Upload the consultant sets it, on bank-add it's recorded automatically). Two source concepts, deliberately separate: `candidates.source` is first-ever touch (immutable); `candidate_roles.source` is per-application (the same person can arrive at Role A via LinkedIn and Role B via referral without one overwriting the other). A candidate can participate in many roles; this table is what makes the role-centric and candidate-centric views two lenses on the same data. When a participation ends it also carries: `outcome` (`joined` / `rejected_by_client` / `withdrawn_by_candidate` / `offer_declined` / `role_cancelled`), `outcome_reason` (the structured rejection picklist value from ADR-012, null for non-rejections), `stage_reached` (the stage the candidate was in when it ended), and `ended_at`. Storing `stage_reached` on the row means stage-wise conversion ("how many reached Round 2 on this desk?") is a direct query, not an event-history replay.

**pending_matches** — the dedup review queue: incoming candidate data, the suspected existing candidate, match type (`exact_email` / `exact_phone` / `fuzzy`), status (`pending` / `merged` / `kept_separate`).

**events** — the foundation. Append-only, never updated, never deleted:
- `event_type` (from the catalog below)
- `actor` (which user; or `system` for AI-generated events)
- object references (candidate / role / client — whichever apply)
- `payload` (JSON details, e.g. from-stage and to-stage)
- `visibility` (`internal` / `client_safe`)
- `severity` (nullable: `nudge` / `warn` / `escalate`) — **designed in now, used from Sprint 3.** Null on ordinary events; carried by flag-type events to support tiered escalation. Adding this column later would mean migrating the foundation mid-build; adding it now costs one nullable field.
- timestamp

### Event catalog for Sprint 1

| Event | Emitted when | Visibility default |
|---|---|---|
| `client_created` | Client added | internal |
| `role_created` | Role saved | client_safe |
| `role_status_changed` | Active/paused/closed transitions, with reason | client_safe |
| `client_reassigned` | A client's assigned consultant changes (ADR-014) | internal |
| `cv_uploaded` | File lands in storage | internal |
| `candidate_created` | New candidate row created | internal |
| `candidate_merged` | Consultant confirms a duplicate match | internal |
| `candidate_linked_to_role` | Participation created (stage = New); payload records origination (`magic_upload` / `bank_add`) | client_safe |
| `stage_changed` | Any stage move (Sprint 1: via dropdown) | client_safe |
| `participation_ended` | A participation reaches an outcome; payload carries `outcome`, `outcome_reason` (structured picklist for rejections), and `stage_reached` | client_safe |
| `ai_profile_parsed` | Parsing completes | internal |
| `ai_summary_generated` | Summary completes | internal |
| `note_added` | Free-text note on candidate or role | internal |

**Reserved event types — in the taxonomy from day one, first emitted in Sprint 3** (no Sprint 1 UI produces them; the Builder defines them in the schema/type catalog so the foundation never needs migrating):

| Reserved event | Will be emitted when (Sprint 3) | Notes |
|---|---|---|
| `commitment_made` | A delivery promise to a client is captured ("batch of 3 CVs by Thursday 6pm") | Payload: what was promised, to whom, deadline. Watched by the nudge engine. |
| `interaction_logged` | One-tap "called candidate" / "chased feedback" | One tap, never typing. Payload: interaction type + object refs. |
| `feedback_recorded` | Interview feedback captured after a round | Attached to a participation + the stage it concerns. **The spine of Sprint 3's feedback nudges** ("feedback pending > 48h") — designing the event now means Sprint 3 adds the capture UI without a schema migration. |
| `flag_raised` | A nudge rule fires | Carries `severity`: `nudge` → `warn` → `escalate`. |

**Product rule (ADR-011), binding on all sprints: no manual task lists anywhere.** The event log records what happened; the system computes what needs doing. Consultants are never asked to create, maintain, or tick off a to-do. The one-tap interaction log is the maximum manual input the product will ever request.

Rule for the Builder: any future feature that cannot be expressed as a producer or consumer of these events does not belong in the architecture. Timelines and metrics are *views over events* — never separately maintained state.

---

## 4. Magic Upload — the cascade (the product's thesis)

Trigger: consultant drags a CV file (PDF/DOCX) onto a role's pipeline, or clicks Upload on the role. Uses **react-dropzone** for the surface, **Vercel AI SDK** with structured output for parsing.

1. **Store** the file in Supabase Storage, linked to the role. Emit `cv_uploaded`.
2. **Parse** the CV via one Anthropic API call → structured profile: name, email, phone, current company, title, total experience, skills, location, education, work history. Emit `ai_profile_parsed`.
3. **Dedup check** against the Candidate Bank:
   - Exact email or phone match → high-confidence flag.
   - **If the CV has neither email nor phone** (parsing missed them or they're absent): skip exact-match entirely, run fuzzy only, and **always** flag for confirmation — a keyless candidate is never silently auto-created, because the one thing worse than a duplicate is a duplicate we made ourselves without asking.
   - Fuzzy match (similar name + same company, or similar name + overlapping history) → low-confidence flag.
   - Any flag pauses the cascade at a one-screen review: "This looks like [existing candidate] — same person?" The screen shows the **full existing profile** — AI summary, contact details, current company, tags, every role they've participated in and the stage reached, and a link to their CV on file — side by side with the incoming CV's freshly parsed data, differences highlighted. Two buttons: **Merge** (new CV and fresher details attach to the existing profile; emit `candidate_merged`) or **Keep separate** (create new). The consultant decides with everything in view; nothing ever merges without a human tap.
   - No match → continue automatically.
4. **Create or update** the candidate row. Emit `candidate_created` (or merge event).
5. **Link** candidate to the role at stage `New`, recording source and consultant. Emit `candidate_linked_to_role`. If the candidate is already active in another role or was previously submitted to this client, show the conflict warning (derived from events — no extra bookkeeping).
6. **Summarize**: second AI call generates the 3–5 sentence candidate summary (background — the consultant doesn't wait for it). Emit `ai_summary_generated`.
7. **Index**: profile becomes full-text searchable (automatic via Postgres).

**Failure behavior (important):** if parsing fails or the file is unreadable, the upload never silently disappears. The candidate card is created in `New` with whatever was extracted (minimum: the file itself and a "needs review" marker), and the consultant sees exactly what's missing. A broken parse must cost the consultant one glance, not a lost candidate.

**The one-minute test:** from file-drop to candidate-card-visible-in-New must feel instant (< 10 seconds for the card; summary may arrive seconds later). Total consultant effort: one drag, possibly one dedup tap. Zero typing.

---

## 5. AI-assisted role creation (JD + transcript)

Creating a role should also involve near-zero typing:

1. Consultant clicks New Role on a client → provides the JD, the onboarding-call transcript, or both — as pasted text or uploaded document files (PDF/DOCX/TXT). **No audio files** — if a recording exists, it must be transcribed to text elsewhere first (audio transcription is on the backlog).
2. One AI call reads whatever was provided and extracts: proposed title, seniority, location, salary range, key requirements, interview process hints (e.g. "two rounds then HM call" → suggests interview_rounds), and anything the transcript adds that the JD lacks (real must-haves vs. paper requirements, urgency, hiring manager preferences) → pre-fills the role form, with a short "what the call added" note saved to the role.
3. Consultant reviews, corrects anything wrong, picks hiring manager, priority, confirms interview rounds, and saves. Emit `role_created`.

The AI proposes; the human confirms. Extracted fields are suggestions, never silently saved.

**Role fields (the confirmed minimum):** title, client, JD, hiring manager, priority, assigned consultant, interview rounds, open date — plus whatever the JD extraction fills (location, salary range, seniority) stored as part of the role's parsed JD data, not as extra required form fields. No field is required beyond title, client, and consultant.

---

## 6. Screens in Sprint 1

Before the first screen is built, run the **design ritual** (per playbook): generate `docs/design-system.md` with the ui-ux-pro-max skill ("internal SaaS operating system for a recruiting agency; calm, editorial, Notion-like; information-dense but unhurried"), reference it from CLAUDE.md, and obey it everywhere. Components: shadcn/ui. Icons: Lucide. Toasts: Sonner.

1. **Sign-in** — Google button. Nothing else.
2. **Client list** — table of clients (TanStack Table foundations), New Client form (name + status + notes + contacts + assigned consultant). **Filters/views: My clients · filter by consultant.**
3. **Client profile** — details, contacts, assigned consultant (with a reassign control — consultant/lead/CEO can change it), roles under this client grouped **Active / Paused**, with a separate tab or link for **Closed roles**.
4. **New Role** — the AI-assisted flow above (JD and/or transcript in, reviewed form out).
5. **Role workspace** — the heart: pipeline columns (canonical stages, enabled rounds only), candidate cards per stage, the upload dropzone, role header (client, priority, consultant, status control, **editable title**). The upload flow asks the consultant for the **role-source** of this candidate (how they came to this role). A roles overview (across clients) offers **My roles · filter by consultant · filter by client** — the focus lenses that replace hard walls in Phase 1.
6. **Candidate card** (on the board) — name, current company, experience, source, time added. Minimal.
7. **Candidate detail panel** (slide-over from the card) — parsed profile, AI summary, tags editor, original CV link, stage dropdown, notes, and the event timeline for this candidate (derived view — our first proof the event log works). Ending a participation happens here: pick an outcome, and for a rejection pick one reason from the fixed one-tap list (ADR-012) — no typing. The stage the candidate is in at that moment is captured automatically as `stage_reached`.
8. **Dedup review screen** — the merge/keep-separate decision with the full existing profile shown alongside the incoming data.

Fully responsive (consultants live on phones), but desktop-first polish.

---

## 7. Explicitly OUT of Sprint 1

Drag-and-drop movement, stage history views and time-in-stage metrics (Sprint 2) · Candidate Bank browsing/search UI **and the Add-from-Bank flow** (Sprint 2 — schema is ready in Sprint 1, UI is not) · My Desk (Sprint 2) · Composers, nudges (Sprint 3) · Dashboards and metrics (Sprint 4) · Context-aware rejection-reason suggestions (Sprint 3+) · Audio transcription of call recordings (backlog — text transcripts only in Sprint 1) · Tag-driven automations (backlog, needs a specific feature case) · Image CV parsing (backlog) · Custom pipeline stages (deliberately not built — ADR-001; the round count is the only per-role variation) · **A global candidate profile score (explicitly not built — no single number rates a candidate; the future relevance engine ranks per-role, never a universal score)** · Anything on the vision doc's not-build list.

Backlog entries to add now: `audio transcription for onboarding calls — speech-to-text service, needs ADR`, `tag-driven AI behaviors — needs concrete feature case`, `image CV parsing — only if daily reality`, `revisit auto-merge for exact dedup matches if manual confirms prove annoying`.

---

## 8. Acceptance test (the founder runs this by hand)

1. Sign in with Google. ✔ You're in.
2. Create a client with two contacts. ✔ Appears in the list; `client_created` event exists.
3. Create a role by pasting a real JD *and* a real onboarding-call transcript. ✔ Fields pre-filled sensibly, the "what the call added" note captures something the JD didn't say; you corrected at most a couple of fields; role appears with the right pipeline columns.
4. Drag a real (anonymized) PDF CV onto the role. ✔ Candidate card appears in New within seconds; open it: parsed profile is right, summary reads well, CV opens, timeline shows the cascade of events.
5. Upload a DOCX CV of the *same person* with the same email to a second role. ✔ System stops and asks "same person?" — you can see the existing full profile (summary, roles, CV) next to the incoming data; tap Merge; the candidate now shows both roles; no second profile exists.
6. Upload the same person to a role at the same client again. ✔ Conflict warning appears.
7. Pause the role with a reason. ✔ Uploads and stage moves are blocked with a clear message, but the role is still right there in the working view under a Paused group with its reason and date; event recorded. Then close the role (reason: on hold indefinitely). ✔ It disappears from the working view and appears in the Closed roles view with its outcome; reopen it and it comes back to Active — every transition in the event log.
8. Try uploading a JPG. ✔ Friendly rejection, nothing breaks.
9. Ask the Builder to show the events table. ✔ Every action above is there, timestamped, attributed, append-only — and the schema shows the severity column and reserved event types waiting for Sprint 3.
10. **Access & filters (open internally):** create a second consultant account and sign in as them. ✔ They can see all clients and roles (Phase 1 is open internally) — then apply **My clients / My roles** and the view narrows to just theirs; clear it and everything returns. Sign in as the Delivery Lead, reassign a client to a different consultant. ✔ The change sticks, a `client_reassigned` event exists, and the "by consultant" filter reflects it.
11. **Outcome + stage reached:** move a candidate to Round 2, then reject them. ✔ You must pick a reason from the one-tap list — the system won't let you reject without one, and there's no text box to type in. Afterwards, ask the Builder to show that participation row: `outcome = rejected_by_client`, `outcome_reason = comp_mismatch` (or whatever you picked), `stage_reached = round_2`, `ended_at` timestamped — and the `participation_ended` event carries the same. This is the proof stage-wise conversion will work in Sprint 4 without replaying history.
12. **Keyless CV:** upload a CV with no email or phone. ✔ The system doesn't silently create a duplicate — it runs a fuzzy check and flags for your confirmation before creating or merging.
13. **Join flags siblings:** put the same candidate in two roles, then mark them `joined` on one. ✔ The system flags their other live participation for one-tap close (it doesn't close it for you).

**Sprint 1 is done when:** all thirteen checks pass with your own hands, the sprint-level metric holds (CV-in-hand → candidate-in-pipeline under one minute, zero duplicate entry), docs are updated, ADRs 001–014 exist in `docs/adr/`, and everything is committed and pushed.

---

## 9. Build order for the Builder (suggested slices)

1. Project scaffold + Supabase schema migrations (all tables, indexes, full event catalog including reserved types and the severity column) + auth with three roles + **row-level security enforcing the org boundary on every org_id table** (org isolation + signed-out = no access; per-consultant walls deliberately deferred — Phase 1 is open internally).
2. Design ritual → `docs/design-system.md`.
3. Clients: list, create, profile, assigned consultant + reassign control, My-clients / by-consultant filters.
4. Roles: AI-assisted creation (JD + transcript) + role workspace shell with pipeline columns + editable title + My-roles / by-consultant / by-client filters.
5. Magic Upload: storage + parsing + candidate creation + linking (with role-source capture; no dedup yet).
6. Dedup flow (including the keyless-CV rule) + conflict warnings.
7. AI summary + candidate detail panel + event timeline.
8. Role status (pause/close) + stage dropdown + participation outcome flow (one-tap rejection picklist, stage_reached capture, join-flags-siblings) + polish + responsive pass.

Each slice: Plan Mode first, small commits, code review + `/ponytail-review` before each commit, founder clicks through before moving on.

---

*Per the HSES constitution: this spec is law for Sprint 1. Scope changes go back to the PM Office, not into the Workshop.*