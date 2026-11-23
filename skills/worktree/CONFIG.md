# Worktree config

Two optional files configure the `worktree` skill. Both use the same keys, in YAML frontmatter:

| File | Scope | Holds |
|---|---|---|
| `~/.claude/worktree.md` | the user, every repo | personal preferences, such as `branch_prefix` and `open` |
| `.claude/worktree.md` in the repo | one repo | the repo's setup, such as `install`, `ports` and `warn_if_changed` |

When both set a key, the repo file wins. `copy` and `warn_if_changed` combine from both files. The Markdown
body below the frontmatter holds anything the keys can't say; the skill reads and follows both bodies.

A repo file is personal setup, so keep it out of version control unless the whole team uses the skill.

## Keys

| Key | Default | Meaning |
|---|---|---|
| `branch_prefix` | none | Prefix for new issue branches, e.g. `feature/` |
| `worktree_dir` | `.claude/worktrees` | Where worktrees go, relative to the main checkout |
| `base` | the branch `origin/HEAD` points to | What new issue branches start from |
| `install` | from the lockfile, see below | Command that installs dependencies in a new worktree |
| `copy` | see below | Extra gitignored files to copy into a new worktree |
| `open` | none | Command that opens a new worktree; `{path}` is its absolute path |
| `ports` | none | Port slots, so several worktrees can run dev servers at once |
| `warn_if_changed` | none | Map of glob to message, reported when the branch changes a matching file |

### install

With no `install` key, pick the command from the lockfile in the repo root:

| Lockfile | Command |
|---|---|
| `bun.lock`, `bun.lockb` | `bun install` |
| `pnpm-lock.yaml` | `pnpm install --frozen-lockfile` |
| `yarn.lock` | `yarn install --frozen-lockfile` |
| `package-lock.json` | `npm ci` |
| `uv.lock` | `uv sync` |
| `poetry.lock` | `poetry install` |
| `Gemfile.lock` | `bundle install` |

Skip the step when no lockfile matches. Set `install: none` to skip it always.

### copy

A new worktree has none of the main checkout's gitignored files. These are always copied when they exist:

- every `.env*` file that `git ls-files -oi --exclude-standard --directory` lists
- `.claude/CLAUDE.local.md`
- `.claude/settings.local.json`

`copy` adds globs to that list, matched against gitignored files in the main checkout.

### ports

Each worktree takes a numbered slot `N`, from 1 to `slots`. The main checkout is slot 0 and uses the repo's
usual ports. Inside the templates, `{N}` is the slot, and `{8000+N}`-style sums are worked out before
writing.

| Sub-key | Default | Meaning |
|---|---|---|
| `slots` | required | How many worktrees can hold a slot at once |
| `file` | `.claude/worktree.env` | Port file written in the worktree |
| `env` | required | Port file contents |
| `start` | none | Command that starts the app with the port file |
| `url` | none | Where the app opens |

Take the lowest slot no other worktree's port file claims. Write the port file with `# worktree slot <N>`
as its first line, then `env`. With every slot taken, tell the user and skip ports.

Then append to the worktree's `.claude/CLAUDE.local.md`, creating it if missing:
`This worktree uses port slot <N>. Start the app with \`<start>\` and open <url>.`

### warn_if_changed

Use it for changes that reach outside the worktree, such as a migration run against a database that every
checkout shares.

## Example

`~/.claude/worktree.md`:

```markdown
---
branch_prefix: feature/
open: code -n {path}
---
```

`.claude/worktree.md` in a repo whose API and web client both take ports:

```markdown
---
ports:
  slots: 3
  env: |
    export API_PORT={8000+N}
    export WEB_PORT={8080+N}
    export API_URL=http://localhost:{8000+N}
  start: source .claude/worktree.env && npm run dev
  url: http://localhost:{8080+N}
warn_if_changed:
  "db/migrations/**": Every checkout shares one local database, and `npm run dev` migrates it.
---

The API accepts requests only from web ports 8081-8083, which caps the slots at 3.
```
