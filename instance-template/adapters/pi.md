# Adapter：Pi

適用 Pi coding agent（`pi` 指令，`earendil-works/pi`）。Pi 核心與可選的第三方 extension `pi-subagents` 分開描述：extension 能力不是 Pi 核心能力。

## 查核基準

- 查核日期：2026-11-15。依據為 Pi 核心 1.5.2 與 `pi-subagents` 0.84.0 隨套件文件，並與下方公開 upstream 交叉核對；未做 live 執行。
- 公開 `main` 文件會變動。初始化時以 `pi --version` 與 installed 文件為準，差異記入 SETUP.md。

## 能力摘要

| 項目 | Pi 核心 | `pi-subagents`（可選） |
|---|---|---|
| Instructions | 原生載入 `AGENTS.md`（或 `CLAUDE.md`） | child 是否繼承由 profile 設定 |
| Skills | `.agents/skills/`、`.pi/skills/` 等 | 可逐角色指定 |
| Native subagent | **無**；提供 SDK 與 extension 範例 | Markdown profiles（`.pi/agents/`） |
| Per-role model／thinking | 無角色登錄；可逐次呼叫指定 | frontmatter 與 `agentOverrides` |
| 排程 | 核心未提供 | durable schedules，但不是 daemon |

## 版本與介面

```sh
pi --version
pi --help
pi --list-models
```

## Instructions

- 每個目錄依序取第一個存在者：`AGENTS.override.md` → `AGENTS.md` → `AGENTS.MD` → `CLAUDE.md` → `CLAUDE.MD`；global、祖先到目前目錄的結果累積。本實例取 `AGENTS.md`，`CLAUDE.md` 不另讀。
- 這是以啟動目錄為準的 startup discovery；從 root 啟動後讀子目錄檔案，不會自動載入該子目錄的 `AGENTS.md`。
- Context discovery 不需要 project trust；`--no-context-files` 才會停用。

## Skills

核心從目前目錄往上搜尋 `.agents/skills/`（有 repository root 時止於 root），本實例 skills 可直接被發現。啟動時只載入 name／description，需要時讀全文；`/skill:home-init` 可明確載入。

## 角色 profile

**核心沒有 native subagent。** 選項（由 owner 決定）：

1. 不安裝 extension：reviewer 改為另開一個獨立 `pi` session，用不同設定手動執行。
2. 安裝第三方 `pi-subagents`（需 owner 同意；安裝前審閱其原始碼並記錄版本）。`--local` 把宣告寫入專案 `.pi/settings.json`，Pi 只在 project trust 後讀取：
   ```sh
   pi install --local npm:pi-subagents@<VERSION>
   ```

使用 `pi-subagents` 時，profile 放 `.pi/agents/<name>.md`：

```markdown
---
name: reviewer
description: Review the assigned diff and sources; report findings only.
tools: read, grep, find, ls
extensions:
model: inherit
systemPromptMode: replace
inheritProjectContext: true
inheritGlobalContext: false
inheritSkills: false
---
Review only the assigned base..head diff and sources. Report findings with file references.
Do not edit files or delegate further work.
```

- Fresh custom agent 預設 `systemPromptMode: replace`，不會自動繼承專案 context；上例以 `inheritProjectContext: true` 載入實例 `AGENTS.md`。
- `tools` 省略代表一般 builtins，空值代表無工具，明列則為 strict allowlist。
- `extensions:` 空值關閉 ambient extensions；extension 必要的 runtime 仍可能載入。
- 拒絕 project trust 時，project agents 與 settings 會被忽略。

## Model 與 reasoning

- `pi --list-models` 或互動 `/model` 列出 Pi 能解析認證的 models；這仍不證明帳號有額度或請求一定成功。
- Thinking 通用值為 `off`、`minimal`、`low`、`medium`、`high`、`xhigh`、`max`，每個 model 只支援其子集合；無 reasoning 的 model 只有 `off`。
- 機器查詢用 RPC，每步等待成功回應：
  ```json
  {"id":"models","type":"get_available_models"}
  {"id":"select","type":"set_model","provider":"<provider>","modelId":"<model-id>"}
  {"id":"levels","type":"get_available_thinking_levels"}
  {"id":"state","type":"get_state"}
  ```
  不支援的 thinking 會被 clamp；以最後 `get_state` 的 `model`／`thinkingLevel` 為有效值。
- `pi-subagents` 角色覆寫：
  ```json
  {"subagents": {"agentOverrides": {"reviewer": {"model": "<provider>/<model-id>", "thinking": "<returned-level>"}}}}
  ```
  優先順序：單次 launch → provider-scoped 角色覆寫 → `agentOverrides` → frontmatter → `subagents.defaultModel` → parent。provider 失敗時不自動換 model。

## 權限邊界

- 核心與 extension 都以啟動帳號的 OS 權限執行；cwd、prompt 與 tool allowlist 都不是 sandbox。
- 核心 `--tools` 對 MCP 有特殊規則；只用核心工具時可加 `--no-extensions --no-mcp`。
- `pi-subagents` 的 `acceptanceRole: read-only` 只是資訊，不撤銷寫入工具。

## 套用與讀回

| 設定 | 套用 | 讀回 |
|---|---|---|
| 核心預設 model／thinking | 專案 `.pi/settings.json` 或使用者 settings，只改 `defaultProvider`、`defaultModel`、`defaultThinkingLevel` 等提案 keys | `/reload` 後開新 session，以 RPC `get_state` 或 `/model` 確認（resume 可能恢復舊 session 的 model） |
| `pi-subagents` profile | 新增 `.pi/agents/<name>.md` | `/reload` 後 `/subagents-models <name>` |
| `pi-subagents` 覆寫 | settings 的 `subagents.agentOverrides` | 同上 |
| Instructions／skills | 無需設定 | 啟動後確認 context 與 `/skill:<name>` |

`/subagents-check-profile` 與 `/subagents-refresh-provider-models` 會做 live probe，不是離線檢查。

## 最小 smoke

需 owner 同意費用後執行：

```sh
pi --print --no-session --no-extensions --no-mcp \
  --no-skills --no-prompt-templates --no-context-files \
  --no-approve --no-tools \
  --model '<provider>/<model-id>' \
  --thinking '<returned-level>' \
  'Reply with exactly OK.'
```

檢查 stdout 與 exit status。成功只證明 model 能回應；角色與 extension 需另外以實際派工驗證。

## 排程

- 核心：未提供排程；可用 `os-scheduler`（既有 cron 或 systemd timer 呼叫 `pi --print ...`），或 `session-due-check`。
- `pi-subagents`：durable schedules 存在 `.pi/subagents/schedules/<id>/`，可跨重啟保存定義，但**不是 daemon**：由同專案某個 Pi session 認領執行，所有 Pi 都關閉時不會觸發；`schedule.run-due` 可供外部 launcher 呼叫。每次執行是 fresh context；overlap 固定 skip。選用時記為 `native`（extension）並寫明這些限制。

## 未驗證與限制

- PATH 上的 `pi` 是否為查核的版本；
- 帳號可用 models、額度與 thinking 實際接受度；
- `pi-subagents` 是否已安裝、啟用，以及 profile 實際載入結果；
- 排程是否在預定時間實際觸發。

## 官方來源

- [Pi README](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/README.md)
- [Configuration](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/configuration.md)
- [Security](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/security.md)
- [Skills](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/skills.md)
- [Settings](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/settings.md)
- [Models](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/models.md)
- [CLI](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/cli.md)
- [RPC commands](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/rpc-commands.md)
- [Packages](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/packages.md)
- [Extensions](https://github.com/earendil-works/pi/blob/main/packages/coding-agent/docs/extensions.md)
- `pi-subagents`（第三方 extension）：[README](https://github.com/nicobailon/pi-subagents/blob/main/README.md)、[agents](https://github.com/nicobailon/pi-subagents/blob/main/docs/agents.md)、[models](https://github.com/nicobailon/pi-subagents/blob/main/docs/models.md)、[missions／schedules](https://github.com/nicobailon/pi-subagents/blob/main/docs/missions.md)
