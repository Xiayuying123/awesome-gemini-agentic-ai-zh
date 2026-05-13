# Mirror Sync TODO — 2026-05 Session Drift

**Date**: 2026-05-13
**Trigger session**: Stage 6 refactor + Stage 7 audit + **Stage 8 creation** + 2026 freshness sweep + GitHub Actions setup
**Strategy**: Path B (zh-TW canonical, mirrors trail). 本 session strategic minimum sync 完成、剩下 deferred 給後續 session（建議用 Gemini delegation per global `CLAUDE.md`、CJK 翻譯主場）。

---

## ✅ Done in this session

| Sync item | en | zh-Hans | Notes |
|---|---|---|---|
| Stage 8 stub mirror | ✅ created (`stages/08-agent-interfaces.en.md`) | ✅ created (`stages/08-agent-interfaces.zh-Hans.md`) | ~50-line placeholders, both link to zh-TW canonical, valid for cross-stage anchor refs |
| README structural | ✅ Stage 8 row + 8-stages + time + Part 5 + dual hub callout | ✅ same | Both keep their existing translated content; only structural sync |
| Glossary §7 Agent Interfaces | ✅ 6 new entries (Computer Use / Browser Use / Sandbox / microVM / Firecracker / gVisor) | ✅ same | Translated, with backref to Stage 8 mirrors |

---

## 🚧 Deferred — future session priorities

### 🔴 HIGH — Stage 8 full translation

`stages/08-agent-interfaces.md` is 547 lines of new content (Computer Use / Browser Use / Sandbox + 9-row terminology glossary + Track A/B dual perspective + Safety 案例 + practice + projects). Currently stub-only in mirrors.

**Estimated translation**: ~1100 lines × 2 locales (en + zh-Hans).

**Suggested delegation**: Gemini (per global `CLAUDE.md` § Complex Task Protocol — CJK translation is Gemini's main use case).

### 🔴 HIGH — Stage 6 catch-up

zh-TW grew from ~320 lines to **576 lines** this session. Mirror still at 318/321 lines. Major net new content not in mirrors:

- §Context Engineering 是什麼（先定位）+ 3-row lineage table + 4-row 概念對照表 (Memory / Embedding / Vector DB / RAG)
- §RAG vs Long Context vs Fine-tuning trade-off table
- §進階 Memory — CoALA framework + Generative Agents 三分數 + 2024-2026 縱覽（including A-MEM / HippoRAG 2 / ScrapMem / Memory Security survey）
- §進階 Reasoning — Path 1 prompt-based table + Path 2 trained-in table (含 R2 / V4 / GPT-5.5 / Opus 4.7 / Gemini 3.1)
- §進階 RAG 技巧 — GraphRAG / Contextual Retrieval / Hybrid Search & Reranking deep dives + 縱覽 (17 techniques)
- §常用 Memory / RAG 工具推薦 + 精選 Projects 單一表

**Estimated translation**: ~260 lines of new content × 2 locales.

### 🔴 HIGH — Stage 7 refactor sync

zh-TW REFACTORED from 462 → 274 lines (net) but the structure is completely different:
- 22 H4 detail blocks consolidated → single Projects table with 適合誰 column
- New §Multi-Agent · Production 是什麼（先定位）opening with discipline lineage
- New §但你真的需要 multi-agent 嗎? (Anthropic + Cognition essays)
- Harness 7→**8** components (added Cost/Latency Optimization)
- New §Agent Benchmark Landscape + ⚠ Berkeley reward-hacking warning
- New §常用工具推薦
- Title softened: "Multi-Agent · Production" → "Multi-Agent · 進階應用"

Mirror still has the OLD structure (22 H4 detail blocks, 7 components, "Production" title).

**Estimated translation**: full restructure ~300 lines × 2 locales.

### 🟡 MEDIUM — Stage 5 forward ref + Stage 4 enhancement + Stage 8 mention

- Stage 4 §什麼時候真的需要 multi-agent gained Anthropic + Cognition essays table
- Stage 5 §自我檢查 gained Stage 8 forward ref
- Stage 5.6 H2/H3 sub-stage updates for 6→8 harness components

**Estimated translation**: ~30 lines diff × 2 locales.

### 🟡 MEDIUM — Stage 1 + Stage 2 small updates

- Stage 1: Claude pricing exercise updated to 4.5/4.6/4.7 trio + 必修閱讀 URL updated
- Stage 2 / Stage 3: small wording updates for 2026 freshness

**Estimated translation**: ~10 lines diff × 2 locales.

### 🟢 LOW — tracks/cli/A3-cli-production.md anchor fixes

3 dead-anchor link target updates after Stage 7 §Observability + §Evaluation Frameworks sections were consolidated into §常用工具推薦 + §Benchmark Landscape.

**Estimated translation**: 3 line diffs × 2 locales.

### 🟢 LOW — Anchor cleanup + enable `--strict` in anchor-validator

`anchor-validator.yml` currently runs in **warn-only** mode. Local run found ~38 legacy broken anchors (Stage 5 sub-headings gained ⭐ extras during this session's template refactor, but cross-references in glossary / cookbook / examples / tracks didn't all sync to the new slugs).

**Action**: one-time anchor cleanup PR to fix the legacy ~38 broken anchors, then flip workflow line from `python scripts/check-anchors.py` to `python scripts/check-anchors.py --strict` to enforce on future PRs. Without this tracker entry the upgrade would silently linger in the workflow YAML's TODO comment.

**Estimated effort**: ~30-50 line diffs across ~10 files, mostly one-line anchor URL fixes.

---

## 📋 Suggested delegation prompt template

When using Gemini (per CJK translation routing in global `CLAUDE.md`):

```
Task: translate Stage 8 (and/or other mirror sync) from zh-TW canonical to en + zh-Hans.

Source files (zh-TW canonical):
- stages/08-agent-interfaces.md (~547 lines)
- stages/06-memory-rag.md (the new §進階 sections)
- stages/07-multi-agent-production.md (full restructure)
- README.md (already structurally synced this session — diff against current mirrors)
- resources/glossary.md §7 Agent Interfaces (already synced this session)

Style:
- Match existing mirror style (see stages/01-llm-basics.en.md / stages/03-tool-use-and-hello-agent.en.md for tone)
- zh-Hans uses 简体中文 (用户 not 使用者; 软件 not 軟體; 网络 not 網路)
- zh-TW canonical uses 繁體中文 (使用者; 軟體; 網路)
- en mirror: technical but casual, matches Anthropic / OpenAI docs tone
- KEEP all cross-stage anchor links valid (use anchor-validator.yml CI to verify)

Constraints:
- Don't translate model names (Opus 4.7 stays Opus 4.7 in all locales)
- Don't translate framework names (browser-use stays browser-use)
- 2026 dates stay literal (2026-05, April 2026)
- Code blocks stay as-is

Output: split into separate commits per file/section for reviewability.
```

---

## 🔁 Automation status

`mirror-sync-reminder.yml` workflow (shipped this session, commit `5cd2dec`)
will now flag any PR that modifies zh-TW canonical without syncing mirrors —
preventing future drift accumulation. The existing drift documented here
is the **legacy** to clear; new drift should be caught at PR time.

---

## 📍 References

- Mirror sync workflow: [`.github/workflows/mirror-sync-reminder.yml`](.github/workflows/mirror-sync-reminder.yml)
- Mirror sync script: [`scripts/check-mirror-sync.py`](scripts/check-mirror-sync.py)
- Anchor validator (helps after translation): [`scripts/check-anchors.py`](scripts/check-anchors.py)
- Freshness check (helps verify 2026 model refs): [`scripts/check-2026-freshness.py`](scripts/check-2026-freshness.py)
- This session's commit range: `dc8b5be..b515e3f` (zh-TW work) + `a3c7406..d8d299d` (freshness + CI)
