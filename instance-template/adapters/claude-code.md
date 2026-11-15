# Adapter：Claude Code

適用 Claude Code CLI（`claude` 指令）；Desktop 與 cloud 只在排程替代方案中提及。

## 查核基準

- 查核日期：2026-11-15，依下方官方來源；未做 live 執行。
- 官方文件持續更新，部分行為有版本門檻（下文註明）。初始化時以 `claude --version` 為準，差異記入 SETUP.md。

## 能力摘要

| 項目 | 官方文件描述 |
|---|---|
| Instructions | `CLAUDE.md`，以 `@AGENTS.md` 引用；新版也原生讀 `AGENTS.md` |
| Skills | 專案 `.claude/skills/<name>/SKILL.md`；**未列出 `.agents/skills/`** |
| Native subagent | 有：`.claude/agents/<name>.md`，不需 extension |
| Per-role model／effort | profile `model`、`effort` |
| Native 排程 | `/loop`、`CronCreate`／`CronList`／`CronDelete`，依附 session |

## 版本與介面

```sh
claude --version
claude --help
```

確認本頁用到的 flags（`--agent`、`--model`、`--effort`、`--permission-mode`、`--tools`、`--disallowedTools`、`--max-turns`、`--output-format`、`-p`）存在。

## Instructions

- 本實例的 `CLAUDE.md` 只有一行 `@AGENTS.md`：`@path` 會真正展開檔案（相對於含 import 的檔案）。官方說明保留這行不會與原生 `AGENTS.md` 支援重複載入。
- 原生讀 `AGENTS.md` 需 v2.1.277 以上，且同目錄或祖先已有 `CLAUDE.md` 時不另讀；因此以 `@AGENTS.md` 為主要接線。
- 讀回：`/memory`，或 `/context all` 查看 Memory files。

## Skills

Claude Code 官方列出的專案位置是 `.claude/skills/`，本實例的 skills 在 `.agents/skills/`，所以：

- **預設（manual read fallback）**：`AGENTS.md` 的路由表已透過 `@AGENTS.md` 載入，agent 依表直接讀 `.agents/skills/<name>/SKILL.md`。不需設定，但這不是 skill auto-discovery。
- **選用**：owner 同意後建立 `.claude/skills` 指向 `../.agents/skills` 的 symlink，再以 `/skills` 讀回。官方文件未說明 symlink 是否支援；讀回時依本實例 `.agents/skills/` 的實際目錄清單逐項核對名稱與來源，包括預載的 `grill-me`、`grilling`、`skill-creator`。同名時 personal 與 enterprise skill 優先於 project skill，所以名稱出現但來源不是本實例也不算通過，衝突依 [初始化指南](../guides/initialization.md) 記錄。任一項看不到或來源無法確認，就移除 symlink、回到預設。新建頂層 skills 目錄後需 `/reload-skills`。GitHub Copilot CLI 也讀 `.claude/skills/`，同時使用時以其 `/skills` 確認沒有重複。

不要把 skills 複製成第二份；單一位置是 `.agents/skills/`。

## 角色 profile

放在 `.claude/agents/<name>.md`。必填 `name`、`description`，Markdown 本文是 system prompt：

```markdown
---
name: reviewer
description: Review the assigned diff and sources; report findings only.
tools: Read, Grep, Glob
model: inherit
permissionMode: dontAsk
maxTurns: 3
---
Review only the assigned base..head diff and sources. Report findings with file references.
Do not edit files, run shell commands, or delegate.
```

- 不提供 Agent 與 Bash 工具，以避免再委派與 shell 寫入。
- 優先順序：managed → `--agents` → project → user → plugin；同一 scope 內避免重名。
- `claude --agent reviewer` 是以 profile 啟動**主 session**；驗證真正的 subagent 派工，要在 parent session 要求使用 `reviewer` 並確認 Agent tool invocation。

## Model 與 reasoning

- 可用 models 以當前 session 的 `/model` 與組織限制為準；`availableModels` 設定只是限制選項，不證明帳號可用。
- Effort 合法值依 model 而定。官方 model-config 頁有對照表；初始化時重新查表，並以 `/model`、`/effort` 確認當前 model。不支援的 effort 可能被自動降級。
- Profile 固定 model 時，把 `inherit` 換成已查證的完整 model ID，需要時加 `effort: <已確認的值>`。
- Subagent model 解析：invocation `model` → profile `model` → `CLAUDE_CODE_SUBAGENT_MODEL` → parent model；`CLAUDE_CODE_SUBAGENT_MODEL_FORCE=1` 會全體覆寫。Effort 另受 `CLAUDE_CODE_EFFORT_LEVEL` 與政策上限影響。
- 沒有 per-subagent extended thinking 設定；不要虛構 `thinking` 欄位。
- 持久設定 `effortLevel` 不接受 `max`。

## 權限邊界

- Profile `tools` 是工具 allowlist，`disallowedTools` 是 denylist；`permissionMode` 決定如何批准，且可能被 parent 的 mode 覆蓋。
- Skill 的 `allowed-tools` 只預先批准，不限制可見工具。
- 主 session 的 `--tools` 只限制 built-in tools，不限制 MCP；需另加 `--disallowedTools "mcp__*"`。
- 以上都不是 OS sandbox；有 Bash 就不能宣稱唯讀。

## 套用與讀回

| 設定 | 套用 | 讀回 |
|---|---|---|
| Agent profile | 新增 `.claude/agents/<name>.md`；同名檔存在時只改提案欄位 | 下一次派工生效；新建首個 agents 目錄後可能需重啟。v2.1.233 以上可 `claude plugin validate .claude/agents`，但仍要自行核對必填欄位 |
| 專案 model／effort | `.claude/settings.json` 只改 `model`、`effortLevel` 等提案 keys | `/status` 查載入的 settings；`claude doctor` 查被拒絕的設定 |
| 個人專案設定 | `.claude/settings.local.json` | 同上 |
| Instructions／skills | 無需設定 | `/memory`、`/skills` |
| 實際派工 model／effort | — | `/tasks`（v2.1.242 以上） |

`/config` 不是完整的有效設定。不輸出整份使用者設定或環境變數。

## 最小 smoke

需 owner 同意費用後執行：

```sh
claude -p \
  --agent reviewer \
  --model "<VERIFIED_MODEL_ID>" \
  --effort "<VERIFIED_EFFORT>" \
  --permission-mode dontAsk \
  --tools "Read,Grep,Glob" \
  --disallowedTools "mcp__*" \
  --max-turns 1 \
  --output-format json \
  "Reply ADAPTER_OK only. Do not use tools."
```

只證明 CLI、profile 與 model 能回應；確認輸出中的實際 model，不能只看到 `ADAPTER_OK` 就宣稱 model／effort 已生效。

## 排程

- Native：`/loop <間隔> <維護指示>`（間隔格式以官方頁為準，例如 `/loop 5m ...`）或 `CronCreate`（五欄 cron、本地時區）。只在 process 運作時觸發；recurring 任務七日到期；`--resume`／`--continue` 只還原尚未過期的 CronCreate 任務，錯過的執行不補跑。記為 `native` 時須寫明這些限制。
- `os-scheduler`：用既有 cron 或 systemd timer 呼叫 `claude -p`；持久性、互斥、記錄與 credentials 由外層負責。
- Desktop／cloud scheduled tasks 是另一個產品介面，不屬本 CLI adapter。
- 都不採用時用 `session-due-check`。

## 未驗證與限制

- installed version 是否達到上文的版本門檻；
- 帳號、provider 與組織政策下實際可用的 models 與 effort；
- `.claude/skills` symlink 是否被發現；
- `/skills` 是否顯示足以判斷每個 skill 來源的位置；
- 實際 subagent 派工時的 model／effort 與權限結果；
- 排程是否在預定時間實際觸發。

## 官方來源

- [Memory](https://code.claude.com/docs/en/memory.md)
- [Skills](https://code.claude.com/docs/en/skills.md)
- [Subagents](https://code.claude.com/docs/en/sub-agents.md)
- [Model configuration](https://code.claude.com/docs/en/model-config.md)
- [Settings](https://code.claude.com/docs/en/settings.md)
- [Scheduled tasks](https://code.claude.com/docs/en/scheduled-tasks.md)
- [CLI reference](https://code.claude.com/docs/en/cli-reference.md)
