# Hyperhunt OS

An AI-in-the-background operating system for recruiting operations.

Internal tool for Hyperhunt today; first product of the Hyperhunt AI SaaS
tomorrow. It replaces the spreadsheets and WhatsApp groups agencies actually
run on — every action produces a structured event, and every derived view
(timelines, dashboards, client briefs, nudges) is a consumer of that event
stream.

**Status:** Sprint 1 in progress — identity, the spine, Magic Upload, and the
event log foundation.

---

## Start here

| If you want to… | Read |
|---|---|
| Understand the product in one sitting | [`docs/founder-product-briefing.md`](docs/founder-product-briefing.md) |
| Know exactly what we're building and why | [`docs/product-vision.md`](docs/product-vision.md) |
| See where the build currently stands | [`docs/status.md`](docs/status.md) |
| Work on the code | [`CLAUDE.md`](CLAUDE.md) — the build rulebook |

## Repo map

```
CLAUDE.md              Build rulebook — read automatically by Claude Code
docs/
  founder-product-briefing.md   Single-doc onboarding: vision → stack → roadmap
  product-vision.md             Source of truth: modules, philosophy, phasing
  status.md                     Current sprint, what's done, what's blocked
  backlog.md                    Everything deferred, with reasons
  cheatcodes.md                 Approved toolkit; reviewed/declined tools
  prompt-library.md             Reusable prompts for sprints and sessions
  START-HERE.md                 Founder's setup and daily operating manual
  specs/                        Approved feature specs, one per sprint
  adr/                          Architecture decision records, numbered
```

## Principles

- **Elimination over addition** — every feature must delete manual work, not
  merely add capability
- **The event log is the foundation** — no separately maintained state; all
  views derive from events
- **Role-centric** — the hiring pipeline belongs to the Role; candidates
  participate in it
- **AI is invisible** — background, event-triggered, no chat surfaces

## Stack

Next.js (App Router, TypeScript) on Vercel · Supabase (Postgres, Auth,
Storage, RLS) · one Anthropic API integration. Additions require an ADR.

## Governance

Built under HSES (Hyperhunt AI Software Engineering System) — see the
separate `HSES` repo for the constitution, operating manual, and improvement
register. In short: documentation before implementation, architecture frozen
during implementation, one sprint one objective, every consequential choice
recorded as an ADR.

---

*When documents disagree: `docs/product-vision.md` wins for product
questions, `CLAUDE.md` wins for build questions.*