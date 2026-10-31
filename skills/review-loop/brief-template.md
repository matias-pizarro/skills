# Review brief: <topic>

The durable part of this topic's Codex review prompt. The `review-loop` skill fills in the commit, the prior response and its finding count, and adds a check for whatever the turn changed. Edit this file when the topic's scope or standing checks change; the skill never edits it.

## Suffix

<`spec`, `tickets`, or `none` when the topic has one artefact kind>

## Scope

Review <what, in one sentence> as of commit <hash> (run: git show --stat <hash>).

## Sources

Read them against:
- <path or glob, one per line>
- the project's `AGENTS.md` and its recorded decisions, such as ADRs, where it has them
- the responses in `.scratch/<topic>/reviews/` (files ending in `-response.md`), the latest being for <prior hash> (left out until a topic has a prior review)

## Standing checks

Check for:
- whether each of the prior review's <n> findings is genuinely resolved, recorded as a deliberate choice, or only reworded (left out until a topic has a prior review);
- <topic-specific check, one per line>
- anything that contradicts `AGENTS.md` or a recorded decision beyond the recorded deliberate choices.

## Citations

Cite every file as a markdown link relative to the review file, which sits in `.scratch/<topic>/reviews/`, with the line in the link text: `[spec.md:35](../spec.md)`. Never an absolute path, and no `:line` in the target; rendered markdown resolves neither. <Any citation rule in the project's `AGENTS.md`, such as files to cite as plain text.>

## Do not re-raise

Do not re-raise <list, one per line, each naming where it is recorded as a deliberate choice> unless you have a new argument.
