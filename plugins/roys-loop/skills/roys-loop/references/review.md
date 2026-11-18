# Code review in Roy's Loop

Read this reference when the coordinator reviews a worker's code change. Roy's
Loop's role, permission, ownership and release rules still apply. Reviewing is
the coordinator's job: workers do not run code reviews. A review is not
permission to merge, deploy, change model settings, start a watcher, or bypass a
required gate.

Since 0.8.0 (2026-11-19) the coordinator's review is Claude Code's built-in
`/code-review`. pstack interrogate is no longer used, and the worker's own
`/review` step is gone. Roy decided both.

## From worker result to final head

1. The owning Orca worker opens the PR, revalidates it and reports the PR, its
   head and the validation evidence. The worker does not review it.
2. The coordinator identifies the PR's current commit, intended behavior and
   base: `gh pr view <PR number> --json headRefOid,baseRefName`. Review the PR
   diff from its base/merge-base to that exact head, and exclude unrelated dirty
   changes. When a finding needs a run, use a safe checkout or isolated worktree
   of that head without disturbing other owners. If a clean matching scope
   cannot be established, report a blocker rather than review a stale or
   contaminated diff.
3. Run the built-in `/code-review <PR number>`. It is the command with effort
   levels (`low` to `max`, plus `ultra`), not the `code-review:code-review`
   plugin skill. From inside another skill, invoke it through the `Skill` tool
   with the PR number as its argument; follow the tool's actual schema and do
   not invent flags.
   - No `--fix`: the coordinator does not fix code, the owning worker does.
   - No `--comment` unless the task asks for PR comments.
   - No `max` or `ultra` unless Roy asks for it. `ultra` is a separately billed
     cloud review that the user triggers.
   - Record the level the review reports it ran at.
   - Read the PR head again after the review. If it moved while the review ran,
     the review covers the old head only.
4. Verify each finding before you route it. Read the cited code at the reviewed
   head and reproduce the failure with a run where you can. Keep verified
   failures that break the intended behavior, merge findings with the same root
   cause, and record dropped findings with the reason. `/code-review` reports
   findings and does not fix them without `--fix`. No verified findings is a
   valid result, but it is not proof that no other defects exist.
5. Route verified findings to the existing owning Orca worker, who fixes and
   revalidates them. Review the new PR head again as "Later heads" describes;
   repeat until no verified findings remain. Then run actual visual E2E on the
   affected flows. A code change after review invalidates that review and
   requires another review of the new head and the affected visual E2E. Before a
   final pass or authorized merge, apply the head check in "Later heads". Keep
   the project's other required checks and release gates.

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
  intact; rule 5's validation of the integrated main covers that.
- **Before a merge.** The current PR head must be the head that CI passed on and
  that the review and the visual E2E cover, directly or by carry-over. Make the
  forge enforce it: merge only with
  `gh pr merge <PR number> --match-head-commit <that head>`, which refuses a head
  that moved in the meantime.

Review clones and worktrees are the coordinator's scratch. Once the PR is merged
or closed, move the repros that the report cites into the report's folder, then
remove the scratch under the Coordinator's cleanup checks. Keep the report.

## Model and platform boundaries

- Worker models follow SKILL.md rule 9: ask Roy when the goal starts, and use the
  newest Claude Opus if he has not answered in 5 minutes. Set no reasoning effort
  for a worker unless Roy names one. Preserve existing owners and the coordinator
  model. Do not change global defaults.
- `/code-review` runs reviewer subagents. These are Claude subagents, separate
  from Orca workers. A subagent started without a model runs on the configured
  default subagent model, which may differ from the session model.
  Report actual models only when observable, and do not claim a model was used
  otherwise. Additional subagents can incur model usage.
- `/code-review` is a Claude Code command. A coordinator in a tool without it
  (Codex, OpenCode) reports the blocker. Do not run that tool's own reviewer,
  such as Codex `/review` or `codex review`, and present it as `/code-review`.
- Workers do not run any reviewer, even when their tool has one, so keep review
  steps out of worker task specs.

## Installation, updates and reload evidence

The single source of Roy's Loop is the public GitHub repository
[`YCSE/roys-loop`](https://github.com/YCSE/roys-loop). Change it there: edit a
clone, raise the `version` in both `plugins/roys-loop/.claude-plugin/plugin.json`
and `plugins/roys-loop/.codex-plugin/plugin.json`, and land it on `main`: through
a PR under rule 3 when a coordinator runs the change, or directly when Roy makes
or orders it outside the loop.
Never edit an installed copy; a cache-only edit is overwritten by the next
update. Keep the repository free of secrets, hostnames, personal paths and work
records, because anyone can read it.

Every device installs from that repository (the README has the install
commands). After a push, bring each device up to date:

- Claude Code: `claude plugin marketplace update roys-loop --json`, then
  `claude plugin update roys-loop@roys-loop --scope user --json`. A device with
  the marketplace's `autoUpdate` on may already have done this itself; check.
- Codex: `codex plugin marketplace upgrade roys-loop`, then
  `codex plugin add roys-loop@roys-loop --json`.
- OpenCode: `git pull` in the clone its skill directory links to.

Do these only when the update is authorized. Verify that each device's active
installed `SKILL.md` and `references/review.md` match the pushed commit; a
command's success message alone is not verification. Respect a denied update
and report it rather than replacing the cache to evade the denial.

Existing sessions do not automatically prove they picked up updated instructions.
At a safe turn boundary use the platform's supported reload if available. In
Claude Code use `/reload-plugins`, record its own acknowledgment and verify the
changed skill text in that session. Do not interrupt an active session. If the
platform has no supported native reload or access is denied, report the exact
session and blocker; a new session or reading a file is not a claimed live reload.
