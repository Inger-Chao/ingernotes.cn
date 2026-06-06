---
title: Claude Code 常用命令与扩展功能指南
tag: Claude
layout: post
category: blog
toc: true
---

> Claude Code 是 Anthropic 官方推出的 CLI 开发工具，将 Claude AI 直接集成到终端工作流中。本文整理日常开发中最实用的命令和扩展能力。

## 一、Slash（斜杠命令）

在对话中输入 `/` 即可触发内置命令：

| 命令 | 说明 |
|------|------|
| `/help` | 获取帮助信息 |
| `/clear` | 清空当前对话上下文 |
| `/compress` | 压缩对话历史以释放上下文空间 |
| `/cost` | 查看当前会话的 API 用量和成本 |
| `/model` | 查看或切换当前使用的 Claude 模型 |
| `/verbose` | 切换详细输出模式（显示工具调用详情） |
| `/feedback` | 提交反馈或报告问题 |


## 二、记忆系统

### `#` 开启记忆模式

Claude Code 提供持久化记忆能力，跨会话保留关键信息：

- **项目记忆** — 存储在 `.claude/CLAUDE.md` 和项目根目录的记忆文件中，记录项目架构、编码规范、技术栈等
- **用户记忆** — 存储在 `~/.claude/` 目录，记录个人偏好、常用工具、开发习惯等

记忆自动加载到对话上下文，减少重复说明的成本。

### 记忆文件约定

```
~/.claude/
├── CLAUDE.md              # 全局指令（所有项目生效）
└── projects/
    └── <project-path>/
        └── memory/
            └── MEMORY.md   # 项目级记忆
```


## 三、Agent 工具（子代理）

Claude Code 支持启动专用子代理处理复杂任务：

| Agent 类型 | 适用场景 |
|-----------|---------|
| `general-purpose` | 通用多步骤任务、代码搜索、复杂研究 |
| `Explore` | 快速探索代码库、查找文件、关键词搜索 |
| `Plan` | 软件架构设计、实现方案规划 |
| `statusline-setup` | 配置 Claude Code 状态栏 |

### 使用示例

```
# 让 Claude 探索整个代码库
"帮我找一下项目中所有与认证相关的代码"

# 让 Claude 规划实现方案
"设计一个用户权限管理系统的实现方案"
```


## 四、MCP 服务器（Model Context Protocol）

MCP 是 Claude Code 与外部服务交互的标准协议，通过 MCP 服务器可以扩展 Claude 的能力边界。

### 常用 MCP 能力

| 能力 | 工具 | 说明 |
|------|------|------|
| 文件搜索 | `Glob` | 按文件名模式快速查找，如 `**/*.java` |
| 内容搜索 | `Grep` | 全文搜索，支持正则表达式 |
| 文件读取 | `Read` | 读取文件内容 |
| 文件编辑 | `Edit` / `Write` | 精确修改或创建文件 |
| 终端执行 | `Bash` | 运行 Shell 命令 |
| 网页抓取 | `WebFetch` / `WebSearch` | 获取网页内容或搜索互联网 |

### MCP 服务器配置

在 `~/.claude.json` 中配置 MCP 服务器：

```json
{
  "mcpServers": {
    "server-name": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-xxx"]
    }
  }
}
```


## 五、Skill 技能系统

Skill 是可复用的专业化工作流，通过 `/skill-name` 触发：

### 开发类技能

| 技能 | 说明 |
|------|------|
| `/simplify` | 代码审查与优化 |
| `/code-review` | 多维度代码评审（与 origin/master 对比） |
| `/commit` | 生成规范的 Git 提交信息 |

### 文档类技能

| 技能 | 说明 |
|------|------|
| `/pdf` | 处理 PDF 文件（创建、读取、合并） |
| `/xlsx` | 处理电子表格 |
| `/docx` | 处理 Word 文档 |
| `/pptx` | 处理 PowerPoint 演示文稿 |

### 前端类技能

| 技能 | 说明 |
|------|------|
| `/frontend-design` | 创建高质量前端界面 |
| `/web-artifacts-builder` | 构建复杂多组件 Web 应用 |


## 六、OpenSpec 规范驱动开发

本项目使用 OpenSpec 进行变更管理，核心工作流：

```bash
# 查看当前变更列表
openspec list

# 查看特定变更详情
openspec show <change-id>

# 验证变更提案
openspec validate <change-id> --strict

# 归档已部署的变更
openspec archive <change-id> --yes
```

### 变更提案流程

1. 创建 `changes/<change-id>/` 目录
2. 编写 `proposal.md`（为什么改、改什么）
3. 编写 `tasks.md`（具体任务清单）
4. 编写 `design.md`（可选，复杂设计说明）
5. 运行 `openspec validate` 验证
6. 实现后归档


## 七、实用技巧

### 上下文管理

- 对话过长时自动压缩，重要信息会被保留
- 使用 `/compress` 主动压缩以节省 token
- 复杂任务使用 Agent 子代理避免主上下文膨胀

### 文件操作

- 优先使用专用工具（Read/Edit/Glob/Grep）而非 Bash
- 编辑文件时 `Edit` 工具比 `Write` 更安全（只改目标行）
- 搜索文件用 `Glob` 模式匹配，搜索内容用 `Grep`

### Git 工作流

- 默认不自动提交，由用户控制提交时机
- 使用 `git status` 和 `git diff` 确认变更范围
- 提交信息遵循 Conventional Commits 规范


---

## 参考资源

- [Claude Code 官方文档](https://docs.anthropic.com/en/docs/claude-code/overview)
- [MCP 协议规范](https://modelcontextprotocol.io/)
- [Anthropic API 文档](https://docs.anthropic.com/)
