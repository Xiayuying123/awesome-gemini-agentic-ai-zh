# Subagent Cookbook — 15 个复制粘贴就能用的派遣 recipe

> [繁體中文](./subagent-cookbook.md) | **简体中文** | [English](./subagent-cookbook.en.md)

> 📋 **这是什么**：[Stage 5.5](../stages/05-gemini-skills-ecosystem.zh-Hans.md#55--subagentsantigravity-cli-原生多-agent-机制) 讲述了 subagent 的核心概念，这份 cookbook 帮助你**今天就能用**——15 个常见场景，每个都包含“该用哪个 subagent + 完整 prompt 模板（复制即可用）+ 何时不用”。
>
> ⚠️ **第一次看？先读 Stage 5.5 的“可派遣 the subagent 有哪些”+“decision table”两节**——理解“什么是 subagent”“Antigravity CLI 内置有哪些”之后再来查 recipe。

---

## 怎么读这份 cookbook

每个 recipe 都采用同样的 4 段结构：

| 结构 | 内容 | 作用 |
|---|---|---|
| **场景** | 你今天工作会遇到的具体场景 | 从“我有 X 问题”找到匹配 the recipe，而不是从“我想用 subagent”找起 |
| **Subagent** | 用哪一个（Antigravity CLI 内置名称） | 直接复制名字，免去挑选烦恼 |
| **Prompt 模板** | 复制粘贴即可用的指令文字 | 无需自己反复打磨提示词 |
| **何时不用** | 比用 subagent 更好的替代方案 | 避免“大材小用”，节省 Token 与时间 |

> 💡 **怎么实际派遣 subagent**：在你的 Antigravity CLI 交互式对话框里，**直接输入（或粘贴）prompt 模板**——仅此应用。Agent 看到指令，会自动通过内部 the Task 调度机制找到对应 subagent 独立启动运行，跑完后向主 session 回报一段摘要。**不需要 slash command，不需要特殊语法**。
>
> 📌 **subagent ≠ slash command**：`/agents` 是列表与管理器命令，**不是调用 subagent 的方式**；派遣 subagent 直接输入普通 prompt 对话文字即可。完整对比表（subagent vs skill / vs slash command）见 [Stage 5.5](../stages/05-gemini-skills-ecosystem.zh-Hans.md#55--subagentsantigravity-cli-原生多-agent-机制)。

---

## 先确认你有哪些 subagent 可用

在你的 Antigravity CLI 会话里运行 `/agents` 这个指令，会列出全部当前可用的 subagent（内置 + 插件 + 自定义）。

**Antigravity CLI 默认内置 7 个** subagent：`general-purpose` / `code-reviewer` / `Explore` / `Plan` / `frontend-developer` / `antigravity-guide` / `statusline-setup`（可能会随版本变动）。

---

## 15 个 Recipe

### Recipe 1: 写完一批新 code，要 commit 前的 review

**场景**：你刚写了 ≥ 50 行新 code，想 commit 但怕有 bug / 漏洞或风格违规。

**Subagent**：`code-reviewer`

**Prompt 模板**：
```
Review the staged changes in this repo (git diff --cached).
Focus on: (1) security issues (hardcoded secrets, SQL injection, XSS),
(2) error handling gaps, (3) missing tests, (4) violations of GEMINI.md
conventions. Per-category PASS/FAIL + concrete fix for each issue with
file:line reference + overall verdict (APPROVE / REQUEST CHANGES).
```

**何时不用**：< 20 行的简单拼写或格式修正（直接自己看一眼就行，无需额外开销）。

---

### Recipe 2: 进入新 repo，不知道该从哪个 file 开始读

**场景**：克隆完一个新 repo，`README.md` 讲不清楚程序入口在哪，不想自己盲目摸索。

**Subagent**：`Explore`

**Prompt 模板**：
```
Map the entry points and core structure of this codebase. Report:
(1) main entry script(s) — what file gets run first,
(2) core module organization — top 3-5 directories and what each does,
(3) test directory layout,
(4) any "where to start reading" guidance in docs/.
Under 300 words with file paths.
```

**何时不用**：你已经知道要看的文件路径（直接使用 Read 工具阅读即可）。

---

### Recipe 3: 设计 refactor / migration plan（先想清楚再动手）

**场景**：要重构一个大模块或进行依赖框架迁移，不确定步骤，想先梳理出规划再开始。

**Subagent**：`Plan`

**Prompt 模板**：
```
Design a step-by-step plan to refactor <module-name> (currently a single
file with tight coupling) into clear, testable components with explicit
interfaces. Include: (1) phased breakdown (≤ 5 phases), (2) which files
touched per phase, (3) what tests gate each phase, (4) rollback strategy
if a phase fails. Don't write code — just the plan.
```

**何时不用**：重构非常轻微，只改动 1-2 个文件。

---

### Recipe 4: 多文件 / 跨语言 parity 审查

**场景**：你修改了不同语言版本的镜像文档，想确认各方内容与排版格式一致。

**Subagent**：`code-reviewer`

**Prompt 模板**：
```
Review the staged diff for cross-locale parity across the 3 locale
variants of <file-stem> (.md / .zh-Hans.md / .en.md). Check: (1) same
section structure (## headers match), (2) same table row counts, (3)
same required terms present in each locale, (4) locale conventions
correct (zh-Hans uses "" not 「」; en uses English). Report per-file
PASS/FAIL.
```

---

### Recipe 5: 多数据源 fact-check / 交叉验证

**场景**：需要写一段引用，不确定多个源文件讲的是否一致，需要交叉验证。

**Subagent**：`general-purpose`

**Prompt 模板**：
```
Fact-check this claim: "<claim>". Search for: (1) primary source / official
docs / paper, (2) 2-3 independent secondary sources, (3) any contradictions
or version differences. Report: confirmed / contradicted / nuanced, with
direct quotes and URLs. Under 400 words.
```

---

### Recipe 6: 找某个 symbol / function 在哪定义

**场景**：代码中频繁导入了某函数，想快速在庞大代码库中定位其定义与作用。

**Subagent**：`Explore`

**Prompt 模板**：
```
Find where `<symbol-name>` is defined in this codebase. Report: (1) the
file:line where it's defined, (2) what it does (1-line summary), (3) which
files import / use it (top 5). Use Grep, not full file reads.
```

---

### Recipe 7: 跨多篇学术论文的比对

**场景**：读完 3-5 篇大体积 PDF，需要横向比对它们的模型方案差异。

**Subagent**：`general-purpose`

**Prompt 模板**：
```
Compare the core methodology across these papers: <file1.pdf>, <file2.pdf>,
<file3.pdf>. Report in a markdown table: (1) model parameters / scale,
(2) hardware setup used for training, (3) primary optimization metric,
(4) stated performance improvement. Keep it concise.
```

---

### Recipe 8: 发版前的代码安全性审计

**场景**：项目即将发布大版本，想在 commit 进主分支前做最后一次漏洞审计。

**Subagent**：`code-reviewer`

**Prompt 模板**：
```
Pre-release security audit for <release-tag> (use git log to find the
relevant commit range since last release). Check: (1) hardcoded
credentials / API keys, (2) eval() / exec() / shell injection risks,
(3) deprecated dependencies with known CVEs, (4) any auth / session
handling changes since last release, (5) public API surface changes
(breaking? documented?). Per-category PASS/FAIL + remediation per
finding.
```

---

### Recipe 9: 评估架构变动的影响范围 (Blast Radius)

**场景**：你想改动一个底层的基类或公共工具函数，需要准确核算受灾面。

**Subagent**：`Plan`

**Prompt 模板**：
```
Assess the blast radius of changing <component>. Report: (1) direct
dependents (which files import this), (2) indirect dependents (transitive
imports), (3) tests that gate the change, (4) suggested rollout order
(safest -> riskiest), (5) feature flag / kill switch strategy if available.
Don't make the change yet — just the impact analysis.
```

---

### Recipe 10: 针对多个目标文件并行派发 Subagent

**场景**：你想对多个文件同时运行独立的风格审查。

**Subagent**：`general-purpose` × N

**Prompt 模板**（每次修改 `<file-path>` 并依次发送）：
```
Audit `<file-path>` for academic-style issues: (1) over-engineering
jargon without first-use explanation, (2) clarity (long sentences,
vague pronouns), (3) unsupported % claims, (4) persona-fit (wrong
technical level for the file's stated target audience — see the
file's banner / intro callout for who it's for). Report 4-category
PASS/FAIL + fix-list with line numbers. Under 500 words.
```

---

### Recipe 11: 找跨仓库的相似实现

**场景**：在项目 A 看到某种优秀的设计实践，想看项目 B 是否有类似逻辑。

**Subagent**：`Explore`

**Prompt 模板**：
```
In <repo-path>, search for code patterns similar to: <description of
pattern, e.g., "retry decorator with exponential backoff">. Report:
(1) up to 5 candidates with file:line, (2) brief description of each,
(3) which is most idiomatic for this codebase's style.
```

---

### Recipe 12: 大批量测试结果自动化评审 (LLM-as-judge)

**场景**：运行完大批量测试用例，想让 LLM 自动评审模型输出是否符合 spec。

**Subagent**：`general-purpose`

**Prompt 模板**：
```
Evaluate the agent outputs in <eval-file.jsonl> against the spec stated
in the file's first 5 lines (or in <spec-file.md>). For each row: PASS /
FAIL with 1-sentence reason. Aggregate: pass rate + top-3 failure modes.
Output as structured JSON: {"case_id": "...", "verdict": "PASS|FAIL",
"reason": "..."}.
```

---

### Recipe 13: UI 组件与无障碍 (Accessibility) 专项设计评审

**场景**：新写了一个交互复杂的 React 组件，要评估其无障碍与响应式指标。

**Subagent**：`frontend-developer`

**Prompt 模板**：
```
Audit <component-file> for: (1) ARIA roles + labels (screen reader
compatibility), (2) keyboard navigation (tab order, Enter / Esc / arrows
behavior), (3) responsive breakpoints (mobile 360px / tablet 768px /
desktop 1280px), (4) color contrast (WCAG AA), (5) touch target size
(>= 44px). Report per-category findings + fixes.
```

---

### Recipe 14: 查询 Antigravity CLI 的功能使用指南

**场景**：忘记了 Hooks 如何配置，或者不确定某种 slash command 的用法。

**Subagent**：`antigravity-guide`

**Prompt 模板**：
```
How do I <specific Antigravity CLI feature question, e.g., "configure a
PreToolUse hook to block dangerous bash commands">? Show: (1) minimum
config in settings.json, (2) example hook script (Python or shell),
(3) 1 common gotcha, (4) where in the official docs to read more.
```

---

### Recipe 15: 编写复杂的 React 表单校验逻辑

**场景**：需要开发一个包含复杂校验的注册表单组件。

**Subagent**：`frontend-developer`

**Prompt 模板**：
```
Implement a React sign-up form with: (1) email format validation
(real-time, debounce 300ms), (2) password strength meter (>= 8 chars,
mixed case, digit, symbol), (3) inline error messages with ARIA
live region, (4) submit button disabled until valid. Use <library, e.g.,
React Hook Form + Zod>. Include: component code, validation schema,
1 happy-path test, 1 error-path test.
```

---

## Recipe 索引（按 subagent 种类反查）

| Subagent | 关联 Recipes |
|---|---|
| `code-reviewer` | **1**（提交前 review）、**4**（多语言比对）、**8**（发版前安全审计） |
| `Explore` | **2**（项目结构探索）、**6**（定位 symbol）、**11**（跨库相似代码查找） |
| `Plan` | **3**（重构计划书）、**9**（影响面 Blast Radius 评估） |
| `general-purpose` | **5**（fact-check）、**7**（论文横向比对）、**10**（并行多目标审查）、**12**（LLM 自动化裁判） |
| `frontend-developer` | **13**（无障碍与适配审计）、**15**（React 表单业务开发） |
| `antigravity-guide` | **14**（CLI 功能与配置查询） |

---

## 何时 NOT 该用 subagent

虽然 Subagent 能实现高度的上下文隔离与任务并行，但在以下场景中**应避免使用**，直接在主会话中运行效率更高：

1. **极其简短、数分钟内即可人工或直接处理的任务**：频繁 spawn subagent 会带来启动开销和 Token 浪费。
2. **需要用户频繁在中间交互确认、提供实时输入反馈的任务**：Subagent 是“全自动派发-最终合并回报”的模式，中途无法进行人机交互。
3. **需要强依赖当前主会话历史记忆的任务**：Subagent 运行在完全隔离的新 context 空间中，无法读取当前主会话“上面我们刚刚说过的...”等局部上文。
4. **决策本身偏向宏观系统架构探讨或发散式讨论**：这种场景下需要反复权衡、人机协作沟通，不适合丢给封闭闭环运行的 subagent。
