> [繁體中文](./setup-guide.md) | [简体中文](./setup-guide.zh-Hans.md) | **English**

# 🚀 From Zero — Setup Guide for People Without a Dev Background

> [← Back to main README](../README.en.md)

> Expected time: 30-45 minutes. You will get your first API key, install Python / uv, and run your first LLM hello world.
> This guide is for people who want to learn AI agents but have not written code before. If you already know Python, git, and the CLI, you can skip to [Stage 1](../stages/01-llm-basics.en.md).

## Decide Two Things First

1. **Do you want to write code yourself?**
   - Yes: finish §A-C, then continue with [Stage 1](../stages/01-llm-basics.en.md) or the [developer branch](../branches/for-developer.en.md).
   - Not sure yet: finish §A-B, then pick the [everyday user](../branches/for-everyday-users.en.md), [teacher](../branches/for-teacher.en.md), [knowledge worker](../branches/for-knowledge-worker.en.md), or [researcher](../branches/for-researcher.en.md) branch.
2. **Which LLM do you want to start with?**
   - Lowest friction: Anthropic Claude. The interface is clear and works well in English and Chinese.
   - Already use ChatGPT often: OpenAI. You still need to create an API key on the API platform.
   - Prefer no cloud API cost at first: local models with Ollama. More steps, but no API key.

An API key is basically a password that lets a program call a model. Treat it like payment information.

---

## §A — Get Your First API Key (About 10 Minutes)

### Anthropic Claude (Recommended First)

1. Open https://console.anthropic.com/
2. Sign up with Google, GitHub, or email.
3. After login, find **API Keys**, then choose **Create Key**.
4. **Copy the key immediately**. Most platforms show it only once.
5. Put it in a local password manager, or briefly in a local text file; the next section moves it into `.env`.

> ⚠️ **Three API-key rules**
> - **Do not paste it** into chat windows, group chats, email, or screenshots.
> - **Do not upload it** to git; GitHub may detect and revoke it.
> - **Do not store it** as a plain text cloud-drive file; syncing creates more exposure.

### Other LLM Options

- **OpenAI**: https://platform.openai.com/api-keys  
  ChatGPT Plus and API access are separate; Plus subscribers still need an API key.
- **Google AI Studio**: https://aistudio.google.com/  
  Useful for trying the Gemini API. Free quota depends on region and account state.
- **Ollama local models**: no API key needed. For the local path, see [Cookbook Recipe 6](cookbook.en.md#6-local-llm--cli-agent-quick-walkthrough).

---

## §B — Install Your Local Environment (About 10 Minutes)

### Install Python 3.10+

- **macOS**: open Terminal and run `brew install python@3.12`. If Homebrew is not installed, start at https://brew.sh.
- **Windows**: download the installer from https://www.python.org/downloads/ and make sure **Add Python to PATH** is checked.
- **Linux**: on Ubuntu, run `sudo apt install python3 python3-venv`; on Fedora, run `sudo dnf install python3`.
- **Verify**: macOS / Linux: `python3 --version`; Windows: `py --version`. You want `Python 3.10` or newer.

### Install uv

uv is a Python package tool. For this guide, think of it as "install the packages I need, then run this script."

```bash
# macOS / Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows PowerShell
irm https://astral.sh/uv/install.ps1 | iex
```

Verify:

```bash
uv --version
```

### Create Your First `.env` File

In the folder where you want to run the script, create a file named `.env`:

```bash
ANTHROPIC_API_KEY=sk-ant-...paste the key you copied
```

`.env` is where local secrets live. Your program can read it, but you should not upload it to GitHub.

### Add `.gitignore`

In the same folder, create `.gitignore`:

```gitignore
.env
__pycache__/
*.pyc
```

This keeps git from recording your `.env` file.

---

## §C — Run Your First `hello-claude.py` (About 5 Minutes)

Create `hello-claude.py`:

```python
from anthropic import Anthropic
from dotenv import load_dotenv

load_dotenv()
client = Anthropic()  # Automatically reads ANTHROPIC_API_KEY

msg = client.messages.create(
    model="claude-sonnet-4-5",
    max_tokens=100,
    messages=[{"role": "user", "content": "Hello, who are you?"}],
)

print(msg.content[0].text)
```

Run it:

```bash
uv run --with anthropic --with python-dotenv python hello-claude.py
```

If Claude introduces itself, your API key, Python, and packages are working.

### Common Errors

| Error | Likely Cause | Fix |
|---|---|---|
| `401 Unauthorized` | API key is missing or mistyped | Copy it again from §A and check the `.env` filename and value |
| `429 Rate limit` | Too many requests too quickly | Wait a few seconds or minutes, then retry |
| `connection refused` | Network or firewall issue | Check your network, company firewall, or school firewall |
| `ModuleNotFoundError` | A package was not installed | Make sure you ran the exact `uv run --with ...` command above |

---

## §D — Install Claude Code for the First Time (About 10 Minutes; Needed for Stage 5 / for-developer)

### Install Node.js First

- **macOS / Linux**: run `brew install node`, or download from https://nodejs.org.
- **Windows**: download the installer from https://nodejs.org.
- **Verify**: run `node --version`; v18 or newer is enough.

### Install Claude Code

```bash
npm install -g @anthropic-ai/claude-code
```

### First Authentication

```bash
claude
```

On first launch, you will usually choose between:

- **Claude subscription**: sign in with your Claude.ai account. This is the simplest path for beginners.
- **API key**: paste the key you created in §A.

### Create Your First `CLAUDE.md`

Create `CLAUDE.md` at the root of your project. Claude Code reads it on startup so it understands how you want help.

```markdown
# Who you are
I am [your name], a [your field, such as teacher / researcher / writer].

# Code style
- Write comments in Traditional Chinese, and code in English
- Prefer type hints when writing functions
- Do not commit automatically; let me run git add myself

# Do not do these
- Do not browse the web unless I explicitly allow it
- Do not modify `.env` or `.gitignore`
- Do not delete folders, including subfolders
```

---

## §E — Your First Skill Example (About 5 Minutes; Needed for Stage 5.3)

A Skill is a reusable prompt package for Claude Code. When your message matches the description, Claude Code loads that instruction automatically.

Create `.claude/skills/hello-skill/SKILL.md`:

```markdown
---
name: hello-skill
description: First hello skill. Trigger when the user says "請打招呼" or "say hi".
---

When the user asks you to greet them, return three things:

1. Say hello once in Traditional Chinese and once in English
2. Mention today's date using system time
3. Give one small daily reminder, randomly chosen from health / learning / mood
```

Run `claude`, then type `say hi`. If Claude returns the three items, the Skill loaded.

> For deeper Skill design, see [Stage 5.3 — Skills](../stages/05-claude-code-ecosystem.en.md#53--skills-claude-code-behavior-layer).
> For copy-and-run examples, see the [Cookbook](cookbook.en.md).

---

## Where to Go Next

| Your Current State | Next Step |
|---|---|
| You want to understand LLMs, APIs, and tokens | [Stage 1 — LLM Basics](../stages/01-llm-basics.en.md) |
| You want to pick a role-based branch | [Everyday users](../branches/for-everyday-users.en.md) / [Teachers](../branches/for-teacher.en.md) / [Knowledge workers](../branches/for-knowledge-worker.en.md) / [Researchers](../branches/for-researcher.en.md) / [Developers](../branches/for-developer.en.md) |
| You want the full Claude Code ecosystem | [Stage 5 — Claude Code Ecosystem](../stages/05-claude-code-ecosystem.en.md) |
| You want local LLMs without a cloud key | [Cookbook Recipe 6](cookbook.en.md#6-local-llm--cli-agent-quick-walkthrough) |
| You want to compare CLI agents | [CLI Agents Comparison Guide](cli-agents-guide.en.md) |
| A term is unclear | [Glossary](glossary.en.md) |
