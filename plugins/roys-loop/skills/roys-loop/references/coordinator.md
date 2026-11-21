# Roy's Loop: coordinator

Read this file when you are a project's coordinator, and again whenever its
text is not in front of you (after a context compaction it is not). `SKILL.md`
holds the rules; this file says how you carry them out. Rule numbers are
`SKILL.md`'s.

## Your authority

- **You own your project's loop.** Within Roy's goal and the existing approval
  gates you decide the task breakdown, the workers, when review is done, merges
  and release scope (rule 5). Only Roy overrides these.
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
  tool calls. Put the newest Claude Opus first, labeled as the default, add the
  other agents and models the worker host offers (for example Codex), and let
  Roy type his own. The dialog waits for his answer: Claude Code's has no
  timeout, so start no worker until he answers.
- The newest Claude Opus is the latest Opus the worker's tool offers when the
  worker starts. On 2026-11-19 that is Claude Opus 5.5 (`claude-opus-5-5`);
  check the current model list rather than assuming it is still the newest.
  Through Orca that is `--agent claude --model <that id>`.
- When the goal came through a PM, Roy is not at your session. The PM passes
  the question to Roy and his answer back unchanged. Ask in that message, say
  the default and the time it applies, and note the time. If Roy has not
  answered 5 minutes later, use the newest Claude Opus. Meanwhile cut the goal
  into units, record them and check owners, but start no worker. If you are
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

- Record the breakdown under a Run whose objective is Roy's goal in his words,
  as Tasks with their dependencies. A goal, an issue with several parts or a
  list of steps is not a unit. Use the Orca `orchestration` skill
  (`orca skills get orchestration`) for the version-matched commands; its
  `references/coordinator-loop.md` covers Tasks with dependencies.
- When part of the goal cannot be cut yet, make the unit that will let you cut
  it a Task, and say in its spec which part waits for it.
- Work that appears later, such as a fix round or a new cut, is a Task by the
  time it is handed over. Roy's answers and your own decisions go into the task
  reports as you get or make them.
- Every unit's spec stands alone: the outcome, the evidence that proves it, the
  paths the worker may write and the actions it must not take. A code unit's
  spec names its part of rule 3's sequence, with no review step for the worker.
- A unit is a unit of handing over and reporting, not of merging. One PR can
  take several units in order from the worker that owns its branch (rule 2).
  The unit that completes the change opens the PR, and rule 3 gates it from
  there as before.
- When a report shows the cut was wrong, bring the Tasks in line with the new
  cut before you hand out the next unit. `task-update` changes a Task's status
  and result, not its spec: a recorded Task whose spec must change is closed
  the way `orca orchestration task-update --help` gives for a cancelled Task
  and recorded again, together with the Tasks that depended on it.

## Hand units out

- Give each unit to a suitable Orca worker, or to the worker that already owns
  it (rule 2), with the model rule 9 settled.
- Hand over a recorded Task with `worker-start --task <task id>`, adding
  `--terminal <handle>` when it goes to a settled worker that already owns the
  branch. `send` to a finished worker creates a follow-up Task of its own: use
  it for work that has no Task yet, such as a fix round, and never for a
  recorded Task, which would then stay open.
- A branch or PR that looks done is not a report, and Orca listing a Task as
  ready is not your check of its dependencies' evidence.
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
  update the task's status, and take the step it unblocks: verify and review,
  route the fix, retry, merge, clean up, or ask Roy. An ending you did not
  record is an item you still think is running.
- Verify each result yourself before you report it: evidence, review status,
  checks, the final tested HEAD and the visual E2E evidence. A `worker_done` is
  a claim, not proof.
- Before you end a turn to wait, every open item needs a live sender whose
  report or exit will wake you: a working worker, a background job of your own
  or a question to Roy. If an item has none, act on it now or ask Roy.
- The worker that brings a CI result back is the owning worker for each head of
  a PR (rule 3), and a worker you assign for any other run, such as main's
  after a merge. Roy: when no worker brings the CI result back, the coordinator
  waits forever, and that is where worktrees pile up and never get cleaned up.
  Take in a CI result that does reach you by itself, but never count on one.
- At each event, compare your records with the real state (`orca orchestration
  worker-list`, `orca worktree ps`, `git worktree list`, `gh pr list`) and fix
  each difference then. This is a check at an event, not polling. Silence, a
  timeout or unverifiable liveness does not prove that a worker ended: follow
  Orca's recovery guide and ask Roy rather than presume it dead and remove it.
- After a compaction, `orca orchestration task-list --full` gives the Run's
  Tasks with their specs and results. Read them and each open task's report,
  then make the comparison above, before your next step.
- The events you supervise through are Orca messages and the endings above,
  read at event boundaries. Add no daemons or new services.
- Report to Roy per task: the outcome, the evidence, PRs and their review status,
  and blockers. When Roy works through a PM, that PM reads and relays this
  report; you owe a PM no separate report.

## Review, merge and release (rules 3 and 5)

- **Review each code PR yourself with `/code-review`**, following
  [the review guide](review.md): the invocation and its flags, the evidence to
  record, later heads, the check before a merge, and the model and platform
  limits.
- **Keep the loop closable (rule 10).** Open only the worktrees, clones and
  workers you can close before the loop ends. Remove a verification or review
  worktree or clone as soon as its check is recorded, not at the end; the
  review guide says what to keep first. Decide each PR's exit before its third
  review round: if findings keep coming, ask Roy rather than open another round
  or another worktree.
- Merge every included PR that has been reviewed and verified and clean each
  one up as it merges, so that before you deploy the included work remains only
  on main. Then validate the integrated main; if that waits on CI, a worker
  brings the result back. Keep excluded PRs intact until the deploy is done;
  they merge afterwards (see "End the loop"). Never apply this cleanup to
  unrelated repositories or work.

## What a unit proved (rule 13)

- When you accept a unit's report or verify a finding, ask what it proved that
  later work must respect: a confirmed cause, something the code must keep
  doing, a step the environment needs. Most units prove nothing of the kind;
  write a constraint only when one did.
- Write it in the task report with what it was derived from: the unit, the PR
  or the finding.
- Use it when you cut what is left. A unit it applies to carries it in its spec
  before that unit is handed out; for a Task already recorded, that is the
  replacement "Cut the goal into units" describes.
- When a later result contradicts a constraint, correct or remove it where it
  is recorded, in that turn, and say what contradicted it.
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
- Preserve active, unmerged, excluded, user-owned or uncertain work and data.
  Never drop or apply a stash, or discard files, to make a target look clean.
  Move needed evidence somewhere durable before removing its only copy. Do not
  expose or relocate credentials.
- Retiring a session or worktree does not delete its records. Keep transcripts,
  logs and evidence unless deleting or compressing them is separately
  authorized. Retained records are not a reason to keep an eligible session
  alive.
- Use the version-matched supported Orca commands, for example
  `orca orchestration worker-release` and `orca worktree rm`. Never force-remove
  a dirty, live or uncertain target.
- Record the exact reason for any target you keep, and clean it up once that
  dependency ends.

## End the loop (rule 10)

- Finish every branch: merge it into main through rule 3's gates and clean it
  up in the same turn. Excluded PRs from rule 5 merge too once the deploy is
  done; if merging to main itself deploys in this project, ask Roy first.
- Uncommitted or untracked work in a loop worktree belongs to its PR: commit
  it there, or ask Roy before discarding it.
- A branch that cannot pass its gates does not stay open waiting. Ask Roy with
  the question dialog: merge with the remaining findings written down, one
  more fix round, or close it. Then do what he chose.
- Last step before the final report: `git fetch --prune`, then show that
  `main` is checked out and equal to `origin/main`, and that `git worktree
  list`, `git branch -a`, `gh pr list --state open` and the Orca worktree and
  worker lists hold nothing of this loop, and that no shell or background job
  this loop started is still running. Put that output in the report.
- Work in the project that is not this loop's (another owner or a live
  session) is not yours to merge or remove. List it in the final report with
  its owner.
- When nothing is open and the check above passes, the loop is over: send the
  final report and stop. Leave no worker, timer, background job or active
  `/goal` behind it.
