# Adapter：Codex CLI

適用 Codex CLI（`codex` 指令），不適用 Codex app 或 cloud。

## 查核基準

- 查核日期：2026-11-15，依下方官方來源；未做 live 執行。查核時官方 `developers.openai.com/codex/` 轉址到下列 `learn.chatgpt.com` 頁面。
- 官方文件明示 0.134.0 起 named profile 格式改變。初始化時以 `codex --version` 判斷適用哪一種格式，差異記入 SETUP.md。

## 能力摘要

| 項目 | 官方文件描述 |
|---|---|
| Instructions | 原生 `AGENTS.override.md`／`AGENTS.md`；`CLAUDE.md` 只在設定 fallback 時讀 |
| Skills | 從目前目錄到 repo root 各層的 `.agents/skills/` |
| Native subagent | 有：`.codex/agents/<name>.toml`，現行版本預設啟用 |
| Per-role model／effort | agent 檔 `model`、`model_reasoning_effort` |
| Native 排程 | 本次未找到 CLI 原生排程；官方 scheduled tasks 頁屬 desktop／web |

## 版本與介面

```sh
codex --version
codex --help
codex exec --help
```

## Instructions

- 每次啟動依序串接：`$CODEX_HOME`（預設 `~/.codex`）的 instructions，再由 project root 往目前目錄，每層取第一個非空的 `AGENTS.override.md` 或 `AGENTS.md`；總量受 `project_doc_max_bytes` 限制。
- 本實例有 `AGENTS.md`，不需任何設定。`CLAUDE.md` 不會被讀，也不需要。
- 沒有官方的 `@AGENTS` include 語法；不要把 Copilot 或 Claude 的 `@` 行為套用到 Codex。

## Skills

`.agents/skills/` 是官方位置，本實例 skills 可直接被發現。Codex 會偵測變動，未更新時重啟。Child 繼承 parent 的 skills，可用 `skills.config` 關閉指定 skills。

## 角色 profile

Custom agent 放在 `.codex/agents/<name>.toml`（或 `~/.codex/agents/`）；必填 `name`、`description`、`developer_instructions`：

```toml
name = "reviewer"
description = "Review the assigned diff and sources; report findings only."
developer_instructions = """
Review only the assigned base..head diff and sources.
Do not edit files or delegate further work.
"""
model = "<MODEL_ID>"
model_reasoning_effort = "<SUPPORTED_EFFORT>"

[features]
shell_tool = false
```

- Placeholder 未替換前不可使用；不需固定時省略 `model` 與 `model_reasoning_effort`，沿用 parent。
- Named config profile 與 custom agent 是不同概念：0.134.0 起 `--profile <name>` 讀 `~/.codex/<name>.config.toml`，不再讀 `[profiles.<name>]`。

## Model 與 reasoning

- 互動：`/model` 選 model 後顯示該 model 的 reasoning 選項。
- 機器介面：`codex app-server` 以 JSONL 交握後查詢，`model/list` 每筆含 `model`、`defaultReasoningEffort`、`supportedReasoningEfforts`（需處理 `nextCursor`），不會啟動 inference：
  ```json
  {"id":1,"method":"initialize","params":{"clientInfo":{"name":"home_repo_probe","version":"0.1.0"}}}
  ```
  收到成功回覆後：
  ```json
  {"method":"initialized","params":{}}
  {"id":2,"method":"model/list","params":{"limit":20,"includeHidden":false}}
  ```
- `model_reasoning_effort` 的合法值是該 model 公布的字串，不是跨 model 的固定 enum。
- 解析順序：explicit spawn → `[agents]` defaults（例如 `default_subagent_model`、`default_subagent_reasoning_effort`）→ parent；agent 檔明設的值最後套用。只設 model 可能沿用先前解析的 effort，兩者一起讀回。

## 權限邊界

- Child 繼承 parent 的 permissions、provider 與 MCP servers；agent 檔中的 `sandbox_mode`、`approval_policy`、`model_provider`、`mcp_servers` **不會覆寫 parent**。
- `features.shell_tool = false` 是官方支援的能力縮減，但不代表所有寫入能力都消失。
- 角色需要不同 sandbox 時，另開 root session，例如 `codex --sandbox read-only -c 'approval_policy="never"'`（`never` 是不互動詢問，不是授予全部權限）。

## 套用與讀回

| 設定 | 套用 | 讀回 |
|---|---|---|
| Custom agent | 新增 `.codex/agents/<name>.toml`；同名檔存在時只改提案 keys | 主 session 要求派工給該 agent 並檢查事件 |
| 使用者層 named profile | `~/.codex/<name>.config.toml`（需 owner 同意；屬實例外設定） | app-server `config/read`（只擷取需要的欄位） |
| Instructions／skills | 無需設定 | 啟動後詢問載入的 instructions 與 skills；以檔案內容比對 |

設定優先順序：CLI overrides → trusted project config → 選取的 profile file → user config → managed／system defaults。`config/read` 是磁碟設定 layering 的結果，不等於執行中 thread 的有效值。

## 最小 smoke

需 owner 同意費用後執行：

```sh
codex exec --json --sandbox read-only \
  'Reply ADAPTER_OK only. Do not invoke tools.'
```

以 JSONL 事件判斷：需看到 `turn.completed`，`turn.failed` 即失敗；不能只憑有 stdout。之後若要驗證角色，再於 session 中要求派工給該 custom agent，並檢查實際 model／effort 證據。

## 排程

- Native：本次查核未找到 Codex CLI 原生排程；官方 scheduled tasks 頁屬 desktop／web，且需電腦與 app 保持運作，不算 CLI 能力。
- `os-scheduler`：用既有 cron 或 systemd timer 呼叫 `codex exec`；要延續特定 session 用 `codex exec resume <SESSION_ID>`，不要用跨工作的 `--last`。Overlap、timeout 由外層處理。
- 都不採用時用 `session-due-check`。

## 未驗證與限制

- installed version 採用哪種 profile 與 agent 格式；
- 帳號可用 models 與 effort（以 `model/list` 實查）；
- Child 是否載入實例的 `AGENTS.md`；
- `shell_tool = false` 等限制的實際效果；
- resume 與 OS 排程的實際行為。

## 官方來源

- [AGENTS.md](https://learn.chatgpt.com/codex/guides/agents-md)
- [Skills](https://learn.chatgpt.com/codex/skills)
- [Subagents](https://learn.chatgpt.com/codex/multi-agent)
- [Config basics](https://learn.chatgpt.com/codex/config-file/config-basic)
- [Advanced configuration](https://learn.chatgpt.com/codex/config-file/config-advanced)
- [Configuration reference](https://learn.chatgpt.com/codex/config-reference)
- [Models](https://learn.chatgpt.com/codex/models)
- [App-server protocol](https://learn.chatgpt.com/codex/app-server)
- [Non-interactive mode](https://learn.chatgpt.com/codex/noninteractive)
- [Scheduled tasks（desktop／web）](https://learn.chatgpt.com/codex/app/automations)
