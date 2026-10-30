# Compatibility builds

This fork exists for one reason: to keep TbSync and EAS-4-TbSync running on
new Thunderbird releases while the upstream maintainer has not shipped an
update for them yet. It does not develop the project further. Nothing here is
a new feature, and every build is meant to be replaced by an official release as
soon as there is one.

The builds are **unofficial** and not released by the upstream maintainer.
They come **without warranty and without support** of any kind; see
sections 6 and 7 of the [Mozilla Public License 2.0](LICENSE). Please do not
report problems with them in the upstream issue trackers.

## The two repositories

Both are always built and released as a pair, because the host and its
provider share the vendored calendar experiment and the protocol version.

| fork | upstream | default branch upstream |
| --- | --- | --- |
| `voigt-it-solutions/TbSync` | `jobisoft/TbSync` | `Main` |
| `voigt-it-solutions/EAS-4-TbSync` | `jobisoft/EAS-4-TbSync` | `master` |

They are checked out side by side, which is also what `common/vendor.sh`
expects:

    voigt-it-solutions/tbsync/
    ├── TbSync/
    └── EAS-4-TbSync/

Remotes in each: `origin` is the fork, `upstream` is jobisoft.

## Branches

- `Main` / `master` mirror upstream unchanged. Never commit to them; update
  them with `git fetch upstream` and a fast-forward push.
- `compat` is what gets built: upstream's last release, the reviewed
  compatibility commits from upstream pull requests, and our own commits
  on top (version, `update_url`, `homepage_url`, release workflow, this
  file). `homepage_url` points at this fork, so the add-on manager leads to
  the notice that the build is unofficial rather than to the upstream issue
  tracker. `author` stays the upstream maintainer, who wrote the add-on.

## What goes into a build

Only reviewed compatibility fixes from upstream pull requests, taken as the
original commits so their authorship and hashes stay intact.

Current state:

| Thunderbird | TbSync | EAS-4-TbSync |
| --- | --- | --- |
| 158 | PR #824, `e7c0975f` | PR #368, `0d35ba3b` |
| 159 | PR #826, `63e8ed05` (contains #824) | PR #371, `e136b9f6` (contains #368) |

**Not included:** EAS-4-TbSync PR #370 (IMAP invitation handling). It adds
new behaviour rather than compatibility fixes and is still a draft upstream.
Ready-made packages attached to the upstream pull requests include it; the
builds here do not.

## Versions

Upstream's version plus a fourth digit that counts our builds:
`5.3.8.3`, `5.3.11.3`, and so on. Bump both repos together, even when only
one of them changed, so a pair is always recognisable.

The fourth digit keeps every build below upstream's next release (`5.3.9`,
`5.3.12`), so an official version always wins once it exists.

## Updates

Both manifests carry

    update_url: https://raw.githubusercontent.com/voigt-it-solutions/TbSync/compat/compat-updates.json

and Thunderbird polls that file instead of addons.thunderbird.net.
`compat-updates.json` lists one entry per add-on; changing it is what makes
installed copies update.

**When upstream releases:** point both entries at the official xpi (the
GitHub release asset of jobisoft's repo) instead of ours. Installed copies
then move to the official build, which has no `update_url`, and from there on
receive updates from addons.thunderbird.net again. After that this fork has
nothing left to do until the next Thunderbird release breaks compatibility.

## Release procedure for a new Thunderbird version

1. Check whether upstream has released already (releases of both repos, and
   `updates.json` on upstream `Main`). If so, do the "when upstream releases"
   step above instead.
2. Find the compatibility pull requests in both repos. Read every commit not
   yet in `compat`. Check that the diff touches nothing beyond compatibility,
   and note in the table above what was taken.
3. Bring the commits into `compat` (`git fetch upstream pull/<n>/head`, then
   cherry-pick or rebase onto them, keeping our own commit last).
4. Bump the version in `package.json`, run `node build.js` so it reaches
   `src/manifest.json`, and run the tests:
   - TbSync: `npm test`
   - EAS-4-TbSync: the command in `.github/workflows/compat-release.yml`,
     which skips the one test that also fails on the unmodified upstream
     release.
5. Update `compat-updates.json`: both versions, both links,
   `strict_max_version`.
6. Commit, push `compat` in both repos, then tag and push the tags:
   `compat-v<version>`. The tag starts `compat-release.yml`, which tests,
   builds, attests and publishes the release.
7. Push `compat-updates.json` last: it must not name a release that does
   not exist yet.

## Verifying a build

Every release asset carries a build-provenance attestation from GitHub
Actions:

    gh attestation verify tbsync_5_3_8_3_atn.xpi -R voigt-it-solutions/TbSync
    gh attestation verify eas-4-tbsync_5_3_11_3_atn.xpi -R voigt-it-solutions/EAS-4-TbSync

It proves the file was built by `compat-release.yml` in that repository, from
the tagged commit. The workflow also unpacks the xpi and checks that it holds
exactly the committed `src/`.

`build.js` stores each file's modification time, so two builds of the same
commit do not have the same hash. To rebuild and compare by hand, compare the
unpacked contents, not the hash.

The xpis are not signed by Mozilla. That is only possible through
addons.thunderbird.net, where these add-on ids belong to the upstream
maintainer, and Thunderbird does not require a signature.
