# Roy's Loop: worker

Read this file when you are a worker, and again whenever its text is not in
front of you (after a context compaction it is not). `SKILL.md` holds the
rules; this file says how you carry them out. Rule numbers are `SKILL.md`'s.

- Do only the assigned purpose. Validate it, then report evidence and blockers
  to the coordinator using the commands in your preamble. Put in the report
  what the next unit needs and only your context holds: the branch and head,
  the decisions you made, what you found and what is left undone.
- Your task is one unit (rule 12). When it turns out to hold several, ask the
  coordinator to cut it, saying what is done and where you would cut, rather
  than push on.
- After a compaction, read your Task's spec again before your next step:
  `orca orchestration dispatch-show --task <task id>` shows a Task's dispatch
  context. Say in your report that a compaction happened, because it tells the
  coordinator the unit was too big.
- Report the real state of the work (rule 11).
  - With a question, ask the coordinator, and keep waiting for the answer
    instead of finishing.
  - When blocked, report blocked with the exact step, and keep independent
    assigned work going (rule 7).
  - Report completed or failed only when that is the actual outcome. Never mark
    waiting, question or blocked work as failed, and never promote it to
    completed.
- For code, do the part of rule 3's sequence that your task names: open the PR,
  revalidate it, wait for CI on that head to finish, and report the PR, its
  head, the CI result and the validation evidence to the coordinator. Fix the
  coordinator's verified findings, revalidate and report each new head the same
  way; after the coordinator re-reviews the final head, run the visual E2E.
  Report the PR number, the validation you ran, final tested HEAD, E2E flows
  and visual evidence. If validation or the visual E2E is blocked, report
  blocked with the exact reason.
  - Wait for CI in one blocking command, for example
    `gh pr checks <PR number> --watch`, not in a polling loop of your own. If
    your tool cuts the command off before CI ends, run it again: that is the
    same wait, not polling. If it shows no checks right after a push, they may
    not have registered yet: run it again before you report that the PR has no
    CI.
  - CI that cannot finish (stuck, queued or waiting for approval) is a blocked
    report with its exact state. Do not wait on it forever.
- Create no worktree, clone or branch beyond your task's. Before you report,
  remove any scratch copy you made and stop the shells and background commands
  you started, unless the task says to leave one running; name any you left.
- No code review means no reviewer of any kind: `/review`, `/code-review`,
  `codex review` or any other, even when your tool has them (rule 1).
