This checkout is a fork used only for compatibility builds of TbSync and
EAS-4-TbSync. The process is in COMPAT.md:

@COMPAT.md

When working on a build:

- Never commit to `Main` (or `master` in EAS-4-TbSync); it mirrors upstream.
- Review every upstream commit before taking it, and record it in the table
  in COMPAT.md. Do not include EAS-4-TbSync PR #370.
- Never edit files under `src/vendor/`; they come from `common/` via
  `common/vendor.sh`.

## Finding the compatibility fixes

A new Thunderbird release usually shows up upstream as an issue titled like
"TbSync is incompatible with Thunderbird <n>", and fixes arrive as pull
requests in both repos. So far each new pull request has replaced the
previous one and contains its commits, so only the commits on top need
reviewing. List them without `gh`:

    curl -s "https://api.github.com/repos/jobisoft/TbSync/pulls?state=all&per_page=10"
    curl -s "https://api.github.com/repos/jobisoft/EAS-4-TbSync/pulls?state=all&per_page=10"

Take the commits from the pull request itself
(`git fetch upstream pull/<n>/head`), never from a ready-made package
attached to it: those have included the excluded PR #370.

## Tests

Run them with `TZ=Europe/Berlin`. Three upstream EAS-4-TbSync tests assume
that timezone and fail under UTC, on the unmodified upstream release as
well; the release workflow sets it for the same reason.

    TZ=Europe/Berlin npm test                       # TbSync
    TZ=Europe/Berlin node --test ...                # EAS-4-TbSync: copy the
                                                    # command from compat-release.yml

## Checking a release before offering it

`compat-updates.json` changes only after both releases exist and pass these
checks:

1. Each release holds the xpi and `SHA256SUMS.txt`, and the checksum matches
   the downloaded xpi.
2. The attestation for that checksum names `compat-release.yml` and the
   commit the tag points to. Without a logged-in `gh`, read it from
   `https://api.github.com/repos/voigt-it-solutions/<repo>/attestations/sha256:<digest>`
   (the DSSE payload is base64 JSON); with one, `gh attestation verify`.
3. The unpacked xpi equals `git archive <tag> src`.
4. The manifest in the xpi has the expected `version`, `update_url` and
   `homepage_url`.
5. Every `update_link` in the new `compat-updates.json` answers a HEAD
   request with 200 before it is pushed.

## When a release workflow fails

The tag is pushed but no release exists. Fix the cause in a new commit on
`compat`, push it, move the tag to that commit locally
(`git tag -d`, then `git tag -s` again) and replace it with
`git push --force origin <tag>`. Replacing a tag is only acceptable while no
release exists for it; once one does, release the next version instead.

## Working with the user

- Commit with the `compat-commit` skill (`.claude/skills/compat-commit/`,
  mirrored in both repos). It holds the commit conventions and says which
  commits must never be reworded.
- Commits and tags are GPG-signed automatically; keep it that way.
- Push branches and tags only when the user asks for it.
- Force-pushes are left to the user: give the exact command instead.
