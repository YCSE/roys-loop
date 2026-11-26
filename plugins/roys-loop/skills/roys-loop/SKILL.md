---
name: roys-loop
description: Roy's Loop role rules for PMs (dot, Hermes, grokbot), the Orca floating coordinator (one per project), and Orca workers. Load it again after a context compaction if you hold one of these roles and no longer have its text. Use when you are a PM reporting loop status or relaying Roy's goal or a GitHub/Tralalero request to a project's coordinator; when you are an Orca coordinator cutting work into units, running parallel workers, assigning, correcting or verifying workers, deciding /goal use and release scope, tracking what is open, cleaning up after merges or ending a loop; when you are an Orca worker, assigned work as a worker or running in an Orca Task or Dispatch context (a preamble with a Task ID); when a goal starts and the worker model must be settled; when Roy corrects how Roy's Loop operates or sessions must reload the updated skill; or when work involves PR-scoped code changes, the worker's /code-review or /review, visual E2E, merges or deployment scope. Also use on "Roy's Loop", "로이 루프" or "로이의 루프".
---

# Roy's Loop

Roy's Loop is a set of role rules, not a process to run. It adds no ledger, relay,
scheduler, polling, daemon or recurring loop. Each role uses the tools and events
it already has (Orca orchestration, GitHub, Tralalero). Loading this skill never
merges, deploys or changes anything by itself.

This file holds the rules that bind every role. How a role carries them out is
in that role's file, in this skill's folder (Claude Code names the folder in
the "Base directory for this skill" line):

- PM: `references/pm.md`
- Coordinator: `references/coordinator.md`, with `references/review.md` when
  you merge a PR and `references/updating.md` when the skill itself changes
  (rule 8)
- Worker: `references/worker.md`, with `references/review.md` when your unit
  has a PR and `references/updating.md` when your task changes this skill

**If your role file's text is not in front of you, read it before your next
step.** That is so when this skill has just loaded, and again after a context
compaction: a tool may keep this file and drop the rest. What you remember
after a compaction is a summary, not a record, so read the records again too
(rule 12).

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
   not do a worker's job itself. A worker does its assigned unit and nothing
   else: unless its task says so it starts no workers, reviews no PR, merges
   nothing and deploys nothing.
2. **Keep existing ownership.** Before you start new work or a new worker, check
   whether a worker, session or PR already owns it. If one does, continue or steer
   that one. Never duplicate a worker or its work.
3. **Every code change is PR-scoped.** The worker opens the PR and reviews it
   once itself with its own tool's review command: in Claude Code
   `/code-review <level> <PR number> --fix` at the level its spec names, in
   Codex or OpenCode `/review`. Where the command applies no fixes, the worker
   makes them. It revalidates and pushes the fixes, waits for CI on that head
   to finish, runs an E2E with actual visual inspection of the affected flows
   on that head, and reports the PR, the head, the command it ran, the
   review's own report, the CI result and the E2E evidence. No later head is
   reviewed again: a review can always find more. The coordinator checks that
   report and decides the merge; it reruns nothing. Findings the review left
   unfixed hold the merge only if the coordinator decides they do, and merging
   with them recorded waives nothing. Whatever else a report leaves open is
   the coordinator's to decide too: another fix, or closing the PR. It asks
   Roy only when the goal's direction has to change.
   - Screenshots alone, unit tests alone, or a build or APK alone are not a
     completed visual E2E. If it cannot be performed, report the exact blocker
     and never mark it passed.
   - A change that affects no screen flow, such as a documents-only PR, has no
     visual E2E to run. The coordinator decides that from the diff, never from
     the worker's word alone: when a changed file could reach a screen, the
     E2E runs. Otherwise the PR's list of changed files at that head is
     recorded in the E2E's place.
   - Merge only after the review, and only a head that CI passed on and that
     the visual E2E covers, and have the forge enforce it:
     `gh pr merge <PR number> --match-head-commit <that head>`. Code changed
     after the visual E2E runs its flows again. When only main moved,
     `references/review.md` has the carry-over.
   - Raising the review's cost is Roy's call: no `max` or `ultra` level unless
     Roy asks for it. `ultra` is a separately billed cloud review that the user
     triggers.
   - This also applies to work done under `/goal`. Keep every check and review
     the project already requires. If a requested command is not available in
     your tool, say so. Never run a different command and present it as the
     requested one.
4. **`/goal` is the coordinator's choice.** Use it only when it suits the task, and
   only after confirming that the current tool actually supports it and that it
   is active. Typing the command is not proof that it ran.
5. **The coordinator decides release scope** within Roy's goal: which PRs are in
   this deployment. Roy can set or change that scope, and a PM cannot. Deciding
   scope is not approval to deploy: a deploy or store submission still needs the
   authorization that Roy's goal and the project's own approval gates give.
6. **Fix root causes for the original purpose.** Do not hide or avoid a problem
   behind a "minimal change" rationale.
7. **Intent and permissions persist.** Do not turn every report into an approval
   request. Existing safety and access boundaries, and project-specific approval
   gates, still apply.
   - Perform already-approved actions directly through permitted existing paths
     first. For an existing approved test account, use existing authentication,
     autofill, safe credential tools and the normal OTP flow for the matching
     account. Blanket "user-only" login holds are not allowed; earlier blanket
     login holds are void. Escalate only the specific step that is impossible or
     needs separate confirmation, with the exact reason and a secret-free
     description of the screen or prompt.
   - A blocked task does not stop independent work. Keep every independent,
     executable task moving, including preparation that does not presume a
     pending decision. Routine coordination within existing authority must not
     become an organization-wide blocker waiting on a user decision.
   - Route permission or account changes, high-risk user steps, and release or
     deploy decisions beyond Roy's goal to Roy. Never waive required review,
     visual E2E or CI. Changing an account stays Roy's. When the coordinator
     knows that Roy changed it and the tool's prompt names that account,
     copying a stopped worker's session into it is the coordinator's, and that
     copy is all it may choose there (`references/coordinator.md`, "Track
     what is open").
   - Classify each blocker as a tool defect, resource conflict or product defect;
     keep diagnosing, fixing and verifying within the existing root-cause scope.
   - Never expose secrets in messages, reports, files, logs or screenshots. Never
     create accounts or credentials, reset passwords, change security settings or
     bypass authentication to get unblocked.
8. **Roy's operating corrections update this skill.** When Roy corrects how Roy's
   Loop operates, the correction reaches the coordinator from Roy directly or
   relayed unchanged by a PM. The coordinator has a worker update the source
   repository `YCSE/roys-loop` and the installed copies, verify that they are
   identical and that a fresh session loads the change, and then has each live
   session reload, as `references/updating.md` describes. A verbal promise is
   not an update, a model saying it knows the rule is not a reload, and a skill
   cannot reload a running session by itself.
9. **Ask Roy for the worker model when a goal starts.** The coordinator asks
   once, as its first step on the goal after reading its role file, and passes
   exactly what Roy names. When Roy names none, the default is `--agent claude
   --model opus`, Claude Code's alias for its latest Opus, and it changes only
   when this rule does. It covers starting workers only. It is not permission
   to switch a running worker's model or provider, for example after a
   provider error: stop and report that instead. Do not change the coordinator
   model or global defaults.
10. **A loop ends with only main.** The coordinator does not report the goal done,
    or stop, while any branch, worktree, PR, temporary clone or worker of this
    loop is still open. Merging a PR and cleaning up after it are one step.
    Cleanup never destroys work: never force-remove a dirty, live or uncertain
    target, never drop or apply a stash or discard files to make one look
    clean, and never take silence as proof that a worker ended. When in doubt,
    keep it and ask Roy.
11. **Track what is open, and take in every ending.** At every turn the
    coordinator knows what this loop still has open and what will close each
    item, from the records it already has and no separate ledger. It takes in
    every ending in the turn it arrives. A worker never stops without a report:
    the coordinator can only track an ending it receives. CI is not a sender to
    wait on, because sometimes it wakes the coordinator and sometimes it does
    not: a worker brings back the result of every CI run the loop waits on.
    Supervise through events, never by polling or watchers.
12. **Hand out small units, and take in a report on each.** Nothing the loop
    needs lives only in a context: a fresh worker could do a unit from its spec
    alone, and a fresh coordinator could carry the loop on from the records
    alone.
    - A unit has one outcome and one piece of evidence that proves it, and a
      worker can finish and report it well inside one context. If its spec
      names several outcomes, cut it again. The steps to one outcome are one
      unit: opening a PR, reviewing it, waiting for CI on that head and
      reporting it is one.
    - Before the first worker starts, the coordinator records the plan: one
      Orca Task that lists the goal's units, the constraints (rule 13), and
      Roy's answers and its own decisions. It keeps that record current, and
      the Run's Tasks show which units are already out.
    - A unit becomes its own Task when it is handed out, after the reports it
      builds on are in and checked, with what they gave written into its spec.
      A worker holds one unit at a time.
    - A unit is finished only when its report has reached the coordinator and
      the coordinator has checked its evidence against the spec.
    - After a compaction, the records to read again are the Task's spec for a
      worker, and the plan and the Run's Tasks for a coordinator. A worker that
      cannot get its spec back asks the coordinator for it rather than work
      from the summary.
13. **Turn what a unit proved into a constraint for later units.** Without this
    a loop is fast and never gets smarter: the same failure comes back in the
    next unit and in the next loop. When the coordinator accepts a unit, it
    writes down what that proved and later work must respect, in one line with
    what it was derived from. A guess, or the unit's
    output restated, is not a constraint. The constraint lands where the work
    is cut: in how the rest is cut and in the spec of every later unit it
    applies to. What holds beyond this goal goes into the project's own agent
    instructions through a PR, so that the next loop starts where this one
    ended. A lesson about Roy's Loop itself goes to Roy, because only Roy's
    word changes this skill (rule 8).
