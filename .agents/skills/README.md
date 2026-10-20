# 电商工作流：仓库级 Codex Skills

这里提供九种工作流及 `ecommerce-image-workflow` 总入口。每个技能使用
`<skill-name>/SKILL.md`，包含 `name`、`description` 元数据，并自带脚本、
参考文档和 MIT 许可证。它们不依赖插件安装或插件命名空间。
原始 `ecommerce-image-workflow 2/` 保留不变；更新原版时需要同步这些副本。

## 在网页版 Codex 调用一比一复刻

将本目录随仓库更改提交并推送到 GitHub，在网页版 Codex 选择包含该更改的
仓库和分支，创建新任务，使技能重新被发现。输入 `$` 选择
`ecommerce-exact-replication`，或在提示词中明确写：

```text
使用 $ecommerce-exact-replication 一比一复刻。
第 1 张是构图参考图，第 2 张起是我的同一商品事实图。
保留参考图构图和光影，替换成我的商品，不继承参考品牌和文案。
先展示图片职责与优化提示词，等我确认提示词和完整计划再生成。
```

在同一消息中附上 1 张参考图和最多 4 张同一商品的事实图。
若当前界面未列出技能，可要求读取
`.agents/skills/ecommerce-exact-replication/SKILL.md` 并遵循其流程；
当前已开始的会话不保证自动刷新技能列表。

也可使用 `$ecommerce-image-workflow`，然后选择第 5 项。

## 图片能力和验证范围

Skill 是操作说明，不会自动授予生图能力。必须以当前会话实际提供的
图片生成/编辑工具为准；没有内置工具时，需要已有且可用的 `codex-imagegen`。
二者都没有时，只能完成分析、提示词和计划。不要自行安装不明 CLI 或调用付费 API。

本次会话提供图片生成/编辑工具，但未实际调用；因此未验证生成成功、额度、
指定模型或 2K 输出。当前机器未发现 `codex-imagegen`。

配置验证通过：10 个技能的元数据、引用与路由、Python 语法及 CLI 帮助；
原有 27 项测试通过。测试只涉及本地文件处理、dry-run 及模拟 provider，
不证明真实生图可用。未安装新依赖，未调用付费 API，未生成图片。

官方规范参考：https://developers.openai.com/codex/skills/
本次访问该页面返回 HTTP 403；OpenAI 官方仓库 `docs/skills.md` 指向此地址。
