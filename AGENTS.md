# 维护规则

- 本仓库是全局 skills 的唯一事实源。所有新增、修改、删除操作只在仓库的 `skills/` 目录中进行，不要直接编辑 `~/.agents/skills` 或任何 agent harness 的 skills 目录。
- 所有 skill 副本平铺存放在 `skills/<name>/` 下，目录名与 `SKILL.md` frontmatter 中的 `name` 保持一致。
- `UPSTREAM.json` 记录所有第三方 skill 对应的上游仓库、路径、版本及是否做过本地修改。自写 skill 不写入 `UPSTREAM.json`。
- 新增、删除第三方 skill 或修改本地适配内容时，同步更新 `UPSTREAM.json`。

# 升级第三方 skill（手动发起）

当用户要求更新某个或某批第三方 skill 时，按以下步骤执行：

1. 读取 `UPSTREAM.json`，获取目标 skill 的 `repo`、`path`、`version` 和 `modified`。
2. 在临时目录 clone 上游仓库，分别取出旧 `version` 和目标版本的 `path` 目录内容。
3. 若 `modified` 为 `false`：先确认本地与旧版本无未记录差异，再用目标版本整体替换 `skills/<name>/`。
4. 若 `modified` 为 `true`：以旧 `version` 为 base、目标版本为 theirs、当前仓库副本为 ours 做三方合并（`git merge-file`），保留本地改动；遇到冲突时停下来与用户确认。
5. 更新 `UPSTREAM.json` 中的 `version`（必要时更新 `note`），检查 `git diff` 无误后提交。
