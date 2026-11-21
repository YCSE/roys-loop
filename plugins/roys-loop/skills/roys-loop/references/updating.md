# Roy's Loop: updating the skill

Read this when Roy corrects how Roy's Loop operates (rule 8), or when a new
version has to reach the devices and the live sessions. Rule numbers are
`SKILL.md`'s.

## Change the source

The single source of Roy's Loop is the public GitHub repository
[`YCSE/roys-loop`](https://github.com/YCSE/roys-loop). Change it there: edit a
clone, raise the `version` in both `plugins/roys-loop/.claude-plugin/plugin.json`
and `plugins/roys-loop/.codex-plugin/plugin.json`, and land it on `main`: through
a PR under rule 3 when a coordinator runs the change, or directly when Roy makes
or orders it outside the loop.
Never edit an installed copy; a cache-only edit is overwritten by the next
update. Keep the repository free of secrets, hostnames, personal paths and work
records, because anyone can read it.

Where a rule goes:

- `SKILL.md` stays at 13,500 bytes or less; check with `wc -c`. After a context
  compaction a tool may keep only the start of a skill: on 2026-11-21 Claude
  Code's documentation says the first 5,000 tokens. At three bytes a token the
  limit is 4,500 tokens, which leaves a tenth spare.
- `SKILL.md` holds what binds every role, and every limit that must hold even
  when a role file was not read: the money guards, the gates that are never
  waived, the secret and account limits, and what a worker never does.
- How one role carries a rule out goes into that role's file. State each rule
  once and point to it from elsewhere.

## Bring each device up to date

Every device installs from that repository (the README has the install
commands). After a push:

- Claude Code: `claude plugin marketplace update roys-loop --json`, then
  `claude plugin update roys-loop@roys-loop --scope user --json`. A device with
  the marketplace's `autoUpdate` on may already have done this itself; check.
- Codex: `codex plugin marketplace upgrade roys-loop`, then
  `codex plugin add roys-loop@roys-loop --json`.
- OpenCode: `git pull` in its clone, then copy the skill folder into OpenCode's
  skills directory with `rsync -a --delete` (see the README).

Do these only when the update is authorized. Verify that each device's active
installed skill folder matches the pushed commit, file for file; a command's
success message alone is not verification. Respect a denied update and report
it rather than replacing the cache to evade the denial.

## Reload the live sessions

- After the final update, the coordinator has each live coordinator and worker
  session that has this skill loaded run its platform's supported reload
  command. Do it at a safe turn boundary, without interrupting or restarting the
  session. In Claude Code the command is `/reload-plugins`; there is no
  `/reload-plugin` command.
- A reload does not replace a role file that the session read earlier, and an
  installed version has its own folder. Have the session load the skill again
  and then read its role file from the folder that load names. On 2026-11-21
  Claude Code's documentation says a skill invoked again is loaded in full when
  its text differs from the copy already in context.
- Record the exact command typed and the tool's own acknowledgment, then verify
  that the session shows the changed instructions. A new session or reading a
  file is not a claimed live reload.
- If the command is unsupported or different on that platform, or access is
  denied, report the platform, the session, the command and the error, plus the
  supported next action. Never claim the literal command ran when another one
  did.
