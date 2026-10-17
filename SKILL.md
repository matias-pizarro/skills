---
name: review-loop
description: Loop until Codex has no findings. Each turn commits pending work, sends it to a Codex review, commits the review, grills the user through each finding, records and commits the response.
disable-model-invocation: true
---

# Review loop

The loop that has shaped this repo's design: commit what is pending, ask Codex for a second-model review of one topic's artefacts, commit the review, walk the user through every finding with the grilling discipline, record what was decided beside the review, commit that, and go again until a review comes back with no findings.

A **turn** is one pass through the three phases, **submit**, **discuss** and **close**. Phases follow each other without pausing; the only waits are the per-finding questions in discuss.

## Invocation

- `/review-loop`: run turns until a review has no findings, at most five.
- `/review-loop <n>`: the same, with `<n>` turns as the cap. `once` is the spelling for one turn.
- `/review-loop [<n>] auto`: unattended discuss; see the discuss phase for what auto does and where it still waits.
- `/review-loop submit`, `/review-loop discuss`, `/review-loop close`: one phase, then stop.

A turn whose review has any finding, low included, is followed by another turn; a review with none ends the loop. When the cap is reached with findings still open, say the topic has not converged and stop; the next invocation continues from the last close.

## Vocabulary

- **Topic**: a directory under `.scratch/` holding one set of artefacts (`sandbox-cli`, `sandbox-impl`, `live-test`). Named by the user, or inferred from the artefacts touched since the last review; ask when two topics are touched.
- **Review file**: `.scratch/<topic>/review-codex-<YYYYmmddHHMM>-<hash>[-<suffix>]-findings.md`, written by Codex. The suffix names the artefact kind when a topic has more than one (`spec`, `tickets`); design docs carry none.
- **Response file**: the review file's path with `-response` in place of `-findings`, written in discuss. The prior response is the newest `*-response.md` in the topic. The two words are chosen so the review sorts before its response by filename, which is the order the docs site lists them in; topics reviewed before this convention keep their older `…[-<suffix>].md` review names.
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
   The prompt always ends with the three fixed sentences: say so plainly if nothing is above low severity; do not edit files; produce a markdown report ordered by severity, citing file and line as a markdown link relative to the review file, with the line in the link text, as in `[spec.md:35](spec.md)`, never as an absolute path or with `:line` in the target, since the docs site resolves neither.
4. Write the prompt to a temporary file and run the review as a subagent with [`run-review`](run-review), from the repo root:
   ```
   .agents/skills/review-loop/run-review <review file> <prompt file>
   ```
   With `OPENAI_API_KEY` set the reviewer is a Pi subagent on an OpenAI model (`REVIEW_MODEL`, default `openai/gpt-5.3-codex`); without it, `codex exec --sandbox read-only`. Inside tmux it runs in its own pane, where its output streams, and the script waits for it; outside tmux it streams in place. Run it with a tool timeout of at least thirty-two minutes, since the script enforces thirty itself. A non-zero exit (124 for the time limit) means no review: report the exit status and stop the turn.
5. Commit the review file alone: `docs: record codex <topic> review`. If the docs build hook refuses it for a link, correct only the link's target, keeping its text and moving a line number from the target into the text, and commit again.
6. Completion: the review file exists, is committed, and has been read; report its finding count and severities in one line.

## discuss

Load the `grilling` and `domain-modeling` skills. Then, for each finding in severity order:

1. State in one or two sentences whether the finding is right, and the fact that settles it, looking things up rather than asking.
2. Put exactly one question in the grilling format, with a recommended answer the user can accept with "yes". When two findings share one root cause, one question may cover both and must say so.
3. Wait. Edit nothing before the answer. In `auto` mode, do not wait: take the recommended answer as the answer and say so in the response entry. The one exception is a recommended answer that is a deliberate choice; auto mode stops at that finding and waits, because a recorded acceptance of a risk or cost carries the user's name and an edit does not.
4. Apply the answer to every file it touches, then append the finding's entry to the response file: its number and title, what was decided, and where it landed. A deliberate choice is also written into the ticket or ADR it concerns, marked as such.
5. A finding that is a transcription error against a decision already made is applied without a question and reported as such.

After the last finding: run a grep for wording the round made stale, fix what it finds, end the response file with a one-line agreement or disagreement with the review's disposition table, and summarise in the form the user has seen all session: one bullet per finding, nothing committed, the next phase named. Print it and continue into close without waiting.

Completion: every finding has a response entry, every edit the entries name exists in the tree, and the stale-wording grep is clean.

## close

1. Commit everything the discuss phase touched, response file included: `docs: respond to the Codex <topic> review of <hash>`, body one line per finding.
2. If the review just closed had no findings, say the topic has converged and stop. Otherwise, if the turn cap is not reached, print the commit line and start the next turn's submit without waiting. If it is reached, or the invocation was a single phase or `once`, print the next `/review-loop` command for this topic, ready to paste, and stop.

## Guardrails

- Review and response files are committed in separate commits, never together with design edits, so `git log` shows what Codex said and what was decided as two events.
- The prompt names files, never pastes them; Codex reads the tree.
- A finding is never resolved by rewording alone; if the honest resolution is to accept the risk, it is written as a deliberate choice where the next reader will look.
