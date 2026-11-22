# Personal Agent Skills

个人全局 skills 仓库，作为唯一事实源单向同步到各 agent harness。

## 同步命令

新设备首次安装或日常全量同步（可重复执行，新设备无需 clone 本仓库）：

```bash
npx skills@latest add Barrowmann/skills -g --skill '*' -y \
  -a claude-code codex cursor antigravity antigravity-cli gemini-cli github-copilot \
     opencode amp cline deepagents kimi-code-cli warp zed
```

> 要求 Node.js >= 22.20（`skills@1.9.0` 要求）。私有仓库会自动使用本机已配置的 Git 凭据、`gh` CLI 或 SSH。

## 日常操作

| 场景 | 命令 / 操作 |
|---|---|
| 新设备初始化 | 运行上面的同步命令 |
| 仓库修改了 skill | 运行同步命令，或 `npx skills update -g` |
| 仓库新增了 skill | 运行同步命令（`update` 只更新已安装的 skill） |
| 仓库删除了 skill | 交互式运行 `npx skills update -g` 按提示清理，或 `npx skills remove -g <name>` |
| 编辑 skill | 仅在仓库 clone 中修改并 push，再同步到各设备 |
| 升级第三方 skill | 按 [AGENTS.md](./AGENTS.md) 流程手动由 agent 更新并同步 `UPSTREAM.json` |

## 仓库结构

- `skills/<name>/`：全部 skill 副本（自写、原样第三方、改编第三方）
- `UPSTREAM.json`：第三方 skill 上游来源与版本记录
- `AGENTS.md`：仓库维护与上游升级说明
