# Stage 5 — Gemini 与 Antigravity CLI 生态系 (Gemini & Antigravity CLI Ecosystem) ⭐⭐

> [繁體中文](./05-gemini-skills-ecosystem.md) | **简体中文** | [English](./05-gemini-skills-ecosystem.en.md)

⏱ **时间估算**：3-4 周（约 15-25 小时）

> 🚪 **进入条件**：**Track A（CLI Power User）** 从 A1-A2 过来、会用 Python + 跑过基本 CLI 即可，从 5.1/5.2 起步；**Track B（Agent Builder）** 建议先完成 [Stage 3](03-tool-use-and-hello-agent.zh-Hans.md)（tool use）+ [Stage 4](04-agent-frameworks.zh-Hans.md)（agent frameworks）再进，把整个 stage 当作“Antigravity CLI 内部如何运作”来深读。

> 💡 整个 stage 围绕 4 个关键词（**MCP / Skills / Plugins / Packaged Distribution**）展开。

**👥 共用 hub**：本 stage 是 Track A（CLI Power User）+ Track B（Agent Builder）两条路径 of the curriculum 的共用中心。

> 📌 **这个 stage 两条轨都用**：
> - **Track A（CLI Power User）**：A2 用 [5.1（Antigravity CLI 基础）](#51--antigravity-cli-基础)；A3 用 [5.2（MCP）](#52--mcpmodel-context-protocol-基础) + 选择性用到 [5.3（Skills）](#53--skillsantigravity-cli-的行为层) 跟 [5.4（Plugins）](#54--plugins-与打包分发)。读的角度是“**怎么用 Antigravity CLI 把工作做好**”。
> - **Track B（Agent Builder）**：把整个 stage 当作“**Antigravity CLI 内部怎么运作**”的深度学习，从 5.1 完整走到 5.4。

> 🗺️ **Antigravity CLI 属于哪种 agent 型态**？→ Type 1（IDE-coupled）+ Type 2（Terminal pair-programmer）。
> 
> ⚠️ **关于运行环境与 API：** Antigravity CLI 支持 Gemini 系列模型，开发需要使用 Gemini API / Google Cloud 凭据，本地调试可以结合本地 endpoint（如 Ollama）或通过 MCP 调用。

> 📋 **本章组成**：6 个子章（5.1 基础 / 5.2 MCP / 5.3 Skills / 5.4 Plugins / 5.5 Subagents / 5.6 Antigravity CLI 源码解剖），每个子章都有“学习目标 → 必修阅读 → 动手练习 → 精选 Projects” → 章末自我检查。
> 🔑 **关键名词**：MCP、Skills、Plugins、Hooks、Subagents。

---

## Stack 一览

由上往下，每一层都建立在底下那一层上：

```
+------------------------------------------+
|       Plugins & Packaged Distribution    |  <- 插件与打包分发层
+------------------------------------------+
|                  Skills                  |  <- 行为策略层（SKILL.md）
+------------------------------------------+
|                   MCP                    |  <- 协议标准化层（Model Context Protocol）
+------------------------------------------+
|                 Tool Use                 |  <- 工具调用层（Gemini Tool Use）
+------------------------------------------+
|                API + SDK                 |  <- 基础模型与接口层（google-antigravity SDK）
+------------------------------------------+
```

每一层各自加上一种能力：
- **API + SDK**：用程序访问 Gemini 大模型。
- **Tool Use**：让模型调用你定义的 function（工具）。
- **MCP**：标准化协议，让任何 agent 客户端（Host）都能使用任何 tool server。
- **Skills**：构建在 CLI 中的行为蓝图，告诉 agent 如何利用工具完成特定工作。
- **Plugins**：把 Skills、hooks、commands、MCP 配置打包成一个单元分发。

这个阶段有 4 个子章节，**请按顺序做**——每一节都建立在前一节之上。

```
5.1 Antigravity CLI 基础  3-5 天 （安装、slash commands、GEMINI.md）
5.2 MCP — 协议层         5-7 天 （编写你的第一个 MCP server）
5.3 Skills — 行为层      5-7 天 （编写你的第一个 SKILL.md）
5.4 Plugins 与打包分发    5-7 天 （打包并发布）
```

---

## 🗺️ 7-Layer Architecture Map（先看这张图、再读 5.1-5.6）

> 📋 **这节是什么**：把 Antigravity CLI 的 7 个 primitive 对应到 **7 个架构层 + 3 个工程学 discipline**。**分层是教学选择，不是绝对真理**。

### 每层 1 句白话 + Gemini/Antigravity 的版本

| Layer | 是什么 | Antigravity CLI/Gemini 的版本 | 谁管 | 学在 |
|---|---|---|---|---|
| **L7 Interface** | 用户和 agent 交谈入口 | `agy` CLI 交互界面 | Harness Engineering | [Stage 5.1](#51--antigravity-cli-基础) |
| **L6 Workflow** | 固定可复用流程模板 | **Skills**（SKILL.md）+ Slash commands + **Plugins**（打包 Skills / hooks / agents）| Prompt Engineering | [Stage 5.3](#53--skillsantigravity-cli-的行为层) / [5.4](#54--plugins-与打包分发) |
| **L5 Coordination** | 多 agent 分工合作 | **Subagents** + Background tasks / agents | Harness Engineering | [Stage 5.5](#55--subagentsantigravity-cli-原生多-agent-机制) |
| **L4 Memory / Context** | 跨对话 / 跨 session 记事情 | History / `/clear` / `/memory add` / Project Rules | Context Engineering | [Stage 6](06-memory-rag.zh-Hans.md) |
| **L3 Control Plane** | tool 执行前/后拦截/校验/阻挡 | **Hooks**（`PreToolUse` / `PostToolUse` / `PreInvocation` 等）| Harness Engineering | [Stage 5.1 hooks 段](#51--antigravity-cli-基础) |
| **L2 Tool Use** | LLM 调用外部 function 的 protocol | Gemini Tool Use / Function Calling | Tool design | [Stage 3](03-tool-use-and-hello-agent.zh-Hans.md) |
| **L2.5 Tool Provider** | 把外部 API 包成 tool 给 Layer 2 用 | **MCP servers**（如 Notion / SQLite / Filesystem 等）| Context Engineering + Tool | [Stage 5.2](#52--mcpmodel-context-protocol-基础) |
| **L1 Foundation** | LLM 本体（system prompt 直接送达这一层）| Gemini API | Prompt Engineering | [Stage 1](01-llm-basics.zh-Hans.md) + [Stage 2](02-prompt-engineering.zh-Hans.md) |

### 3 工程学 Discipline overlay（核心 insight）

Prompt / Context / Harness 是**不同层的 discipline**——学会其中一个，不会自动会另一个：

| Discipline | 负责哪些 layer | 1 句话 | 学在 |
|---|---|---|---|
| **Prompt Engineering** | L1 + L6 | "送进 LLM 的字符串怎么设计" | [Stage 2](02-prompt-engineering.zh-Hans.md) |
| **Context Engineering** | L4 + L2.5 | "context window 装什么信息" | [Stage 6](06-memory-rag.zh-Hans.md) |
| **Harness Engineering** | L3 + L5 + L7 | "LLM 外面的 runtime scaffolding" | [Stage 7 §Harness Engineering](07-multi-agent-production.zh-Hans.md) |

### 跨 CLI vendor mini-comparison（2026-06 snapshot）

| 层 | Antigravity CLI (`agy`) | Claude Code | OpenAI Codex | Gemini CLI (legacy) |
|---|---|---|---|---|
| L5 Coordination（multi-agent）| ✅ Subagents / Tasks | ✅ Subagents | ❌ single-agent | ❌ |
| L3 Control Plane（Hooks）| ✅ Hooks (`hooks.json`) | ✅ Hooks | ❌ | ❌ |
| L2.5 Tool Provider（MCP）| ✅ MCP Natively | ✅ | ✅ | ✅（需手动配置） |
| L6 Workflow（Skills）| ✅ SKILL.md / Plugins | ✅ SKILL.md | AGENTS.md | GEMINI.md |

---

## 5.1 — Antigravity CLI 基础

### Antigravity CLI 是什么（先定位）

**Antigravity CLI (shorthand: `agy`) = 跑在你 terminal 里的 Gemini coding agent**——拥有完整的文件系统访问权限、可执行 shell 脚本、直接操作 git、运行子进程，并能**自主完成多步骤工作**。

它与其他 Gemini 界面的角色区分如下：

| 界面 | 跑在哪 | 能做什么 | 用法 |
|---|---|---|---|
| **gemini.google.com**（web） | 浏览器 | 纯对话、支持多模态输入，无法直接操作系统与文件 | 灵感碰撞、单点提问、文档理解 |
| **Google AI Studio**（web） | 浏览器 | 调试 prompt、测试系统指令、调试 parameters | 快速原型设计、API 测试 |
| **Gemini API**（programmatic） | 你的 server / script | LLM 调用，需要开发者自己包装 agent 循环 | 生产级后台系统开发 |
| **Google Antigravity SDK** | 你的 Python 环境 | 程序控制的多轮 Agent 运行时，支持 hooks 与 tools | 定制化 Agent 工作流开发 |
| **Antigravity CLI**（**本节**） | 你的 terminal | **全功能 OS-level AI 助手**，集成 Skills、Hooks、Subagents 生态 | **工程师日常编程的主力搭档** |

进 5.2-5.6 之前你会在这节学到 **4 个 Antigravity CLI 核心结构**：`GEMINI.md`（记忆与规则层）/ slash commands（交互控制层）/ `~/.gemini/antigravity-cli/` 目录（全局配置层）/ `settings.json`（行为设置）。

### 学习目标

完成本节后你会：
- 理解 Antigravity CLI 相比 web 界面或原生 API 的独特优势
- 完成 `agy` 客户端的安装、完成 Google Cloud / Gemini API 认证
- 熟练使用常用 slash command 来控制 agent 行为与状态
- 编写一份符合规范的项目级 `GEMINI.md` 规则手册
- 掌握 `~/.gemini/antigravity-cli/` 目录结构（配置文件、全局 skills、plugins 所在位置）

### 必修阅读
1. [**Google — Antigravity CLI Quickstart**](https://antigravity.google/docs/quickstart) — 官方安装指南
2. [**Google — GEMINI.md Best Practices**](https://antigravity.google/docs/memory) — 如何编写高效的项目指令集
3. [**Google — Slash Commands**](https://antigravity.google/docs/slash-commands) — 官方完整命令列表
4. [**Google — Settings Schema**](https://antigravity.google/docs/settings) — `settings.json` 的配置参数详解

---

### 📋 GEMINI.md 设计 prompts（依 5 原则）

当你要生成或审计项目中的 `GEMINI.md` 时，可以使用以下 prompts：

#### Prompt 1 — Audit 你现有的 GEMINI.md

```
我有一个 GEMINI.md，请依下面 5 个 harness engineering 原则审计：

1. Legibility — 用 markdown header 分区吗？约定写得具体（"用 2 空格缩进"）还是模糊（"格式需规范"）？
2. Progressive Disclosure — < 200 行吗？有用 @-import 拆分吗？
3. System of Record — GEMINI.md 是否作为 entry map，将更具体的规则指向子目录下的 rules 文件夹（如 .agents/rules/）？
4. Taste Invariants — 规则是否可由外部验证（"在提交前运行 pytest"）？还是说一堆「遵循最佳实践」的空话？
5. Transparency — 有要求 agent 展示规划步骤吗？还是预期它默默做完？

每一条给出 PASS / FAIL / PARTIAL + 原因 + 改进建议。
```

#### Prompt 2 — 生成新的 GEMINI.md（依 5 原则）

```
我要为一个 [描述项目，例如：Python Fast API 服务 / 前端 React App / 数据处理 Monorepo] 编写 GEMINI.md。请依下面 5 个 harness engineering 原则生成：

- 整体控制在 200 行以内
- 作为 entry map，将具体规范拆分至 `.agents/rules/<topic>.md` 模块
- 每条规则清晰可验证
- 增加 1-2 条透明度规则（例如：「修改超过 50 行代码时，必须先展示 plan 规划」）
- 标注哪些规则留在主文件，哪些建议拆分出去

输出：
1. 完整的 GEMINI.md 内容
2. 建议的 `.agents/rules/` 目录结构与子主题列表
3. 1 个代表性的示例规则 `.agents/rules/linting.md`
```

---

### 常用 slash commands（交互指令）

| Command | 用途 | 何时使用 |
|---|---|---|
| `/help` 或 `?` | 列出所有可用命令与快捷键 | 对 CLI 操作不熟悉或查看命令说明时 |
| `/clear` | 清空对话历史并重置上下文 | 对话过长，想在一个干净的新上下文中继续 |
| `/fork` 或 `/branch` | 将当前会话克隆（分支）到一个新线程 | 探索另一条技术路线时，避免破坏原有的会话上下文 |
| `/config` 或 `/settings` | 弹出交互式配置面板（Settings Editor）| 修改全局设置（如 sandbox 状态、颜色风格等） |
| `/permissions` | 查看并切换当前会话的权限模式 | 当需要调整文件写权限或命令行执行权限时 |
| `/model` | 切换正在运行 of the Gemini 模型 | 想节省 token 换轻量模型，或需要更强的模型进行深度推理 |
| `/mcp` | 开启 Model Context Protocol 服务器管理器 | 调试、添加或移除 MCP 连接器 |
| `/agents` | 打开子 Agent 与后台任务管理器 | 查看并控制当前正在并行运作的 Subagents |
| `/memory add <fact>` | 往 `GEMINI.md` 追加并固定一条事实/常识 | 希望 agent 牢记关于当前项目的特定信息 |
| `/exit` | 关闭 TUI 界面并退出 | 结束工作会话 |

> 💡 **Quick Tips:**
> - 输入 `!` 开头可以直接临时执行一条本地 shell 指令（如 `!npm test`）
> - 输入 `@` 可以触发文件路径自动补全
> - 双击 `esc` 可清空当前输入框

---

### `~/.gemini/antigravity-cli/` 目录结构（Mental Map）

```
~/.gemini/antigravity-cli/         ← 全局 user-level 目录
├── settings.json                   ← 全局偏好设置（预设模型、沙箱环境、安全策略）
├── settings.local.json             ← 机器特异性本地配置（如局部 token，忽略 git 提交）
├── GEMINI.md                       ← 全局通用 baseline 规则（任何 session 都会首先载入）
├── skills/                         ← 全局共享的自定义技能目录
│   └── <name>/SKILL.md
├── agents/                         ← 全局 Subagents 模版定义目录
│   └── <name>.md
├── plugins/                        ← 全局已安装插件目录
└── hooks/                          ← 全局挂钩事件触发脚本

<project-root>/.agents/            ← 项目级 project-level 目录（随代码仓库版本控制）
├── settings.json                   ← 项目专属配置与权限清单
├── skills/                         ← 项目专属技能（优先级高于全局 skills）
│   └── <name>/SKILL.md
├── agents/                         ← 项目专属 Subagents（项目私有定义）
├── rules/                          ← 模块化项目规则文档（对应 GEMINI.md 中 @-import 引入的规则）
│   └── <topic>.md
└── hooks.json                      ← 项目级 Hooks 触发定义文件
```

**优先级规则**（冲突时冲突消解顺序）：`项目级 (.agents/)` > `用户全局 (~/.gemini/antigravity-cli/)` > `内建预设 (built-in)`。

### 动手练习
- **练习 1：建立首个会话** — 安装 CLI 后，在本地测试仓库中运行 `agy`，尝试提问“分析此仓库的代码结构”，观察它通过 tools 读取文件的流程。
- **练习 2：项目规则约束** — 在仓库根目录下创建 `GEMINI.md` 并写入一条“不能在 commit 消息中包含 emoji，代码缩进必须为 Tab”等规则，进行修改并 commit 提交，看 agent 是否会被约束。
- **练习 3：玩转 Slash Commands** — 会话过程中，体验 `/config` 调出菜单修改主题，用 `/permissions` 调低或调高敏感权限。
- **练习 4：全局目录探索** — 进到 `~/.gemini/antigravity-cli/`，查看 `settings.json` 中的字段含义。

---

## 5.2 — MCP（Model Context Protocol）基础

### MCP 是什么（先定位）

**MCP (Model Context Protocol) = “让 LLM 能够连接到任何外部工具与数据源”的开放标准化协议**。

在没有 MCP 之前，每家大模型服务商都在各自闭门造车，定义专属的 Tool Call Schema，导致很多集成工具无法复用。MCP 完成了这一层的**标准化解耦**：开发者只要写一个符合 MCP 规范的 Server，这个 Server 提供的工具集就可以无缝接入任何支持 MCP 的 Host（如 Cursor、Claude Desktop、Antigravity CLI 等）。

```
+---------------+     MCP Standard Protocol      +---------------+
|   MCP Host    | <============================> |  MCP Server   |
| (agy / Cursor)|                                | (Notion/Fetch)|
+---------------+                                +---------------+
```

**MCP 的三大主要抽象概念**：
1. **Tools (工具)**：LLM 能够主动触发 the Function（例如：`run_query(sql)`、`get_api_status()`）。大多数 MCP 实践中 90% 的场景都是使用 Tools。
2. **Resources (资源)**：供 LLM 被动读取的静态或动态数据源（例如：`file:///workspace/data.csv`、`db://tables/users`）。
3. **Prompts (提示模板)**：Server 提供给 LLM 的快捷提示词样板（例如：`review-pull-request`）。

---

### 动手练习
- **练习：配置一个 MCP 插件** — 将内置或社区开源的 filesystem MCP 服务接入你的 Antigravity CLI 会话中。运行 `/mcp` 命令观察其连接状态与暴露 the tools。
- **练习：写一个你自己的 Python MCP Server** — 利用 Python SDK 编写一个极简的温度单位转换或日志解析工具。
  - 第一步：`pip install mcp`
  - 第二步：编写包含 `@mcp.tool()` 装饰器的 Python 服务。
  - 第三步：通过 `agy` 配置或插件管理器将其引入。

---

## 5.3 — Skills（Antigravity CLI 的行为层）

### Skill 是什么（先定位）

Skill 实际上是一个**特殊的 markdown 文件 (`SKILL.md`)**，定义了在遇到什么情境时，引导 agent 使用什么样的标准动作来解决它。

Antigravity CLI 每次执行前会评估现有 Skills 中的 `description` 描述 frontmatter，如果与当前任务的情境契合，系统就会**自动将该 SKILL.md 载入当前的系统上下文**，来充实 agent 的具体业务知识。

### Skill vs GEMINI.md vs MCP vs Plugin vs Subagent 对照表

| 组件 | 是什么 | 何时使用 | 触发机制 | 例子 |
|---|---|---|---|---|
| **GEMINI.md** | 项目通用的 baseline 准则与规章 | 定义全局代码规范、命名约定与项目定位 | **每个会话启动时默认无条件载入** | `GEMINI.md` |
| **MCP server** | 标准化工具数据桥梁，提供具体 API 接口 | 需要大模型向外部系统（数据库/第三方 SaaS API）发起通信 | 启动后被作为一个常驻 Tool，按需调用 | `github-mcp` |
| **Skill** | 针对**特定情境与任务**的行为指南流程 | 某种复杂甚至多步骤的固定流程（如：“把 PDF 转换为规范 markdown”） | **按 Frontmatter 中的 description 自动匹配并注入上下文** | `.agents/skills/pdf-parser/SKILL.md` |
| **Plugin** | 将多个 skills、hooks、mcp 配置组装成的包 | 需要将一个相对独立且复杂的子工具套件打包发布或在团队内推广 | 通过 `agy plugin install` 一键下载与激活 | `frontend-developer-plugin` |
| **Subagent** | 拥有**完全隔离上下文**的独立 agent 进程 | 当某子任务庞大且消耗 Token，需要单独开辟工作区（如：“对整个 codebase 的每一个文件做漏洞审查”） | description 规则触发自动派单，或通过 slash 命令唤醒 | `.agents/agents/sec-auditor.md` |

---

### 📋 SKILL.md 设计 prompts（依 5 原则）

#### Prompt 1 — Audit 你现有的 SKILL.md

```
我有一个 SKILL.md 文件，请根据以下 5 个 principles（Legibility, Progressive Disclosure, System of Record, Taste Invariants, Throughput/Merge）对它进行审核，指明 PASS / FAIL / PARTIAL 以及具体的重构建议：

1. 触发精确度：Frontmatter description 是否准确圈定了它会被匹配的情景？
2. 长度与结构：SKILL.md 主文件是否在 200 行以内？复杂场景例子是否已提取到 references 文件夹？
3. 数据唯一性：是否存在在 SKILL.md 中重复叙述的外部知识？
4. 验证闭环：是否定义了对于成功的可检验标准（Success Criteria）？
5. 门禁检查：是否规划了针对该技能输出质量的 eval 测试或者 validation preset？
```

#### Prompt 2 — 生成新的 SKILL.md

```
我想为以下场景编写一个 SKILL.md 文件：[描述任务，例如：抓取某公开网站 API 并做格式校准]。请按 harness engineering 的 5 原则生成：

- 头部包含包含规范 yaml frontmatter（含 triggering, rules 和 tools）
- 主文件 < 200 行，多余的上下文知识作为 reference 拆分
- 给出建议的 references 结构
- 提供清晰的 success criteria（成功判定）
- 编写一套简易的验收 eval（如 python test 命令或 lint 命令）
```

---

## 5.4 — Plugins 与打包分发

### Plugin 是什么（先定位）

**Plugin（插件）是 Antigravity CLI 的一种高内聚打包机制**。

当你在 5.2 与 5.3 中完成了 MCP server 开发或写出了好用的 Skills 后，如果想在团队中共享，可以使用 Plugin 机制。它可以将 Skills、Hooks、Subagents 模版、MCP 服务器配置整合成一个单一目录。

### Antigravity Plugin 规范目录结构

```
my-antigravity-plugin/
├── plugin.json              ← (必填) 插件配置文件与元数据元信息
├── hooks.json               ← (可选) 声明在此插件作用域内运行的 Event Hooks
├── mcp_config.json          ← (可选) 声明绑定的 MCP Servers 配置
├── skills/                  ← (可选) 附带的 SKILL.md 技能目录
│   └── code-cleaner/
│       └── SKILL.md
├── agents/                  ← (可选) 子 Agent 定义文件
│   └── refactor-helper.md
└── rules/                   ← (可选) 代码规范与规则定义文件
```

### 插件管理与迁移命令

- **列出当前已安装的所有插件**：
  ```bash
  agy plugin list
  ```
- **从本地路径或 GitHub URL 安装插件**：
  ```bash
  agy plugin install /path/to/local/plugin
  ```
- **旧版 Gemini CLI 迁移指令**：
  如果你之前为 Gemini CLI 编写过遗留扩展配置，可以直接一键导入并将其转化为 Antigravity 插件格式：
  ```bash
  agy plugin import gemini
  ```

---

## 5.5 — Subagents（Antigravity CLI 原生多 Agent 机制）⭐

在日常开发中，有些任务（如大规模架构重构、深入日志分析、漏洞大面积排查）会消耗极高数量 of the Token，如果直接在主会话中由单个 Agent 处理，会导致上下文瞬间被垃圾信息塞满，使得主 Agent 失去原有的开发上下文记忆。

**Subagent（子 Agent）通过开启具有独立 Context Window 的新子进程来完美解决这一痛点**。

```
                    +--------------------+
                    |  agy Main Session  |
                    +--------------------+
                              |
                    (Spawn via Task Tool)
                              |
            +-----------------+-----------------+
            |                                   |
  +------------------+                 +------------------+
  |    Subagent A    |                 |    Subagent B    |
  | (Explore Code)   |                 | (code-reviewer)  |
  | Context Window A |                 | Context Window B |
  +------------------+                 +------------------+
```

### Subagent 与框架型多 Agent（如 CrewAI / LangGraph）的对比

| 比较维度 | LangGraph / CrewAI 路线 | Antigravity CLI Subagent 机制 |
|---|---|---|
| **启动阻力** | 需安装复杂 Python 包、手动编排节点边与逻辑关系 | 仅需在 `.agents/agents/` 下编写一份 markdown 蓝图 |
| **运行时维护** | 开发者需要自主提供 Python 程序作为宿主运行环境 | 由 CLI 内部的进程调度系统直接托管与派发 |
| **内存隔离** | 需由框架管理多会话状态与交互 | **天生隔离**，子进程间各自维护完全独立的 context window |
| **应用甜蜜点** | 复杂的跨 LLM API 的多角色业务系统开发 | 紧密契合当前开发工作流、日常编程的专项任务提效 |

### Antigravity CLI 中的三种多 Agent 协调模式

| 模式 | 特点与应用场景 | 启用与管理方法 |
|---|---|---|
| **Subagents (子 Agent)** | 隔离大上下文，并行处理不同的审查与检索任务，最终在主会话中汇总报告。 | 在 `.agents/agents/` 中定义 markdown。主 Agent 根据任务描述**自适应挑选调度**，也可以使用 `/agents` 查看。 |
| **Agent Teams (团队协同)** | 多模型、多视角碰撞。一个扮演架构师，一个扮演安全极客，一个唱反调，三者可直接 debate。 | 会话中发起指令：“建立一个技术分析小组，一个从前端易用性探讨，一个从数据稳定性论证”，并在界面中使用快捷键切换不同角色。 |
| **Background Agents (后台静默任务)**| 不需要同步等待，将大任务静默放置在后台处理（如重构、单元测试编写），用户继续在 CLI 中输入其他指令。| 会话中使用 `/bg` 命令将当前任务推入后台，或直接通过命令行 `agy --bg "实现对模块 X 的单元测试编写"`，并在 `/tasks` 面板监控状态。 |

---

### 👉 经典子 Agent 文件样板 (code-reviewer.md)

`.agents/agents/code-reviewer.md`：
```markdown
---
name: code-reviewer
description: 审计当前仓库中未 commit 或 staged 的代码改动，排查安全隐患、代码风格冲突并检测测试覆盖度。在用户说 "审查我的代码" 或输入 `/review` 时触发。
tools:
  - Read
  - Grep
  - Bash
model: gemini-2.5-flash # 强制指定更轻量、更划算的模型去跑审查，节约 Token 成本
---

你是一个资深的安全审计与代码评审专家。请严格按照以下步骤工作：
1. 运行 `git diff --cached` 获取当前暂存区的代码变动。
2. 重点审查：明文密钥硬编码、SQL 注入隐患、未捕获的异常、测试遗漏。
3. 产出报告：PASS 或输出包含具体 file:line 的修改意见列表。
```

---

## 5.6 — Antigravity CLI 源码解剖（Harness Engineering 案例分析） ⭐

> 🎯 **本节定位**：本节并不是从头讲 Harness Engineering 的理论概念，而是将 **Google Antigravity SDK** 作为一个经典的生产级 Harness 实践案例进行解构，帮助你在代码中建立直观的感知。

### 必修阅读
1. [**Google — google-antigravity SDK 源码库**](https://github.com/google/google-antigravity) — 核心阅读文件：`google_antigravity/_internal/client.py` （包含 Agent 主循环）以及 `query.py`。
2. [**Community — Awesome Harness Engineering**](https://github.com/ai-boost/awesome-harness-engineering) — 汇集了各种大模型运行机制与安全阻断的参考实现。

### 🛠️ 动手练习：解剖 Agent 核心主循环

我们来阅读 SDK 中的 `google_antigravity/_internal/client.py`（也可以通过 `pip show google-antigravity` 找到你本地 Python 环境中的位置），找出构成生产级 Agent 骨架的 **6 个核心内部运行时元件**：

1. **Agent Loop (Agent 执行循环)**：定位实际在什么位置重复地接收模型响应、检测是否有 tool call 请求。
2. **Tool Registry & Dispatcher (工具注册与分发)**：LLM 响应了 tool_use 后，Runtime 是如何动态路由到对应的真实工具函数（如执行本地 python 文件或 Shell）？
3. **Context Manager & Window Compacter (上下文管理器与压缩机制)**：分析工具执行结果如何拼接回 message 历史，以及当会话超出 context 窗口限制时，如何做到自动摘要。
4. **Safety Layer & Policy Gate (安全校验层)**：在哪里对高危操作（比如写系统文件、执行写命令）进行了拦截与权限二次确认？
5. **Retry & Error Recovery (容错机制)**：当工具报错时（如 shell执行失败），系统是如何将 Traceback 反哺给模型让其自我修正，还是抛出异常？
6. **Telemetry & Metering (遥测与计量)**：统计当前会话所消耗的 Token 数量、计算 Latency 延迟以及上报 Metrics 的点埋在了哪里？

---

## 5.7 — SDK：把 Antigravity CLI 拆开来自己组 ⭐

当你的使用场景不再局限于在终端跟 Agent 敲字交互，而是需要：
- 将 agent 嵌进公司现有的 Web 门户或 CI/CD 流水线中
- 通过定时任务（Cron）自动拉起并完成代码检测
- 对外开放包装好的 API 接口，并加装自定义的权限网关

你就需要放弃 CLI（`agy`），改用更底层的 **Google Antigravity SDK**。

### 极简 SDK 演示（只需 6 行 Python）

```python
import asyncio
from google_antigravity import query

async def run_agent():
    # 这一行就能启动带有完整 tool permissions & context management 的运行时
    async for msg in query(prompt="分析当前仓库并列出所有 python 依赖文件"):
        # 实时打印流式返回。可以通过 isinstance 区分系统消息与助手回复
        print(msg)

if __name__ == "__main__":
    asyncio.run(run_agent())
```

---

## ✅ 进入 Stage 6 前的自我检查

在进入下一阶段学习前，请确认你是否已掌握以下清单：
- [ ] 成功安装并运行了 `agy`，并且使用过 `/clear`, `/fork`, `/permissions` 等至少 5 个 slash commands。
- [ ] 学会使用 `/mcp` 调试与配置 MCP 服务，并能在会话中自如使用这些工具。
- [ ] 在项目根目录根据 5 原则编写并测试了 `GEMINI.md`，限制了 agent 的常规开发习惯。
- [ ] 亲手编写了一个符合 YAML frontmatter 规范的自定义 `SKILL.md`，并在描述命中时看到了自动装载。
- [ ] 了解 `plugin.json` 架构，知道如何把 skills、hooks 编写为可分发的 plugin 包。
- [ ] 编写了本地 Subagent 定义，观察到它在处理超长任务时展现出的 Context Window 隔离效果。
- [ ] 阅读了 `google-antigravity` SDK 核心主循环源码，在纸上或者笔记中能够大致指出 tool dispatch 与 safety gate 的执行顺序。
