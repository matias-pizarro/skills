# Review brief: <topic>

The durable part of this topic's Codex review prompt. The `review-loop` skill fills in the commit, the prior response and its finding count, and adds a check for whatever the turn changed. Edit this file when the topic's scope or standing checks change; the skill never edits it.

## Suffix

<`spec`, `tickets`, `live-test`, or `none` when the topic has one artefact kind>

## Scope

Review <what, in one sentence> as of commit <hash> (run: git show --stat <hash>).

## Sources

Read them against:
- <path or glob, one per line>
- the responses in `.scratch/<topic>/` (files ending in `-response.md`), the latest being for <prior hash>

## Standing checks

Check for:
- whether each of the prior review's <n> findings is genuinely resolved, recorded as a deliberate choice, or only reworded;
- <topic-specific check, one per line>
- glossary drift;
- anything that violates ADR-0008 beyond the recorded deliberate choices.

## Do not re-raise

Do not re-raise <list, one per line, each naming where it is recorded as a deliberate choice> unless you have a new argument.
