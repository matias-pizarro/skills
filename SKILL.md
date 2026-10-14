---
name: review-loop
description: Commit, send the work to a Codex review, commit the review, grill the user through each finding, record the response.
disable-model-invocation: true
---

# Review loop

One turn of the loop that has shaped this repo's design: commit what is pending, ask Codex for a second-model review of one topic's artefacts, commit the review, walk the user through every finding with the grilling discipline, and record what was decided beside the review. Each turn ends by printing the next turn's command, so the loop is one keystroke to continue.

The loop has three phases. `/review-loop` runs **submit** then **discuss**. `/review-loop submit`, `/review-loop discuss` and `/review-loop close` run one phase.

## Vocabulary

- **Topic**: a directory under `.scratch/` holding one set of artefacts (`sandbox-cli`, `sandbox-impl`, `live-test`). Named by the user, or inferred from the artefacts touched since the last review; ask when two topics are touched.
- **Review file**: `.scratch/<topic>/review-codex-<YYYYmmddHHMM>-<hash>[-<suffix>].md`, written by Codex. The suffix names the artefact kind when a topic has more than one (`spec`, `tickets`); design docs carry none.
- **Response file**: the review file's path with `-response` before `.md`, written in discuss. The prior response is the newest `*-response.md` in the topic.
- **Topic brief**: `.scratch/<topic>/review-brief.md`, the durable part of the review prompt. Created from [`brief-template.md`](brief-template.md) on a topic's first run.
- **Finding**: one numbered item in a review. **Deliberate choice**: a finding the user resolves by accepting the risk or cost; it is recorded in the ticket or ADR it concerns, not only in the response.

## submit

1. If the tree is dirty, commit it. Read the diff and write a one-line conventional subject with a short body; never commit a review or response file in this step.
2. Identify the topic and suffix. If the topic brief is missing, create it from the template, filling the scope, sources and checks from the artefacts you can see, and show it to the user before continuing.
3. Build the prompt from the brief, adapting it to this turn:
   - the commit hash and `git show --stat <hash>` line;
   - the prior response file by path and its finding count, read from the file, with the disposition-table instruction naming that count; on a first run, omit the disposition table;
   - the brief's do-not-re-raise list plus any deliberate choice recorded in the prior response;
   - a check for whatever this turn changed that the brief's standing checks do not name.
   The prompt always ends with the three fixed sentences: say so plainly if nothing is above low severity; do not edit files; produce a markdown report ordered by severity, citing file and line.
4. Run it, streaming output, with a timeout of thirty minutes:
   ```
   codex exec --sandbox read-only -o <review file> "<prompt>"
   ```
5. Commit the review file alone: `docs: record codex <topic> review`.
6. Completion: the review file exists, is committed, and has been read; report its finding count and severities in one line.

## discuss

Load the `grilling` and `domain-modeling` skills. Then, for each finding in severity order:

1. State in one or two sentences whether the finding is right, and the fact that settles it, looking things up rather than asking.
2. Put exactly one question in the grilling format, with a recommended answer the user can accept with "yes". When two findings share one root cause, one question may cover both and must say so.
3. Wait. Edit nothing before the answer.
4. Apply the answer to every file it touches, then append the finding's entry to the response file: its number and title, what was decided, and where it landed. A deliberate choice is also written into the ticket or ADR it concerns, marked as such.
5. A finding that is a transcription error against a decision already made is applied without a question and reported as such.

After the last finding: run a grep for wording the round made stale, fix what it finds, end the response file with a one-line agreement or disagreement with the review's disposition table, and summarise in the form the user has seen all session: one bullet per finding, nothing committed, the next phase named.

Completion: every finding has a response entry, every edit the entries name exists in the tree, and the stale-wording grep is clean.

## close

1. Commit everything the discuss phase touched, response file included: `docs: respond to the Codex <topic> review of <hash>`, body one line per finding.
2. Print the next `submit` command for this topic, ready to paste, unless the review just closed had no findings, in which case say the topic has converged and stop.

## Guardrails

- Review and response files are committed in separate commits, never together with design edits, so `git log` shows what Codex said and what was decided as two events.
- The prompt names files, never pastes them; Codex reads the tree.
- A finding is never resolved by rewording alone; if the honest resolution is to accept the risk, it is written as a deliberate choice where the next reader will look.
