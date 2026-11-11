# Second Brain —— 蓝图

> 这是本项目的起源文档。它描述了这个模式、需求以及我们的实现方式。你可以把它当作从零构建自己版本的蓝图，也可以直接安装我们的实现（见 README.md）。

## 起源

Andrej Karpathy 发了一个帖子，讲的是用 LLM 构建个人知识库 —— 把原始资料丢进一个文件夹，让 LLM 把它们编译成结构化的 wiki，再用 Obsidian 浏览全部内容。

https://x.com/karpathy/status/2039805659525644595

几天后，他通过一份刻意保持模糊以留出创作空间的想法文件，把这个理论扩展成了一个模式。想法文件在这里：https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f ，同时本地也存了一份在 `llm-wiki.md`。

---

## 这个模式

你最终会得到三个文件夹：

- **raw-原始资料/** 是你的收件箱。文章、论文、笔记、转录稿 —— 统统丢进来，永远不用考虑整理它们。
- **wiki-知识库/** 是 LLM 为你构建的东西。它读取 raw 里的所有内容，撰写文章、创建主题文件夹、把概念链接起来、维护一份索引。你几乎不用碰这个文件夹。
- **output-输出/** 是报告和查询结果的去处。

一旦跑起来，你就可以针对 wiki 向 LLM 提问，它会浏览文件、提取相关材料并作答。不需要花哨的 RAG 搭建 —— 只需要 markdown 文件和一段好的提示词。

魔力在于 vault 根目录的 agent 配置文件（CLAUDE.md、AGENTS.md 等）。那就是你告诉 LLM 如何扮演图书管理员的地方 —— 它必须遵守的架构、操作、页面格式和规则。

---

## 构建它需要什么

**Obsidian** —— 前端。它是一个 markdown 编辑器，把一个装满 .md 文件的文件夹当作 wiki 对待，带反向链接和图谱视图。
> 从 obsidian.md 下载。

**一个 AI agent（编码智能体）** —— 读取资料、撰写 wiki 页面、维护一切的 LLM。任何能读写文件的 agent 都可以。你通过 vault 根目录的配置文件告诉它规则：
- Claude Code → `CLAUDE.md`
- OpenAI Codex → `AGENTS.md`
- Cursor → `.cursor/rules/*.mdc`
- Gemini CLI → `GEMINI.md`

**Obsidian Web Clipper** —— 一个浏览器扩展，把网页文章保存为干净的 markdown 文件，直接存进 vault 的 `raw-原始资料/` 文件夹。这是喂入源材料的主要方式。
> https://chromewebstore.google.com/detail/obsidian-web-clipper/cnjifjpddelmedmihgijeibhnjfabmlf

---

## 四个操作

无论你自己构建还是使用我们的技能，这个系统都有四个操作：

**初始化（Onboarding）** —— 搭建 vault 目录结构并生成 agent 配置文件。创建 `raw-原始资料/`、`wiki-知识库/`（含 sources-来源、entities-实体、concepts-概念、synthesis-综合 子目录）和 `output-输出/`。生成 `wiki-知识库/index.md` 和 `wiki-知识库/log.md` 的初始文件。

**摄取（Ingest）** —— 把一份原始资料处理成 wiki 页面。读取资料，在 `wiki-知识库/sources-来源/` 创建摘要，创建或更新实体页和概念页，在相关页面之间添加 wikilink，更新索引和日志。一份资料通常会触及 10-15 个 wiki 页面。

**查询（Query）** —— 针对 wiki 回答问题。读索引找到相关页面，用 wikilink 引用综合出答案，主动提出可以把有价值的结果保存为综合页。

**体检（Lint）** —— 对 wiki 做健康检查。扫描失效的 wikilink、孤儿页面、矛盾、过时的论断、缺失的交叉引用和数据缺口。按严重程度报告并提供修复。

---

## Agent 配置文件

agent 配置是整个系统的大脑。它精确地告诉 LLM 该如何表现。关键章节：

- **架构** —— 三个目录（raw-原始资料、wiki-知识库、output-输出），wiki 子目录（sources-来源、entities-实体、concepts-概念、synthesis-综合），两个特殊文件（index.md、log.md）
- **页面格式** —— YAML frontmatter（tags、sources、created、updated）+ wikilink 语法
- **操作** —— 摄取、查询和体检的分步工作流
- **规则** —— 约束 LLM 行为的 10 条规则（绝不修改 raw、始终更新索引等）

你可以手写这份配置（概念基础见 `llm-wiki.md`），也可以让我们的引导向导生成。权威规则在 `skills/second-brain/references/wiki-schema.md`。

---

## 多 Agent 支持

这个 wiki 模式与具体 agent 无关 —— 它只是 markdown 文件和一组约定。同一套规则在任何 agent 的配置文件里都适用。这意味着你可以：
- 用 Claude Code 搭好一个 vault，然后再从 Cursor 使用它
- 不重建 vault 的情况下切换 agent
- 让多个 agent 在同一个 vault 上工作（它们遵循相同的规则）

---

## 一切如何拼在一起

Karpathy 的模式：丢入原始资料，让 LLM 编译 wiki，在 Obsidian 里浏览。

| 概念 | 实现 |
|---|---|
| "丢入原始资料" | `raw-原始资料/` 目录 + Obsidian Web Clipper |
| "LLM 编译 wiki" | 摄取操作 —— 读取资料、创建/更新 wiki 页面、维护索引和日志 |
| "在 Obsidian 浏览" | Obsidian 以反向链接和图谱视图读取 `wiki-知识库/` |
| "提出问题" | 查询操作 —— 搜索 wiki，带引用地综合答案 |
| "保持质量" | 体检操作 —— 审计矛盾、孤儿页面、过时论断 |
| "完成搭建" | 初始化操作 —— 交互式向导搭建一切 |
| "提示词" | 根据 wiki-schema 规则生成的 agent 配置文件 |

想法文件（`llm-wiki.md`）是创意的种子。本文档是蓝图。`skills/` 目录是可执行的实现 —— 但你也完全可以基于这份蓝图，用任何 LLM 和任何你顺手的工具构建自己的版本。
