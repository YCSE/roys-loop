# Roy's Loop

Role rules for running software work through agents:

- **PMs** report the loop's state (running, stopped, blockers) and relay requests. They do not steer.
- **One Orca floating coordinator per project** owns the loop: it breaks work down, starts workers, reviews each PR with `/code-review`, and decides merges and release scope.
- **Orca workers** open PRs, revalidate, fix the coordinator's verified findings and run the visual E2E. They run no reviews.

It is a set of rules, not a process. It adds no scheduler, polling or daemon, and loading it changes nothing by itself. The rules are in [`plugins/roys-loop/skills/roys-loop/SKILL.md`](plugins/roys-loop/skills/roys-loop/SKILL.md), and the review procedure is in [`references/review.md`](plugins/roys-loop/skills/roys-loop/references/review.md).

This repository is the single source. Every device installs from it.

## Install

### Claude Code

```sh
claude plugin marketplace add YCSE/roys-loop
claude plugin install roys-loop@roys-loop --scope user
```

To let Claude Code update it on its own, turn on auto-update for the `roys-loop` marketplace in `/plugin`, or set `"autoUpdate": true` on its entry under `extraKnownMarketplaces` in `~/.claude/settings.json`:

```json
"roys-loop": {
  "source": { "source": "github", "repo": "YCSE/roys-loop" },
  "autoUpdate": true
}
```

### Codex

```sh
codex plugin marketplace add YCSE/roys-loop
codex plugin add roys-loop@roys-loop
```

### OpenCode

OpenCode reads skills from a real folder, so keep a clone and copy the skill out of it:

```sh
git clone https://github.com/YCSE/roys-loop ~/.local/share/roys-loop
mkdir -p ~/.config/opencode/skills
rsync -a --delete ~/.local/share/roys-loop/plugins/roys-loop/skills/roys-loop/ ~/.config/opencode/skills/roys-loop/
```

## Update

After a new version is pushed:

| Tool | Command |
| --- | --- |
| Claude Code | `claude plugin marketplace update roys-loop` then `claude plugin update roys-loop@roys-loop --scope user` (or let auto-update do it) |
| Codex | `codex plugin marketplace upgrade roys-loop` then `codex plugin add roys-loop@roys-loop` |
| OpenCode | `git -C ~/.local/share/roys-loop pull`, then the `rsync` line above |

Running sessions keep the old text until they reload. In Claude Code, run `/reload-plugins`.

## Changing the rules

Edit a clone and raise `version` in both `plugins/roys-loop/.claude-plugin/plugin.json` and `plugins/roys-loop/.codex-plugin/plugin.json`, so that `plugin update` sees a new release. Land the change on `main`: through a PR reviewed by the coordinator when the change runs inside the loop (rule 3), or directly when Roy makes or orders it outside the loop.

The repository is public. Keep secrets, hostnames, personal paths and work records out of it.

## License

MIT. See [LICENSE](LICENSE).
