# 定期維護

定期維護包含 Memory Dreaming、Wiki ingest／lint、來源完整性核對與第一性原理反思。狀態與授權記錄在 [maintenance/state.md](../maintenance/state.md)。

## 啟用與授權

- `status: not-enabled` 或 `paused`：不執行。沒有 owner 同意就沒有定期維護。
- 啟用由 `home-init` 在初始化（或 owner 之後要求）時完成：owner 一次同意 `state.md`「授權」的每個欄位（頻率、時區、harness／model／effort、每次預算、資料範圍、排除範圍、可自動修改類型、本機 commit、同步、排程模式），才改為 `status: enabled`。
- 啟用後，在授權範圍內執行**不再逐次詢問**。超出範圍的變更走提案或 owner decision；範圍本身要改時重新同意。
- 每次執行都要求開始時工作樹乾淨。取得同意時向 owner 說明：本機 commit 為 `not-allowed` 時，每次維護後要等 owner 檢查並 commit，下一次才會執行；無人值守的持續維護需要本機 commit 授權。不因此自動擴大 commit 授權。

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

1. **讀狀態**：讀 `state.md`。
2. **固定範圍**：工作樹必須乾淨，否則停止並回報。`剩餘清單` 非空時，沿用其中記錄的批次 `base`／`head`，不改用目前 `HEAD`，先處理清單項目；否則取 `base`＝checkpoint 的 commit（首次為 `none`），`head`＝目前 `HEAD`。輸入只取資料檔的變更：
   ```sh
   git diff --name-status --no-renames <base> <head> -- MEMORY.md memory wiki sources reflections REPOS.md
   ```
   首次執行改用 `git ls-tree -r --name-only <head> -- <同上路徑>` 列出全部檔案。`--no-renames` 讓改名拆成 `D` 與 `A` 兩項。讀取版本依變更類型決定：`D`（已刪除）讀 `base` 版本（`git show <base>:<path>`），其餘內容一律讀 `head` 版本（`git show <head>:<path>`）。兩者都不讀工作樹，所以批次之間的新 commit 不會讓 offset 錯位；`head` 之後的變更留給下一批。`guides/`、`adapters/`、`.agents/`、範本檔不是 Memory 輸入，不在每次維護中重讀。
3. **套用預算**：依授權的檔案數、bytes、token 或費用上限挑選本次處理項目；大檔按區段處理並記 offset。超出預算的項目寫入剩餘清單，不跳過、不全 repo dump。每個項目記錄變更類型、讀取版本（`base` 或 `head`）、路徑、`git rev-parse <讀取版本>:<path>` 的 blob id 與 offset；續批先核對 blob id，不一致就停止並回報，不猜測續讀位置。
4. **執行工作**：
   - 輸入中的刪除（`D`）：讀 `base` 版本並核對 `git diff <base> <head> -- <path>`，確認索引與連結不再指向該檔；被刪的是證據、異議或失敗紀錄時列入 owner decision，不自行還原。核對完成即算已處理。
   - Memory Dreaming：合併等價重複、找出可能過時的條目（請 owner 確認，不自行刪）、精簡措辭、從近期內容找 Reflection 候選。
   - Wiki：處理尚未 ingest 的來源；依 [wiki.md](wiki.md) lint。
   - 來源：核對 raw 檔 sha256 與索引一致；授權允許網路存取時，核對外部來源版本是否更新，更新者標 `needs-recheck`。
   - 反思：對候選或重複出現的問題依 [reflection.md](reflection.md) 做第一性原理分析。
5. **只做授權的修改**：修改套用在目前檔案；該區段在 `head` 之後已被改動時不自動修改，留給下一批。等價重複、機械性壞連結、索引與格式可依授權直接修。語意不確定的合併、刪除重要證據或異議、過時政策、有效結論改變，改為提案（[review.md](review.md)）或列入 owner decision 清單。
6. **維護本身的刪除**：只刪除在 `<base>..<head>` 範圍內已確認不再需要的內容；不刪 source 原檔（除非 owner 另行確認 retention 規則）、不刪異議、失敗紀錄或證據。
7. **更新狀態**：本批固定範圍全部處理完，才把 checkpoint 前移到批次 `head` 並清空剩餘清單；部分完成或失敗時保留原 base，在剩餘清單寫入批次 `base`／`head` 與未完成項目（類型、讀取版本、路徑、blob id、offset），並記錄失敗原因。在執行紀錄追加一行：日期、base→head、處理數／預算、修改、提案、待 owner 決定、剩餘數、結果。
8. **commit 與同步**：依授權把本次修改與狀態更新一起做一次本機 commit，讓工作樹回到乾淨；同步只在授權的 remote／branch 上進行。Git merge 成功不等於語意一致，同步後若有合併，下次執行先 lint。本機 commit 為 `not-allowed` 時，修改與狀態留在工作樹給 owner 檢查並 commit，下一次執行會在第 2 步停止並回報。
9. **回報**：回報修改、提案、待 owner 決定與剩餘項目。

## 精簡與去重的界線

| 可依授權直接做 | 必須提案或請 owner 決定 |
|---|---|
| 內容等價的重複合併 | 語意相近但不完全相同的合併 |
| 壞連結、索引、格式修正 | 刪除重要證據、異議或失敗紀錄 |
| 新來源的 draft 摘要與 Wiki 草稿 | 宣告過時、改變有效結論或政策 |
| | 修改 guides、skills、分類或結構 |

精簡的目的是讓內容更容易用，不是讓 repository 看起來更乾淨：不得為了精簡破壞來源、吞掉異議或抹除失敗。
