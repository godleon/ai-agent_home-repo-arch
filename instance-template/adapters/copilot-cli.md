# Adapter：GitHub Copilot CLI

適用 GitHub Copilot CLI（`copilot` 指令），不適用 IDE 或 cloud agent。

## 查核基準

- 查核日期：2026-11-15，依下方官方來源；未做 live 執行。
- 官方文件持續更新，不等於你的 installed version。初始化時以實際 `--version`／`--help` 為準，差異記入 SETUP.md。

## 能力摘要

| 項目 | 官方文件描述 |
|---|---|
| Instructions | 原生讀取 `AGENTS.md`、`CLAUDE.md` 等；`@relative/path` 引用 |
| Skills | 專案 `.agents/skills/`（也支援 `.github/skills/`、`.claude/skills/`） |
| Native subagent | 有：Markdown agent profile，不需 extension |
| Per-role model／effort | `model`／`models`、`reasoningEffort` |
| Native 排程 | experimental `/after`、`/every`，僅限目前 session |

## 版本與介面

```sh
copilot --version
copilot --help
```

確認本頁用到的 flags（`--agent`、`--available-tools`、`--allow-tool`、`--deny-tool`、`--effort`、`--output-format`、`-p`）在 `--help` 中存在；不存在的就不用。

## Instructions

- 原生讀取 repository root、目前目錄與兩者之間的 `AGENTS.md`、`CLAUDE.md`。多份 instructions 會合併，官方說明沒有通用的優先順序。
- 本實例同時有 `AGENTS.md` 與內容為 `@AGENTS.md` 的 `CLAUDE.md`；官方未說明是否去重，同一內容可能載入兩次。以 `/instructions` 讀回確認。
- **Custom subagent 預設不接收 repository instructions**：profile 需設 `include-custom-instructions: true`。

## Skills

`.agents/skills/` 是官方列出的專案 skills 位置，本實例的 skills 可直接被發現，不需額外接線。讀回：

```text
/skills reload
/skills info home-init
```

`allowed-tools` 是免逐次確認的工具授權，不是 sandbox；本實例 skills 不使用它。

## 角色 profile

放在 `.github/agents/<name>.agent.md`（或使用者層級 `~/.copilot/agents/`）。最小 reviewer：

```markdown
---
name: reviewer
description: Review the assigned diff and report findings with file references.
tools: ["view", "glob", "grep"]
include-custom-instructions: true
---
Review only the assigned base..head diff and sources. Report findings; do not edit files or delegate.
```

- `description` 必填；`tools` 省略代表全部工具，任一位置出現 `"*"` 也代表全部。
- 啟動方式：`copilot --agent reviewer`、互動 `/agent`，或由主 session 委派。

## Model 與 reasoning

- 用互動 `/model` 查看帳號可用 models 與各 model 可選的 effort。官方 reference 的全域 effort 列舉（`low`、`medium`、`high`、`xhigh`、`max`）**不是每個 model 都合法**；本次查核未找到能列出每個 model 合法 effort 的機器介面，所以以 `/model` 畫面為準。
- Profile 可加入（placeholder 必須換成 `/model` 查到的值）：
  ```yaml
  model: "<MODEL_ID>"
  reasoningEffort: "<SUPPORTED_EFFORT>"
  modelPolicy: "required"
  ```
- 優先順序：單次呼叫指定 → `subagents` 設定覆寫 → agent profile → parent session。無法滿足時一般 fallback 到 session 的 model；`modelPolicy: "required"` 時改為拒絕派工。
- 官方文件同時出現 `modelPolicy` 與 `model-policy` 兩種拼法；未在 installed version 讀回確認前，不宣稱 model 已被強制。
- Session 使用 Auto 時，child 繼承當次解析的 model，不能假設角色 model 生效。

## 權限邊界

- `--available-tools`／`--excluded-tools` 決定模型看得到哪些工具；`--allow-tool`／`--deny-tool` 決定能否執行。deny 優先於 allow（含 `--allow-all`）。
- Profile 的 `tools` 清單與文字指示都不是 OS sandbox。

## 套用與讀回

| 設定 | 套用 | 讀回 |
|---|---|---|
| Agent profile | 新增 `.github/agents/<name>.agent.md`；同名檔已存在時先讀出並只改提案欄位 | `/agent` 清單 |
| Repository 預設 model | `/model --repo` | `/settings show model` 或 `copilot config model`；有效 session 值以 `/model` 確認 |
| 每角色 model 覆寫 | `/subagents` | `/subagents` |
| Instructions／skills | 無需設定 | `/instructions`、`/skills info <name>` |

`/model` 不加參數只改目前 session；`--global` 會改使用者預設，需 owner 另外同意。

## 最小 smoke

需 owner 同意費用後執行：

```sh
copilot --agent reviewer \
  --available-tools='view,glob,grep' \
  --allow-tool='read' \
  --deny-tool='write,shell' \
  --output-format=json \
  -p 'Reply ADAPTER_OK only. Do not use tools.'
```

成功只證明 CLI、profile 與 model 能回應；不證明 instructions 遵循、真正的 subagent 派工或權限邊界。派工另需在主 session 要求委派給 `reviewer` 並確認實際使用的 model。

## 排程

- Native：experimental `/after`、`/every`，只在目前 session 有效，官方未說明重啟或 resume 後保留。若選用，記為 `native`，並說明 session 結束即停止。
- `os-scheduler`：用既有 cron 或 systemd timer 呼叫 `copilot -p '<維護指示>'`；續用特定 session 時保存並指定該 session，避免盲用 `--continue`。
- 都不採用時用 `session-due-check`。

## 未驗證與限制

- `modelPolicy` 拼法與實際強制行為；
- 每個 model 的合法 effort 集合；
- experimental 排程在重啟後的行為；
- 帳號 entitlement、組織政策對 models 與 tools 的限制；
- `AGENTS.md` 與 `@AGENTS.md` 同時存在時是否重複載入。

## 官方來源

- [Using GitHub Copilot CLI](https://docs.github.com/en/copilot/how-tos/use-copilot-agents/use-copilot-cli)
- [Adding custom instructions for Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-custom-instructions)
- [Adding agent skills for Copilot CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-skills)
- [Copilot CLI command reference](https://docs.github.com/en/copilot/reference/cli-command-reference)
