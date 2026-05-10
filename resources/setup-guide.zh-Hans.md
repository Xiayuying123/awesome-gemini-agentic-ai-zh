> [繁體中文](./setup-guide.md) | **简体中文** | [English](./setup-guide.en.md)

# 🚀 从零开始 — 给没有开发背景的设置指南

> [← 回主路线 README](../README.zh-Hans.md)

> 预估时间：30-45 分钟。你会申请第一个 API key、装好 Python / uv，并跑出第一个 LLM hello world。
> 这份文档写给“想学 AI agent，但还没写过 code”的人。已经熟 Python / git / CLI 的开发者，可以直接跳 [Stage 1](../stages/01-llm-basics.zh-Hans.md)。

## 先决定两件事

1. **你想自己写 code 吗？**
   - 想：走完 §A-C，然后接 [Stage 1](../stages/01-llm-basics.zh-Hans.md) 或 [for-developer](../branches/for-developer.zh-Hans.md)。
   - 还不确定：先走完 §A-B，再按身份看 [日常用户](../branches/for-everyday-users.zh-Hans.md)、[教师](../branches/for-teacher.zh-Hans.md)、[知识工作者](../branches/for-knowledge-worker.zh-Hans.md)、[研究者](../branches/for-researcher.zh-Hans.md)。
2. **你想先用哪家 LLM？**
   - 最省心：Anthropic Claude，界面清楚，英文和中文都好用。
   - 已经常用 ChatGPT：OpenAI，需要另外到 API 平台申请 key。
   - 想先不付云端费用：Ollama 本地模型，步骤多一点，但不需要 API key。

API key 可以先理解成“让程序调用模型的密码”。请把它当成信用卡资料保管。

---

## §A — 申请第一个 API key（约 10 分钟）

### Anthropic Claude（推荐第一次）

1. 打开 https://console.anthropic.com/
2. 用 Google、GitHub 或 email 注册。
3. 进入账号后找到 **API Keys**，点 **Create Key**。
4. **立刻复制显示出的 key**。多数平台只会显示一次。
5. 先放在本机密码管理器，或短暂放在本机文本文件；下一节会移到 `.env`。

> ⚠️ **API key 三不规则**
> - **不贴**到 chat 窗口、群组、email 或截图。
> - **不上传**到 git；GitHub 可能扫到后自动撤销。
> - **不放**云端硬盘纯文本文件；同步到其他设备等于多一份风险。

### 其他 LLM 选项

- **OpenAI**：https://platform.openai.com/api-keys  
  ChatGPT Plus 和 API key 是两件事；订阅 Plus 仍要另外申请 API key。
- **Google AI Studio**：https://aistudio.google.com/  
  适合先试 Gemini API，免费额度会依地区和账号状态不同。
- **Ollama 本地模型**：不用 API key。走本地路线请看 [Cookbook Recipe 6](cookbook.zh-Hans.md#6-本地-llm--cli-agent-快速-walkthrough)。

---

## §B — 装本机环境（约 10 分钟）

### 装 Python 3.10+

- **macOS**：打开 Terminal，输入 `brew install python@3.12`。如果还没有 Homebrew，先看 https://brew.sh。
- **Windows**：到 https://www.python.org/downloads/ 下载 installer，安装时一定要勾 **Add Python to PATH**。
- **Linux**：Ubuntu 用 `sudo apt install python3 python3-venv`，Fedora 用 `sudo dnf install python3`。
- **验证**：macOS / Linux 输入 `python3 --version`；Windows 输入 `py --version`。看到 `Python 3.10` 以上即可。

### 装 uv

uv 是 Python 包管理工具。你可以把它想成“帮你临时装好需要的包再执行”的工具。

```bash
# macOS / Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows PowerShell
irm https://astral.sh/uv/install.ps1 | iex
```

验证：

```bash
uv --version
```

### 建立第一个 `.env` 文件

在你要跑 script 的文件夹里，建立一个文件名叫 `.env` 的文件：

```bash
ANTHROPIC_API_KEY=sk-ant-...贴上你刚才复制的 key
```

`.env` 是专门放本机秘密信息的文件。程序会读它，但你不应该把它上传到 GitHub。

### 加上 `.gitignore`

同一个文件夹建立 `.gitignore`：

```gitignore
.env
__pycache__/
*.pyc
```

这样 git 就不会把 `.env` 收进版本记录。

---

## §C — 跑第一个 `hello-claude.py`（约 5 分钟）

建立 `hello-claude.py`：

```python
from anthropic import Anthropic
from dotenv import load_dotenv

load_dotenv()
client = Anthropic()  # 自动读取 ANTHROPIC_API_KEY

msg = client.messages.create(
    model="claude-sonnet-4-5",
    max_tokens=100,
    messages=[{"role": "user", "content": "Hello, who are you?"}],
)

print(msg.content[0].text)
```

执行：

```bash
uv run --with anthropic --with python-dotenv python hello-claude.py
```

看到 Claude 回复自我介绍，就代表你的 API key、Python、包都通了。

### 常见错误

| 错误信息 | 常见原因 | 解法 |
|---|---|---|
| `401 Unauthorized` | API key 没读到或打错 | 回 §A 重新复制，确认 `.env` 文件名和内容 |
| `429 Rate limit` | 太快发太多请求 | 等几秒或几分钟再跑 |
| `connection refused` | 网络或防火墙问题 | 确认网络、公司或学校防火墙 |
| `ModuleNotFoundError` | 包没有被安装 | 确认执行的是上面的 `uv run --with ...` 命令 |

---

## §D — 第一次装 Claude Code（约 10 分钟；Stage 5 / for-developer 会用到）

### 先装 Node.js

- **macOS / Linux**：`brew install node`，或从 https://nodejs.org 下载。
- **Windows**：从 https://nodejs.org 下载安装包。
- **验证**：输入 `node --version`，看到 v18 以上即可。

### 装 Claude Code

```bash
npm install -g @anthropic-ai/claude-code
```

### 第一次认证

```bash
claude
```

第一次启动时通常会让你选：

- **Claude subscription**：用 Claude.ai 账号登录，对初学者最省事。
- **API key**：贴上 §A 申请到的 key。

### 建立第一份 `CLAUDE.md`

在你的 project 根目录建立 `CLAUDE.md`。Claude Code 启动时会读它，理解你希望它怎么协助。

```markdown
# 你是谁
我是 [你的名字]，[你的领域，例如：教师 / 研究者 / 写作者]。

# Code style
- 注释用简体中文写，code 用英文
- 写 function 时优先加 type hint
- 不要主动 commit；改完让我手动 git add

# 不准做的事
- 不要联网查资料，除非我明确说可以
- 不要动 `.env` 或 `.gitignore`
- 不要删文件夹，包括子文件夹
```

---

## §E — 第一个 Skill 示例（约 5 分钟；Stage 5.3 会用到）

Skill 是 Claude Code 的“可复用 prompt 包”。当你的消息符合描述，Claude Code 会自动加载那份指示。

建立 `.claude/skills/hello-skill/SKILL.md`：

```markdown
---
name: hello-skill
description: 第一个 hello skill。当用户说“请打招呼”或“say hi”时触发。
---

当用户请你打招呼时，回三件事：

1. 用简体中文和英文各说一次 hello
2. 提现在的日期（用 system 时间）
3. 给一个今日小提醒（随机选健康 / 学习 / 心情建议）
```

跑 `claude`，输入“请打招呼”。如果 Claude 回复三件事，就代表 Skill 被加载了。

> 想看更完整的 Skill 设计：看 [Stage 5.3 — Skills](../stages/05-claude-code-ecosystem.zh-Hans.md#53--skillsclaude-code-的行为层)。
> 想看可以照做的示例：看 [Cookbook](cookbook.zh-Hans.md)。

---

## 接下来去哪

| 你现在的状态 | 下一步 |
|---|---|
| 想正式理解 LLM、API、token | [Stage 1 — LLM 基础](../stages/01-llm-basics.zh-Hans.md) |
| 想直接挑身份分支 | [日常用户](../branches/for-everyday-users.zh-Hans.md) / [教师](../branches/for-teacher.zh-Hans.md) / [知识工作者](../branches/for-knowledge-worker.zh-Hans.md) / [研究者](../branches/for-researcher.zh-Hans.md) / [开发者](../branches/for-developer.zh-Hans.md) |
| 想看 Claude Code 完整生态 | [Stage 5 — Claude Code 生态](../stages/05-claude-code-ecosystem.zh-Hans.md) |
| 想本地 LLM、不用云端 key | [Cookbook Recipe 6](cookbook.zh-Hans.md#6-本地-llm--cli-agent-快速-walkthrough) |
| 想比较 CLI agent | [CLI Agents 比较指南](cli-agents-guide.zh-Hans.md) |
| 不懂某个用词 | [Glossary](glossary.zh-Hans.md) |
