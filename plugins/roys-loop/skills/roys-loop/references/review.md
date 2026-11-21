# Code review in Roy's Loop

Read this reference when the coordinator reviews or merges a worker's code
change. Roy's Loop's role, permission, ownership and release rules still apply,
and rule numbers are `SKILL.md`'s. A review is not permission to merge, deploy,
change model settings, start a watcher, or bypass a required gate.

Since 0.8.0 (2026-11-19) the coordinator's review is Claude Code's built-in
`/code-review`. pstack interrogate is no longer used, and the worker's own
`/review` step is gone. Roy decided both.

## From worker result to final head

1. The owning Orca worker opens the PR, revalidates it, waits for CI on that
   head and reports the PR, its head, the CI result and the validation evidence.
2. The coordinator identifies the PR's current commit, intended behavior and
   base: `gh pr view <PR number> --json headRefOid,baseRefName`. Review the PR
   diff from its base/merge-base to that exact head, and exclude unrelated dirty
   changes. When a finding needs a run, use a safe checkout or isolated worktree
   of that head without disturbing other owners. If a clean matching scope
   cannot be established, report a blocker rather than review a stale or
   contaminated diff.
3. Run the built-in `/code-review <PR number>`. It is the command with effort
   levels, not the `code-review:code-review` plugin skill. From inside another
   skill, invoke it through the `Skill` tool with the PR number as its
   argument; follow the tool's actual schema and do not invent flags.
   - Do not pass `--fix`: the review reports findings and the owning worker
     fixes them. Do not pass `--comment` unless the task asks for PR comments.
     The level stays within rule 3's limit.
   - Record the level the review reports it ran at.
   - Read the PR head again after the review. If it moved while the review ran,
     the review covers the old head only.
4. Verify each finding before you route it: a finding is a claim. Read the
   cited code at the reviewed head and reproduce the failure with a run where
   you can. Keep verified failures that break the intended behavior, merge
   findings with the same root cause, and record dropped findings with the
   reason. No verified findings is a valid result, but it is not proof that no
   other defects exist.
5. Route verified findings to the existing owning Orca worker, who fixes and
   revalidates them and reports the new head with its CI result. Review that
   head as "Later heads" describes; repeat until no verified findings remain.
   Then the owning worker runs the visual E2E on the affected flows. When the
   worker reports that no flow is affected, compare its list of changed files
   with `gh pr diff <PR number> --name-only` before you accept that. Before a
   final pass or authorized merge, apply the head check in "Later heads". Keep
   the
   project's other required checks and release gates.

Record the PR and full reviewed head, scope and intent, the exact invocation and
level, observable reviewer model details, verified repro commands and outcomes,
dropped-finding counts and reasons, fix/re-review heads, the kind of each
re-review (full `/code-review`, delta review or carry-over), visual E2E flows and
evidence, and blockers. Avoid secrets in repros and evidence. A typed command is
not evidence that the review ran; its own report is. If a required step cannot
be run or proved, report it as blocked, not passed.

## Later heads

The full `/code-review` runs once, at the first code-ready head. Classify each
later head before you review it.

- **The owner changed code.** For a fix round, run a delta review of the new
  head: rerun each verified finding's command, then review the diff from the
  reviewed head to the new head. Record it as a delta review with both heads. It
  is not a full `/code-review`, so never label it as one. The same holds for a
  mutation check or any other variant: record it under its own name. New
  behavior, a redesign or a large new diff gets a full `/code-review` again.
  Re-run the affected visual E2E after the code change.
- **Only main moved.** When main was merged into the branch, or the branch was
  rebased onto main, compare the PR's own patch at the two heads. Fetch first so
  that `origin/main` is current.

  ```sh
  git fetch origin
  git diff --binary "$(git merge-base origin/main <reviewed head>)" <reviewed head> | git patch-id --stable
  git diff --binary "$(git merge-base origin/main <new head>)" <new head> | git patch-id --stable
  ```

  When the first field of both outputs is the same, the patch is identical. The
  review and visual E2E then carry over to the new head, and only CI runs again,
  on that exact head. Record both heads and the patch id. When the ids differ,
  an output is empty or a command fails, treat the head as changed code. The id
  also changes when main touched the context lines around the PR's hunks. An
  identical patch does not show that main's own changes leave the PR's behavior
  intact; the validation of the integrated main in `coordinator.md` covers
  that.
- **Before a merge.** The current PR head must be the head that CI passed on and
  that the review and the visual E2E cover, directly or by carry-over. Make the
  forge enforce it: merge only with
  `gh pr merge <PR number> --match-head-commit <that head>`, which refuses a head
  that moved in the meantime.

Review clones and worktrees are the coordinator's scratch. As soon as the
review's findings are recorded, move the repros that the report cites into the
report's folder, then remove the scratch once nothing still uses it and nothing
unsaved is left in it. Keep the report.

## Model and platform boundaries

- Worker models follow rule 9. Preserve existing owners.
- `/code-review`'s reviewers are Claude subagents, not Orca workers. They run
  on the configured default subagent model, which may differ from the session
  model, and may incur additional model usage. Report actual models only when
  observable.
- If the review cannot run on the exact PR head, report the exact blocker. Do
  not silently relax an explicit model restriction or claim the review passed.
- `/code-review` is a Claude Code command. A coordinator in a tool without it
  (Codex, OpenCode) reports the blocker. Do not run that tool's own reviewer,
  such as Codex `/review` or `codex review`, and present it as `/code-review`.
