# cb-skills

A Claude Code plugin. Each skill is a folder under `skills/` with a `SKILL.md`.

- Bump `version` in `.claude-plugin/plugin.json` with every change to a skill. Installed copies update only
  when the version changes.
- Add each new skill to the Skills table in `README.md`.
- Run `claude plugin validate .` before pushing. Done when it prints `Validation passed`.
- Keep skills generic: no personal preferences and nothing that names a particular repo or project.
- To try a skill without reinstalling the plugin, symlink its folder into `~/.claude/skills/`, and remove the
  link when done.
