> **繁體中文** | [简体中文](./setup-guide.zh-Hans.md) | [English](./setup-guide.en.md)

# 🚀 從零開始 — 給沒有開發背景的設定指南

> [← 回主路線 README](../README.md)

> 預估時程：30-45 分鐘。你會申請第一個 API key、裝好 Python / uv，並跑出第一個 LLM hello world。
> 這份文件寫給「想學 AI agent，但還沒寫過 code」的人。已經熟 Python / git / CLI 的開發者，可以直接跳 [Stage 1](../stages/01-llm-basics.md)。

## 先決定兩件事

1. **你想自己寫 code 嗎？**
   - 想：走完 §A-C，然後接 [Stage 1](../stages/01-llm-basics.md) 或 [for-developer](../branches/for-developer.md)。
   - 還不確定：先走完 §A-B，再依身分看 [日常使用者](../branches/for-everyday-users.md)、[教師](../branches/for-teacher.md)、[知識工作者](../branches/for-knowledge-worker.md)、[研究者](../branches/for-researcher.md)。
2. **你想先用哪一家 LLM？**
   - 最省心：Anthropic Claude，介面清楚，英文和中文都好用。
   - 已經常用 ChatGPT：OpenAI，需要另外到 API 平台申請 key。
   - 想先不付雲端費用：Ollama 本機模型，步驟多一點，但不需要 API key。

API key 可以先理解成「讓程式呼叫模型的密碼」。請把它當成信用卡資料保管。

---

## §A — 申請第一個 API key（約 10 分鐘）

### Anthropic Claude（推薦第一次）

1. 開 https://console.anthropic.com/
2. 用 Google、GitHub 或 email 註冊。
3. 進帳號後找到 **API Keys**，按 **Create Key**。
4. **立刻複製顯示出的 key**。多數平台只會顯示一次。
5. 先放在本機密碼管理器，或短暫放在本機文字檔；下一節會移到 `.env`。

> ⚠️ **API key 三不規則**
> - **不貼**到 chat 視窗、群組、email 或截圖。
> - **不上傳**到 git；GitHub 可能掃到後自動撤銷。
> - **不放**雲端硬碟純文字檔；同步到其他裝置等於多一份風險。

### 其他 LLM 選項

- **OpenAI**：https://platform.openai.com/api-keys  
  ChatGPT Plus 和 API key 是兩件事；訂閱 Plus 仍要另外申請 API key。
- **Google AI Studio**：https://aistudio.google.com/  
  適合先試 Gemini API，免費額度會依地區和帳號狀態不同。
- **Ollama 本機模型**：不用 API key。走本機路線請看 [Cookbook Recipe 6](cookbook.md#6-本機-llm--cli-agent-快速-walkthrough)。

---

## §B — 裝本機環境（約 10 分鐘）

### 裝 Python 3.10+

- **macOS**：開 Terminal，輸入 `brew install python@3.12`。如果還沒有 Homebrew，先看 https://brew.sh。
- **Windows**：到 https://www.python.org/downloads/ 下載 installer，安裝時一定要勾 **Add Python to PATH**。
- **Linux**：Ubuntu 用 `sudo apt install python3 python3-venv`，Fedora 用 `sudo dnf install python3`。
- **驗證**：macOS / Linux 輸入 `python3 --version`；Windows 輸入 `py --version`。看到 `Python 3.10` 以上即可。

### 裝 uv

uv 是 Python 套件管理工具。你可以把它想成「幫你臨時裝好需要套件再執行」的工具。

```bash
# macOS / Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows PowerShell
irm https://astral.sh/uv/install.ps1 | iex
```

驗證：

```bash
uv --version
```

### 建立第一個 `.env` 檔

在你要跑 script 的資料夾裡，建立一個檔名叫 `.env` 的檔案：

```bash
ANTHROPIC_API_KEY=sk-ant-...貼上你剛才複製的 key
```

`.env` 是專門放本機祕密資訊的檔案。程式會讀它，但你不應該把它上傳到 GitHub。

### 加上 `.gitignore`

同一個資料夾建立 `.gitignore`：

```gitignore
.env
__pycache__/
*.pyc
```

這樣 git 就不會把 `.env` 收進版本紀錄。

---

## §C — 跑第一個 `hello-claude.py`（約 5 分鐘）

建立 `hello-claude.py`：

```python
from anthropic import Anthropic
from dotenv import load_dotenv

load_dotenv()
client = Anthropic()  # 自動讀取 ANTHROPIC_API_KEY

msg = client.messages.create(
    model="claude-sonnet-4-5",
    max_tokens=100,
    messages=[{"role": "user", "content": "Hello, who are you?"}],
)

print(msg.content[0].text)
```

執行：

```bash
uv run --with anthropic --with python-dotenv python hello-claude.py
```

看到 Claude 回覆自我介紹，就代表你的 API key、Python、套件都通了。

### 常見錯誤

| 錯誤訊息 | 常見原因 | 解法 |
|---|---|---|
| `401 Unauthorized` | API key 沒讀到或打錯 | 回 §A 重新複製，確認 `.env` 檔名和內容 |
| `429 Rate limit` | 太快送太多請求 | 等幾秒或幾分鐘再跑 |
| `connection refused` | 連線或防火牆問題 | 確認網路、公司或學校防火牆 |
| `ModuleNotFoundError` | 套件沒有被安裝 | 確認執行的是上面的 `uv run --with ...` 指令 |

---

## §D — 第一次裝 Claude Code（約 10 分鐘；Stage 5 / for-developer 會用到）

### 先裝 Node.js

- **macOS / Linux**：`brew install node`，或從 https://nodejs.org 下載。
- **Windows**：從 https://nodejs.org 下載 installer。
- **驗證**：輸入 `node --version`，看到 v18 以上即可。

### 裝 Claude Code

```bash
npm install -g @anthropic-ai/claude-code
```

### 第一次認證

```bash
claude
```

第一次啟動時通常會讓你選：

- **Claude subscription**：用 Claude.ai 帳號登入，對初學者最省事。
- **API key**：貼上 §A 申請到的 key。

### 建立第一份 `CLAUDE.md`

在你的 project 根目錄建立 `CLAUDE.md`。Claude Code 啟動時會讀它，理解你希望它怎麼協助。

```markdown
# 你是誰
我是 [你的名字]，[你的領域，例如：教師 / 研究者 / 寫作者]。

# Code style
- 註解用繁體中文寫，code 用英文
- 寫 function 時優先加 type hint
- 不要主動 commit；改完讓我手動 git add

# 不准做的事
- 不要連網查資料，除非我明確說可以
- 不要動 `.env` 或 `.gitignore`
- 不要刪資料夾，包括子資料夾
```

---

## §E — 第一個 Skill 範例（約 5 分鐘；Stage 5.3 會用到）

Skill 是 Claude Code 的「可重用 prompt 包」。當你的訊息符合描述，Claude Code 會自動載入那份指示。

建立 `.claude/skills/hello-skill/SKILL.md`：

```markdown
---
name: hello-skill
description: 第一個 hello skill。當使用者說「請打招呼」或「say hi」時觸發。
---

當使用者請你打招呼時，回三件事：

1. 用繁體中文跟英文各說一次 hello
2. 提現在的日期（用 system 時間）
3. 給一個今日小提醒（隨機選健康 / 學習 / 心情建議）
```

跑 `claude`，輸入「請打招呼」。如果 Claude 回覆三件事，就代表 Skill 被載入了。

> 想看更完整的 Skill 設計：看 [Stage 5.3 — Skills](../stages/05-claude-code-ecosystem.md#53--skillsclaude-code-的行為層)。
> 想看可以照做的範例：看 [Cookbook](cookbook.md)。

---

## 接下來去哪

| 你現在的狀態 | 下一步 |
|---|---|
| 想正式理解 LLM、API、token | [Stage 1 — LLM 基礎](../stages/01-llm-basics.md) |
| 想直接挑身分分支 | [日常使用者](../branches/for-everyday-users.md) / [教師](../branches/for-teacher.md) / [知識工作者](../branches/for-knowledge-worker.md) / [研究者](../branches/for-researcher.md) / [開發者](../branches/for-developer.md) |
| 想看 Claude Code 完整生態 | [Stage 5 — Claude Code 生態系](../stages/05-claude-code-ecosystem.md) |
| 想本機 LLM、不用雲端 key | [Cookbook Recipe 6](cookbook.md#6-本機-llm--cli-agent-快速-walkthrough) |
| 想比較 CLI agent | [CLI Agents 比較指南](cli-agents-guide.md) |
| 不懂某個用詞 | [Glossary](glossary.md) |
