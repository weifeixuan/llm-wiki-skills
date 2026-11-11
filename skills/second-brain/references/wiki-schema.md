# Wiki Schema（Wiki 规范）

LLM 维护的知识库 wiki 的权威规则。这是唯一事实来源 —— agent 配置模板都从这里取材。

## 架构

三个目录，三种角色：

- **raw-原始资料/** —— 不可变的源文档。LLM 从这里读取，但绝不修改这些文件。
- **wiki-知识库/** —— LLM 的工作区。在这里创建、更新和维护所有文件。
- **output-输出/** —— 报告、查询结果和生成的产物放在这里。

Wiki 子目录：
- `wiki-知识库/sources-来源/` —— 每份已摄取资料对应一份摘要页
- `wiki-知识库/entities-实体/` —— 人物、组织、产品、工具的页面
- `wiki-知识库/concepts-概念/` —— 想法、框架、理论、模式的页面
- `wiki-知识库/synthesis-综合/` —— 对比、分析、横切主题

两个特殊文件：
- `wiki-知识库/index.md` —— 所有 wiki 页面的总目录，按类别组织。每次摄取都要更新。
- `wiki-知识库/log.md` —— 只追加的时间顺序记录。绝不修改已有条目。

## 页面格式

每个 wiki 页面必须包含 YAML frontmatter：

    ---
    tags: [tag1, tag2]
    sources: [source-filename-1.md, source-filename-2.md]
    created: YYYY-MM-DD
    updated: YYYY-MM-DD
    ---

所有内部链接使用 `[[wikilink]]` 语法。当你提到一个拥有自己页面的概念、实体或资料时，就链接它。

## 操作

### 摄取（处理新资料）

当用户把文件添加到 raw-原始资料/ 并要求处理时：

1. 完整阅读资料
2. 与用户讨论核心要点
3. 在 `wiki-知识库/sources-来源/` 创建资料摘要页，包含：标题、资料元数据、关键论断和结构化摘要
4. 识别资料中提到的所有实体和概念。对每一个：
   - 如果 wiki 页面已存在：用这份资料的新信息更新它，并注明来源
   - 如果 wiki 页面不存在：在相应的子目录创建一个
5. 在所有相关页面之间添加 `[[wikilink]]`
6. 把新页面更新进 `wiki-知识库/index.md`
7. 追加到 `wiki-知识库/log.md`：`## [YYYY-MM-DD] ingest | 资料标题`

一份资料可能触及 10-15 个 wiki 页面。这是正常的。

### 查询（回答问题）

当用户提出问题时：

1. 阅读 `wiki-知识库/index.md` 找到相关页面
2. 阅读相关的 wiki 页面
3. 综合出答案，用 `[[wikilink]]` 引用 wiki 页面
4. 如果答案产生了有价值的产物（对比、分析、新关联），主动提出把它保存为 `wiki-知识库/synthesis-综合/` 中的新页面
5. 如果保存了新页面，更新索引和日志

### 体检（健康检查）

当用户要求对 wiki 做 lint 或健康检查时：

1. 扫描页面之间的矛盾
2. 找出已被更新的资料取代的过时论断
3. 识别孤儿页面（没有入链）
4. 找出被提及但还没有自己页面的重要概念
5. 检查缺失的交叉引用
6. 提出可以通过网络搜索填补的数据缺口
7. 报告发现并提供修复选项
8. 记录本次体检：`## [YYYY-MM-DD] lint | 发现摘要`

## 索引格式

`wiki-知识库/index.md` 中每条记录占一行：

    - [[页面名称]] —— 一行摘要

按类别标题组织：来源（Sources）、实体（Entities）、概念（Concepts）、综合（Synthesis）。

## 日志格式

`wiki-知识库/log.md` 中每条记录：

    ## [YYYY-MM-DD] 操作名 | 标题
    所做事项的简要描述。

## 页面命名

文件名使用 **kebab-case**（小写短横线），带 `.md` 扩展名。文件内的页面标题使用 **Title Case**。

- 资料页：`wiki-知识库/sources-来源/article-title-here.md` → `# Article Title Here`
- 实体页：`wiki-知识库/entities-实体/entity-name.md` → `# Entity Name`
- 概念页：`wiki-知识库/concepts-概念/concept-name.md` → `# Concept Name`
- 综合页：`wiki-知识库/synthesis-综合/comparison-topic.md` → `# Comparison Topic`

创建 `[[wikilink]]` 时，使用页面标题（Title Case），而不是文件名：
- 正确：`[[Entity Name]]`
- 错误：`[[entity-name]]`

把标题转成文件名的方法（slugify）：转小写、空格替换为短横线、移除特殊字符、截断到合理长度。

## 图片处理

网页剪藏的文章通常包含图片。按以下方式处理：

1. **把图片下载到本地。** 在 Obsidian 设置 Settings → Files and links 中，把"Attachment folder path"设为 `raw-原始资料/assets-附件/`。剪藏文章后使用"为当前文件下载附件"（建议绑定快捷键，如 Ctrl+Shift+D）。
2. **wiki 页面引用图片** 使用标准 markdown：`![描述](../raw-原始资料/assets-附件/image-name.png)`。图片保留在 `raw-原始资料/assets-附件/` —— 绝不把图片复制进 `wiki-知识库/`。
3. **摄取时**，注意资料中的图片。如果某张图片包含重要信息（图表、示意图、数据），在 wiki 页面中用文字描述其内容，确保知识以文本形式留存。

## 体检频率

按以下节奏运行体检（`/second-brain-lint`）：
- **每摄取 10 份资料后** —— 趁记忆新鲜时发现交叉引用的缺口
- **至少每月一次** —— 发现随时间积累的过时论断和孤儿页面
- **任何大型查询或综合分析之前** —— 确保依赖 wiki 做分析之前它是健康的

## 规则

1. 绝不修改 `raw-原始资料/` 中的文件。它们是不可变的源材料。
2. 创建或删除页面时，始终更新 `wiki-知识库/index.md`。
3. 执行操作时，始终在 `wiki-知识库/log.md` 追加记录。
4. 所有内部引用使用 `[[wikilink]]`。绝不在页面内容中使用原始文件路径。
5. 每个 wiki 页面必须有包含 tags、sources、created、updated 字段的 YAML frontmatter。
6. 当新信息与现有 wiki 内容矛盾时，更新 wiki 页面并标注矛盾，同时引用双方资料。
7. 资料摘要页保持客观事实。解释和综合留给概念页和综合页。
8. 被问到问题时，先搜索 wiki。只有 wiki 没有答案时才去看原始资料。
9. 优先更新已有页面，而不是创建新页面。只有当主题足够独立时才创建新页面。
10. 保持 `wiki-知识库/index.md` 简洁 —— 每页一行，每条不超过 120 字符。
