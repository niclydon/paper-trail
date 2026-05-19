# paper-trail

Documentation discipline for Claude Code (and other AI coding agents) that captures the reasoning path, not just the final state.

Two files per repo:
- **`CHANGES.md`** at the root: chronological log, newest entry on top. Append as work happens.
- **`docs/narrative/<YYYY-MM-DD>-<topic>.md`**: long-form records for migrations, incidents, rewrites, and other multi-session arcs.

## Why this exists

When an AI agent does most of the typing, the reasoning path is the thing that disappears. The agent has no memory of the alternatives it rejected. You forget too. Six months later something suggests a fix you already turned down and there is no record of why.

Most documentation captures the final state. The why-this-approach-and-not-the-other-three, the verification log, the operator pushback that changed the design: those vanish at session end.

Paper-trail is a portable ruleset that makes Claude Code write at "Netflix-documentary" depth: tradeoffs, false starts, operator decisions, verification numbers, and the exact details that would otherwise be lost.

## Install

1. Copy `DOCUMENTARY_STYLE_DOCUMENTATION.md` into your projects root (e.g. `~/projects/`).
2. In your top-level `CLAUDE.md` (Claude Code reads `CLAUDE.md` files along the path to your working directory), add:

   ```markdown
   @DOCUMENTARY_STYLE_DOCUMENTATION.md
   ```

3. In each project's `CLAUDE.md`, paste the per-project boilerplate from [`templates/per-project-boilerplate.md`](./templates/per-project-boilerplate.md).
4. Create an empty `CHANGES.md` at each project root using [`templates/CHANGES.md`](./templates/CHANGES.md).
5. For the first non-trivial migration, incident, or multi-phase task, create a narrative doc at `docs/narrative/<YYYY-MM-DD>-<topic>.md` using the skeleton in [`templates/docs/narrative/YYYY-MM-DD-example.md`](./templates/docs/narrative/YYYY-MM-DD-example.md).

That's it. Claude will start appending CHANGES.md entries on its next session in that tree.

### Tight on context? Use the lite variants

If you're running on a smaller model, working inside sub-agents, or just want a smaller footprint in your CLAUDE.md chain, swap in the `-LITE` versions:

- `DOCUMENTARY_STYLE_DOCUMENTATION-LITE.md` instead of the full ruleset (~75% smaller, same discipline).
- `templates/per-project-boilerplate-lite.md` instead of the standard one.

The full version is canonical. Lite is the operational summary, kept in sync. Use full when you can afford it; switch to lite when context is tight.

## What good looks like

See [`examples/`](./examples) for a real before-and-after pair:

- [`examples/changes-entry.md`](./examples/changes-entry.md): a real `CHANGES.md` entry. One paragraph plus a six-line code block plus four metadata lines. Names the dormant pipeline that was re-lit, the build number that shipped, the cross-repo dependency.
- [`examples/narrative-doc.md`](./examples/narrative-doc.md): the matching narrative doc for the same event. Titled like an episode ("The Settings Row That Brought Music Back"). Sections include "The Trigger," "The Diff," "What Almost Happened," and "What's Unblocked." Reads like a documentary, not a diff.

The CHANGES.md entry is the chronological index. The narrative doc is the episode.

## The audit posture

> Treat every session as if a future documentary crew will want to interview the agent-plus-user pair about what happened this hour. "We did X" is the story. "We considered A, B, C; rejected A because of Y and B because of Z; chose C; here are the numbers that validated C" is the documentary. Write the documentary version.

Full spec in [`DOCUMENTARY_STYLE_DOCUMENTATION.md`](./DOCUMENTARY_STYLE_DOCUMENTATION.md).

## License

MIT. See [LICENSE](./LICENSE).
