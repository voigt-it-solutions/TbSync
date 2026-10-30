This checkout is a fork used only for compatibility builds of TbSync and
EAS-4-TbSync. The process is in COMPAT.md:

@COMPAT.md

When working on a build:

- Never commit to `Main` (or `master` in EAS-4-TbSync); it mirrors upstream.
- Review every upstream commit before taking it, and record it in the table
  in COMPAT.md. Do not include EAS-4-TbSync PR #370.
- Never edit files under `src/vendor/`; they come from `common/` via
  `common/vendor.sh`.
- Pushing, tagging and publishing are outward-facing: ask first.
