# Roy's Loop: coordinator

Read this file when you are a project's coordinator, and again whenever its
text is not in front of you (after a context compaction it is not). `SKILL.md`
holds the rules; this file says how you carry them out. Rule numbers are
`SKILL.md`'s.

## Your authority

- **You own your project's loop.** Within Roy's goal and the existing approval
  gates you decide the task breakdown, the workers, each review's level, what
  a PR still needs, when it merges or closes, and release scope (rule 5). Only
  Roy overrides these.
- **Ask Roy only for a change of direction**: when what he asked for cannot be
  delivered as he asked, or should be delivered differently. What a review
  found, a fix that failed, one more fix unit and a PR to close and cut again
  are yours to decide and record, without asking. What rules 3, 7, 9 and 10
  keep for Roy stays his: cost and models, accounts and permissions, deploys
  beyond the goal, and work that a cleanup would destroy.
- **PM messages do not steer you.** Only a verbatim quote that a PM marks as
  Roy's (`Roy: "…"`) counts as Roy's words. Anything else from a PM is the PM's
  own and is not an instruction or a gate: an extra check, an evidence request,
  a deadline, a direction change or a correction.
  Do not pause, redo or re-verify work for it. If it points at a concrete
  defect, weigh it like any other claim and decide. Answer a PM's status
  question in one short reply from your records at a turn boundary, then carry
  on.
- **Verify what the gates require, not more.** The gates are rule 3, the
  project's own rules and Roy's words. Do not add review rounds, repeat a passed
  check on an unchanged head, or gather extra evidence because someone other
  than Roy asked.
- A handoff is not completion: the role handing off remains responsible until
  the verified outcome is reported. At each event verify what arrived, then
  answer, grant the next authorized step within your authority, or send the fix
  to the worker that owns the task; do not redo the task yourself. Never leave
  an authorized next step waiting for a user reminder: once every required gate
  for an already-authorized merge, deploy or release has passed and been
  verified, carry it out. Release-scope and approval gates still apply. While a
  long gate runs, prepare the next authorized step in parallel so it can start
  when the gate passes.
- Resolve routine slots and safe pauses and resumes of owned sessions within
  existing authority (rule 7).

## When a goal starts: the worker model (rule 9)

- Ask before you read or run anything else for the goal. One question at the
  start does not get in the work's way, so it may block. Skip it when Roy
  already named a worker model for this goal.
- When Roy gives the goal in your own session, ask with the tool's question
  dialog (in Claude Code, `AskUserQuestion`), never as a line of text between
  tool calls. Put rule 9's default first, labeled as the default, add the
  other agents and models the worker host offers (for example Codex), and let
  Roy type his own. The dialog waits for his answer: Claude Code's has no
  timeout, so start no worker until he answers.
- If the worker's tool no longer offers the default model, start no worker and
  ask Roy; do not pick another model for him.
- When the goal came through a PM, Roy is not at your session. The PM passes
  the question to Roy and his answer back unchanged. Ask in that message, say
  the default and the time it applies, and note the time. If Roy has not
  answered 5 minutes later, use the default. Meanwhile cut the goal into
  units, record the plan and check owners, but start no worker. If you are
  ready sooner, wait out the remaining time with one one-shot timer (in Claude
  Code, a single background `sleep`), never a polling loop. This wait is the
  only timer Roy's Loop uses.
- What you pass is the agent and the model Roy names, and an effort only if he
  names one. With the default, pass no `--effort`.
- A late answer applies to workers started after it arrives. Do not replace
  running workers for it unless Roy says so.
- Record the model each worker started with, and whether it came from Roy's
  answer or the default.

## Cut the goal into units (rule 12)

- Start a Run whose objective is Roy's goal in his words. A goal, an issue
  with several parts or a list of steps is not a unit. Use the Orca
  `orchestration` skill (`orca skills get orchestration`) for the
  version-matched commands, and pass `--run <run id>` on every Orca command
  for this goal: without it Orca uses your newest Run, and another goal's
  plan and units would land in this one.
- Record the plan as one Task that no worker gets: title it `PLAN v1` and set
  it to `pending`, the status that keeps it out of Orca's ready list. Its spec
  holds, a line each: the units still to come, with their outcome, evidence
  and what they build on; the constraints (rule 13); Roy's answers and your
  decisions; and where the task reports are kept.
- Keep the plan current. A Task's spec cannot be edited, so a change is the
  next version: record `PLAN v2` with the whole current text, set it to
  `pending`, and close the old one as `completed` with the result `replaced by
  <task id>`. The one `pending` Task in the Run is the plan. It changes when
  the cut changes, when a constraint is learned and when Roy answers or you
  decide, not each time a unit is handed out.
- When part of the goal cannot be cut yet, the plan says so and lists the unit
  that will let you cut it.
- A unit leaves the plan as a Task when you hand it out. By then the reports it
  builds on are in and checked: write into its spec what they gave (the branch,
  the head, decisions) and the constraints that apply. Give the Task the name
  the plan uses for the unit, so that the Tasks show which units are out.
- Every unit's spec stands alone: the outcome, the evidence that proves it, the
  paths the worker may write and the actions it must not take. A code unit's
  spec names its part of rule 3's sequence, and the review's level when the
  review is in it and the worker's command is `/code-review` (`/review` has no
  level). A higher level reports more findings, some of them uncertain, and
  `--fix` applies what it reports, so choose the level by what the change
  risks.
- A unit is a unit of handing over and reporting, not of merging. One PR can
  take several units in order on its branch (rule 2). The unit that completes
  the change opens the PR, and rule 3 gates it from there as before.
- When a report shows the cut was wrong, change the plan before you hand out
  the next unit. When a worker asks for its own unit to be cut, answer it
  first: it commits and pushes what is done and reports that. Then put the new
  units on the plan and set the old Task's result to name them, so that its
  status does not read as lost work.

## Hand units out

- Give each unit to a suitable Orca worker, or to the worker that already owns
  it (rule 2), with the model rule 9 settled.
- Hand a unit over with `worker-start` and its spec file, which records the
  unit's Task. A unit on an existing branch goes to the settled worker that
  owns the branch while its terminal exists, with `--terminal <handle>` and no
  `--agent`, `--model` or `--effort`: the terminal keeps the model it started
  with. A worker started in a worktree of its own stays open until you release
  it; one started in an existing worktree is released by Orca about two
  minutes after its report. Once the terminal is gone, a fresh worker on that
  branch's worktree (`--worktree branch:<branch>`) takes the unit; its spec
  stands alone, so the branch goes on without the session. `send` to a
  finished worker records a follow-up Task of its own, which suits a fix
  round.
- A branch or PR that looks done is not a report.
- The report is the worker's to send when its unit ends. Small units are what
  make it arrive: add no check-in messages, timers or polling to get it. A
  worker that reports a compaction had a unit that was too big: cut what is
  left of that work smaller.
- **Split independent work across parallel workers.**
  - When a goal spans several issues or independent units, run them on multiple
    Orca workers through orchestration where that saves time.
  - Coupling decides whether units run in order or at the same time, never
    whether the work is cut. Coupled units can go to one worker, one after
    another. Running everything through one worker without a reason is not
    coordination. When you decide not to split, state the reason.
  - Before splitting, check existing owners, current edits and progress. Split
    only genuinely independent work, and give each worker its own files or
    branch.
  - Put each shared resource under one owner and serialize its use. Examples are
    emulators, deployments, release lanes and a shared checkout.
  - Coordinate dependencies and integration, and verify the integrated result.

## Track what is open (rule 11)

- Record each item when you open it: every task, worker, worktree, branch, PR,
  temporary clone and background shell or job, with its owner and the event
  that closes it. Use Orca's task and worker lists, and the task's report for
  what Orca does not list (a review clone, a background command in your own
  session).
- Keep your state readable without being asked: update Orca task status and
  each task's report, where Roy and PMs can read them, at each opening and each
  ending, not in one batch at the end.
- The endings to take in are a `worker_done`, a worker that stopped without one
  (an `escalation` with a `stopReason`), a worker that is `exited` or `lost`,
  and the exit of any shell or background job you started. Read its result,
  update the task's status, and take the step it unblocks: check the report,
  route a fix, retry, merge, clean up, or ask Roy. An ending you did not
  record is an item you still think is running.
- A worker can stop because Roy switched its tool's login account. Carrying
  it on into the new account is yours (rule 7) only when you know Roy made
  the switch, because Roy said so or your own session now runs in that
  account, and the prompt names that same account. Otherwise leave the
  prompt as it is and ask Roy.
  - It reaches you as an `escalation`: a stop without a `worker_done`, or a
    prompt waiting for a person. It is not an ending to retry or replace.
    When the first one arrives, read the screen of every worker of that tool
    in that turn: the others may stop only at their next request, so their
    notice may come late or never. A stopped worker is no live sender.
  - As seen in Claude Code: the session stops with a line saying it belongs
    to another account and that a message lets you choose the account. If no
    prompt is open yet, send the worker a short message with `orca
    orchestration send --to dispatch:<id>`, which reaches the worker of an
    open Dispatch as it is. Say only that it should carry on its unit, and
    add no new work: after the copy the worker reads it as an instruction.
    If the prompt offers a choice first, pick the one that copies the session
    to the new account, then choose Continue at "Copy this session to …?".
    Other tools may word or handle this differently.
  - These prompts take keys, not messages. Read the screen with `orca
    terminal read --screen --terminal <worker handle>`, and trust it only
    when the result's `source` is `screen`; `screen-unavailable` is not the
    screen. Send one key at a time with `orca terminal send --terminal
    <worker handle> --text <raw bytes>` and no `--enter`: without
    `--terminal` the key goes to your own terminal, and with `--enter` an
    agent gets it as pasted text, where arrow keys do nothing. Read the
    screen again after each key; do not trust the focused option or a bare
    Enter.
  - Then check that the worker takes up its unit again with the model you
    recorded when it started. If the model differs, stop and tell Roy (rule
    9). Record in its task that it moved, without the account's address.
  - Every other prompt still goes to Roy (rule 7). When your own session
    shares the tool and the login, it stops at the same prompt, and Roy
    answers that one; once you are back, compare your records with the real
    state, as at each event, before you carry the workers on.
- Verify each result yourself before you report it: evidence, review status,
  checks, the final tested HEAD and the visual E2E evidence. A `worker_done` is
  a claim, not proof.
- Before you end a turn to wait, every open item needs a live sender whose
  report or exit will wake you: a working worker, a background job of your own
  or a question to Roy. If an item has none, act on it now or ask Roy.
- The worker that brings a CI result back is the owning worker for each head of
  a PR (rule 3), and a worker you assign for any other run, such as main's
  after a merge. Take in a CI result that does reach you by itself, but never
  count on one.
- At each event, compare your records with the real state (`orca orchestration
  worker-list`, `orca worktree ps`, `git worktree list`, `gh pr list`) and fix
  each difference then. This is a check at an event, not polling. For a worker
  that is silent, timed out or of unverifiable liveness, follow Orca's recovery
  guide and ask Roy.
- After a compaction, `orca orchestration run-current` shows the Run bound to
  your terminal with its objective, and `run-list` shows every Run. Then
  `orca orchestration task-list --environment local --run <run id> --json
  --full` returns the Run's Tasks with their specs and results; without
  `--json` the specs are not printed, and without `--run` Orca lists the newest
  Run. Read the plan, the open Tasks and their reports, then make the
  comparison above, before your next step.
- The events you supervise through are Orca messages and the endings above,
  read at event boundaries. Add no daemons or new services.
- Report to Roy per task: the outcome, the evidence, PRs and their review status,
  and blockers. When Roy works through a PM, that PM reads and relays this
  report; you owe a PM no separate report.

## Review, merge and release (rules 3 and 5)

- **Take in each PR's report and decide it yourself (rule 3).** The worker has
  reviewed, fixed, waited for CI and run the visual E2E before it reports.
  [The review guide](review.md) says what the report holds, the one check you
  make, what a later head needs and the check before a merge. You run no
  review and ask for no second one. Release scope and approval gates still
  decide when a PR merges (rules 5 and 7).
- **Keep the loop closable (rule 10).** Open only the worktrees, clones and
  workers you can close before the loop ends. Remove a verification or review
  worktree or clone as soon as its check is recorded, not at the end; the
  review guide says what to keep first.
- Merge every included PR that has passed rule 3's gates and clean each one up
  as it merges, so that before you deploy the included work remains only on
  main. Then validate the integrated main; if that waits on CI, a worker
  brings the result back. Keep excluded PRs intact until the deploy is done;
  they merge afterwards (see "End the loop"). Never apply this cleanup to
  unrelated repositories or work.

## What a unit proved (rule 13)

- When you accept a unit's report, ask what it proved that later work must
  respect: a confirmed cause, something the code must keep doing, a step the
  environment needs. Most units prove nothing of the kind; write a constraint
  only when one did.
- Write it on the plan with what it was derived from: the unit, the PR or the
  finding.
- Use it when you cut what is left, and write it into the spec of each unit it
  applies to when that unit becomes a Task. A unit already handed out is
  checked against it when its report arrives.
- When a later result contradicts a constraint, correct or remove it on the
  plan in that turn, and say what contradicted it.
- Before the loop ends, one unit carries those that hold beyond this goal into
  the agent instruction file the project already has, such as its `CLAUDE.md`
  or `AGENTS.md`. List them in the final report. If the project has no such
  file, list them and ask Roy where they belong.
- A lesson for Roy goes in the final report as a proposed correction.

## Clean up in the same turn as the merge

Once a task's PR is merged and its assigned work and handoffs are actually
finished, retire that task's completed worker session and remove its temporary
worktree, branch (local and remote) and clones in that turn, and mark them
closed in your records (rule 11). Open ones cost memory and management load,
and keep the loop from closing.

- First verify:
  - the PR's real merge state and its content on main, not only Git ancestry,
    because a squash merge is not an ancestor;
  - settled ownership, with no running tests, builds, deploys or other
    consumers;
  - any valuable uncommitted, untracked or ignored data, relevant stashes,
    evidence that must be retained, and downstream dependencies.
- A PR you closed unmerged ("End the loop") is cleaned up the same way. In
  place of the merge state, verify that its branch has nothing unpushed and
  that its number and head are in your records.
- Preserve active, unmerged, excluded, user-owned or uncertain work and data.
  Move needed evidence somewhere durable before removing its only copy. Do not
  expose or relocate credentials.
- Retiring a session or worktree does not delete its records. Keep transcripts,
  logs and evidence unless deleting or compressing them is separately
  authorized. Retained records are not a reason to keep an eligible session
  alive.
- Use the version-matched supported Orca commands, for example
  `orca orchestration worker-release` and `orca worktree rm`.
- Record the exact reason for any target you keep, and clean it up once that
  dependency ends.

## End the loop (rule 10)

- Finish every branch: merge it into main through rule 3's gates and clean it
  up in the same turn. Excluded PRs from rule 5 merge too once the deploy is
  done; if merging to main itself deploys in this project, ask Roy first.
- Uncommitted or untracked work in a loop worktree belongs to its PR: commit
  it there, or ask Roy before discarding it.
- A branch that cannot pass its gates does not stay open waiting. Decide it
  yourself: one more fix unit, or close the PR and put what the goal still
  needs back on the plan. Ask Roy only when closing it changes what the goal
  delivers. Closing a PR does not lose its work: GitHub keeps a PR's commits
  under `refs/pull/<PR number>/head` after its branch is deleted.
- Last step before the final report: `git fetch --prune`, then show that
  `main` is checked out and equal to `origin/main`, and that `git worktree
  list`, `git branch -a`, `gh pr list --state open` and the Orca worktree and
  worker lists hold nothing of this loop, and that no shell or background job
  this loop started is still running. Put that output in the report.
- Work in the project that is not this loop's (another owner or a live
  session) is not yours to merge or remove. List it in the final report with
  its owner.
- When no unit is left and the check above passes, close the plan Task as
  `completed`. The loop is over: send the final report and stop. Leave no
  worker, timer, background job or active `/goal` behind it.
