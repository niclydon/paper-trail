# Documentary-Style Documentation Rules (lite)

A condensed version of `DOCUMENTARY_STYLE_DOCUMENTATION.md` for context-constrained sessions. Same standard, fewer tokens. The full doc is canonical; use it when context allows.

## Two files per repo

**`CHANGES.md`** at root, newest entry on top. Append as work happens, not retroactively. Each entry: date, what changed (specifics: file paths, names, numbers), why (motivation, not diff), what was decided, what was rejected and why, verification numbers, what's still open. Link to a narrative doc if one exists.

**`docs/narrative/<YYYY-MM-DD>-<topic>.md`** for migrations, incidents, rewrites, source onboarding, multi-phase work. Sections: trigger, starting state, alternatives considered and rejected, the approach, verification with real numbers, "what almost happened" (reversals, mistakes caught), what's now unblocked.

## The standard

Write so a future reader (you in 6 months, a teammate, an AI rejoining the project) can reconstruct *what* was done, *why*, *what was considered and rejected*, and *what numbers validated it*.

- Name specifics: file paths, commit hashes, model names, port numbers, row counts, tok/s, RTT, latency.
- Name alternatives rejected and why.
- Name the back-and-forth: pushback, what was proposed, what was agreed.
- Name the numbers verbatim. Not "fast" but "48 tok/s decode, 0.3s TTFT".
- Narrative prose for non-trivial decisions, not just bullets.
- No buzzwords. No em-dashes.

## Commits

1-2 sentence messages focused on **why**, not what. Specific file staging (`git add file1 file2`). Never amend; create new commits. Push immediately.

## The test

"We did X" is the story. "We considered A, B, C; rejected A because Y; rejected B because Z; chose C; here are the numbers that validated C" is the documentary. Write the documentary version.
