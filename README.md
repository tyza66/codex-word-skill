# Codex Word Skill

一个将"阅读 Word 文档"与"精确修改 Word 内容和格式"两项能力统一封装的 Codex skill。

## 功能

- **读取** `.docx`：提取并返回结构化文本（摘要、问答、定向查找）。
- **精确编辑**：修改措辞、调整结构、修复样式/标题/表格/页眉页脚/间距等细节，并通过渲染验证确保结果无缺陷。
- **内置工具与示例**：附带通用版 Python 脚本，可直接用于渲染验证、文档审计和精确表格编辑。

## 结构

```text
codex-word-skill/
|-- SKILL.md                       入口：能力边界、工具契约、黄金路径
|-- agents/
|   `-- openai.yaml                UI 元数据
|-- scripts/
|   |-- render_docx.py            通用版 DOCX 渲染器（LibreOffice headless，输出 page-N.png）
|   |-- word_audit.py             通用版文档审计器（标题层级、编号、直接格式覆盖、字体使用）
|   `-- table_geometry.py        通用版表格几何辅助（tblW/tblGrid/tcW 同步写精确列宽）
|-- examples/
|   |-- read_content.py           读取并转储段落、表格、页眉页脚、节几何
|   |-- edit_content.py           跨 run 和表格单元格的精确查找替换
|   |-- edit_tables.py            按内容权重设置列宽、单元格边距和可见边框
|   `-- tracked_changes.py        通过原始 OOXML 补丁添加修订标记
`-- references/
    |-- content-precision.md      内容精度检查清单（措辞、术语、数字、交叉引用）
    `-- format-precision.md       格式精度检查清单（间距、边框、对齐、表格布局）
```

## 工作原理

该 skill 是编排层，不重复实现底层能力，而是路由到环境中已有的两个 skill：

- **读取 Word 内容**：通过 `read-documents` skill 路由到 `documents` skill。
- **精确编辑内容与格式**：通过 `documents` skill，使用 `python-docx` 处理常规段落/样式/表格，OOXML 补丁处理批注、修订、超链接、域等高级能力。

## 使用方式

在 Codex 中用自然语言描述 Word 相关需求即可，例如：

- "帮我读一下这个 docx"
- "把第三章的标题改成黑体"
- "这个表格的列宽和边框调一下"

skill 会自动命中并按黄金路径执行：先读取掌握上下文，再编辑，然后渲染为 PNG 逐页检查，循环至无缺陷后交付最终 `.docx`。

脚本与示例可直接在 Python 环境中运行（建议通过 `load_workspace_dependencies` 解析运行时）：

```bash
# 渲染验证
python scripts/render_docx.py input.docx --output_dir out/

# 编辑前审计
python scripts/word_audit.py input.docx

# 精确表格编辑
python examples/edit_tables.py input.docx output.docx
```

## 适用边界

- 适用于 `.docx` 的内容阅读与精确编辑。
- 不适用于创建电子表格、幻灯片、PDF（分别使用 `spreadsheets`、`presentations`、`pdf` skill）。
- 不适用于手绘或插图类素材。

## 运行需要

通过 `load_workspace_dependencies` 获取 Node/Python 运行时，不依赖系统全局安装。

## 许可证

MIT
