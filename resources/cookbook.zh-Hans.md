# Cookbook — 把概念变成可执行的 recipe

> [繁體中文](./cookbook.md) | **简体中文** | [English](./cookbook.en.md)

> Stage 5（Antigravity CLI 生态）与 [`mcp-skills-catalog.md`](mcp-skills-catalog.zh-Hans.md) 讲述了“概念”与“有哪些工具”。这份 cookbook 补充中间缺失的环节：“**怎么动手做出来**”。每个 recipe 是一份 step-by-step + sample code + 常见 pitfall，约 30-50 分钟可以完成一个。
>
> 这不是单纯的 reference，也不是长篇 tutorial——它是 recipe，挑你需要的那道烹饪即可。

---

## 📋 目录

1. [写你的第一个 Skill（SKILL.md 结构）](#1-写你的第一个-skill)
2. [写你的第一个 MCP server（Python SDK）](#2-写你的第一个-mcp-server)
3. [Word / Excel / PowerPoint workflow](#3-office-docs-workflow)
4. [NotebookLM workflow](#4-notebooklm-workflow)
5. [Zotero workflow](#5-zotero-workflow)
6. [本地 LLM + CLI Agent 快速 walkthrough](#6-本地-llm--cli-agent-快速-walkthrough)

---

## 1. 写你的第一个 Skill

> Skill = 一个包含 `SKILL.md` 的文件夹，Antigravity CLI 启动时会自动加载，并按情境触发。最简 viable 版本 50 行就能运行。
>
> 📚 **这份是“30 分钟跑出第一个”实操版。想看“Skill 怎么写得好”深度讨论** → [Hello-Agents Extra08：如何写出好的 Skill](https://github.com/datawhalechina/hello-agents/blob/main/Extra-Chapter/Extra08-如何写出好的Skill.md)（讨论 description 写法、references / scripts 设计等）。

### 为什么

编写 Skill 与“在 prompt 里加几段 instruction”的差别在于：
- Skill 是 **per-domain (特定领域)** 的，不会污染所有会话上下文。
- 可以打包跨项目/团队共用。
- Agent 自己决定何时加载（看 frontmatter description 匹配程度）。

### 步骤

#### Step 1：创建 skill 文件夹

两个位置可以放（看你需要用户级还是项目级）：

```bash
# 用户级（所有项目共用）
mkdir -p ~/.gemini/antigravity-cli/skills/my-first-skill
cd ~/.gemini/antigravity-cli/skills/my-first-skill

# 或 项目级（只在当前 repo 触发）
mkdir -p .agents/skills/my-first-skill
cd .agents/skills/my-first-skill
```

#### Step 2：写 `SKILL.md`

最小可工作的模板：

```markdown
---
name: my-first-skill
description: When the user asks for [SPECIFIC SITUATION], use this skill to [WHAT IT DOES]. Examples include [2-3 trigger phrases]. Do NOT use for [WHAT IT'S NOT FOR].
---

# My First Skill

You are now in the [domain] context.

## When the user asks X, do these steps:

1. First, [action A]
2. Then, [action B]
3. Verify with [check]

## Don't do:

- [anti-pattern 1]
- [anti-pattern 2]

## Reference

- (optional) link to a doc / paper / API spec
```

具体例子：“整理 Python 代码的 import 顺序”

```markdown
---
name: python-import-organizer
description: When the user pastes Python code or asks to clean up imports / format code / sort imports, organize the imports following PEP 8 + isort order: stdlib first, then third-party, then local. Do NOT use for non-Python code.
---

# Python Import Organizer

When the user wants Python imports cleaned up:

1. Group imports into 3 sections: stdlib / third-party / local
2. Within each group, sort alphabetically
3. Add a blank line between groups
4. Remove unused imports (only if user explicitly asks; otherwise just sort)

## Don't:
- Don't change function code, only the import block
- Don't auto-remove imports without asking
```

#### Step 3：测试

```bash
# 重启 Antigravity CLI (agy) 让它重新加载 skills
# 在会话中输入触发句
# 例如：「帮我整理一下这段 Python 的 imports」
# 观察 agent 是否按照 SKILL.md 的步骤执行
```

#### Step 4（进阶）：加 evals

在 skill 文件夹内加 `evals/evals.json`：

```json
{
  "evals": [
    {
      "input": "整理一下这段 Python 的 imports: import os\nimport requests\nfrom mypackage import foo",
      "expected_behavior": ["按 stdlib / third-party / local 分组", "alphabetical 排序"]
    }
  ]
}
```

之后可以用自动化测试工具跑集成测试。

### 常见 pitfall

| 症状 | 原因 | 解法 |
|---|---|---|
| Agent 从不触发我的 skill | description 写得太笼统，匹配不到用户的 query | description 加 2-3 个具体 trigger phrase（"when the user asks X / Y / Z"） |
| 触发了但行为不对 | SKILL.md 步骤太抽象 | 改成 numbered list、每步明确动作 |
| 触发了不该触发的情境 | description 太宽，匹配到不相关 query | 加 "Do NOT use for X" 收敛触发范围 |

### 进一步

- 查看 [Stage 5.3](../stages/05-gemini-skills-ecosystem.zh-Hans.md#53--skillsantigravity-cli-的行为层) 的 Skill 结构详解
- 查看 [`google/antigravity-skills`](https://github.com/google/antigravity-skills) 官方 skill 模板写法
- 多个 skill 打包成 plugin → [Stage 5.4](../stages/05-gemini-skills-ecosystem.zh-Hans.md#54--plugins-与打包分发)

---

## 2. 写你的第一个 MCP server

> MCP server = 一个独立进程，跑起来提供 tool / resource / prompt 给 LLM host（如 Antigravity CLI 客户端）。最小可运行版 < 50 行 Python。

### 为什么

- Skill 是给 Agent 的“角色 + 规则”；MCP 是给 Agent 赋予的“**外部工具与 API**”。
- Skill 自身无法独立发送网络请求或操作复杂系统 API，而 MCP server 可以运行任何复杂的本地或远程代码。
- Skill 绑定在当前 CLI 客户端，而 MCP 可以在任何支持 MCP 的 Host（如 Cursor、Cursor-agent 等）间复用。

### 步骤

#### Step 1：安装官方 SDK

```bash
pip install mcp
```

#### Step 2：写 `server.py`

最小模板——一个会回 echo 的 tool：

```python
# server.py
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp.types import Tool, TextContent

app = Server("hello-mcp")

@app.list_tools()
async def list_tools() -> list[Tool]:
    return [
        Tool(
            name="echo",
            description="Echo the input text back to the user.",
            inputSchema={
                "type": "object",
                "properties": {
                    "text": {
                        "type": "string",
                        "description": "Text to echo back",
                    }
                },
                "required": ["text"],
            },
        )
    ]

@app.call_tool()
async def call_tool(name: str, arguments: dict) -> list[TextContent]:
    if name == "echo":
        return [TextContent(type="text", text=f"Echo: {arguments['text']}")]
    raise ValueError(f"Unknown tool: {name}")

async def main():
    async with stdio_server() as (read_stream, write_stream):
        await app.run(read_stream, write_stream, app.create_initialization_options())

if __name__ == "__main__":
    import asyncio
    asyncio.run(main())
```

#### Step 3：将 MCP Server 添加到 Antigravity CLI

你可以通过在 `~/.gemini/antigravity-cli/settings.json`（或项目局部的 `.agents/settings.json`）中写入配置，或者在一个 plugin 目录下的 `mcp_config.json` 中添加它：

```json
{
  "mcpServers": {
    "hello-mcp": {
      "command": "python",
      "args": ["/absolute/path/to/server.py"]
    }
  }
}
```

或者使用 `/mcp` 交互式命令面板来管理 MCP 服务。

#### Step 4：测试

启动 `agy`，尝试向它发起提问：

```
你问：用 echo 工具把 "hello world" 发送给我
Agent 回：Echo: hello world (同时会展示调用工具图标)
```

### 常见 pitfall

| 症状 | 原因 | 解法 |
|---|---|---|
| Agent 找不到该 tool | `server.py` 路径错或启动失败 | 终端尝试手动 `python server.py` 看是否有语法报错或异常 |
| tool 列出来但执行失败 | `inputSchema` 格式错 | 查看 [`schema-design-cheatsheet.md`](schema-design-cheatsheet.zh-Hans.md) 的标准 JSON Schema |
| Agent 拒绝自动调用 tool | tool description 描述不清晰 | 将 description 改写为“When the user asks X, use this tool”等更明确的语意触发 |

### 进一步

- 查看 [Stage 5.2](../stages/05-gemini-skills-ecosystem.zh-Hans.md#52--mcpmodel-context-protocol-基础) 的 MCP 规范
- 查看 [`modelcontextprotocol/servers`](https://github.com/modelcontextprotocol/servers) 官方示例（filesystem、sqlite 等）

---

## 3. Office Docs Workflow

> 让 Agent 读写 Word / Excel / PowerPoint / PDF，许多现成的 Skills 和插件已经提供了优秀的实现。

### 为什么

最常见场景包括：
- 从 Markdown / 大纲自动生成格式美观的 Word 或 PPT 幻灯片。
- 读取 PDF / Excel 表格进行关键数字分析。
- 自动转换中文字符集，重排样式并保留原本的 track changes。
- 自动化生成商业分析报告。

### 步骤

#### Step 1：安装办公技能包

例如，克隆兼容的办公文档解析技能包到你的配置目录：

```bash
git clone https://github.com/google/antigravity-skills.git ~/.gemini/antigravity-cli/skills/office-skills
```

#### Step 2：重启会话

在 `agy` 中激活以下办公领域技能：
- 读写 Word (.docx)
- 读写 Excel (.xlsx)
- 读写 PPT (.pptx)
- 读写 PDF

#### Step 3：实用 Prompt 模版

**从大纲生 PPT**：
```
读取我的 outline.md，照此大纲生成一份 PPT 幻灯片：
- 封面页
- 每页展示一个 H2 主题，精简 bullet points
- 保存至 ./output/presentation.pptx
```

**从 Excel 汇总**：
```
分析 ./data/sales.xlsx 的第一张工作表，计算每个大区的 Q4 营业额，并将其写入 Markdown 表格文件 ./output/q4-summary.md 中。
```

---

## 4. NotebookLM Workflow

> NotebookLM 是 Google 的研究与 RAG 工具。虽然 Antigravity CLI 没有内建对 NotebookLM 的官方桌面集成，但我们可以使用社区方案把它的能力带进 CLI。

### 为什么

NotebookLM 的强大功能：
- 上传超大 PDF 文件群，构建精确无幻灯的 RAG 知识库。
- 问答引用自动锚定到具体文档的页码。
- 自动生成 Podcast 音频。

社区提供了两种结合方式：
1. **PleasePrompto/notebooklm-skill**：通过浏览器自动化脚本读取。
2. **teng-lin/notebooklm-py**：基于 Python API 接口的操作。

### 步骤

#### Step 1：使用 notebooklm-py
```bash
pip install notebooklm-py
```

#### Step 2：程序化脚本调用

```python
from notebooklm import NotebookLM
nlm = NotebookLM() # 完成 Google Cloud OAuth 认证

# 创建一个新的笔记本空间
nb = nlm.create_notebook("AI Agent Research")

# 批量添加文献
nb.add_source("papers/attention_is_all_you_need.pdf")

# 提出问题
response = nb.query("Explain the core contributions of this paper.")
print(response.text)
print(response.citations)
```

---

## 5. Zotero Workflow

> 科研文献管理工具 Zotero 可以结合 Zotero Skills 为 Antigravity CLI 赋能。

### 步骤

#### Step 1：开启 Zotero 本地 API 服务
Zotero 客户端默认需要配置开启：
- **Edit → Preferences → Advanced → Config Editor**
- 将 `extensions.zotero.httpServer.enabled` 设为 `true`
- 将 `extensions.zotero.httpServer.port` 设置为 `23119` (默认)

#### Step 2：安装 Zotero 技能包

```bash
git clone https://github.com/WenyuChiou/zotero-skills ~/.gemini/antigravity-cli/skills/zotero-skills
```

根据 repo 文档配置你的 API Key 以便进行远程库交互。

#### Step 3：运行 Prompt 测试

```
搜索我 Zotero 库中 2024 年以后、关于 Multi-Agent 架构的所有论文，并按时间排序，输出为 markdown 表格。
```

---

## 6. 本地 LLM + CLI Agent 快速 walkthrough

> 很多开发者在本地跑了 Ollama 等模型，也想通过 terminal 玩转 agentic coding 逻辑。

### 为什么

需要指出的是，**Antigravity CLI 原生设计是作为 Gemini API 的 Harness**，需要提供 Google API 密钥或云端 OAuth 认证。如果你想完全在本地离线运行 Agentic CLI 并结合本地模型，目前最直接的方案是使用 **OpenCode / Aider / goose** 等支持 BYO LLM 的开源客户端。

### 步骤

#### Step 1：使用 Ollama 启动本地模型
```bash
# 运行 Ollama
ollama pull qwen2.5:7b
ollama serve
```

确认本地接口是否能正常响应：
```bash
curl http://localhost:11434/v1/chat/completions \
  -H "Content-Type: application/json" \
  -d '{"model":"qwen2.5:7b","messages":[{"role":"user","content":"解释什么是 ReAct agent"}]}'
```

#### Step 2：使用 Aider 或 OpenCode 连接本地模型
以 **Aider** 为例：
```bash
pip install aider-chat
aider --model ollama/qwen2.5:7b --no-show-model-warnings
```

以 **goose** 为例：
```bash
goose configure # 选择 Ollama 作为 Provider，关联 qwen2.5:7b
goose session start
```

#### Step 3：与 Antigravity CLI 的对比

| 面向 | Antigravity CLI (agy) | OpenCode / Aider + Ollama |
|---|---|---|
| **底座 LLM** | Gemini Cloud 系列 (Flash/Pro) | 本地模型 (如 Qwen / Llama) |
| **推理与上下文** | 原生百万上下文支持，多轮长逻辑稳定 | 受本地显存与上下文窗口硬件限制 |
| **隐私性** | 云端托管，符合企业云数据规范 | `100%` 留在本地硬盘与内存 |
| **适用工作** | 整个复杂代码库分析、多文件重构 | 离线演示、隐私敏感文件操作、低成本实验 |
