# HYPERHUNT OS — START HERE
### The complete setup & work document. Everything in one place, in execution order.

---

# PART 0 — Your Kit (5 files + this one)

| File | Where it goes | Purpose |
|---|---|---|
| `hyperhunt-os-product-vision.md` | Repo → `docs/product-vision.md` AND Claude Project knowledge | Source of truth for WHAT we're building |
| `CLAUDE.md` | Repo root | The Builder's operating instructions |
| `founder-build-playbook.md` | Claude Project knowledge | Daily working rhythm |
| `complete-setup-guide.md` | Your desktop | Detailed click-by-click backup of Part 1–5 below |
| `cheatcodes.md` | Repo → `docs/cheatcodes.md` AND Claude Project knowledge | Curated accelerators; CLAUDE.md instructs the Builder to use them |
| **This file** | Print it / keep open | The master sequence |

**The system in one paragraph:** Claude.ai (a Project called "Hyperhunt OS —
Build") is your Guide & Project Manager — you plan there. Claude Code inside
VS Code is your Builder — it writes all code. GitHub is permanent memory —
nothing is real until committed. ChatGPT is not part of this system. You are
not learning to be an engineer; you are running an engineering process, and
the process is this document.

---

# PART 1 — Accounts (~15 min)

☑ **1.1 GitHub** — you have it; be logged in. ✓ done

☑ **1.2 Supabase** (database, free): supabase.com → sign in with GitHub →
New project → name `hyperhunt-os-dev` → **Generate** password and SAVE IT
in your password manager → region **Mumbai (ap-south-1)** → Create. Close tab. ✓ done

☑ **1.3 Vercel** (hosting, free): vercel.com → Sign up → Continue with
GitHub. Done, close tab. ✓ done

☐ **1.4 Meta verification** ⏳ PENDING — do soon; its weeks-long clock only starts on submission (background, needed months from now):
business.facebook.com → Settings → Business info → Start verification →
submit documents. Takes weeks, runs by itself. Start it, forget it.

---

# PART 2 — Tools on your Mac (~20 min)

☑ **2.1** Open Terminal (Cmd+Space → type `terminal` → Enter). ✓ done

☑ **2.2** Check Git: paste `git --version` → Enter. See a version number?
Move on. Popup asking to install developer tools? Click Install, wait, retry. ✓ done

☑ **2.3** Node.js: browser → **nodejs.org** → download **LTS** for macOS →
install like any app. Verify in Terminal: `node --version`. ✓ done

☑ **2.4** Claude Code: in Terminal:
```
npm install -g @anthropic-ai/claude-code
```
Verify: `claude --version`. (Trouble? docs.claude.com → Claude Code, or ask
me and paste the error.) ✓ done

☑ **2.5** VS Code (you have it): Extensions icon (left sidebar) → search
**Claude Code** → Install the Anthropic one. ✓ done

---

# PART 3 — Create the project (~20 min)

☐ **3.1** github.com → **+** → New repository → name `hyperhunt-os` →
**Private** → tick "Add a README" → Create.

☐ **3.2** Terminal, one line at a time (swap in YOUR-USERNAME):
```
mkdir -p ~/Developer
cd ~/Developer
git clone https://github.com/YOUR-USERNAME/hyperhunt-os.git
```

☐ **3.3** VS Code → File → Open Folder → Developer → hyperhunt-os.

☐ **3.4** Create the structure (right-click in the file panel):
- New File `CLAUDE.md` → paste the CLAUDE.md content → save (Cmd+S)
- New Folder `docs` → inside it:
  - New File `product-vision.md` → paste the vision doc → save
  - New File `cheatcodes.md` → paste the cheatcodes doc → save
  - New Folder `specs` · New Folder `adr`
  - New File `backlog.md` (empty) · New File `status.md` (empty)

☐ **3.5** First commit: Source Control icon (left sidebar) → message
`chore: baseline — governance and vision` → **Commit** → **Sync Changes**.
Refresh the repo on github.com; your files should be there. ✅

---

# PART 4 — Wake the Builder (~20 min)

☐ **4.1** In VS Code open the Claude Code panel (Claude icon, or
Terminal → New Terminal → type `claude`). Sign in via browser when prompted.

☐ **4.2** Brief it. Type:
```
Read CLAUDE.md and docs/product-vision.md. Summarize in plain language what
we are building and what rules you must follow.
```
If it correctly describes Hyperhunt OS and the rules → your Builder is alive. ✅

☐ **4.3** Install the day-one cheatcodes. In Claude Code:
```
/plugin
```
From the official marketplace install (skip any you can't find — none block
starting): **TypeScript LSP** · a **git commit helper** · **Playwright
browser testing**.

Also install the official **code review** plugin if listed (correctness
eyes before every commit).

Then the design skill:
```
/plugin marketplace add nextlevelbuilder/ui-ux-pro-max-skill
/plugin install ui-ux-pro-max@ui-ux-pro-max-skill
```
And ponytail — your elimination philosophy, injected into the Builder:
```
/plugin marketplace add DietrichGebert/ponytail
/plugin install ponytail@ponytail
```
Also grab the **frontend-design** skill from github.com/anthropics/skills
(ask the Builder: "install the frontend-design skill from anthropics/skills
into .claude/skills/").

☐ **4.4** Connect Supabase MCP: supabase.com → your project → follow their
"Connect → MCP" instructions for Claude Code (or ask the Builder to walk you
through it — connect the **dev** project only).

*Everything else in cheatcodes.md waits for its sprint. One at a time.*

---

# PART 5 — Set up your Project Manager (~10 min)

☐ **5.1** Claude.ai → Projects → **Create project** → name
`Hyperhunt OS — Build`.

☐ **5.2** Project knowledge → Add content → upload:
`product-vision.md` · `founder-build-playbook.md` · `cheatcodes.md`.

☐ **5.3** Open a new chat in the Project and paste:
```
You are my project manager for building Hyperhunt OS (vision doc, playbook,
and cheatcodes are in project knowledge). I am a non-technical founder.
Claude Code in VS Code is my builder. We work sprint by sprint per the vision
doc's phasing. Track progress, keep me on scope, explain everything in plain
language. Setup is complete. First task: write the Sprint 1 feature
specification (Spine + Magic Upload + event log).
```

**🎉 SETUP COMPLETE. The clock starts here.**

---

# PART 6 — The Build Loop (your job, on repeat, until shipped)

1. **PLAN** — Project chat: "What's next?" → work until you have a spec you
   can explain in plain words.
2. **COMMIT THE SPEC** — save as `docs/specs/sprint-X-name.md` → Source
   Control → Commit → Sync. The spec is now law.
3. **BUILD** — Claude Code → Shift+Tab to **Plan Mode** → paste:
   `Read CLAUDE.md and docs/specs/[filename]. Propose your implementation
   plan before writing any code.` → question it → approve only when it makes
   sense to you.
4. **TEST** — after each piece, run the app and click through it like a
   consultant. Wrong or confusing = say so now.
5. **SAVE** — commit working pieces, small and often. Never end the day
   with unsaved work.
6. **CLOSE** — "Confirm definition of done: works, tested, docs updated,
   committed." → then `/clear`.
7. **REPORT** — tell the Project chat what's done → "What's next?" → loop.

### First build session only — the Design Ritual
Before any screen exists, in Claude Code:
```
Use the ui-ux-pro-max skill to generate a complete design system for:
"internal SaaS operating system for a recruiting agency; calm, editorial,
Notion-like; information-dense but unhurried." Save it as
docs/design-system.md and reference it from CLAUDE.md.
```
One design decision, frozen, obeyed by every future screen.

---

# PART 7 — Rhythms & Memory

**Daily (even 30 min):** Project chat → "Where are we? Today's single
objective?" → do that one thing → commit → new ideas get one line in
`docs/backlog.md`, nothing more.

**Weekly (30 min):** sprint progress vs. timeline · process learnings →
HSES improvement register · next week's ONE objective.

**Context management (how Claude never forgets):**
- Chats are disposable; the Project is permanent. New chat per objective.
- `docs/status.md` = the baton: half a page — current sprint, done, in
  progress, next, blockers. End every PM session with *"Update status.md
  for handoff"* → commit it → re-upload to project knowledge when it
  changes meaningfully.
- Claude Code side: `/clear` per feature; CLAUDE.md + the spec reload
  context each session.
- Missing context in a new chat? The fix is never "find the old chat" —
  it's "that should have been a document." Write it, commit it, move on.

**When stuck:** Builder circling → `/clear`, re-point at spec · scary error
→ "explain in plain language before fixing" · unsure on a decision → PM
chat · mid-build idea → backlog.md.

---

# PART 8 — The Road Ahead (from the vision doc)

| Sprint | Kills | Focused (~20h/wk) |
|---|---|---|
| 0 — Foundation | — | This document, done ✓ |
| 1 — Spine + Magic Upload | Duplicate data entry | 3–4 weeks |
| 2 — Movement + History + Bank view + add-from-Bank + My Desk v1 | Status bookkeeping | 2 weeks |
| 3 — Composers (Briefs · Packs · Messages) + 7 nudges | Update-typing, CV reformatting | ~2 weeks |
| 4 — Decision Layer (My Desk matured + dashboards) | Asking | 2–2.5 weeks |
| Rollout | Sheets | 2 weeks |
| *Post-Sprint-4 (logged, not scheduled)* | *Read-only MCP server — ask live data questions from Claude* | *~2–3 days, once history exists* |

**≈ 3 months to consultants living in it daily** (≈ 5–5.5 months at ~10 h/wk).
Then one quarter of trusted event history = internal phase exit.

### Four rules that protect this timeline
1. No spec, no code. 2. One session, one objective. 3. You test everything
with your own hands. 4. Commit small, commit often.

---

*Setup is Parts 1–5, once. Work is Part 6, repeated. Memory is Part 7,
maintained. Everything else is already written down. Begin.*