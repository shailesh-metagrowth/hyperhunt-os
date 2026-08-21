# Hyperhunt OS — Backlog

The pressure valve. Every good idea that doesn't belong in the current sprint
lands here with one line — brain unclogged, "one sprint, one objective"
protected.

Format: `- [Idea] — [why deferred, briefly]`
Sorted loosely by when they might genuinely earn a promotion.

## Deferred from Phase 1 (per vision doc non-goals)

- User invitation and management UI — CEO creates the 3 accounts in Supabase dashboard for now; build a UI when a fourth+ account is a real recurring chore.
- Password reset UI — not needed; Google sign-in (Sprint 1) handles authentication, so there are no passwords to reset.
- AI sourcing / relevance engine — Idea 1; attaches later to the calibration slot and outcome events (this is the future big bet, deliberately not first).
- Calendar sync and self-scheduling links — swampy; interview date entered once as an event is enough for Phase 1.
- Email/WhatsApp inbox inside the product — comms stay where they live; the composers meet them there.
- In-app team chat — Hyperhunt already lives in WhatsApp; adding chat inside the OS eliminates no chore.
- Auto-dialers — same reasoning.
- A manual task manager — the computed Today queue is the task manager.
- Gamified leaderboards — process health, not person-ranking.
- Client logins and portals — internal phase has no external users.
- Configurable BI, custom fields, custom dashboards, metric builder — the incumbents' bloat disease.
- Notifications infrastructure beyond in-app badges — three consultants in one office and one WhatsApp; the fancy version can wait.
- Candidate portal / community / marketplace — later phases or never.
- Mobile app — web app must be fully responsive; native app is external-phase at best.
- Billing — internal phase has no billing.
- AI chat interface, AI interviewing — invisible-AI philosophy; no chat surface.
- Permissions beyond three roles (Senior Consultant tier, custom permissions) — only if a real permission difference emerges.
- Per-consultant data walls (consultants restricted to only their own clients/candidates) — Phase 1 is open internally: all internal users see everything, focus comes from filters. Hard walls wait for the external/multi-tenant phase.

## Non-essential plugins (add when they earn their install)

- Additional official-marketplace plugins beyond the day-one set — install when a specific need appears, not preemptively.

## Product improvements captured during vision drafting

- Cross-client benchmark nudges (external phase) — the data network effect that requires multiple orgs.
- One-tap interaction log for calls / WhatsApp messages — resolves the "wrong nudge" tension; decide before Sprint 3 whether one tap counts as data entry or a heartbeat (Open Decision #6 in vision doc).
- WhatsApp Cloud API delivery of client briefs — replaces copy-to-paste when OBA verification completes.
- Direct interactive client portal for the intelligence layer — first external-phase differentiator.

## Deferred during Sprint 1 spec work

- Audio transcription of onboarding calls — speech-to-text is a new service; needs an ADR. Text transcripts only for now.
- Image/scanned CV parsing — only if it becomes daily reality.
- Tag-driven AI behaviours — needs a concrete feature case first.
- Auto-merge on exact dedup matches — revisit if manual confirmations prove annoying in practice.
- Context-aware rejection-reason suggestions (tailored by role, client, candidate) — Sprint 3+, once enough rejection data exists to make suggestions meaningful.
- AI best-match candidate recommendations when adding from the Bank — relevance engine territory; Sprint 2 ships attribute-based ranking only.
- Global candidate profile score — rejected on principle: relevance is relational, so a single number answers no real question.

## Post-Sprint-4 (internal phase, once event history exists)

- **Hyperhunt OS MCP server — conversational strategy layer over live data.**
  A read-only MCP server exposing query tools (search_candidates,
  get_role_pipeline, get_metrics, list_stalled_roles) so the founder and
  leadership can ask strategic questions in Claude against live Hyperhunt
  data: "which roles have had no client feedback in two weeks?", "what
  rejection reasons dominate for fintech clients this quarter?".
  *Why deferred:* an MCP over an empty database answers nothing — its value
  comes entirely from accumulated events. Needs a quarter of real history.
  *Why it fits:* the conversation lives in Claude, not in the product, so it
  respects the no-chat-surface rule. It is another consumer of the event
  stream — no new data required.
  *Cost when built:* ~2-3 days. The Sprint 1 schema (event log, org_id, RLS,
  structured outcomes) is already MCP-ready; no schema change needed.
  *Rules when built:* read-only always (no write access via MCP); the MCP
  authenticates as a specific user and inherits their RLS scope.
  *Strategic note:* "ask your recruiting data anything, from Claude" is a
  differentiated external-phase selling point no incumbent can match.
  Logged Jul 2026.

## Ideas that surfaced but explicitly rejected

- (Add here anything considered and refused, with the reason, so we don't re-litigate later.)

---

*Add one line per idea, no essays. The backlog is a pressure valve, not a
product spec. Anything that genuinely belongs in a future sprint gets
promoted from here when its time comes.*