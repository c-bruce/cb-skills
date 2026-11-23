# cb-skills

Generic skills for Claude Code, packaged as a plugin.

## Install

```bash
claude plugin marketplace add c-bruce/cb-skills
claude plugin install cb-skills@cb-skills
```

Update with `claude plugin marketplace update cb-skills`, then `claude plugin update cb-skills@cb-skills`.

## Skills

| Skill | Use it to |
|---|---|
| [`worktree`](skills/worktree/SKILL.md) | Set up a git worktree to start an issue, work on a PR or review one, and remove it when done. Configured per user and per repo: see [CONFIG.md](skills/worktree/CONFIG.md). |

## Writing a skill

Each skill is a folder under `skills/` with a `SKILL.md`. To try changes without reinstalling, symlink the
folder into `~/.claude/skills/`, then remove the link when you're done.

Check the plugin before you push:

```bash
claude plugin validate .
```
