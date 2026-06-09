# Subagent 进阶使用 — Description 写法 / Composition / Debug

> [繁體中文](./subagent-advanced.md) | **简体中文** | [English](./subagent-advanced.en.md)

> 📋 **这份是给谁看的**：你已经掌握了内置 subagent 的基本使用（[Stage 5.5](../stages/05-gemini-skills-ecosystem.zh-Hans.md#55--subagentsantigravity-cli-原生多-agent-机制) 和 [cookbook](./subagent-cookbook.zh-Hans.md) 已经阅读完毕），准备开始：(1) **自己编写一个** 自定义 subagent、(2) **组合与编排多个** subagent 协作，或者 (3) **排查与调试** 行为异常的 subagent。
>
> ⚠️ **先决条件**：阅读完 Stage 5.5 核心概念与 cookbook 的 15 个 recipe 后再继续。

---

## 怎么读这份文件

本篇包含 3 个独立的进阶主题，按需查阅：

| 遇到的具体问题 | 对应章节 |
|---|---|
| 编写了一个 subagent，但主 Agent 始终不自动派遣它 | [§1 Description 怎么写才会被主会话自动 spawn](#1-description-怎么写才会被主会话自动-spawn) |
| 想要串联或并行运行 2-3 个 subagent 协同工作 | [§2 Composition pattern 怎么设计](#2-composition-pattern-怎么设计) |
| subagent 执行报错、越权或执行步骤异常 | [§3 自定义 subagent 的调试与 Debug](#3-自定义-subagent-的调试与-debug) |

---

## §1 Description 怎么写才会被主会话自动 spawn

主会话在阅读你的输入时，如何匹配并路由给特定的子 Agent 呢？它会扫描配置文件（如 `.agents/agents/<name>.md`）头部 YAML frontmatter 中的 **`description`** 字段。

**Description 的写作细节决定了路由匹配的准确率**。下面是 4 个常见 Bug 与改进写法：

### Bug 1: 描述过于空洞抽象，Agent 无法抓取触发时机

❌ **不推荐写法**：
```yaml
description: A helpful code reviewer.
```
**分析**：`helpful` 属于主观描述，`reviewer` 范围太广。Agent 无法据此推断在什么代码语境或提问场景下应该唤醒它，只能依靠用户以命令方式指名道姓调用。

✅ **建议改写为**（包含明确触发条件 + 执行边界）：
```yaml
description: Use PROACTIVELY when the user has staged ≥ 50 lines of changes and is about to commit. Reviews staged diff for security issues, style violations, missing error handling, and test gaps. Returns per-category PASS/FAIL + concrete fix list.
```
**优势**：
1. 使用了 `PROACTIVELY`（主动派遣）强提示词。
2. 规定了具体的阈值“修改并暂存代码 ≥ 50 行且准备 commit”。
3. 阐明了它的核心审计内容和返回的结构化格式。

---

### Bug 2: 滥用 `PROACTIVELY`，导致频繁打扰主线开发

❌ **不推荐写法**：
```yaml
description: Use PROACTIVELY for all code-related tasks.
```
**分析**：“all code-related” 范围过宽，主 Agent 会在每一次细微修改（哪怕只是修改拼写）时频繁派发 subagent，造成严重的交互干扰和 Token 损耗。

✅ **建议改写为**（精确化限定范围）：
```yaml
description: Use PROACTIVELY when a commit is about to land that modifies authentication, database queries, or API routes — these are high-risk surface areas needing extra review.
```
**优势**：限定在高风险区域（认证/数据库查询/API路由），大幅降低冗余触发。

---

### Bug 3: 未配置 `PROACTIVELY`，陷入纯被动

❌ **不推荐写法**：
```yaml
description: Reviews code when asked.
```
**分析**：由于没有配置 `PROACTIVELY`，只有当用户在 prompt 里明确写出“让 code-reviewer 审计这段代码”时才会运行，无法作为自动化安全守卫在后台静默拦截。

✅ **建议改写为**：
```yaml
description: Code reviewer. Use PROACTIVELY when staged changes touch test files but the test count didn't increase — likely missing test coverage for new logic.
```

> 💡 **主动与被动的定位选择**：
> - **Always-on 的安全门禁或规范审查**（如漏洞扫描、静态 lint 强制校验） → 配置 `PROACTIVELY` + 明确的 trigger。
> - **复杂、耗费大量 Token 的深入研究**（如交叉对比多篇大文档） → 避免配置 `PROACTIVELY`，采用被动触发模式（如 `Use when user asks for ...`）。

---

### Bug 4: 描述冗长复杂，干扰决策器

❌ **不推荐写法**（写成了洋洋洒洒的说明文档）：
```yaml
description: This subagent performs comprehensive code review including security analysis, performance profiling, style enforcement, type checking, dependency auditing, license compliance, documentation completeness verification, test coverage assessment, accessibility validation, internationalization checks... (此处省略几百字)
```
**分析**：虽然系统对字段长度没有极其严格的物理限制，但过长的说明会占用不必要的系统上下文（System Prompt Budget），导致调度路由器在解析时失焦。

✅ **建议改写为**（保留最精准的 2-3 句话）：
```yaml
description: Use PROACTIVELY before commits touching auth or payment code. Checks: hardcoded secrets, missing input validation, SQL injection risk. Returns issue list with file:line.
```

---

## §2 Composition pattern 怎么设计

当你想同时让 2 个或更多子 Agent 协作时，通常可以参考以下三种被社区验证的编排模式 (Composition Patterns)：

```
Pattern A: Parallel (平行隔离)   ---> Spawn Subagent 1 / 2 / 3 同步独立运行
Pattern B: Pipeline (多步管道)   ---> Subagent 1 Output -> Subagent 2 Input
Pattern C: Meta-Agent (自我迭代) ---> Subagent 自动生成/修改其他 Subagent (不推荐)
```

### Pattern A — 平行隔离 (Parallel)

**适用情境**：多个子任务彼此完全**独立**，互不依赖。
- 例如：同时对 4 个不同的模块文件进行并发式的规范合规性检测。
- 在 prompt 中列出这 4 个目标文件，主会话会分析并单回合多次调用 Task 接口，实现并发执行。

### Pattern B — Pipeline 串联 (Pipeline)

**适用情境**：任务流程有明显的**上下游步骤关系**，前一步的产出作为下一步的输入。
- 例如：输入文献 -> 运行 researcher 子代理提取关键段落 -> 运行 reconciler 整合终稿。
- **推荐实践**：编写一个技能（Skill）或配置插件（如 `agent-collab-workspace`）来定义并触发它的顺序派发逻辑，避免用户手动在终端逐个复制传递结果。

### Pattern C — Meta-Agent (自我生成，不推荐)

**避坑提示**：让一个 subagent 自动去写另一个 subagent。在实际开发中，这极易导致上下文嵌套爆炸、权限漏洞失控、且极其难以调试。**如有重复性或动态模板需求，请改用 Skill 或预设 template 实现**。

---

## §3 自定义 subagent 的调试与 Debug

当你在 `.agents/agents/<name>.md` 中写好配置，但在 `agy` 交互中执行效果不对时，请依次排查以下 5 个切点：

### 切点 1: 确认 CLI 正常加载了该配置文件

```bash
# 在 agy 对话框中输入：
/agents
```
**检查结果**：
- 如果列表里**没有**你新写的 Agent 名称：
  - 检查文件位置是否放错。项目级应在 `./.agents/agents/` 下，全局在 `~/.gemini/antigravity-cli/agents/` 下。
  - 检查 YAML frontmatter 格式，头尾的 `---` 是否正确包裹。
  - 检查是否存在名字冲突。

### 切点 2: 审计 Description 的路由逻辑
如果 Agent 在列表中但无法自动被唤醒，请尝试输入：
```
描述一个会触发该 subagent 的代码修改情境，但不要直接输入它的名字，看路由器是否能指派正确。
```
若主 Agent 始终没有 spawn 动作，需要优化 frontmatter 中的 `description`，加入更清晰的 `PROACTIVELY` 触发边界。

### 切点 3: 确认工具白名单 (tools whitelist)
如果 subagent 报错“I don't have access to tool X”：
检查 YAML frontmatter 中的 `tools:` 白名单。子 Agent 的权限是默认受限的，如果你需要它读取文件、使用 grep 搜索或运行本地测试，必须明写赋予权限：

```yaml
tools:
  - Read
  - Grep
  - Glob
  - Bash        # 用于运行本地测试命令 (如 pytest)
  - WebFetch    # 允许其读取外部网页
```
> ⚠️ **安全 Gotcha**：如果将 `tools:` 字段省略或写空，subagent 将继承主会话的**全部**工具权限（包括直接修改文件等高危操作），存在沙箱安全隐患。

### 切点 4: 模型与 Token 成本控制
默认情况下，如果没有在 frontmatter 中指定 `model:`，子 Agent 将继承当前主会话所配置的模型。为防止消耗过大，建议为不需要超强推理的审查或检索工具配置轻量模型：

```yaml
model: gemini-2.5-flash   # 绝大多数常规检索和检查使用 Flash 模型即可
```

### 切点 5: 确保 Prompt 具有独立性 (Self-Contained)
**子 Agent 的进程是完全没有主会话历史记忆的**。

❌ **错误写法**：
```
帮我审查一下我们刚才讨论的修改。
```
子 Agent 不知道“刚才讨论的”是什么。

✅ **正确写法**：
```
Review the staged changes in this repo (git diff --cached) for any security vulnerabilities.
```
这保证了 Prompt 传入后，子进程凭借这段话本身就足以运行。
