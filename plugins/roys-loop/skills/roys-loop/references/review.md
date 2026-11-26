# Code review in Roy's Loop

Read this reference when your unit has a PR (worker) or when one is reported
to you (coordinator). Roy's Loop's role, permission, ownership and release
rules still apply, and rule numbers are `SKILL.md`'s. A review is not
permission to merge, deploy, change model settings, start a watcher, or bypass
a required gate.

pstack interrogate is not used for this review.

## The worker: review, fix, report

1. Open the PR and read its head:
   `gh pr view <PR number> --json headRefOid,baseRefName`. Your worktree is on
   that head with nothing uncommitted: the review's fixes go into the working
   tree, and anything else there would be mixed into them. Fetch first. A
   review that takes in commits that are not the PR's, as one against a stale
   base does, is a blocker to report, not a review.
2. Review the PR once, with your own tool's review command. Use the options
   the command actually has; do not invent flags.
   - **Claude Code**: the built-in `/code-review <level> <PR number> --fix`.
     It is the command with effort levels, not the `code-review:code-review`
     plugin skill. From inside another skill, invoke it through the `Skill`
     tool with those arguments. Always pass the level your spec names: without
     one the command reuses the level typed last, which nobody chose for this
     PR. If your spec names none, ask the coordinator before you run it. The
     level stays within rule 3's limit. Do not pass `--comment` unless the
     task asks for PR comments.
   - **Codex**: `/review`. From a shell it is
     `codex review --base <base branch>`, which reviews the branch against the
     PR's base.
   - **OpenCode**: `/review <PR number>`.
   - Where the command applied no fixes, make them yourself, once: fix what
     the review found in this PR's change. A finding you leave goes in your
     report with the reason.
3. Read what changed and revalidate it. A fix that breaks the intended
   behavior or the project's checks is corrected or dropped, and one outside
   the paths your spec lets you write is dropped; say which in your report.
   Commit and push.
4. Wait for CI on that head and run the visual E2E on it. When either shows a
   break, fix it and run both again on the new head. A break that is still
   there after your fix is a report, not another try: the coordinator decides.
   Do not run the review again, on that head or any later one.
5. Report the PR and its full final head, the exact command you ran and the
   level when the review reports one, the review's own report (what it found,
   what was fixed, what was left and why), observable reviewer model details,
   the validation you ran, the CI result, the visual E2E flows and evidence,
   and blockers. Avoid secrets in the report. A typed command is not evidence
   that the review ran; its own report is. If a step cannot be run or proved,
   report it as blocked, not passed.

## The coordinator: check the report, decide

- Check the report against the unit's spec: the review's own report is there,
  and the head that CI passed on and the visual E2E covers is the PR's current
  head. One call reads that head and the results of its checks from the forge:
  `gh pr view <PR number> --json headRefOid,statusCheckRollup`. For a PR the
  worker reports as having no CI, that list is empty. That is the whole check.
  Do not run the review, a fix or the E2E again, verify the review's findings
  or add a check of your own.
- When the worker reports that no flow is affected, read
  `gh pr diff <PR number>` yourself: if a change could reach a screen, the E2E
  runs. Otherwise record the list from `gh pr diff <PR number> --name-only` in
  the E2E's place.
- Then merge, within the project's other required checks and release gates.
- When a report leaves something open, decide what it needs and record the
  decision: merging as it is, a fix unit to the owner, or closing the PR and
  putting what the goal still needs back on the plan. Merging as it is is for
  findings the review left unfixed: they hold a merge only if you decide they
  do. A review that did not run, CI that failed or has not finished on that
  head, and a visual E2E that failed or does not cover it always hold it
  (rules 3 and 7). Getting a blocked one run is yours to arrange.

## Later heads

The review runs once per PR, and no later head gets another. New behavior is a
new unit with its own PR, not a later head of this one.

- **The owner changed code.** After a fix, CI and the affected visual E2E run
  again on that head.
- **Only main moved.** When main was merged into the branch, or the branch was
  rebased onto main, the worker that did it compares the PR's own patch at the
  two heads and reports both heads and the patch id. Fetch first so that
  `origin/main` is current.

  ```sh
  git fetch origin
  git diff --binary "$(git merge-base origin/main <reported head>)" <reported head> | git patch-id --stable
  git diff --binary "$(git merge-base origin/main <new head>)" <new head> | git patch-id --stable
  ```

  When the first field of both outputs is the same, the patch is identical. The
  visual E2E then carries over to the new head, and only CI runs again, on that
  exact head. When the ids differ, an
  output is empty or a command fails, treat the head as changed code. The id
  also changes when main touched the context lines around the PR's hunks. An
  identical patch does not show that main's own changes leave the PR's behavior
  intact; the validation of the integrated main in `coordinator.md` covers
  that.
- **Before a merge.** For rule 3's head condition, a visual E2E that carried
  over covers the new head. Rule 3's merge command refuses a head that moved in
  the meantime.

A clone or worktree made for a check is scratch. Once the check is recorded and
nothing unsaved is left in it, remove it. Keep the report.

## Model and platform boundaries

- Worker models follow rule 9. Preserve existing owners.
- A review is additional model usage. `/code-review`'s reviewers are Claude
  subagents, not Orca workers, and run on the configured default subagent
  model, which may differ from the session model. `/review` runs on what its
  tool is configured with. Pass no model or configuration override to a review
  command (rule 9). Report actual models only when observable.
- If the review cannot run on the PR, or you cannot start the command from
  inside your session, report the exact blocker. Do not silently relax an
  explicit model restriction or claim the review passed.
