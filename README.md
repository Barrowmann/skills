# Personal agent skills

This repository is for personal use. It is the single source of truth for syncing global skills across devices and agent harnesses.

## Sync command

Run this command on a new device or to sync all skills. You do not need to clone this repository on a new device.

```bash
npx skills@latest add Barrowmann/skills -g --skill '*' -y \
  -a claude-code codex cursor antigravity antigravity-cli gemini-cli github-copilot \
     opencode amp cline deepagents kimi-code-cli warp zed
```

`skills@1.9.0` requires Node.js 22.20 or later. Private repository access uses your configured Git credentials, the `gh` CLI, or SSH.

## Common tasks

| Task | Action |
|---|---|
| Set up a new device | Run the sync command above. |
| Pull edits to installed skills | Run the sync command, or run `npx skills update -g`. |
| Pull newly added skills | Run the sync command. `npx skills update -g` only updates already installed skills. |
| Remove skills deleted from this repo | Run `npx skills update -g` interactively and confirm deletion, or run `npx skills remove -g <name>`. |
| Edit a skill | Edit the file in a local clone of this repository, push the commit, then sync each device. |
| Update third-party skills | Follow [AGENTS.md](./AGENTS.md) and update `UPSTREAM.json`. |

## Repository layout

- `skills/<name>/` holds a full copy of each skill. It contains personal skills, unmodified third-party skills, and locally adapted skills.
- `UPSTREAM.json` records the upstream repository, path, version, and modification status for each third-party skill.
- `AGENTS.md` defines the maintenance rules and the manual upgrade process for third-party skills.
