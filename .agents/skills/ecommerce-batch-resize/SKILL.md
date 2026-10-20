---
name: ecommerce-batch-resize
description: >-
  使用预制素材文件夹批量读取 1-20 张原图，并将每张图智能重排到一个或多个目标画幅。用户提到批量改尺寸、多张图片统一适配比例、批量横转竖或平台尺寸适配时使用。单张图片调整和复刻风格不要使用本技能。
---

## 仓库级 Codex 兼容说明

- 本技能位于 `.agents/skills/`。以本 `SKILL.md` 所在目录为技能根目录；下文所有 `scripts/` 和 `references/` 路径均相对此目录，执行命令前切换到此目录，包含空格的路径须加引号。
- 公共文档在本目录的 `references/`：`project-schema.md`、`gpt-image-2-prompt-optimizer.md`、`prompt-guidelines.md`、`quality-checklist.md`；自定分批另读 `custom-batch-presets.md`。首次执行工作流前读取相关文档。
- Python 图片处理需要 Pillow。先检查现有 Python 环境；不要自动安装依赖。当前云环境可使用 `/workspace/.venvs/qiyu-image-workflow/bin/python`（若存在），或激活该环境后运行下文命令。
- 真正出图前先检查当前会话是否提供图片生成/编辑工具；工具不可用时再检查 `codex-imagegen`。两者均无时停止在提示词与计划阶段，明确报告无法出图，不安装未知 CLI，不以占位图冒充结果。
- 文档中的 `gpt-image-2`、`high`、`2K` 是工作流期望值，不代表当前工具支持这些选项；仅传入工具 schema 实际支持的参数，若无法满足用户要求先说明限制。目录配置不会提供模型、API 权限或额度。
- 保持用户确认提示词及完整计划的要求；未经确认不调用生成工具。不得自动下载依赖、调用外部付费 API 或记录密钥。

# 电商图片批量改尺寸

保持内联执行，不使用子代理。这是文件夹批处理工作流。

## 素材投递

先获取项目名称，再运行：

```bash
python3 scripts/prepare_inputs.py batch-resize --workspace <workspace> --project-slug <slug>
```

报告创建目录的绝对路径。目录内的 `请先阅读.md` 会要求用户把 1-20 张图片放入 `01-待处理原图/`，并在 `02-目标尺寸.txt` 中每行填写一个画幅。用户回复“已放好”后运行扫描器。

## 工作流

1. 定位当前 `SKILL.md` 所在的技能根目录并读取公共 `gpt-image-2-prompt-optimizer.md`、项目 schema 和质检规则。
2. 扫描支持的图片、自然排序、目标画幅和重复文件；集中报告问题。
3. 分析每张图的主体、文字安全区、边缘风险和可扩展背景。
4. 创建 `batch-resize` 项目，把每张原图登记为 `source`，在 `workflow_config.targets` 保存去重后的目标画幅。
5. 为“每张原图 × 每个目标画幅”建立独立任务，按原图顺序再按目标画幅顺序排列；每个任务只引用一张 source，保持主体、文字和内容不变，只智能重排。
6. 展示输入图、目标画幅、重排策略和输出名映射，请客户确认。
7. 用内置优化器生成批量改尺寸提示词；同一重排策略可批量批准，异常图单独校验。
8. 生成后检查拉伸、白边、关键文字裁切、主体缺失和画幅尺寸，只重做失败任务。
