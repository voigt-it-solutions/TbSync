---
name: compat-commit
description: Commit in the voigt-it-solutions TbSync or EAS-4-TbSync compatibility fork, following its Conventional Commits rules. Use for every commit in either repository, whenever changes are committed or the user asks to commit.
allowed-tools: Bash
---

# Committing in the compatibility forks

This file is mirrored in both forks, `TbSync` and `EAS-4-TbSync`, under
`.claude/skills/compat-commit/`. The two copies must stay identical: change
one, copy it to the other, and commit both.

## Which commits this applies to

Only our own commits on the `compat` branch. Two kinds of commit are never
reworded, amended or squashed:

- **Commits taken from upstream pull requests.** COMPAT.md identifies them by
  hash, and rewording changes the hash. They keep their author and message.
- **Commits already pushed.** The history is public and release tags point
  into it. Check with `git log origin/compat..compat` before amending
  anything; only what it lists may be changed.

Never commit to `Main` (TbSync) or `master` (EAS-4-TbSync); they mirror
upstream.

## Format

[Conventional Commits 1.0](https://www.conventionalcommits.org/en/v1.0.0/):

    <type>(<scope>): <description>

    <body>

| type | used for |
| --- | --- |
| `build` | versions and manifest changes; scope `release` for a build's version bump |
| `chore` | the update feed `compat-updates.json`; scope `updates` |
| `ci` | the release workflow |
| `docs` | COMPAT.md, CLAUDE.md, README, this skill |
| `fix` | a compatibility fix of our own, only if one is ever needed instead of an upstream one |

Nothing else occurs in these forks: there are no features (`feat`), and the
code is upstream's, so no `refactor`, `perf` or `style` either. A change
that seems to need one of those does not belong in this fork.

- Description: lower case, imperative, no full stop, at most 72 characters.
- Body: why the change is made, wrapped at 72 columns. Leave it out only
  when the description already says everything.
- A change touching both repos gets one commit in each, with the same
  subject.

Examples:

    build(release): 5.3.8.5 / 5.3.11.5 for Thunderbird 160
    chore(updates): offer 5.3.8.5 / 5.3.11.5
    ci: pin the timezone the tests run in
    docs: describe the checks before a release is offered

## Steps

1. `git status` and `git diff` (or `git diff --staged`): see what is
   actually changing.
2. Stage files by name. Never stage `dist/`, xpis, or anything under
   `.claude/` other than `.claude/skills/`.
3. One logical change per commit. A version bump and the update feed that
   offers it are separate commits, because the feed is pushed only after the
   release exists (see COMPAT.md).
4. Commit with a heredoc so the body keeps its line breaks:

       git commit -m "$(cat <<'EOF'
       docs: describe the checks before a release is offered

       Why the change is made.
       EOF
       )"

5. Confirm the signature: `git log -1 --format='%G? %s'` must show `G`.
   Commits are GPG-signed by the repository's git configuration; never pass
   `--no-gpg-sign` and never change the git configuration.

Committing is local. Pushing is a separate step, done only when the user
asks for it.
