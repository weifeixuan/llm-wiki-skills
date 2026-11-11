---
name: second-brain
description: >
  用 LLM Wiki 模式搭建一个新的 Obsidian 知识库。当用户想创建第二大脑（second brain）、
  初始化一个 vault、搭建个人知识库，或说出"初始化/onboard"时使用。通过交互式
  向导引导配置 vault 名称、位置和领域。
allowed-tools: Bash Read Write Glob Grep
---

# Second Brain —— 初始化向导

用 LLM Wiki 模式搭建一个新的 Obsidian 知识库。LLM 扮演图书管理员 —— 读取原始资料，把它们编译成结构化、互链的 wiki，并长期维护。

## 向导流程

引导用户完成以下 3 个步骤。每次只问一个问题。每一步都有合理的默认值 —— 用户可以接受默认值，也可以自己提供。

### 第 1 步：Vault 名称

询问：
> "你想给你的知识库起什么名字？这将作为文件夹名称。"
> 默认：`second-brain`

接受用户提供的任何名称。它将成为文件夹名称和 agent 配置中的标题。

### 第 2 步：Vault 位置

询问：
> "应该在哪里创建？给我一个路径，否则我用默认值。"
> 默认：`~/Documents/`

接受任何绝对或相对路径。把 `~` 解析为用户主目录。最终的 vault 路径是 `{位置}/{vault名称}/`。

### 第 3 步：领域 / 主题

询问：
> "这个知识库是关于什么的？这有助于我设置相关的标签并描述 vault 的用途。"
>
> 示例："AI 研究"、"金融科技创业公司的竞争情报"、"个人健康与健身"

接受自由文本。用它来：
- 为 agent 配置写一句领域描述
- 生成 5-8 个与领域相关的建议标签

## 向导之后：搭建 Vault

收集完所有答案后，按顺序执行以下步骤。**本技能不依赖任何脚本** —— 所有目录和文件由你亲自创建。

### 幂等规则（必须遵守）

- 创建任何目录或文件**之前**，先检查它是否已存在
- 已存在的文件**绝不覆盖**（尤其是 `wiki-知识库/index.md` 和 `wiki-知识库/log.md`，里面可能有用户数据）
- 已存在的目录直接跳过，不做任何改动
- 因此本搭建流程可以安全地重复运行

### 1. 创建目录结构

在 `{位置}/{vault名称}/` 下创建以下目录（已存在的跳过）：

```
raw-原始资料/assets-附件/
wiki-知识库/sources-来源/
wiki-知识库/entities-实体/
wiki-知识库/concepts-概念/
wiki-知识库/synthesis-综合/
output-输出/
```

### 2. 创建初始文件

检查以下两个文件，**不存在时才创建**，内容如下；已存在则原样保留：

`wiki-知识库/index.md`：

```markdown
# 索引

所有 wiki 页面的总目录。每次摄取时更新。

## 来源（Sources）

## 实体（Entities）

## 概念（Concepts）

## 综合（Synthesis）
```

`wiki-知识库/log.md`：

```markdown
# 日志

所有操作的时间顺序记录。
```

### 3. 生成 agent 配置文件

不询问、不检测，**固定生成一份配置文件**：`AGENTS.md`，写在 vault 根目录（已存在则跳过，遵守幂等规则）。Claude Code 2.1.277+ 原生读取 AGENTS.md，无需单独的 CLAUDE.md。

读取 `<skill目录>/references/agent-config-template.md`，按模板生成内容，替换占位符：

- `{{VAULT_NAME}}` → 第 1 步得到的 vault 名称
- `{{DOMAIN_DESCRIPTION}}` → 由第 3 步提炼的一句领域描述
- `{{DOMAIN_TAGS}}` → 根据第 3 步的领域生成 5-8 个领域相关标签，以列表形式呈现
- `{{WIKI_SCHEMA}}` → 读取 `<skill目录>/references/wiki-schema.md`，插入从 `## 架构` 开始的全部内容

配置写入 vault。如果用户明确说还用旧版 Claude Code（< 2.1.277）/ Cursor 等其他工具，再按模板中的"其他工具对照"追加生成对应文件。

### 4. 更新 wiki-知识库/log.md

追加一条设置记录：

```
## [YYYY-MM-DD] setup | Vault 初始化完成
为 {{DOMAIN_DESCRIPTION}} 创建了 vault "{{VAULT_NAME}}"。
Agent 配置：{{生成的配置文件列表}}。
```

### 5. 输出摘要

向用户展示：

1. **创建了什么** —— 目录树和配置文件
2. **必要的下一步** —— 安装 Obsidian Web Clipper 浏览器扩展：
   > 安装 Obsidian Web Clipper，轻松把网页文章保存进你的 vault：
   > https://chromewebstore.google.com/detail/obsidian-web-clipper/cnjifjpddelmedmihgijeibhnjfabmlf
3. **如何开始** —— 在 Obsidian 中打开 vault 文件夹，剪藏一篇文章到 `raw-原始资料/`，然后运行 `/second-brain-ingest`

## 参考文件

以下文件随本技能附带，位于 `<skill目录>/references/`：

- `wiki-schema.md` —— 权威 wiki 规则（所有 agent 配置的唯一事实来源）
- `agent-config-template.md` —— agent 配置文件模板（各 agent 共用）

## 后续步骤

设置完成后，用户的工作流是：

1. **剪藏文章** 到 `raw-原始资料/`（使用 Obsidian Web Clipper）
2. **摄取资料** 用 `/second-brain-ingest` —— 把原始文件处理成 wiki 页面
3. **提出问题** 用 `/second-brain-query` —— 搜索 wiki 并综合答案
4. **健康检查** 用 `/second-brain-lint` —— 每摄取 10 份资料或每月运行一次
