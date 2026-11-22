# Maintenance rules

- This repository is for personal use and is the single source of truth for syncing global skills across devices and agent harnesses. Make all additions, edits, and deletions under `skills/` in this repository. Do not edit `~/.agents/skills` or any agent harness skills directory directly.
- Store every skill flat under `skills/<name>/`. Match the directory name to the `name` field in `SKILL.md` frontmatter.
- Record every third-party skill in `UPSTREAM.json` with its upstream repository, path, version, and modification flag. Omit personal skills from `UPSTREAM.json`.
- Update `UPSTREAM.json` whenever you add, remove, or locally adapt a third-party skill.

## Updating third-party skills

Run this process only when the user asks to update one or more third-party skills.

1. Read `UPSTREAM.json` to get `repo`, `path`, `version`, and `modified` for each target skill.
2. Clone the upstream repository into a temporary directory, then extract `path` at both the recorded `version` and the target version.
3. When `modified` is `false`, verify that the local copy has no unrecorded changes against the recorded `version`, then replace `skills/<name>/` with the target version.
4. When `modified` is `true`, run a three-way merge with `git merge-file` using the recorded `version` as base, the target version as theirs, and the current repository copy as ours. Keep local adaptations. Stop and ask the user if any conflict occurs.
5. Update `version` in `UPSTREAM.json`, update `note` if needed, review `git diff`, and commit the changes.
