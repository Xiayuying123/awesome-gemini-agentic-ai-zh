# Project Memory — awesome-agentic-ai-zh

> Standing instructions for any AI agent (Gemini, Antigravity CLI, others) working on this repo. Read this **first** before touching exercises or model recommendations.

## 📍 Repo positioning — read before adding anything

**This repo's role**: **learning roadmap + 145+ curated resources + simple illustrative cases.**

**Benchmark for "what we are NOT"**: [`datawhalechina/hello-agents`](https://github.com/datawhalechina/hello-agents) is the canonical chapter-length zh-TW depth tutorial (16 production capabilities, chapter format). **We don't compete with it; we route to it.**

**Implications when contributing**:

| Decision | Rule |
|---|---|
| New stage-level exercise folder | OK if it adds **a roadmap node + dual-path SDK demo + 1-line punchline**. 70-150 lines starter is the right size. |
| Expanding a starter beyond ~150 lines | **Push back**. If it's growing into chapter-length, add a 📚 callout pointing to hello-agents instead. |
| Adding a 5th `extension` to README | Diminishing return. Keep README tight (under ~200 lines); extra depth goes to the 📚 callout. |
| New resource (lib / paper / tool / framework) | Almost always YES — add to the relevant `精選 Projects` section or `resources/` catalog. Curation is the primary value. |
| New chapter-length tutorial inside this repo | **Push back**. If the topic deserves chapter-length, the right move is: write a 1-page summary + simple illustrative case + 📚 callout to a canonical source (hello-agents / Google Cookbooks / framework's own docs). |
| Trilingual mirror priority | zh-TW canonical first; en + zh-Hans mirror when capacity allows. Don't block shipping waiting for 3-lang. |

**One-line summary**: **route → depth, not reinvent**. Every exercise folder ends with 📚 "want chapter-length? go to hello-agents X + [extra ref]".

---

## Canonical Ollama models (verified against user's `ollama list`)

| Model tag | When to use | Notes |
|---|---|---|
| **`gemma4:e4b`** | Stage 1 + 2 (plain chat, prompt engineering) | Effective 4B params, ~7.5 GB download, CPU-friendly. **The `:e4b` tag matters** — NOT `gemma3n:e4b`, NOT `gemma3:4b`, NOT `gemma4:latest`. |
| **`gemma4:e2b`** | Low-RAM-machine alternative for Stage 1+2 | ~4 GB, runs on 4 GB RAM machines |
| **`qwen2.5:3b`** | Stage 3+ (tool use / agent / ReAct) | 1.9 GB, **reliable tool-use support** (OpenAI function-calling format), default for any agent / function-calling exercise |
| **`llama3.2:3b`** | `qwen2.5:3b` alternative for tool use | 2.0 GB, similar capability |
| **`mistral-nemo:12b`** | Higher-quality local fallback | 7.1 GB, closer-to-cloud quality |

---

## Canonical Gemini models

| Model | Use case | Pricing (per 1M tokens) |
|---|---|---|
| **`gemini-2.5-flash`** | Cheapest cloud option, default for all exercises | Very cost-effective |
| **`gemini-2.5-pro`** | Production default, agent development, complex reasoning | High quality, 1M+ context |

---

## Framing rules (do not violate)

1. **Gemini is the canonical / production reference** in documentation positioning.
2. **Ollama is the practice default** because of cost — students should not be blocked by API fees during learning.
3. **Every exercise must ship BOTH paths**:
   - Path A (Ollama, `<details open>`, primary practice runnable)
   - Path B (Gemini API, `<details>`, optional cloud-quality comparison)
4. **Every exercise must mention budget explicitly** — single-run cost + total stage cost.
5. **Local LLMs must appear in any model recommendation list** — never list cloud-only options.

---

## Exercise file conventions

- `starter.py` = Ollama / OpenAI-compatible default (Path A)
- `starter_gemini.py` = Gemini SDK version (Path B)
- `test.py` = mock-based tests for the Ollama starter (OpenAI-compat response shape)
- `test_gemini.py` = mock-based tests for the Gemini starter
- `requirements.txt` = both `openai` and `google-genai` pinned
- `README.md` = trilingual switcher + 怎麼跑（兩條 path）+ budget per path + walkthrough + common pitfalls
- Each starter ends with `# === 自我驗證 ===` block containing 2+ `assert` statements
- Each Python file headers Windows-cp950 UTF-8 reconfigure:
  ```python
  import sys
  if hasattr(sys.stdout, "reconfigure"):
      sys.stdout.reconfigure(encoding="utf-8", errors="replace")
  ```

---

## Translation rules

- **zh-TW canonical** (`.md` without language suffix). zh-Hans + en mirror.
- **Gemini does translations** — do NOT delegate to other models.

---

## Antigravity CLI commit rules

- Antigravity CLI (`agy`) runs the local agent workspace.
- The agent boundary corresponds to the commit boundary per `GEMINI.md` rulebook.
