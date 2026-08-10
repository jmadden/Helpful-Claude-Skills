# conventional-commits

Turns a diff, staged changes, or a plain description of your work into one
spec-compliant [Conventional Commits 1.0.0](https://www.conventionalcommits.org/en/v1.0.0/)
message.

## Why this exists

Most agents can produce something that looks like a Conventional Commit. The
failure modes are consistent and annoying:

- Writing `feat: add search and fix the login redirect` for a diff that should
  have been two commits.
- Calling a behavior fix a `refactor` because the diff reads like restructuring.
- Inventing `Closes #142` when no issue number appears anywhere.
- Using the body to narrate the diff in prose, which the diff already shows.
- Defaulting to `chore` for anything the agent cannot classify.
- Producing a message that the repo's own commitlint config then rejects.

This skill closes each of those specifically.

## What it does

**Checks atomicity first.** Before typing a message, it decides whether the
diff is one logical change. If it is not, it refuses to write a combined
message, lists the distinct changes it sees, and proposes a message for each.

**Applies a type precedence rule.** When a change plausibly fits multiple
types, user-facing behavior wins: `feat` > `fix` > `perf` > `refactor` >
`build`/`ci` > `docs`/`style`/`test`/`chore`. This is the single biggest
source of wrong types.

**Uses the imperative test.** The description must complete the sentence
"If applied, this commit will ___". More reliable than telling an agent to
"use present tense", which it routinely misreads.

**Never invents facts.** No fabricated issue numbers, ticket IDs, or
co-authors. If the reference does not appear in the diff, branch name, or your
description, the footer is omitted.

**Reads your commitlint config.** If `commitlint.config.*`, `.commitlintrc*`,
or a semantic-release config is present in the repo, those rules override the
skill's defaults. The goal is zero CI rejections.

**Covers the parts most prompts skip.** `revert` type with the SHA footer,
git trailer syntax for footers, `BREAKING-CHANGE` as an accepted hyphenated
synonym, and the fact that a breaking change can accompany any type, not just
`feat`.

## Install

Project scope, one repo:

```bash
cd <your-repo>
mkdir -p .claude/skills
cp -R conventional-commits .claude/skills/
```

Personal scope, every project on your machine:

```bash
mkdir -p ~/.claude/skills
cp -R conventional-commits ~/.claude/skills/
```

Verify with `/skills` in a Claude Code session.

## Use

It triggers on its own. Any of these will do it:

- "write me a commit message"
- "commit this"
- pasting a diff and asking what it should say
- "reword my last commit"

Or force it with `/conventional-commits`.

## Example output

Standard feature:

```
feat(auth): add email validation to registration
```

Fix with body and footer:

```
fix(reviews): handle null review list in product render

The renderer crashed with an uncaught null pointer when a product
had no reviews. Default to an empty array so the empty state
renders instead.

Fixes #482
```

Breaking change:

```
feat(api)!: drop support for v1 endpoints

BREAKING CHANGE: The /api/v1 routes have been removed. All clients
must migrate to /api/v2. See docs/migration-v2.md for the mapping
of old to new endpoints.
```

Non-atomic diff, where it flags instead of committing:

> This diff contains two unrelated changes: a new CSV export feature in
> `reports/` and a fix for the session timeout in `auth/`. These should be
> separate commits:
>
> 1. `feat(reports): add CSV export for monthly summaries`
> 2. `fix(auth): extend session timeout to honor remember-me flag`
>
> Split with `git add -p`, or tell me to proceed with one combined message
> anyway.

## Configuring it for your repo

Two optional additions, both worth the two minutes.

### Restrict the scopes

Free-form scopes drift. Within a month you have `auth`, `authentication`, and
`login` all meaning the same thing. Add an allowed list to your repo's
`CLAUDE.md`:

```markdown
## Commits

Conventional Commits. Allowed scopes:

`api` · `auth` · `ui` · `db` · `ci` · `docs`
```

The skill reads it and stops guessing. If no scope fits, it omits the scope
rather than inventing one.

### Add repo-specific rules

Anything unusual about how your repo commits belongs next to the scope list.
For example, if a version-manifest bump always ships with the change that
required it:

```markdown
- A `plugin.json` version bump that accompanies a skill edit belongs in
  the SAME commit, not a separate `chore`.
```

Without a rule like that, the atomicity check will try to split the two.

## House rules that are not in the spec

Two things this skill enforces that Conventional Commits 1.0.0 itself does
not specify. Both are labeled as house rules inside `SKILL.md`, so you can
change them.

**Header length: 72 characters.** The spec sets no limit at all. The widely
cited 50-character rule comes from general git convention, not from
Conventional Commits, and it is brutal once you add a scope. 72 matches the
common commitlint default. If your config says otherwise, your config wins.

**Both `!` and the footer for significant breaks.** The spec accepts either
one alone. Using both is more visible to humans skimming a log.

## Requirements

None. Single file, no dependencies, no network access, no scripts.

## Feedback

If the skill produces a wrong type, invents a reference, or fails to trigger
when it should, open an issue with the diff (or a redacted version of it) and
what it produced. Triggering failures are the most useful kind of report.
