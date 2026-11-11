# Agent 配置模板

vault 根目录固定生成**一份**配置文件：`AGENTS.md`。

- **AGENTS.md** —— 事实标准：Codex、Cursor、Gemini CLI、Trae、WorkBuddy（官方兼容，自动读取）等大多数工具认这个名字
- **Claude Code** —— 2.1.277 版（2026-09-18）起原生读取 AGENTS.md（当目录中没有 CLAUDE.md 时自动回退）；更旧的版本可按下方对照表补一份 CLAUDE.md

## 生成内容

把所有 `{{placeholder}}` 替换为用户在向导中的回答后，写入 vault 根目录的 `AGENTS.md`：

```
# {{VAULT_NAME}}

> {{DOMAIN_DESCRIPTION}}

## 建议标签

{{DOMAIN_TAGS}}

## 知识库规则

你是一个个人知识库的图书管理员和 wiki 维护者。你阅读原始资料，把它们编译成结构化的 wiki 页面，并长期维护这个 wiki。你绝不即兴发挥结构 —— 你严格遵循以下规则。

{{WIKI_SCHEMA}}
```

## 其他工具对照（仅当用户明确提出时才生成）

| 工具 | 输出位置 | 说明 |
|---|---|---|
| Claude Code（旧版 < 2.1.277） | `<vault>/CLAUDE.md` | 内容相同，直接复制 |
| Cursor | `<vault>/.cursor/rules/second-brain.mdc` | 需在文件最前面加 frontmatter（见下） |
| Trae | `<vault>/.trae/rules/project_rules.md` | 内容相同，直接复制 |
| 其他工具 | 按该工具的项目规则约定 | 内容相同，复制过去即可 |

Cursor 版本需在内容最前面附加：

```
---
description: {{VAULT_NAME}} 的知识库图书管理员规则
globs:
alwaysApply: true
---
```

## 占位符说明

- `{{VAULT_NAME}}` —— 向导第 1 步得到的 vault 名称
- `{{DOMAIN_DESCRIPTION}}` —— 向导第 3 步的领域/主题，整理成一句描述
- `{{DOMAIN_TAGS}}` —— 由 LLM 根据领域描述生成的 5-8 个相关建议标签列表
- `{{WIKI_SCHEMA}}` —— `wiki-schema.md` 的全部内容，从 `## 架构` 章节开始
