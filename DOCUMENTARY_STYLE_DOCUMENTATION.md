# Documentary-Style Documentation Rules (for Claude Code)

This file is a portable ruleset that makes Claude Code track changes and write narrative records at a "Netflix-documentary" level of depth across all your project work. Drop it into a project root as `CLAUDE.md`, or reference it from an existing `CLAUDE.md` via `@DOCUMENTARY_STYLE_DOCUMENTATION.md`.

Adapted from a multi-project personal setup where these docs feed a downstream AI drafting pipeline. If you don't have a drafting pipeline, the rules still stand on their own. They just make the docs good for any future reader: you in six months, a teammate, an AI assistant rejoining the project.

---

## Documentation as Record-Keeping

**Work under documentary-level scrutiny.** Every session, across every project, should be documented in enough detail that a reader six months later (human or AI) can reconstruct *what* you did, *why* you did it, *what you considered and rejected*, and *what numbers you saw along the way*. If the docs are shallow, anything downstream (a blog post, a handoff, a postmortem, an AI summary) comes out generic. If the docs capture the real reasoning and specifics, the output sounds authentic.

Every project maintains:

- **`CLAUDE.md`**: current-state overview, stack, commands, architecture, key decisions. Updated whenever the current state changes.
- **`CHANGES.md`**: chronological work log, newest entry on top. Append as you go, not retroactively.
- **`docs/narrative/<YYYY-MM-DD>-<topic>.md`** (for any non-trivial migration, rewrite, incident, source onboarding, or multi-phase initiative): the narrative record. Starting state, insight that drove the plan, alternatives considered and rejected (with reasons), phase-by-phase execution, hard numbers, rollback posture. This is where back-and-forth between the user and the agent lives. Keep it after the work lands.

### What "documentary-level" means in practice

Write for a future reader who wasn't in the room:

- **Name specifics.** Port numbers, model names, file paths, commit hashes, row counts, VRAM figures, tok/s, RTT in ms, latency in seconds. Concrete detail beats generic claims.
- **Name the alternatives you rejected.** "We chose Qwen3-Embedding-8B over Qwen3-Embedding-4B because the 4B to 8B jump is one of the largest single-model deltas on MTEB" beats "upgraded the embedder." Record the full decision matrix. A table works well.
- **Name the back-and-forth.** If the user pushed back on an initial plan and you changed it, that's the interesting part. Record what was proposed, what was objected to, and what was agreed. These quality-bump moments are what make downstream summaries sound authentic.
- **Name the numbers you saw.** When you ran the benchmark, what did it print? Don't summarize to "fast." Write "48 tok/s decode, 0.3s TTFT for a 100-token completion on the Qwen3-Next-80B-A3B Q4_K_M build at commit a3f2c8e91". Numbers age well; adjectives don't.
- **Narrative prose, not just bullets.** Voice and context live in prose. Write at least one prose paragraph per non-trivial decision explaining why, before you move on to bullets or tables.
- **Hardware and network context.** Record which machine ran it, what RAM/VRAM config, what network path (local, VPN, etc.), and any BIOS or config setting that mattered.
- **Avoid buzzwords and em-dashes.** Corporate voice reads as generic and gets flattened by anything downstream. Use plain words.

### `CHANGES.md` entry shape

Each entry has: **Date**, **What changed** (specifics: table names, file paths, numbers), **Why** (motivation, not diff. "pgbouncer decommissioned because it was never actually used" beats "removed pgbouncer"), **Design decisions** (tradeoffs, rejected alternatives), **Numbers** (row counts, timing, cost), and a **link to the narrative doc** if one exists. One-line entries are fine for small changes; multi-paragraph entries for anything users or other systems would notice.

Newest entry on top. Append as you go, not retroactively at the end of a session.

### The audit posture

Treat every session as if a future documentary crew will want to interview the agent-plus-user pair about what happened this hour. "We did X" is the story. "We considered A, B, C; rejected A because of Y and B because of Z; chose C; here are the numbers that validated C" is the documentary. Write the documentary version.

---

## Git Commit Standards (Supporting)

Commits support the documentary posture by being *why*-focused rather than *what*-focused. The diff already shows the what.

- Write concise commit messages (1-2 sentences) focused on **why**, not what.
- "pgbouncer decommissioned because it was never actually used" beats "removed pgbouncer".
- Prefer specific file staging (`git add file1 file2`) over `git add -A` or `git add .`.
- Never amend a commit unless explicitly asked. Always create new commits. If a pre-commit hook fails, fix the issue and create a **new** commit (the failed commit didn't happen).
- Push immediately after committing. Don't leave unpushed commits sitting on a branch.

---

## Per-Project Boilerplate

Add this paragraph to any repo's `CLAUDE.md` to reinforce the rule for that specific repo (useful when a repo's `CLAUDE.md` doesn't import this file directly):

> Maintain a `CHANGES.md` file in this repo. Document all significant changes as they are made: new features, bug fixes, refactors, config changes, dependency updates, and removals. Group by date, keep it scannable. This is a historical record of what changed and why. For any non-trivial migration, rewrite, incident, or multi-phase initiative, also create a narrative record at `docs/narrative/<YYYY-MM-DD>-<topic>.md` capturing the starting state, decisions made, alternatives rejected, verification, and final state.

---

## Optional: CLI Workflow Capture

If you also want to capture *meta-level* patterns about how Claude Code itself was used (for a workshop, talk, blog post, or your own learning), maintain a separate top-level file (e.g. `WORKFLOW_LOG.md`) that sits alongside `CHANGES.md`. This indexes the tool patterns, not the work.

**Append an entry whenever any of these happen:**

- A custom slash command is invoked. Note what it did and why.
- A skill is invoked. The skill name and the task it solved.
- Plan mode is used. When, how long, whether the plan got executed, deferred, or abandoned.
- A `.claude/plans/` file is created, updated, or resumed across sessions.
- Multiple Agent tool calls are issued in parallel in a single message (unusual and demoable).
- A specialized subagent type (`Explore`, `Plan`, `statusline-setup`, etc.) is used.
- An MCP server tool is called. Which server, which tool, what it enabled.
- A hook fires (PreToolUse, SessionEnd, etc.) and changes the session.
- A background task is launched with `run_in_background`, especially when the same session continues doing other work.
- The `Monitor` tool is used for event-triggered notifications.
- `ScheduleWakeup` or `CronCreate` is used for deferred or recurring work.
- A deferred tool is loaded via `ToolSearch` (a pattern a lot of users don't know exists).
- The memory system captures something that will compound.
- The `!` prefix is used to run a shell command directly in the prompt stream.
- A layered CLAUDE.md import pattern (`@../CODE.md`) shapes behavior.
- A permission denial happens and how the session adapted.
- A session is resumed with `--continue` or `--resume <session>` and context survives.
- Context compression or session handoff happens. What got preserved, what didn't.
- The agent pushes back on a plan, the user pushes back on an approach, or a quality-bump moment occurs in the dialogue itself. These are the demos that feel human.

**Entry format** (one per pattern occurrence, keep it tight, append newest on top):

```markdown
## YYYY-MM-DD: <Pattern name>

**Context:** one sentence about what project or task this happened in.

**What happened:** concrete steps. Quote commands, tool names, file paths.

**Why it mattered:** what the pattern unlocked. Speed, context savings, correctness, a demoable "look at this" moment.

**Workshop angle:** how this would slot into a talk (lead story, b-roll example, counter-example, etc.).

**Screenshot-worthy?** yes or no. Does a visual of the CLI output or the .claude/ file structure sell it.
```

Do not sanitize, abstract, or generalize. Capture the real transcript: the actual commands typed, the actual file names, the real numbers. Specificity is the whole point. "I use Claude Code with parallel subagents" is a claim; "here's the exact tool call that fanned out Explore plus Explore to audit two repos in 90 seconds instead of 10 minutes of sequential grepping" is content.

---

## How to Install in a New Setup

1. Save this file at your projects root (e.g. `~/projects/DOCUMENTARY_STYLE_DOCUMENTATION.md`).
2. In your top-level `~/projects/CLAUDE.md` (create one if you don't have one. Claude Code reads `CLAUDE.md` files along the path to your working directory), add this line under a "Standards" heading:

   ```markdown
   @DOCUMENTARY_STYLE_DOCUMENTATION.md
   ```

3. In each individual project's `CLAUDE.md`, add the per-project boilerplate paragraph above so each repo has an explicit `CHANGES.md` expectation.
4. Create `CHANGES.md` in each active project with a header and leave it empty. Claude will start appending entries.
5. For the first non-trivial migration, incident, rewrite, or multi-phase task in any project, create `docs/narrative/<YYYY-MM-DD>-<topic>.md` and let Claude narrate the work there as it goes.

The rules take effect on the next Claude Code session in that directory tree.
