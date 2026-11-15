# 定期維護

定期維護包含 Memory Dreaming、Wiki ingest／lint、來源完整性核對與第一性原理反思。狀態與授權記錄在 [maintenance/state.md](../maintenance/state.md)。

## 啟用與授權

- `status: not-enabled` 或 `paused`：不執行。沒有 owner 同意就沒有定期維護。
- 啟用由 `home-init` 在初始化（或 owner 之後要求）時完成：owner 一次同意 `state.md`「授權」的每個欄位（頻率、時區、harness／model／effort、每次預算、資料範圍、排除範圍、可自動修改類型、本機 commit、同步、排程模式），才改為 `status: enabled`。
- 啟用後，在授權範圍內執行**不再逐次詢問**。超出範圍的變更走提案或 owner decision；範圍本身要改時重新同意。

## 排程模式

依 [adapters](../adapters/) 中目前 harness 已查證的能力選擇，並寫入 `state.md`：

| 模式 | 意義 | 限制 |
|---|---|---|
| `native` | harness 原生排程 | 各 harness 的持久性不同，多數只在 session 運作時觸發；照 adapter 的限制說明 |
| `os-scheduler` | 既有的 OS 排程（例如 cron、systemd timer）呼叫 harness 的非互動指令 | 需 owner 同意安裝；權限、overlap、記錄由該排程與 harness 指令負責 |
| `session-due-check` | 每次 session 開始時由 `AGENTS.md` 檢查是否到期 | **不是背景排程**：沒有 session 就不會執行 |

驗證分三項記錄，不互相代替：

1. **設定完成**：排程已建立並讀回（指令輸出或設定檔內容）。
2. **人工 dry run**：手動跑一次維護流程，結果正確。
3. **實際觸發**：排程在預定時間自行觸發並完成一次。首次觸發前填「下次預期時間」，狀態維持 `not-run`。

設定檔存在、指令 exit 0 或 dry run 成功，都不代表排程已實際觸發。

## 每次執行

1. **讀狀態**：讀 `state.md`。`剩餘清單` 非空時先處理它。
2. **固定範圍**：工作樹必須乾淨，否則停止並回報。取 `base`＝checkpoint 的 commit（首次為 `none`），`head`＝目前 `HEAD`。輸入只取資料檔的變更：
   ```sh
   git diff --name-status <base> <head> -- MEMORY.md memory wiki sources reflections REPOS.md
   ```
   首次執行列出上述路徑的全部檔案。`guides/`、`adapters/`、`.agents/`、範本檔不是 Memory 輸入，不在每次維護中重讀。
3. **套用預算**：依授權的檔案數、bytes、token 或費用上限挑選本次處理項目；大檔按區段處理並記 offset。超出預算的項目寫入剩餘清單，不跳過、不全 repo dump。
4. **執行工作**：
   - Memory Dreaming：合併等價重複、找出可能過時的條目（請 owner 確認，不自行刪）、精簡措辭、從近期內容找 Reflection 候選。
   - Wiki：處理尚未 ingest 的來源；依 [wiki.md](wiki.md) lint。
   - 來源：核對 raw 檔 sha256 與索引一致；授權允許網路存取時，核對外部來源版本是否更新，更新者標 `needs-recheck`。
   - 反思：對候選或重複出現的問題依 [reflection.md](reflection.md) 做第一性原理分析。
5. **只做授權的修改**：等價重複、機械性壞連結、索引與格式可依授權直接修。語意不確定的合併、刪除重要證據或異議、過時政策、有效結論改變，改為提案（[review.md](review.md)）或列入 owner decision 清單。
6. **刪除前核對**：只刪除在 `<base>..<head>` 範圍內已確認不再需要的內容；不刪 source 原檔（除非 owner 另行確認 retention 規則）、不刪異議、失敗紀錄或證據。
7. **commit 與同步**：依授權做本機 commit；同步只在授權的 remote／branch 上進行。Git merge 成功不等於語意一致，同步後若有合併，下次執行先 lint。
8. **更新 checkpoint**：本次範圍全部處理完才把 base 前移到 `head`；部分完成或失敗時保留原 base，把未完成項目與 offset 寫入剩餘清單，並記錄失敗原因。
9. **回報**：在執行紀錄追加一行：日期、base→head、處理數／預算、修改、提案、待 owner 決定、剩餘數、結果。

## 精簡與去重的界線

| 可依授權直接做 | 必須提案或請 owner 決定 |
|---|---|
| 內容等價的重複合併 | 語意相近但不完全相同的合併 |
| 壞連結、索引、格式修正 | 刪除重要證據、異議或失敗紀錄 |
| 新來源的 draft 摘要與 Wiki 草稿 | 宣告過時、改變有效結論或政策 |
| | 修改 guides、skills、分類或結構 |

精簡的目的是讓內容更容易用，不是讓 repository 看起來更乾淨：不得為了精簡破壞來源、吞掉異議或抹除失敗。
