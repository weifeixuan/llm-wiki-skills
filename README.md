# Second Brain（第二大脑）

一个由 LLM 维护的个人知识库，基于 [LLM Wiki 模式](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)。把原始资料丢进文件夹，让 LLM 将它们编译成结构化的 wiki，然后在 Obsidian 里浏览这一切。

## 工作原理

你把原始材料（文章、论文、笔记、转录稿）放进 `raw-原始资料/` 文件夹。LLM 读取所有内容，撰写结构化的 wiki 页面，建立交叉引用，并维护一份索引。你在 Obsidian 里浏览成果——沿着链接跳转、探索图谱视图、提出问题。

LLM 是图书管理员，你是策展人。

## 前置条件

- **[Obsidian](https://obsidian.md)** —— 用来浏览 wiki 的 markdown 编辑器
- **一个 AI agent（编码智能体）** —— [Claude Code](https://claude.ai/code)、[Codex](https://openai.com/codex)、[Cursor](https://cursor.com)、[Gemini CLI](https://github.com/google-gemini/gemini-cli)，或任何支持 [Agent Skills](https://agentskills.io) 的智能体

> 本项目不依赖 Node.js、npm 或任何脚本：安装只需复制文件夹，初始化由 AI agent 按技能说明直接完成。

## 安装

把本仓库 `git clone`（或下载 ZIP 解压）后，将 `skills/` 目录下的四个技能文件夹复制到你的 AI agent 的技能目录中即可：

```bash
git clone https://github.com/weifeixuan/llm-wiki-skills.git
cd llm-wiki-skills

# Claude Code / CodeBuddy 系
cp -r skills/second-brain* ~/.claude/skills/

# WorkBuddy
cp -r skills/second-brain* ~/.workbuddy/skills/

# 其他 agent：把技能文件夹放进各自的技能目录即可
```

这样会安装四个技能：

| 技能                     | 作用                |
| ---------------------- | ----------------- |
| `/second-brain`        | 新建一个 vault（引导式向导） |
| `/second-brain-ingest` | 把原始资料处理成 wiki 页面  |
| `/second-brain-query`  | 针对你的 wiki 提问      |
| `/second-brain-lint`   | 对 wiki 做健康检查      |

## 快速开始

1. **安装技能**（见上文）（或交由AI agent完成安装这组技能）
2. **运行向导：** 在你的 AI agent 里输入 `/second-brain` —— 回答 3 个问题（名称、位置、领域），其余全自动完成
3. **安装 Web Clipper：** [Obsidian Web Clipper](https://chromewebstore.google.com/detail/obsidian-web-clipper/cnjifjpddelmedmihgijeibhnjfabmlf) —— 配置它把剪藏内容保存到 vault 的 `raw-原始资料/` 文件夹
4. **在 Obsidian 中打开** —— 启动 Obsidian，选择"打开文件夹作为仓库"，选中你的 vault 文件夹
5. **剪藏第一篇文章** 到 `raw-原始资料/`，然后运行 `/second-brain-ingest` —— LLM 会和你讨论核心要点并构建 wiki 页面
6. **在 Obsidian 里浏览 wiki** —— 跟随 `[[wikilink]]`、探索图谱视图、查看 `wiki-知识库/index.md`
7. **持续使用** —— 用 `/second-brain-query` 提问，用 `/second-brain-lint` 做健康检查

## 你会得到什么

```
your-vault/
├── raw-原始资料/            # 你的收件箱 —— 原始资料放这里
│   └── assets-附件/         # 图片和附件
├── wiki-知识库/             # LLM 维护的 wiki
│   ├── sources-来源/        # 每份已处理资料对应一份摘要
│   ├── entities-实体/       # 人物、组织、产品、工具
│   ├── concepts-概念/       # 想法、框架、理论
│   ├── synthesis-综合/      # 对比、分析、主题
│   ├── index.md             # 所有页面的总目录
│   └── log.md               # 按时间顺序记录的操作日志
├── output-输出/             # 报告和生成的产物
└── AGENTS.md                # agent 配置文件（行业事实标准，WorkBuddy / Codex / Trae / Claude Code 2.1.277+ 均读取）
```

## FAQ

**向导失败了，或者我需要重新运行设置。**  
再运行一次 `/second-brain` 即可 —— 引导流程是幂等的。它不会覆盖已有文件，所以你的数据是安全的。如果想要全新开始，删除 vault 文件夹后重新运行。

**我不小心修改了 `raw-原始资料/` 里的文件。**  
没关系。wiki 是根据原始内容构建的。如果需要找回原文，查看 git 历史（如果 vault 是 git 仓库）或重新剪藏。wiki 页面不受影响。

**`wiki-知识库/index.md` 和实际页面不同步了。**  
运行 `/second-brain-lint` —— 它会检查索引一致性并提供修复选项。

**重命名页面后 wikilink 失效了。**  
运行 `/second-brain-lint` —— 它会扫描失效的 `[[wikilink]]` 并报告哪些文件需要更新。

**wiki 变得很大，查询变慢了。**  
这是正常的 —— LLM 会先读 `wiki-知识库/index.md` 定位页面再深入阅读。规模太大时可以考虑给 vault 建立更细的分类结构。

**可以同时用多个 AI agent 吗？**  
可以。配置统一写在 vault 根目录的 `AGENTS.md` —— 它是行业事实标准，WorkBuddy、Codex、Trae、Claude Code（2.1.277+）等大多数工具都自动读取，多个 agent 可以在同一个 vault 上工作。少数不认 AGENTS.md 的工具，把这份文件复制到它约定的位置即可（Claude Code 旧版复制为 `CLAUDE.md`，Cursor 放到 `.cursor/rules/`）。

**剪藏文章里的图片怎么处理？**  
在 Obsidian 中，设置 Settings → Files and links → Attachment folder path 为 `raw-原始资料/assets-附件/`。剪藏文章后，使用"为当前文件下载附件"把图片保存到本地。

**多久 lint 一次？**  
每处理 10 份资料或每月一次 —— 以先到者为准。在做大型查询或综合分析之前也建议跑一次。

## 基于以下项目

- [Andrej Karpathy 的 LLM Wiki 模式](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)
- [Agent Skills 开放标准](https://agentskills.io)
- [蓝图与起源故事](docs/REQUIREMENTS.md) —— 本项目的奠基文档
