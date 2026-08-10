# Helpful Claude Skills 🛠️

A free, open collection of Agent Skills for Claude Code and Claude Desktop.

Every skill here is a single folder with a `SKILL.md` inside. Drop it in the
right directory and Claude picks it up automatically when the work matches.
No configuration, no dependencies, no accounts.

## The skills

| Skill | What it does | Scope | Runs in |
|---|---|---|---|
| [`conventional-commits`](skills/conventional-commits/) | Turns a diff or a description of your work into a spec-compliant Conventional Commit message. Flags non-atomic diffs instead of writing "feat: add X and fix Y". | Per project | Claude Code |

More coming. Suggestions and pull requests welcome.

**Scope** is where to install the skill.

- **Per project** means put it in the repo's `.claude/skills/`. Use this when
  the skill only makes sense inside a codebase, or when its behavior should
  be tuned per repo.
- **Global** means put it in `~/.claude/skills/` and get it everywhere. Use
  this only for skills you want loaded in every session regardless of what
  you are working on.

**Runs in** is which app the skill actually works in.

- **Claude Code** is the terminal agent. Reads both scopes.
- **Claude Desktop** reads the same `~/.claude/skills/` path as Claude Code,
  so global skills work in both.
- **Cowork / cloud** sessions do not read `~/.claude/skills/` from your
  machine. Cloud sessions do load project skills committed to the cloned
  repo. If you need a skill in Cowork, enable it on your claude.ai account
  instead.

If a skill lists fewer apps than you expected, that is usually because it
needs something only those apps have. `conventional-commits` needs a git
repo and a real diff, so it is a Claude Code skill in practice even though
nothing stops you copying it elsewhere.

## Install a skill

Pick a scope first. This is the only decision you have to make.

### Project scope (recommended)

The skill lives inside one repo and works only there. It travels with the
repo, so anyone who clones it gets the same skill.

```bash
cd <your-repo>
mkdir -p .claude/skills
cp -R path/to/helpful-claude-skills/skills/conventional-commits .claude/skills/
```

Commit `.claude/skills/` if you want teammates to have it. Add it to
`.gitignore` if it is just for you.

### Personal scope

The skill is available in every project on your machine.

```bash
mkdir -p ~/.claude/skills
cp -R path/to/helpful-claude-skills/skills/conventional-commits ~/.claude/skills/
```

Use this only for skills you genuinely want everywhere. Personal skills load
in every session whether the work is relevant or not.

### Or clone and copy

```bash
git clone https://github.com/<your-username>/helpful-claude-skills.git
cd helpful-claude-skills
cp -R skills/conventional-commits ~/path/to/target/.claude/skills/
```

## Verify it loaded

Start a Claude Code session and run:

```
/skills
```

The skill should be listed. If it is not, check three things: the folder is
directly under `.claude/skills/`, the file is named `SKILL.md` exactly, and
the YAML frontmatter at the top parses (opening and closing `---` lines).

If you created the `.claude/skills/` directory while a session was already
running, restart the session once.

## How skills actually trigger

Two ways:

1. **Automatically.** Claude reads every installed skill's `description` and
   loads the skill when your request matches. You never type its name.
2. **Manually.** Type `/skill-name` to force it.

The `description` field is the whole triggering mechanism. If a skill in this
repo is not firing when you expect it to, that is a bug worth reporting.

## A note on symlinks

You can symlink a skill folder into a repo instead of copying it, and Claude
Code will follow the link. Useful if you maintain one copy and want edits to
propagate.

The catch: git commits the symlink, not the file. Anyone else who clones the
repo gets a pointer to a path that does not exist on their machine, and cloud
sessions break the same way. Symlink for yourself, copy for your team.

## Scope and limitations

- These skills work in **Claude Code** and **Claude Desktop**, which read the
  same `~/.claude/skills/` path.
- Cowork and cloud sessions do not read `~/.claude/skills/` from your local
  machine. For those, either commit the skill into the repo or enable it on
  your claude.ai account.
- Some skills read from other agents that use the same `SKILL.md` format.
  That is untested here; no promises.

## Contributing

New skills are welcome. Please keep to the shape of what is already here:

- One folder per skill, kebab-case, named after the skill.
- `SKILL.md` is required. `README.md` in the skill folder is strongly
  encouraged (explain what it does for a human, not for Claude).
- The `description` frontmatter is third person, states what the skill does
  AND when to trigger it, and includes concrete trigger phrases.
- No external dependencies unless there is no way around it. If a skill
  needs a binary or a package, say so at the top of its README.
- No telemetry, no network calls, no data collection. Ever.
- Keep `SKILL.md` under roughly 500 lines. Longer material goes in a
  `reference/` subfolder that `SKILL.md` points at.

Open an issue first if you want to discuss an idea before building it.

## License

MIT. Use these however you like, including commercially. Attribution
appreciated but not required.
