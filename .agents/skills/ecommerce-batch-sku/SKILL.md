---
name: ecommerce-batch-sku
description: >-
  把 1-20 张不同 SKU 商品图逐一套入同一个模板，先用内置 gpt-image-2 模块为每个 SKU 优化提示词并请求客户校验，保持整批风格一致，并支持一一对应的商品名称、型号、尺寸或规格文本。用户提到批量 SKU、同模板换产品、批量更新型号名称、统一商品卡片时使用。
---

## 仓库级 Codex 兼容说明

- 本技能位于 `.agents/skills/`。以本 `SKILL.md` 所在目录为技能根目录；下文所有 `scripts/` 和 `references/` 路径均相对此目录，执行命令前切换到此目录，包含空格的路径须加引号。
- 公共文档在本目录的 `references/`：`project-schema.md`、`gpt-image-2-prompt-optimizer.md`、`prompt-guidelines.md`、`quality-checklist.md`；自定分批另读 `custom-batch-presets.md`。首次执行工作流前读取相关文档。
- Python 图片处理需要 Pillow。先检查现有 Python 环境；不要自动安装依赖。当前云环境可使用 `/workspace/.venvs/qiyu-image-workflow/bin/python`（若存在），或激活该环境后运行下文命令。
- 真正出图前先检查当前会话是否提供图片生成/编辑工具；工具不可用时再检查 `codex-imagegen`。两者均无时停止在提示词与计划阶段，明确报告无法出图，不安装未知 CLI，不以占位图冒充结果。
- 文档中的 `gpt-image-2`、`high`、`2K` 是工作流期望值，不代表当前工具支持这些选项；仅传入工具 schema 实际支持的参数，若无法满足用户要求先说明限制。目录配置不会提供模型、API 权限或额度。
- 保持用户确认提示词及完整计划的要求；未经确认不调用生成工具。不得自动下载依赖、调用外部付费 API 或记录密钥。

# 电商批量 SKU

保持内联执行，不使用子代理。

## 输入与硬性映射

这是文件夹批处理工作流。先获取项目名称，运行：

```bash
python3 scripts/prepare_inputs.py batch-sku --workspace <workspace> --project-slug <slug>
```

创建素材投递目录和说明文件。用户一次放入 1-20 张商品图、正好 1 张模板和可选映射表，不逐张上传。用户回复“已放好”后运行：

```bash
python3 scripts/scan_inputs.py batch-sku <input-dir> --out <project-dir>/work/input-report.json
```

SKU 文本可不提供；一旦提供，使用 `sku-mapping.csv` 按 `product_file` 文件名明确匹配，不依赖文件顺序。不要从文件名猜测认证、性能、尺寸或规格。

每个任务中 `image 1` 是当前商品，`image 2` 是统一模板。只替换模板里的商品身份，并按当前任务的 SKU 文本更新明确列出的字段；背景、构图、字体层级和视觉风格保持一致。

## 执行

1. 定位当前 `SKILL.md` 所在的技能根目录并读取公共参考，包括 `gpt-image-2-prompt-optimizer.md`。
2. 创建素材文件夹并报告绝对路径。用户回复“已放好”后运行扫描器，集中检查图片格式、数量、模板数量、CSV 文件名映射和重复文件。
3. 创建 `batch-sku` 项目，在 `workflow_config` 中记录模板 ID 和按明确文件名映射的 SKU 字段。
4. 每个商品创建一个任务，`asset_ids` 严格为当前商品、模板；`original_request` 保存共用替换要求和该商品对应的 SKU 文本。
5. 先展示“商品文件 → SKU 字段 → 输出文件”映射表并请求确认，再优化一份全批次共用提示词。共用提示词批准后为每个 SKU 注入对应字段；只有特殊 SKU 才单独校验提示词。
6. 运行公共校验器。文本映射错误或任一必要提示词未批准时停止，不做模糊匹配。
7. 生成前展示参数和目录，等待完整计划确认。
8. 批准后使用内置图片能力或公共 CLI 执行器。
9. 逐项检查商品身份和对应文本，只重画错误 SKU，不重跑整批。
