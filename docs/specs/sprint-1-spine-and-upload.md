# Sprint 1 Spec — The Spine + Magic Upload + Event Log Foundation

**Status:** Approved for build
**Sprint objective (one line):** A real CV uploaded to a real role produces the full automatic cascade — parsed, deduplicated, in the pipeline, summarized, searchable — with every step recorded as an event, in under one minute, with zero duplicate typing.
**Kills:** Duplicate data entry.
**Timeline:** 3–4 weeks at ~20h/week.
**Governance:** Built under CLAUDE.md rules. Stack is frozen: Next.js (App Router, TypeScript) on Vercel, Supabase (Postgres, Auth, Storage), Anthropic API via Vercel AI SDK. No other frameworks without an ADR.

---

## 1. Decisions resolved in this spec (each becomes a short ADR)

| # | Decision | Resolution |
|---|---|---|
| ADR-001 | Pipeline stages | Fixed canonical set. No custom stages. Per-role choice of interview round count (1–3). |
| ADR-002 | Role lifecycle | Role status: `active` / `paused` / `closed`. **Paused freezes the pipeline** (no uploads, no stage moves) but the role stays visible in the working view under a Paused group, with reason and date, so follow-up is never missed. **Closed removes the role from the working view** into a separate Closed roles view, with a recorded outcome and reason ("client put it on hold indefinitely" is one of them). Closed roles can be reopened. Not a pipeline stage. |
| ADR-003 | Candidate terminal outcomes | `joined` / `rejected` only. Both are first-class outcome events. |
| ADR-004 | Deduplication | Nothing merges automatically. All suspected duplicates (exact and fuzzy) are flagged for one-tap consultant confirmation. |
| ADR-005 | Auth | Google sign-in via Supabase Auth. Three roles from day one: `consultant`, `delivery_lead`, `ceo_admin`. Sprint 1 screens are identical across roles. |
| ADR-006 | CV formats | PDF and DOCX only. Images/others rejected with a clear message. |
| ADR-007 | Data migration | Clean start. No import of existing sheets. |
| ADR-008 | Candidate tags | Free-form tags field exists on candidates from day one (searchable later). No automation or AI reads tags in Phase 1. |
| ADR-009 | AI-assisted role creation | Creating a role accepts a JD and/or an onboarding-call transcript — pasted text or a document file. AI pre-fills role fields from whatever is provided; consultant reviews and confirms before save. Audio files are not accepted (speech-to-text is a new service — fails the stack test); audio transcription goes to the backlog. |

---

## 2. The canonical pipeline

Fixed stage set, identical semantics on every role:

```
New → Screening → Submitted → Interview 1 → Interview 2 → Interview 3 → Offer → Joined / Rejected
```

- **Rounds are not a separate concept from stages.** Interview 1, 2, and 3 *are* stages in the fixed list above. Each role's `interview_rounds` setting (1, 2, or 3; default 2, editable while active) only controls how many of those interview columns the role shows. A one-interview role shows one interview column; the stages themselves — and what they mean — are identical on every role, which is what keeps metrics comparable across roles and clients.
- `Joined` and `Rejected` are terminal. A rejection may carry an optional reason (short text) and the stage it happened from — captured in the event, not as a required form.
- **Sprint 1 movement rule:** Magic Upload places candidates in `New`. Manual stage movement (drag-and-drop, history, time-in-stage) is Sprint 2. In Sprint 1 a candidate can only be moved via a simple stage dropdown on the candidate detail panel — minimal, no drag-and-drop, no metrics — so the pipeline isn't a dead end during testing. Every such move still emits a proper stage-change event (the event log must be complete from day one even if the UI is minimal).

### Role status (not a stage)

- `active` — normal operation. Lives in the main role lists.
- `paused` — **frozen but in sight.** No CVs flow, no stage moves — uploads and movement are blocked with a friendly "this role is paused" message. Pausing requires a one-line reason, captured in the event. The role remains in the working view, grouped under a **Paused** section with its reason and pause date, so consultants see it every day and can't forget to follow up. (Sprint 3 hook: "role paused for N+ days — check status with client" becomes a nudge rule over this event, for free.)
- `closed` — the ending. The role **leaves the working view** and moves to a separate **Closed roles** view (the archive). Closing asks one question: filled through us (which candidate joined) / filled elsewhere / cancelled / put on hold indefinitely — with an optional reason. Captured as an outcome event. A closed role can be reopened (status back to active, event recorded) if the client revives it.

---

## 3. Data model (plain language)

Every table carries `org_id` from day one (single org "Hyperhunt" for now). All schema changes via migration files only — never dashboard edits.

**users** — name, email, role (`consultant` / `delivery_lead` / `ceo_admin`). Created via Google sign-in; role assigned by admin.

**clients** — name, status (`active` / `inactive`), free-text notes.

**client_contacts** — belongs to a client: name, title, email, phone. The role's hiring manager is picked from these.

**roles** — belongs to a client: title, JD (text + optional original file), onboarding transcript (text + optional original file, if provided), hiring manager (contact reference), priority (`high` / `medium` / `low`), assigned consultant, status (`active` / `paused` / `closed`), interview_rounds (1–3), open date, closed date, close outcome. Includes the **nullable `calibration` field — reserved, unused, do not touch** (future relevance engine).

**candidates** — the Candidate Bank. One person, one row, forever: full name, email, phone, current company, total experience, location, skills (list), tags (free-form list), source (`referral` / `linkedin` / `job_board` / `direct` / `other`), original CV file reference, parsed profile (structured JSON from the AI), AI candidate summary (text). Full-text search indexes are created on name, company, skills, and summary in this sprint, even though the search UI ships in Sprint 2.

**candidate_roles** — the participation link: candidate ↔ role, current stage, who added them, when. A candidate can participate in many roles; this table is what makes the role-centric and candidate-centric views two lenses on the same data.

**pending_matches** — the dedup review queue: incoming candidate data, the suspected existing candidate, match type (`exact_email` / `exact_phone` / `fuzzy`), status (`pending` / `merged` / `kept_separate`).

**events** — the foundation. Append-only, never updated, never deleted:
- `event_type` (from the catalog below)
- `actor` (which user; or `system` for AI-generated events)
- object references (candidate / role / client — whichever apply)
- `payload` (JSON details, e.g. from-stage and to-stage)
- `visibility` (`internal` / `client_safe`)
- timestamp

### Event catalog for Sprint 1

| Event | Emitted when | Visibility default |
|---|---|---|
| `client_created` | Client added | internal |
| `role_created` | Role saved | client_safe |
| `role_status_changed` | Active/paused/closed transitions, with reason | client_safe |
| `cv_uploaded` | File lands in storage | internal |
| `candidate_created` | New candidate row created | internal |
| `candidate_merged` | Consultant confirms a duplicate match | internal |
| `candidate_linked_to_role` | Participation created (stage = New) | client_safe |
| `stage_changed` | Any stage move (Sprint 1: via dropdown) | client_safe |
| `candidate_rejected` / `candidate_joined` | Terminal outcomes | client_safe |
| `ai_profile_parsed` | Parsing completes | internal |
| `ai_summary_generated` | Summary completes | internal |
| `note_added` | Free-text note on candidate or role | internal |

Rule for the Builder: any future feature that cannot be expressed as a producer or consumer of these events does not belong in the architecture. Timelines and metrics are *views over events* — never separately maintained state.

---

## 4. Magic Upload — the cascade (the product's thesis)

Trigger: consultant drags a CV file (PDF/DOCX) onto a role's pipeline, or clicks Upload on the role. Uses **react-dropzone** for the surface, **Vercel AI SDK** with structured output for parsing.

1. **Store** the file in Supabase Storage, linked to the role. Emit `cv_uploaded`.
2. **Parse** the CV via one Anthropic API call → structured profile: name, email, phone, current company, title, total experience, skills, location, education, work history. Emit `ai_profile_parsed`.
3. **Dedup check** against the Candidate Bank:
   - Exact email or phone match → high-confidence flag.
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
2. **Client list** — table of clients (TanStack Table foundations), New Client form (name + status + notes + contacts).
3. **Client profile** — details, contacts, roles under this client grouped **Active / Paused**, with a separate tab or link for **Closed roles**.
4. **New Role** — the AI-assisted flow above (JD and/or transcript in, reviewed form out).
5. **Role workspace** — the heart: pipeline columns (canonical stages, enabled rounds only), candidate cards per stage, the upload dropzone, role header (client, priority, consultant, status control).
6. **Candidate card** (on the board) — name, current company, experience, source, time added. Minimal.
7. **Candidate detail panel** (slide-over from the card) — parsed profile, AI summary, tags editor, original CV link, stage dropdown, notes, and the event timeline for this candidate (derived view — our first proof the event log works).
8. **Dedup review screen** — the merge/keep-separate decision with the full existing profile shown alongside the incoming data.

Fully responsive (consultants live on phones), but desktop-first polish.

---

## 7. Explicitly OUT of Sprint 1

Drag-and-drop movement, stage history views and time-in-stage metrics (Sprint 2) · Candidate Bank browsing/search UI (Sprint 2) · My Desk (Sprint 2) · Composers, nudges (Sprint 3) · Dashboards and metrics (Sprint 4) · Audio transcription of call recordings (backlog — text transcripts only in Sprint 1) · Tag-driven automations (backlog, needs a specific feature case) · Image CV parsing (backlog) · Any custom pipeline stages (requires an ADR with a real case) · Anything on the vision doc's not-build list.

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
9. Ask the Builder to show the events table. ✔ Every action above is there, timestamped, attributed, append-only.

**Sprint 1 is done when:** all nine checks pass with your own hands, the sprint-level metric holds (CV-in-hand → candidate-in-pipeline under one minute, zero duplicate entry), docs are updated, ADRs 001–009 exist in `docs/adr/`, and everything is committed and pushed.

---

## 9. Build order for the Builder (suggested slices)

1. Project scaffold + Supabase schema migrations (all tables, indexes, event catalog) + auth with three roles.
2. Design ritual → `docs/design-system.md`.
3. Clients: list, create, profile.
4. Roles: AI-assisted creation (JD + transcript) + role workspace shell with pipeline columns.
5. Magic Upload: storage + parsing + candidate creation + linking (no dedup yet).
6. Dedup flow + conflict warnings.
7. AI summary + candidate detail panel + event timeline.
8. Role status (pause/close) + stage dropdown + polish + responsive pass.

Each slice: Plan Mode first, small commits, code review + `/ponytail-review` before each commit, founder clicks through before moving on.

---

*Per the HSES constitution: this spec is law for Sprint 1. Scope changes go back to the PM Office, not into the Workshop.*