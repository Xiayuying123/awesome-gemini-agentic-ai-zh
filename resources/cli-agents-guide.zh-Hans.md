> [繁體中文](./cli-agents-guide.md) | **简体中文** | [English](./cli-agents-guide.en.md)

# CLI Agents 比较指南

> [← 回主路线 README](../README.md)

> 📌 **这份是 reference doc**（深度比较、选择逻辑、坑、推荐搭配）。
> 第一次接触 CLI agent、想要 step-by-step 上手 → 看 [`tracks/cli/A1-cli-intro.zh-Hans.md`](../tracks/cli/A1-cli-intro.zh-Hans.md)（Track A 第一站）。
> 想先理解“为什么有的 agent 在 terminal、有的在 Telegram、有的在 Jetson”这层 mental model → 看 [`resources/agent-paradigms.zh-Hans.md`](agent-paradigms.zh-Hans.md)（5 种 agent 型态）。
> 已经在用、想决定 / 比较 / 升级 → 留在这份。

跨 5 个 branch + Track A 共用的参考——**Antigravity CLI / Claude Code / Codex / OpenCode / goose / Aider / Hermes Agent 之间怎么挑？** Track A（A1-A3）的 CLI workflow 设计、5 条 branch 内的 CLI 引用都连到这份；每个 branch 都会用到 CLI agent，但没有一个 branch 真的“拥有”这份比较，所以放在 `resources/`。

## 📋 7 个主流 CLI agent

只列在 terminal 跑的（IDE-based 如 Cursor / Cline / Continue 不在这份；那些放在 [for-developer](../branches/for-developer.zh-Hans.md)）。前 6 个数字 `gh api` 验证于 2026-06-01；Hermes Agent 验证于 2026-06-05。

| 工具 | 提供者 | License | 主推 LLM | 认证 / 计费 | Stars |
|---|---|---|---|---|---|
| [Antigravity CLI](https://antigravity.google) | Google（官方） | Apache-2.0 | Gemini 系列 | Google Cloud 凭据或 Gemini API key | ★ 110k+ |
| [Claude Code](https://github.com/anthropics/claude-code) | Anthropic（官方） | NOASSERTION | Claude | Claude 订阅 **或** Anthropic Console API key | ★ 120k+ |
| [Codex](https://github.com/openai/codex) | OpenAI（官方） | Apache-2.0 | GPT 系列 | ChatGPT 账号登录 **或** OpenAI API key | ★ 80k+ |
| [OpenCode](https://github.com/sst/opencode) | 社群（repo 已迁至 `anomalyco/opencode`） | MIT | 任意（多 provider） | BYO API key 或 OpenCode Zen 内建 hosted | ★ 155k+ |
| [goose](https://github.com/block/goose) | Agentic AI Foundation（repo 已迁至 `aaif-goose/goose`） | Apache-2.0 | 15+ provider（含 Ollama） | BYO API key 或既有 ACP 订阅 | ★ 43k+ |
| [Aider](https://github.com/Aider-AI/aider) | Aider-AI（社群） | Apache-2.0 | BYO API key | ★ 44k+ |
| [Hermes Agent](https://github.com/NousResearch/hermes-agent) | Nous Research | MIT | 多模型接入 | BYO API key（多 provider） | ★ 175k+ |

---

## 🎯 该选哪个？依 use case 决定

### 使用 Google 生态 + 想要 1M+ token 超长 context
**首推**：Antigravity CLI (`agy`)。它针对 Gemini 模型进行深度优化，拥有百万 tokens 的上下文支持，并提供优秀的 Hooks (`hooks.json`) 与原生 Subagents 并发调度机制。

### 写 paper / 文献 / 研究
**首推**：Antigravity CLI 或 Claude Code（长 context、reasoning 强、安全限制优秀）。Gemini 模型的超长 token 适合一次性丢整本 PDF 或大型数据集进去问。

### 写 code / 改 codebase
**首推**：Aider（git-native——每次改完自动 commit，方便 revert）或 Antigravity CLI。OpenCode 适合需要在多 LLM 间切的场景。

### 隐私 / offline / 不送云端
**首推**：goose 或 OpenCode + 本地 Ollama。两个都支持 BYO LLM，可以接 `http://localhost:11434/v1`（Ollama 默认）。

---

## 📝 跨 CLI 都通用的 prompt 写法

如果想让 prompt 在不同 CLI 之间 portable（或想随时换工具不重写），照这几条原则：

1. **明确指定文件路径**——“修改 `src/auth.py`”比“修改那个 auth 档”好。
2. **要求多步骤拆解**——`先列 plan、确认后再动手`，所有 CLI 都吃这个结构。
3. **避免依赖特定 CLI 的 magic 指令**——`/resume` 是特定的，不同客户端不支持。
4. **用项目规则文件规范偏好**——Claude Code 用 `CLAUDE.md`，Antigravity CLI 使用 `GEMINI.md`，Codex 用 `AGENTS.md`，**内容可以相通**。
5. **明确要 review 的 scope**——“只 review 我这次的 diff” vs “review 整个 repo”。

---

## 🔧 实用搭配（real-world setup）

### Setup A：Antigravity CLI 主推 + OpenCode 备援
- Antigravity CLI 处理日常 90% 的云端 Gemini 推理与大范围 codebase 分析。
- OpenCode + Ollama，处理完全隐私敏感的本地数据。

### Setup B：Antigravity CLI 与 Aider 配合
- 利用 Antigravity CLI 的超长 context 做大模块重构前的架构规划与分析。
- 配合 Aider 完成局部的 git-native 代码提交与反复微调。
