---
name: roys-loop
description: Roy's Loop role rules for PMs (dot, Hermes, grokbot), the Orca floating coordinator (one per project), and Orca workers. Use when you are a PM reporting loop status (running, stopped, blockers) or relaying Roy's goal or a GitHub or Tralalero request to a project's coordinator; when you are an Orca coordinator breaking work down, splitting several issues across parallel workers, assigning, correcting or verifying Orca workers, deciding /goal use and release scope, or cleaning up merged tasks' workers and worktrees; when you are an Orca worker, meaning you were explicitly assigned work as a worker or run in an Orca Task or Dispatch context such as a preamble with a Task ID; when a goal starts and Roy must be asked which model the workers use; when Roy corrects how Roy's Loop operates or live sessions must reload an updated skill; or when work involves PR-scoped code changes, the coordinator's /code-review and its findings, visual E2E checks, merging included PRs, or deployment scope. Also use on "Roy's Loop", "로이 루프" or "로이의 루프".
---

# Roy's Loop

Roy's Loop is a set of role rules, not a process to run. It adds no ledger, relay,
scheduler, polling, daemon or recurring loop. Each role uses the tools and events
it already has (Orca orchestration, GitHub, Tralalero). Loading this skill never
merges, deploys or changes anything by itself.

## Find your role

- **PM (dot, Hermes, grokbot)**: you report the loop's state to Roy (the user):
  what is running, what has stopped, and the blockers. You may relay Roy's goals
  and new GitHub/Tralalero requests to a project's coordinator. You do not steer
  the work.
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
3. **Every code change is PR-scoped.** The worker opens the PR, revalidates it
   and reports the PR and its head. Workers do not run code reviews; the
   coordinator does. The coordinator reviews the PR's actual current head with
   `/code-review <PR number>` (see Coordinator), verifies the findings, routes
   verified findings back to the owner, and reviews the updated head again.
   Repeat until no verified findings remain. Then run an E2E with actual visual
   inspection of the affected flows on the final head.
   - Screenshots alone, unit tests alone, or a build or APK alone are not a
     completed visual E2E.
   - If the visual E2E cannot be performed, report the exact blocker and never mark
     it passed.
   - Any code change after review invalidates that review. The coordinator
     reviews the new head again: a delta review for a fix round, a full
     `/code-review` for new behavior, a redesign or a large new diff (see
     Coordinator). Address verified findings and revalidate. Any later code
     change also requires re-running the affected visual E2E.
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
   give. Merge every included PR that has been reviewed and verified, then
   validate the integrated main. Before you deploy, clean up each PR included in
   this release, and only after both of these checks:
   - The PR is actually merged into main. Check the real PR merge state and its
     content on main, not only Git ancestry, because a squash merge is not an
     ancestor.
   - Its worktree is clean and has no live owner.

   Then remove that PR's temporary branch and worktree, so that the included work
   remains only on main. Keep excluded PRs intact. Also keep any uncommitted,
   untracked, unmerged or live-owned work intact. If something cannot be cleaned
   up safely, keep it and report the cleanup blocker. Never apply this cleanup to
   unrelated repositories or work. Merged tasks are also cleaned up promptly, not
   only at release time; see the Coordinator's post-merge cleanup.
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
     deploy decisions beyond Roy's goal to Roy. Release scope within the goal is
     the coordinator's (rule 5). Never waive required review, visual E2E or CI.
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
   session loads the change, and reports. A verbal promise is not an update. Do not
   use automatic watchers or services for this.
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
   - The coordinator asks Roy which model the workers should use, in its first
     message about the goal. When the goal came through a PM, the PM passes the
     question to Roy and his answer back unchanged. Skip the question when Roy
     already named a worker model for this goal. Ask in a plain message that does
     not block the work, say what the default is and the time it applies, and
     note the time you asked.
   - If Roy has not answered 5 minutes after the question, use the newest Claude
     Opus: the latest Opus the worker's tool offers when the worker starts. On
     2026-11-19 that is Claude Opus 5.5 (`claude-opus-5-5`); check the current
     model list rather than assuming it is still the newest. Through Orca that is
     `--agent claude --model <that id>`.
   - Pass exactly what Roy names: the agent, the model, and an effort only if he
     names one. With the 5-minute default, pass no `--effort`.
   - The question does not block preparation: break the goal down and check
     owners meanwhile. Start no worker before Roy answers or the 5 minutes pass.
     If you are ready sooner, wait out the remaining time with one one-shot timer
     (in Claude Code, a single background `sleep`), never a polling loop. This
     wait is the only timer Roy's Loop uses.
   - A late answer applies to workers started after it arrives. Do not replace
     running workers for it unless Roy says so.
   - Record the model each worker started with, and whether it came from Roy's
     answer or the 5-minute default.
   - This default is Roy's standing instruction for starting workers only. It is
     not permission to switch a running worker's model or provider, for example
     after a provider error: stop and report that instead.

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
  coordinator's reports, GitHub PR and CI state, Tralalero cards. Add no
  watchers or polling. If the records cannot tell, report the state as unknown
  and say what you checked. Ask a coordinator a status question only when Roy
  tells you to, and ask it once.
- **Report new requests.** When you find a new GitHub or Tralalero request,
  report it to Roy (the project, the link and its impact) and pass the link to
  that project's coordinator. This is a report, not an approval request, and it
  carries no direction of yours.
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
- Keep your state readable without being asked: keep Orca task status current
  and write each task's report where Roy and PMs can read it.
- Break the goal into tasks and give each one to a suitable Orca worker. Use the
  Orca `orchestration` skill (`orca skills get orchestration`) for the
  version-matched commands. If a worker already owns a task, send it the
  follow-up instead of starting a new worker.
- Start each worker with the model rule 9 settles: Roy's answer, or the newest
  Claude Opus once 5 minutes pass without one. Set no reasoning effort unless Roy
  names one. Continue existing owners rather than replacing them, and do not
  change the coordinator model or global defaults.
- **Review each code PR yourself with `/code-review`.** Read
  [the review guide](references/review.md) for the invocation, evidence,
  later-head, model and platform details. After the worker reports the PR and its
  head, identify the actual PR head and run Claude Code's built-in
  `/code-review <PR number>` on it. That is the built-in command with effort
  levels, not the `code-review:code-review` plugin skill. Record the reviewed head,
  the level it ran at, verified findings, repro evidence, dropped findings and any
  limits. Route fixes to the owning Orca worker and review the new head again
  before visual E2E. For a fix round that is a delta review: rerun each verified
  finding's command and review the diff since the reviewed head, and record it as
  a delta review, not as a full `/code-review`. New behavior, a redesign or a
  large new diff gets a full `/code-review` again.
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
  - Reviewing is the coordinator's job. Never put a review step in a worker's
    task.
  - If `/code-review` is not available in your tool, or the review cannot run on
    the exact PR head, report the exact blocker. Do not substitute a different
    reviewer and present it as `/code-review`, silently relax an explicit model
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
  the verified outcome is reported. Consume results, evidence and questions
  promptly at the event; verify them, then answer, grant the next authorized step
  within your authority, or route the fix.
  Never leave an authorized next step waiting for a user reminder.
- Once every required gate for an already-authorized merge, deploy or release has
  passed and been verified, carry it out without a user reminder. Release-scope
  and approval gates still apply. While a long gate runs, prepare the next
  authorized step in parallel so it can start when the gate passes.
- Supervise through existing events, including Orca messages, worker reports and
  CI results read at event boundaries. Do not use watchers, polling loops, daemons
  or new services.
- Correct direction by sending the fix to the worker that owns the task. Do not
  redo the task yourself.
- Put rule 3's full sequence in every code task's spec: PR and revalidation by the
  worker with no worker-run review, the coordinator's `/code-review` with fixes and
  re-review, then the visual E2E on the final head. Each task spec also names the
  evidence that proves the task done, the paths the worker may write and the
  actions it must not take. Before you report a pass,
  verify the final tested HEAD and the visual E2E evidence. An unperformed visual
  E2E is a blocker, not a pass.
- Verify each result yourself before you report it: evidence, review status and
  checks. A `worker_done` is a claim, not proof.
- Report to Roy per task: the outcome, the evidence, PRs and their review status,
  and blockers. When Roy works through a PM, that PM reads and relays this
  report; you owe a PM no separate report.
- **Clean up promptly after a merge.** Once a task's PR is merged and its assigned
  work and handoffs are actually finished, retire that task's completed worker
  session and its temporary worktree and branch without delay. Open ones cost
  memory and management load.
  - First verify:
    - the actual PR merge and its content on main, including squash merges;
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
  - The clones and worktrees that your reviews created are yours to clean
    up too, once the PR is merged or closed. First move the repros that the
    review report cites into the report's folder, and keep the report.
  - This extends rule 5's release cleanup; it does not weaken it.

## Worker

- Do only the assigned purpose. Validate it, then report evidence and blockers
  to the coordinator using the commands in your preamble.
- Report only the exact blocked step and keep independent assigned work going (rule 7).
- Report the real state of the work:
  - With a question, ask the coordinator, and keep waiting for the answer
    instead of finishing.
  - When blocked, report blocked with the exact blocker.
  - Report completed or failed only when that is the actual outcome.

  Never mark waiting, question or blocked work as failed, and never promote it to
  completed.
- For code: open the PR, revalidate it and report the PR, its head and the
  validation evidence to the coordinator, who reviews it. Fix the coordinator's
  verified findings and revalidate as directed; after the coordinator re-reviews
  the final head, run the visual E2E. Report the PR number, the validation you
  ran, final tested HEAD, E2E flows and visual evidence. If validation or the
  visual E2E is blocked, report blocked with the exact reason.
- Do not start other workers, duplicate work, merge or deploy unless your task
  says to.
- Do not run code reviews as a worker (`/review`, `/code-review`, `codex review`
  or any other reviewer), even when your tool has them. The coordinator reviews.
