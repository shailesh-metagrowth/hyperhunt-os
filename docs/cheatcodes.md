# Hyperhunt OS — Cheatcodes

Curated accelerators, each mapped to the module it serves and the sprint that
unlocks it. Rule zero: a cheatcode is installed when its sprint needs it,
not before. An arsenal is not a backpack.

---

## 1. Design & UI (makes the product look like a product)

| Cheatcode                                                           | What it does                                                                                                                                       | Used for                                                                                                                                     | When                              |
| ------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------- |
| **frontend-design skill** (Anthropic, github.com/anthropics/skills) | Steers Claude away from generic AI-looking UI toward intentional typography, spacing, hierarchy                                                    | The overall calm, Notion-like feel                                                                                                           | Sprint 1, day one                 |
| **UI/UX Pro Max skill** (community, 100k+ stars)                    | Design intelligence: 60+ UI styles, 161 palettes, 57 font pairings, 99 UX rules, chart guidance; generates a full design system from a description | Generating the Hyperhunt OS design system once, then enforcing it everywhere                                                                 | Sprint 1, before the first screen |
| **shadcn/ui** (ui.shadcn.com)                                       | Copy-paste React components (tables, dialogs, dropdowns, forms) — code you own, not a dependency; the default pairing with Next.js + Tailwind      | Every screen's building blocks                                                                                                               | Sprint 1                          |
| **21st.dev + Magic MCP**                                            | Marketplace of polished shadcn-style components + an MCP that generates components from a text prompt inside Claude Code                           | When a screen needs something shadcn lacks (fancy empty states, stat cards)                                                                  | Sprint 2+ as needed               |
| **Motion** (motion.dev, formerly Framer Motion)                     | Animation library                                                                                                                                  | Drag-and-drop feel, stage-move transitions, Today-queue reordering. Use sparingly — a calm OS animates like a well-made drawer, not a casino | Sprint 2                          |
| **Lucide icons** (lucide.dev)                                       | The icon set (ships with shadcn)                                                                                                                   | Everywhere                                                                                                                                   | Sprint 1                          |

**Install commands (run inside Claude Code):**

```
/plugin marketplace add nextlevelbuilder/ui-ux-pro-max-skill
/plugin install ui-ux-pro-max@ui-ux-pro-max-skill
```

shadcn, Motion, Lucide: just tell the Builder to use them — they're npm
libraries it installs itself.

**Sprint 1 design ritual:** before the first screen, run UI/UX Pro Max to
generate the design system (prompt it with: "internal SaaS operating system
for a recruiting agency; calm, editorial, Notion-like; information-dense but
unhurried"), save the output as `docs/design-system.md`, and reference it in
CLAUDE.md so every future screen obeys it. One decision, enforced forever.

---

## 2. Building blocks for OUR specific modules

| Cheatcode                                         | Serves                                                      | Why this one                                                                                             |
| ------------------------------------------------- | ----------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| **dnd-kit** (dndkit.com)                          | Module 4: Pipeline drag-and-drop                            | The standard React drag-drop library — accessible, smooth, made for kanban boards                        |
| **TanStack Table**                                | Modules 2 & 5: Candidate Bank lists, My Desk role portfolio | Headless tables: sorting, filtering, grouping — pairs with shadcn's table styles                         |
| **Recharts**                                      | Module 7: Delivery Lead & CEO dashboards                    | Simple declarative charts; enough for six metrics, no BI platform energy                                 |
| **react-dropzone**                                | Module 3: Magic Upload                                      | The drag-a-CV-anywhere upload surface                                                                    |
| **Vercel AI SDK**                                 | Module 9: Background AI                                     | Clean wrapper for the Anthropic API calls (parsing, summaries, composers) with structured output support |
| **@react-pdf/renderer** (or server-side HTML→PDF) | Module 6: Submission Packs                                  | Generating the branded, contact-stripped submission CV as a PDF                                          |
| **Sonner**                                        | Everywhere                                                  | The toast notifications shadcn recommends — "Candidate added to pipeline ✓"                              |

You don't install these yourself — you name them in sprint specs, and the
Builder uses them. Their presence in the spec is what stops the Builder from
inventing bespoke solutions to solved problems.

---

## 3. Claude Code power moves (the Builder's own cheatcodes)

| Cheatcode                                        | What it unlocks                                                                                                                                                                                                                                                                                                                                                                       |
| ------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Plan Mode** (Shift+Tab)                        | Research-and-propose before any file is touched. Non-negotiable session opener                                                                                                                                                                                                                                                                                                        |
| **Context7 MCP** (Upstash)                       | Feeds Claude _current_ docs for any library — kills the "trained on last year's API" bug class. Say "use context7" when working with Supabase/Next.js APIs                                                                                                                                                                                                                            |
| **Supabase MCP** (official)                      | Builder reads your schema, writes migrations, queries dev data directly                                                                                                                                                                                                                                                                                                               |
| **Playwright plugin** (official marketplace)     | Builder opens the running app and clicks through flows itself — your QA co-pilot                                                                                                                                                                                                                                                                                                      |
| **TypeScript LSP plugin** (official marketplace) | Builder catches its own type errors while writing, not at build time                                                                                                                                                                                                                                                                                                                  |
| **Ponytail** (DietrichGebert/ponytail, 84k+ ★)   | Your elimination philosophy injected into the Builder: a pre-write decision ladder (does this need to exist → stdlib → native feature → existing dep → one line → minimum that works). Benchmarked ~54% less code, ~20% cheaper, ~27% faster on real agentic runs. `/ponytail-review` hunts over-engineering in diffs. Runs two small lifecycle hooks (security-scanned, open source) |
| **Code review plugin** (official marketplace)    | Correctness-focused review before commits — the second set of eyes a non-technical founder doesn't otherwise have. Pairs with `/ponytail-review`: one hunts wrongness, the other hunts complexity                                                                                                                                                                                     |
| **Custom skills** (`.claude/skills/`)            | When a pattern repeats twice (event-emitting migrations, composer prompts), have the Builder write it up as a project skill — your improvement register, made executable                                                                                                                                                                                                              |
| **Hooks**                                        | Automation on events, e.g. run typecheck after every edit. Add only when a repeated annoyance justifies it                                                                                                                                                                                                                                                                            |
| **/plugin marketplace**                          | The official directory (claude-plugins-official) is pre-connected; community marketplaces can be added by repo name. **Trust hierarchy: official marketplace first, high-adoption community second, obscure never without a skeptical read**                                                                                                                                          |

---

## 4. Rules of engagement (so cheatcodes don't become the new bloat)

1. **One at a time, when the sprint demands it.** Installing ten skills on
   day one bloats the Builder's context and dilutes its attention.
2. **Community skills are third-party instructions.** Before installing any
   non-Anthropic skill, have the Builder read its SKILL.md aloud and confirm
   it only contains design/coding guidance. High-star, widely-used ones
   (like UI/UX Pro Max) are safer bets; obscure ones need a skeptical read.
3. **Every adopted library gets one ADR line.** "Chose dnd-kit for pipeline
   drag-drop; alternative was react-beautiful-dnd (unmaintained)." Ten
   seconds now; no archaeology later.
4. **The design system is generated once, then frozen.** Re-running design
   generators per-screen produces a patchwork. One system, referenced in
   CLAUDE.md, obeyed everywhere.
5. **If a cheatcode fights the stack, the stack wins.** Everything here was
   chosen to sit inside Next.js + Tailwind + shadcn + Supabase. Anything
   demanding its own runtime, framework, or service fails the audition.

---

## 5. Reviewed & declined (so we don't re-litigate)

| Candidate                                         | Verdict                | Why                                                                                                                                                                                                                                                                                                                                                                            |
| ------------------------------------------------- | ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **claude-mem** (auto session memory)              | Declined               | Makes chats secretly permanent — unreviewed transcript sediment injected into future sessions. Our memory is deliberately curated (CLAUDE.md, specs, ADRs, status.md): every persistent context is a document a human approved. Revisit only if re-explaining across sessions becomes a real, repeated pain — and even then, the first fix is a missing document, not a plugin |
| **Obsidian skill/vault** (project knowledge base) | Declined for the build | The repo is the single source of truth (constitution rule); a vault is a second knowledge store, and split truth is how docs rot. Fine as personal note-taking, but nothing project-critical lives there and the Builder doesn't read it                                                                                                                                       |
