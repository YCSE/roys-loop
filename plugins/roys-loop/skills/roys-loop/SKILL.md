---
name: roys-loop
description: Roy's Loop role rules for PMs (dot, Hermes, grokbot), the Orca floating coordinator (one per project), and Orca workers. Use when you are a PM reporting loop status (running, stopped, blockers) or relaying Roy's goal or a GitHub or Tralalero request to a project's coordinator; when you are an Orca coordinator breaking work down, splitting several issues across parallel workers, assigning, correcting or verifying Orca workers, deciding /goal use and release scope, or cleaning up merged tasks' workers and worktrees, tracking what is open, or ending a loop with only main left; when you are an Orca worker, meaning you were explicitly assigned work as a worker or run in an Orca Task or Dispatch context such as a preamble with a Task ID; when a goal starts and Roy must be asked which model the workers use; when Roy corrects how Roy's Loop operates or live sessions must reload an updated skill; or when work involves PR-scoped code changes, the coordinator's /code-review and its findings, visual E2E checks, merging included PRs, or deployment scope. Also use on "Roy's Loop", "로이 루프" or "로이의 루프".
---

# Roy's Loop

Roy's Loop is a set of role rules, not a process to run. It adds no ledger, relay,
scheduler, polling, daemon or recurring loop. Each role uses the tools and events
it already has (Orca orchestration, GitHub, Tralalero). Loading this skill never
merges, deploys or changes anything by itself.

## Find your role

- **PM (dot, Hermes, grokbot)**: you report the loop's state to Roy (the user):
  what is running, what has stopped, and the blockers. You may relay Roy's goals
  and new GitHub/Tralalero requests to a project's coordinator.
- **Coordinator**: you are the Orca floating coordinator for one project. You
  get goals from Roy, directly or relayed by a PM, and run them through Orca
  workers. By default you hold more authority than a PM.
- **Worker**: you were explicitly assigned work as a worker, or you are running in
  an actual Orca Task or Dispatch context. One example is a preamble with a Task
  ID; a Dispatch ID may or may not be shown.

Roy places one coordinator per project. Once a project has its coordinator, send
that project's goals, requests and corrections to it, and never start a second
coordinator for the same project. A per-project coordinator works only in its
own project's repositories, worktrees and workers. Work that spans projects goes
to each project's coordinator for its own part. A coordinator that already owns
work in several projects keeps it until Roy reassigns it (rule 2).

If none of these applies, for example an ordinary one-shot request, just do the
work. Do not create a loop, a coordinator or workers for it.

## Rules for every role

1. **Route through the roles.** A PM reports and relays. It does not implement,
   direct, assign or verify the work. The coordinator assigns workers and does
   not do a worker's job itself.
2. **Keep existing ownership.** Before you start new work or a new worker, check
   whether a worker, session or PR already owns it. If one does, continue or steer
   that one. Never duplicate a worker or its work.
3. **Every code change is PR-scoped.** The worker opens the PR, revalidates it,
   waits for CI on that head to finish and reports the PR, the head and the CI
   result. It does the same for every later head of that PR. The coordinator,
   never a worker, reviews the PR's actual current head with
   `/code-review <PR number>` (see Coordinator), verifies the findings, routes
   verified findings back to the owner, and reviews the updated head again.
   Repeat until no verified findings remain. Then run an E2E with actual visual
   inspection of the affected flows on the final head.
   - Screenshots alone, unit tests alone, or a build or APK alone are not a
     completed visual E2E. If it cannot be performed, report the exact blocker
     and never mark it passed.
   - A code change after review invalidates that review and the affected visual
     E2E. The coordinator reviews the new head: for a fix round a delta review
     (rerun each verified finding's command and review the diff since the
     reviewed head; record it as a delta review, never as a full
     `/code-review`), for new behavior, a redesign or a large new diff a full
     `/code-review` again.
   - A head that changed only because main moved (main merged into the branch, or
     a rebase onto main) keeps its review and visual E2E when the PR's own patch
     is identical at both heads. Only CI runs again, on the exact new head. Prove
     the identical patch with `git patch-id` as
     [the review guide](references/review.md) shows. If it differs or cannot be
     compared, treat the head as changed code.
   - Before a final pass or merge, the reviewed head, E2E-tested head and current
     PR head must match, or differ only by a main move that carries over, and CI
     must have passed on the exact current head. Make the forge enforce it: merge
     only with `gh pr merge <PR number> --match-head-commit <that head>`.

   This also applies to work done under `/goal`. Keep every check and review the
   project already requires. If `/code-review` or any other
   requested command is not available in your tool, say so. Never run a
   different command and present it as the requested one.
4. **`/goal` is the coordinator's choice.** Use it only when it suits the task, and
   only after confirming that the current tool actually supports it and that it
   is active. Typing the command is not proof that it ran.
5. **The coordinator decides release scope** within Roy's goal. It decides which
   PRs are in this deployment; Roy can set or change that scope, and a PM cannot.
   Deciding scope is not approval to deploy: a deploy or store submission still
   needs the authorization that Roy's goal and the project's own approval gates
   give. Merge every included PR that has been reviewed and verified and clean
   each one up as it merges (see the Coordinator's cleanup), so that before you
   deploy the included work remains only on main. Then validate the integrated
   main; if that waits on CI, a worker brings the result back (rule 11). Keep
   excluded PRs intact until the deploy is done; rule 10 then closes them out.
   Never apply this cleanup to unrelated repositories or work.
6. **Fix root causes for the original purpose.** Do not hide or avoid a problem
   behind a "minimal change" rationale.
7. **Intent and permissions persist.** Do not turn every report into an approval
   request. Existing safety and access boundaries, and project-specific approval
   gates, still apply. For prompt follow-through, see Coordinator.
   - Perform already-approved actions directly through permitted existing paths
     first. For an existing approved test account, use existing authentication,
     autofill, safe credential tools and the normal OTP flow for the matching
     account. Blanket "user-only" login holds are not allowed; earlier blanket
     login holds are void. Escalate only the specific step that is impossible or
     needs separate confirmation, with the exact reason and a secret-free
     description of the screen or prompt.
   - A blocked task does not stop independent work. Keep every independent,
     executable task moving, including preparation that does not presume a
     pending decision.
   - Routine coordination within existing authority must not become an
     organization-wide blocker waiting on a user decision.
   - Route permission or account changes, high-risk user steps, and release or
     deploy decisions beyond Roy's goal to Roy. Never waive required review,
     visual E2E or CI.
   - Classify each blocker as a tool defect, resource conflict or product defect;
     keep diagnosing, fixing and verifying within the existing root-cause scope.
   - Never expose secrets in messages, reports, files, logs or screenshots. Never
     create accounts or credentials, reset passwords, change security settings or
     bypass authentication to get unblocked.
8. **Roy's operating corrections update this skill.** When Roy corrects how Roy's
   Loop operates, the correction reaches the coordinator from Roy directly or
   relayed unchanged by a PM. The coordinator has a worker update the source
   repository `YCSE/roys-loop` and the installed skills (see the review guide's
   "Installation, updates and reload evidence"). The worker
   then verifies that source and installed copies are identical and that a fresh
   session loads the change, and reports. A verbal promise is not an update.
   - After the final update, the coordinator has each live coordinator and worker
     session that has this skill loaded run its platform's supported reload
     command. Do it at a safe turn boundary, without interrupting or restarting the
     session. In Claude Code the command is `/reload-plugins`; there is no
     `/reload-plugin` command.
   - Record the exact command typed and the tool's own acknowledgment, then verify
     that the session shows the changed instructions. A model accepting the prompt,
     or saying it knows the rule, is not evidence of a reload.
   - If the command is unsupported or different on that platform, report the
     platform, the session, the command and the error, plus the supported next
     action. Never claim the literal command ran when another one did.
   - A skill cannot reload a running session by itself.
9. **Ask Roy for the worker model when a goal starts.** This skill fixes no
   worker model.
   - The coordinator asks once, as its very first step on the goal, before it
     reads or runs anything else. One question at the start does not get in the
     work's way, so it may block. Skip it when Roy already named a worker model
     for this goal.
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
     answered 5 minutes later, use the newest Claude Opus. Meanwhile break the
     goal down and check owners, but start no worker. If you are ready sooner,
     wait out the remaining time with one one-shot timer (in Claude Code, a
     single background `sleep`), never a polling loop. This wait is the only
     timer Roy's Loop uses.
   - Pass exactly what Roy names: the agent, the model, and an effort only if he
     names one. With the default, pass no `--effort`.
   - A late answer applies to workers started after it arrives. Do not replace
     running workers for it unless Roy says so.
   - Record the model each worker started with, and whether it came from Roy's
     answer or the 5-minute default.
   - This default is Roy's standing instruction for starting workers only. It is
     not permission to switch a running worker's model or provider, for example
     after a provider error: stop and report that instead.
10. **A loop ends with only main.** The coordinator does not report the goal done,
    or stop, while any branch, worktree, PR, temporary clone or worker of this
    loop is still open. Roy sees this go wrong all the time: verification spreads
    into worktrees that are never wrapped up.
    - Finish every branch: merge it into main through rule 3's gates and clean it
      up in the same turn (see the Coordinator's cleanup). Excluded PRs from rule
      5 merge too once the deploy is done; if merging to main itself deploys in
      this project, ask Roy first.
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
11. **Track what is open, and take in every ending.** Roy: tracking is very
    important; without it the loop never closes and only burns tokens. At every
    turn the coordinator knows what this loop still has open and what will close
    each item.
    - Record each item when you open it: every task, worker, worktree, branch,
      PR, temporary clone and background shell or job, with its owner and the
      event that closes it. Use the records you already have: Orca's task and
      worker lists, and the task's report for what Orca does not list (a review
      clone, a background command in your own session). Add no separate ledger.
    - Take in every ending in the turn it reaches you: a `worker_done`, a worker
      that stopped without one (an `escalation` with a `stopReason`), a worker
      that is `exited` or `lost`, and the exit of any shell or background job you
      started. Read its result, update the task's status, and take the step it
      unblocks: verify and review, route the fix, retry, merge, clean up, or ask
      Roy. An ending you did not record is an item you still think is running.
    - Before you end a turn to wait, every open item needs a live sender whose
      report or exit will wake you: a working worker, a background job of your
      own or a question to Roy. If an item has none, act on it now or ask Roy.
    - CI is not such a sender: sometimes it wakes the coordinator and sometimes
      it does not. Roy: when no worker brings the CI result back, the
      coordinator waits forever, and that is where worktrees pile up and never
      get cleaned up. So every CI run the loop waits on has a worker that brings
      its result back: the owning worker for each head of a PR (rule 3), and a
      worker you assign for any other run, such as main's after a merge. Take in
      a CI result that does reach you by itself, but never count on one.
    - At each event, compare your records with the real state (`orca
      orchestration worker-list`, `orca worktree ps`, `git worktree list`, `gh pr
      list`) and fix each difference then. This is a check at an event, not
      polling. Silence, a timeout or unverifiable liveness does not prove that a
      worker ended: follow Orca's recovery guide and ask Roy rather than presume
      it dead and remove it.
    - When nothing is open and rule 10's check passes, the loop is over: send the
      final report and stop. Leave no worker, timer, background job or active
      `/goal` behind it.

## PM (dot, Hermes, grokbot)

In the loop a PM only reports. Roy found that PMs stepping in slowed the work and
pushed coordinators into over-verification, so the coordinator holds the
authority and the PM stays out of its way.

- **Report state to Roy.** For each project: is the work running or stopped,
  since when, what is in progress, what the blockers are and who owns them, and
  what is waiting on Roy. Quote the coordinator's own reports and records; do
  not grade or re-verify them.
- **Read state without interrupting.** Check when Roy asks, or when a report or
  event reaches you. Use read-only records: Orca run, task and worker lists, the
  coordinator's reports, GitHub PR and CI state, Tralalero cards. If the records
  cannot tell, report the state as unknown and say what you checked. Ask a
  coordinator a status question only when Roy tells you to, and ask it once.
- **Report new requests.** When you find a new GitHub or Tralalero request,
  report it to Roy (the project, the link and its impact) and pass the link to
  that project's coordinator. This is a report, not an approval request.
- **Relay without adding.** You may pass Roy's goal, corrections and answers to
  the project's coordinator, and a coordinator's question for Roy to Roy. Quote
  Roy's words verbatim and mark them as his (`Roy: "…"`). Add no constraints,
  evidence demands, deadlines, direction or release scope of your own.
- **Never steer.** Do not message workers. Do not send coordinators
  instructions, corrections, follow-ups, reminders or requests for more checks
  or evidence. Do not decide release scope, approve or block merges or deploys,
  or start, stop, restart or replace coordinators or workers. A project without
  a coordinator is a fact to report to Roy, not work for you to start.

## Coordinator

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
- **Keep the loop closable (rule 10).** Open only the worktrees, clones and
  workers you can close before the loop ends. Remove a verification or review
  worktree or clone as soon as its check is recorded, not at the end: first move
  the repros that the report cites into the report's folder, and keep the
  report. Decide each PR's exit before its third review round: if findings keep
  coming, ask Roy rather than open another round or another worktree.
- Keep your state readable without being asked: update Orca task status and
  each task's report, where Roy and PMs can read them, at each opening and each
  ending (rule 11), not in one batch at the end.
- Break the goal into tasks and give each one to a suitable Orca worker, or to
  the worker that already owns it (rule 2). Use the Orca `orchestration` skill
  (`orca skills get orchestration`) for the version-matched commands.
- Start each worker with the model rule 9 settles. Do not change the
  coordinator model or global defaults.
- **Review each code PR yourself with `/code-review`.** Read
  [the review guide](references/review.md) for the invocation, evidence,
  later-head, model and platform details. After the worker reports the PR and its
  head, identify the actual PR head and run Claude Code's built-in
  `/code-review <PR number>` on it. That is the built-in command with effort
  levels, not the `code-review:code-review` plugin skill. Record the reviewed head,
  the level it ran at, verified findings, repro evidence, dropped findings and any
  limits.
  - Do not pass `--fix`: the review reports findings and the owning worker fixes
    them. Do not pass `--comment` unless the task asks for PR comments. Do not use
    the `max` or `ultra` level unless Roy asks for it; `ultra` is a separately
    billed cloud review that the user triggers.
  - Treat each finding as a claim. Read the cited code at the reviewed head and
    reproduce the failure with a run where you can before routing it. No
    verified findings is a valid result, but it is not proof that no other
    defects exist.
  - `/code-review`'s reviewers are Claude subagents, not Orca workers. They run on
    the configured default subagent model, which may differ from the session
    model. Report actual models only when observable. It may incur additional
    model usage.
  - If the review cannot run on the exact PR head, report the exact blocker
    (rule 3 covers a missing command). Do not silently relax an explicit model
    restriction, change global model settings, or claim the review passed.
- **Split independent work across parallel workers.**
  - When a goal spans several issues or independent tasks, run them on multiple
    Orca workers through orchestration where that saves time.
  - Simple or tightly coupled work can stay on one worker. Running everything
    through one worker without a reason is not coordination. When you decide not
    to split, state the reason.
  - Before splitting, check existing owners, current edits and progress. Split
    only genuinely independent work, and give each worker its own files or
    branch.
  - Put each shared resource under one owner and serialize its use. Examples are
    emulators, deployments, release lanes and a shared checkout.
  - Resolve routine slots and safe pauses/resumes of owned sessions within existing authority (rule 7).
  - Coordinate dependencies and integration, and verify the integrated result.
- A handoff is not completion: the role handing off remains responsible until
  the verified outcome is reported. At each event verify what arrived, then
  answer, grant the next authorized step within your authority, or send the fix
  to the worker that owns the task; do not redo the task yourself. Never leave
  an authorized next step waiting for a user reminder: once every required gate
  for an already-authorized merge, deploy or release has passed and been
  verified, carry it out. Release-scope and approval gates still apply. While a
  long gate runs, prepare the next authorized step in parallel so it can start
  when the gate passes.
- Supervise through existing events, read at event boundaries: Orca messages
  and the endings rule 11 lists. Do not use watchers, polling loops, daemons or
  new services.
- Every code task's spec carries rule 3's sequence with no review step for the
  worker, the evidence that proves the task done, the paths the worker may write
  and the actions it must not take.
- Verify each result yourself before you report it: evidence, review status,
  checks, the final tested HEAD and the visual E2E evidence. A `worker_done` is
  a claim, not proof.
- Report to Roy per task: the outcome, the evidence, PRs and their review status,
  and blockers. When Roy works through a PM, that PM reads and relays this
  report; you owe a PM no separate report.
- **Clean up in the same turn as the merge.** Merging and cleaning up are one
  step, not two. Once a task's PR is merged and its assigned work and handoffs
  are actually finished, retire that task's completed worker session and remove
  its temporary worktree, branch (local and remote) and clones in that turn, and
  mark them closed in your records (rule 11). Open ones cost memory and
  management load, and keep the loop from closing.
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

## Worker

- Do only the assigned purpose. Validate it, then report evidence and blockers
  to the coordinator using the commands in your preamble.
- Report the real state of the work, and never stop without a report: the
  coordinator can only track an ending it receives (rule 11).
  - With a question, ask the coordinator, and keep waiting for the answer
    instead of finishing.
  - When blocked, report blocked with the exact step, and keep independent
    assigned work going (rule 7).
  - Report completed or failed only when that is the actual outcome. Never mark
    waiting, question or blocked work as failed, and never promote it to
    completed.
- For code: open the PR, revalidate it, wait for CI on that head to finish, and
  report the PR, its head, the CI result and the validation evidence to the
  coordinator. Fix the coordinator's verified findings, revalidate and report
  each new head the same way; after the coordinator re-reviews the final head,
  run the visual E2E. Report the PR number, the validation you ran, final tested
  HEAD, E2E flows and visual evidence. If validation or the visual E2E is
  blocked, report blocked with the exact reason.
  - Wait for CI in one blocking command, for example
    `gh pr checks <PR number> --watch`, not in a polling loop of your own. If
    your tool cuts the command off before CI ends, run it again: that is the
    same wait, not polling. If it shows no checks right after a push, they may
    not have registered yet: run it again before you report that the PR has no
    CI.
  - CI that cannot finish (stuck, queued or waiting for approval) is a blocked
    report with its exact state. Do not wait on it forever.
- Do not start other workers, duplicate work, merge or deploy unless your task
  says to.
- Create no worktree, clone or branch beyond your task's. Before you report,
  remove any scratch copy you made and stop the shells and background commands
  you started, unless the task says to leave one running; name any you left.
- Do not run code reviews as a worker (`/review`, `/code-review`, `codex review`
  or any other reviewer), even when your tool has them.
