# Adapter：OpenCode

適用 OpenCode CLI（`opencode` 指令）；desktop、web 與 Console 不在本頁範圍。

## 查核基準

- 查核日期：2026-11-16，依下方官方 V2 文件，並對照官方 repository `v2.3.0` 的 CLI 原始碼；未做 live 執行。
- 官方網站現以 V2 為主。V1 文件另在 `opencode.ai/v1/docs/`，兩版欄位不同。V1 與 V2 共用 `opencode` 指令與設定位置。
- 初始化先跑 `opencode --version` 判斷版本，再決定讀哪一版：
  - `2.x`：照本頁；和本頁不符時以該版 `--help` 為準，差異記入 SETUP.md。
  - `1.x`：改查 V1 文件與該版 `--help`，不套用本頁的 V2 欄位。
  - 無法判斷：不寫任何 OpenCode 設定，只用 `AGENTS.md` 與直接讀 skill 檔的文字流程，版本記為 `unknown`。skill 被拒絕的判斷仍照下方「Skills」的權限一項。
- 不安裝、不升級、不 migrate 既有設定（官方 V1→V2 轉換是選用步驟）；需要時另提案。

## 能力摘要

| 項目 | 官方文件描述（V2） |
|---|---|
| Instructions | 只讀 `AGENTS.md`：全域檔加上由目前目錄往上的每一層；V2 不以 `CLAUDE.md` fallback |
| Skills | 專案 `.agents/skills/` 屬相容位置，可直接被發現；原生位置是 `.opencode/skills/` |
| Native subagent | 有：`.opencode/agents/<name>.md`，`mode: subagent` |
| Per-role model／effort | agent `model: <provider>/<model>#<variant>`；variant 由各 model 的 catalog 決定 |
| Native 排程 | 指南頁未列出；API reference 有 experimental 的 session 排程 |

## 版本與介面

```sh
opencode --version
opencode --help
opencode run --help
opencode debug --help
```

只用 `--help` 中實際存在的 flags。`opencode run`、`opencode models`、`opencode debug agents`、`opencode debug config` 會連到使用者帳號的共用背景 service；service 沒在執行時會自動啟動它。執行前先告知 owner；不想啟動共用 service 時，若 `--help` 列有 `--standalone`，加上它改用暫時的私有 server。

## Instructions

- 載入順序：先載入全域 `~/.config/opencode/AGENTS.md`，再載入目前目錄往上每一層的 `AGENTS.md`。目錄在 home 底下時一路找到 home；在 home 以外時找到 project root 為止。設定中的 `instructions` 會在這些檔案之後追加。
- 這些內容會合併，不會互相覆蓋，衝突也不會被解決。因此上層目錄與全域的規則也會生效，不能把載入結果當成只有本實例的規則。盤點時列出路徑上實際存在的 `AGENTS.md`（例如用 `ls` 逐層確認），以及設定中的 `instructions`，記入 SETUP.md；內容不另外複製。
- 從本實例根目錄啟動，本實例的 `AGENTS.md` 會被載入，不需設定。V2 不讀 `CLAUDE.md`，本實例的 `CLAUDE.md` 對 OpenCode 沒有作用。
- 子目錄的 `AGENTS.md` 要等讀到該區域才載入；載入後的修改不會自動生效，需要開新 session。
- 讀回：沒有官方指令能列出已載入的 instructions。用上面的檔案清單（`visible`）和 `opencode debug config` 確認有沒有額外的 `instructions`；agent 實際遵循與否要靠後續行為觀察，不以模型自述為證據。

## Skills

本實例的 `.agents/skills/` 是 V2 列出的專案相容位置。OpenCode 從目前目錄往上找到 project root，各層的 skills 都會被發現，不需複製或建立 symlink。

- **ID**：skill ID 由路徑決定，`.agents/skills/grill-me/SKILL.md` 的 ID 是 `grill-me`。frontmatter 的 `name` 只是顯示名稱。本實例的 skills 都有 `description`，也都沒有設定 `disable-model-invocation` 或 `metadata.opencode/autoinvoke: false`；只要 `skill` 權限允許，就會列給模型。
- **同 ID 衝突**：兩個來源有相同 ID 時，後註冊的來源生效。註冊順序（越後面優先序越高）：
  1. built-in；
  2. `.claude/skills`；
  3. `.agents/skills`（2、3 都是全域先，再由最遠的上層往目前目錄）；
  4. `~/.config/opencode/skills`；
  5. 專案 `.opencode/skills`；
  6. 設定中的 `skills` 陣列。

  所以全域 `~/.agents/skills` 與 `~/.claude/skills` 的同名 skill 會被本實例覆蓋，但 4–6 會覆蓋本實例。若依 [Claude Code adapter](claude-code.md) 建立了 `.claude/skills` symlink，OpenCode 也會發現它；同 ID 時由後註冊的 `.agents/skills` 生效，兩者指向同一份檔案。系統設定目錄也會載入 `skills/`，它在這個順序中的位置文件沒有說明，有同 ID 時視為來源無法確認。
- **讀回**：
  1. 在 4–6 與系統目錄中，逐一檢查有沒有和本實例 `.agents/skills/` 實際目錄同 ID 的 skill。`skills` 陣列用 `opencode debug config` 確認。
  2. 在 session 中用 `@<skill-id>` 明確載入（例如 `@grill-me`），確認 skill 工具回報的 base directory 是本實例的 `.agents/skills/<skill-id>/`。

  只看到名稱不算通過。有同 ID 衝突或來源無法確認時，依[初始化指南](../guides/initialization.md)直接讀本實例的檔案，並記錄衝突。
- **權限**：`skill` 權限或 `skill.use` policy 拒絕某個 skill 時，該 skill 會被隱藏或拒絕載入。V1 的 `permission.skill` 設為 `deny` 也一樣；V1 agent 設 `tools.skill: false` 會停用整個 skill 工具，這是刻意停用，不算「沒有原生 skill 工具」。這些都是 owner 或管理者設定的限制：記為受阻並回報，不改用直接讀檔繞過。
  - 「沒出現在清單」不等於「沒被發現」。V1 與 V2 都會發現專案路徑上的 `.agents/skills/`，本實例 skills 也都有 `description`。從本實例內啟動、skill 清單卻沒有 `grill-me`、`grilling` 等本實例 skill 時，依官方疑難排解無法排除權限或 policy 拒絕（V1 另列同名衝突），一律記為受阻，請 owner 檢查權限與同名來源。
  - 受阻時不自行修改權限。是否調整由 owner 決定；調整屬設定變更，依初始化第 5 步提案、套用與讀回。managed 或系統 policy 的拒絕無法由專案設定解除。
- skill 目錄中的 `agents/openai.yaml` 是 Codex 用的 metadata，OpenCode 文件沒有提到，不把它當成 OpenCode 設定。

## 角色 profile

V2 的 agent 檔放在 `.opencode/agents/<name>.md`（使用者層級是 `~/.config/opencode/agents/`），也可以寫在設定的 `agents` 物件。Markdown 本文就是 system prompt。最小 reviewer：

```markdown
---
description: Review the assigned diff and sources; report findings only.
mode: subagent
permissions:
  - action: "*"
    resource: "*"
    effect: deny
  - action: read
    resource: "*"
    effect: allow
  - action: glob
    resource: "*"
    effect: allow
  - action: grep
    resource: "*"
    effect: allow
  - action: read
    resource: "*.env"
    effect: deny
  - action: read
    resource: "*.env.*"
    effect: deny
---
Review only the assigned base..head diff and sources. Report findings with file references.
Do not edit files or delegate further work.
```

- 上例沒有 `model`，subagent 會沿用 parent session 的 model。要固定時加 `model: <provider>/<model-id>#<variant>`，只用第 3 步查到的值；placeholder 未替換前不可使用。
- 規則依序比對，最後符合的那條生效。上例先全部拒絕，再開放 read、glob、grep，最後拒絕讀取 `.env` 類檔案。被拒絕的動作包括 `shell`（只禁 `edit` 不夠，shell 仍可寫檔）、`subagent`、網路，以及專案外的 `external_directory`。
- V2 欄位是 `permissions`（有序陣列）、`system`、`disabled`，`model` 中以 `#variant` 指定 variant；V1 是 `permission`、`prompt`、`disable` 與獨立的 `variant`。同一個 agent 不要混用兩版欄位：檔案只要出現 `permissions` 就整份以 V2 解讀，其中的 V1 欄位會被忽略。
- 啟動方式：在主 session 要求「use the reviewer subagent」。主 agent 的 `subagent` 權限決定它能派哪些 agent。subagent 在背景執行，預設不能再派下一層。

## Model 與 reasoning

- 可用 models：`opencode models` 列出目前專案可用的 `provider/model`，或在 TUI 用 `/models` 選取。官方 catalog 與文件只算 `docs`，不代表帳號可用；可用 models 也依專案而定。
- Reasoning 以 variant 表示，例如 `#high`。合法名稱來自所選 model 目前的 catalog metadata，不是固定 enum，名稱不存在會產生 model-resolution error。`opencode models` 不顯示 variants（`v2.3.0` 原始碼只輸出 `provider/model`），因此 variant 未經實際執行接受前，記為 `docs` 或 `unknown`。
- 根設定 `model` 只保存 provider 與 model，不保存 variant；variant 要設在 agent、command 或單次執行。Session 已選的 model 優先於設定預設值。

## 權限邊界

- 每個 agent 的基本 policy 都是全部 allow（`.env` 類讀取改為 ask），再依序附加全域規則與 agent 規則。Custom subagent 使用自己的 permissions，不是 parent 權限的子集合，所以唯讀角色要自己明確拒絕。
- V2 預設允許 `external_directory`：讀寫專案以外的路徑不會詢問。外部專案的邊界依 [projects.md](../guides/projects.md) 與 owner 授權處理，不能宣稱工具會自動限制；owner 同意時可以提案加上 `{ "action": "external_directory", "resource": "*", "effect": "ask" }`。
- `shell` 以啟動帳號的檔案、process 與網路權限執行，對目錄的推斷也只是盡力而為。以上都不是 OS sandbox。
- `hidden` 只影響是否顯示，不是安全限制。選擇「Allow always」會留下 project 範圍的永久 allow 規則。`opencode run --auto` 會自動批准所有沒有被明確拒絕的權限：沒有 owner 明確授權時不使用。
- Managed、系統或全域 policy 可以強制拒絕專案允許的動作，但不能授予權限。

## 套用與讀回

| 設定 | 套用 | 讀回 |
|---|---|---|
| Agent profile | 新增 `.opencode/agents/<name>.md`；同名檔存在時只改提案欄位 | `opencode debug agents` 輸出的 JSON 中，只擷取該 agent 的必要欄位（以實際輸出為準） |
| 專案預設 model | 專案 `opencode.jsonc` 只改 `model` 等提案 keys | `opencode debug config`（輸出已遮罩；只擷取相關 keys），並確認新 session 的實際 model |
| 使用者層設定 | `~/.config/opencode/opencode.json(c)`（需 owner 同意；屬實例外設定，只記錄改了哪些 keys） | 同上 |
| Instructions／skills | 無需設定 | 見上方 Instructions 與 Skills 的讀回 |

- 設定從目前目錄一路找到檔案系統根目錄：上層目錄的 `opencode.json(c)` 也會套用。所有 `.opencode/opencode.json(c)` 都覆蓋直接放置的 `opencode.json(c)`，系統與 managed 設定的優先序最高。同一棵目錄樹只用一種形式。
- 讀回值和提案不同時，依[初始化指南](../guides/initialization.md)第 6 步記錄差異，不改用其他值。

## 最小 smoke

需 owner 同意費用後執行。在工作樹乾淨的本實例根目錄執行：

```sh
opencode run --agent plan --format json \
  --model '<provider>/<model-id>#<variant>' \
  'Reply ADAPTER_OK only. Do not use tools.'
```

- 不需要 variant 時，省略 `#<variant>`。
- 內建 `plan` agent 會拒絕一般檔案修改，但在預設權限下仍可執行 shell。文件列出的 `opencode run` flags 中沒有關閉全部工具的選項，所以執行後要確認 `git status` 仍然乾淨。
- 檢查 exit status 與輸出。JSON 事件格式文件沒有列出；名稱錯誤的 variant 會以 model-resolution error 失敗。
- 要驗證角色時，在 session 中要求派工給該 subagent，並確認實際使用的 model；`opencode stats --models` 可以看各 model 的用量。看到 `ADAPTER_OK` 不代表 model 或 variant 已生效。

## 排程

- Native：V2 的指南頁沒有排程功能。[API reference](https://opencode.ai/docs/api/) 列有 experimental 的 `experimental.schedule.*`，可以在延遲後、指定時間、固定間隔或依 cron 喚醒某個 session。使用前要在設定中開啟 `experimental.schedules`（schema 為 boolean），官方標示它可能隨時改變。Service 停止時會不會觸發、錯過的執行會不會補跑，文件都沒有說明。採用時要先取得 owner 對開啟 experimental 設定的同意，記為 `native`（experimental）並寫明這些限制。不自行建立其他排程機制。
- `os-scheduler`：用既有的 cron 或 systemd timer 呼叫 `opencode run`。要延續特定 session 時，用 `--session <SESSION_ID>`，不要用 `--continue`（它會接續「最後一個」session，可能跨到別的工作）。無人值守時，遇到需要詢問的權限會怎麼處理，文件沒有說明（未驗證）；不要用 `--auto` 擴大權限。Overlap、timeout 與記錄由外層處理。
- 都不採用時用 `session-due-check`。

## 擴充

- 第三方擴充依 [extensions.md](../guides/extensions.md) 盤點與提案，包括 plugins、MCP servers，以及設定 `skills` 陣列中的外部目錄或 HTTP catalog。
- 盤點來源：`opencode plugin list`、`opencode mcp list`、`opencode debug config` 中的相關 keys，以及 `.opencode/plugins/` 目錄。
- 在設定中加入 plugin 項目，server 啟動時就會在背景安裝缺少的套件，所以寫入設定就等於安裝，必須先完成 review 與授權。版本要固定（精確 npm 版本或完整 commit hash）。`opencode plugin add` 會寫入**全域**設定，屬 user scope。
- V2 的 plugin API 和 V1 不相容：V1 plugin 在 V2 不會執行，也不會自動轉換。

## 未驗證與限制

- PATH 上的 `opencode` 版本，以及它是否就是查核的 V2；
- 帳號在本專案實際可用的 models，以及每個 model 實際接受的 variant；
- 上層目錄與全域 `AGENTS.md`、設定與 skills 的實際影響；
- skill 工具回報的 base directory 在 TUI 中是否看得到；
- `opencode debug agents` 的輸出欄位；
- subagent 實際使用的 model、variant 與權限結果；
- 無人值守時的權限處理，以及排程（OS 或 experimental API）是否實際觸發。

## 官方來源

- [V2 文件索引（llms.txt）](https://opencode.ai/docs/llms.txt)
- [Intro](https://opencode.ai/docs/)
- [Instructions](https://opencode.ai/docs/instructions/)
- [Skills](https://opencode.ai/docs/skills/)
- [Agents](https://opencode.ai/docs/agents/)
- [Config](https://opencode.ai/docs/config/)
- [Models](https://opencode.ai/docs/models/)
- [Permissions](https://opencode.ai/docs/permissions/)
- [Policies](https://opencode.ai/docs/policies/)
- [Plugins](https://opencode.ai/docs/plugins/)
- [Migrate from V1](https://opencode.ai/docs/migrate-v1/)
- [CLI intro](https://opencode.ai/docs/cli/)
- [CLI commands](https://opencode.ai/docs/cli/commands/)
- [Troubleshooting](https://opencode.ai/docs/troubleshooting/)
- [API reference](https://opencode.ai/docs/api/)
- [Config schema](https://opencode.ai/config.json)
- [V1 文件](https://opencode.ai/v1/docs/)、[V1 Skills](https://opencode.ai/v1/docs/skills/)
- [官方 repository（CLI 原始碼，`v2.3.0`）](https://github.com/anomalyco/opencode/tree/v2.3.0/packages/cli/src/commands)
