# 外部專案

外部專案是 owner 的其他 repositories 與其原始碼。Home Repo 只保存導航指標（[REPOS.md](../REPOS.md)）與對專案的理解分析；專案本身在 Home Repo 之外。這是 agent 遵守的約定，不是 OS 或 filesystem 強制的隔離。

## 存放邊界

[AGENTS.md](../AGENTS.md)「邊界」的外部專案規則涵蓋 Home Repo 的整個工作樹，包括 gitignored 與 untracked 的位置。

不算外部專案、可以放在 Home Repo：

- Home Repo 自身的 skills、extension 設定、templates 與必要輔助程式；
- 依 [extensions.md](extensions.md) 以 harness 原生機制安裝的第三方擴充。

Wiki 與 sources 保存理解、分析與來源。引用程式碼時優先寫 repository＋commit＋path；確有必要才附最小片段，並標同樣的來源。不放整份專案、完整檔案鏡像、依賴或建置產物。[sources.md](sources.md) 可保存的公開文章不受影響；外部專案的程式碼適用本節。

## 位置檢查

外部專案根目錄、REPOS.md 的路徑與 clone 目的地，都要確認在 Home Repo 之外：

1. **Home Repo**：`git rev-parse --show-toplevel` 解析 symlink 後的真實路徑。
2. **候選路徑**：已存在者解析 symlink 後的真實路徑；尚不存在者，取最近一個已存在的祖先解析為真實路徑，再接上預定的子路徑（子路徑不含 `..`）。
3. **比較**：逐個路徑元件比較。候選路徑等於 Home Repo，或以 Home Repo 的全部元件開頭，就是在 Home Repo 內，拒絕。字串前綴不算包含：`/work/home-other` 不在 `/work/home` 內。

建立目錄前與實際操作時各再檢查一次；期間 symlink 或目錄可能已經改變。

## 外部專案根目錄

初始化時詢問 owner，通過位置檢查後記在 [SETUP.md](../SETUP.md)「外部專案根目錄」，作為新 clone 的預設位置。未設定或檢查失敗時，只停止依賴它的專案工作；Memory、Wiki 與其他任務照常進行。

## 索引

[REPOS.md](../REPOS.md) 每列一個專案：名稱／用途、何時使用、外部本機路徑、remote、入口指引。只列 owner 同意列入的專案，路徑須通過位置檢查。列入索引或 clone 都不授予寫入、push、merge 或發布權限。本機路徑與私有 remote 只寫在使用者自己的實例中。

## 專案工作

外部專案的讀碼、修改、測試、debug 與跨 repository 規劃，每次都依序：

1. **找目標**：讀 REPOS.md，依用途與「何時使用」找到目標。沒有條目或有多個候選時，向 owner 確認或依可觀察證據辨認，並在已有授權內補登索引；不猜測目標。
2. **核對身分**：路徑通過位置檢查且存在，`git rev-parse --show-toplevel` 等於該路徑，`git remote -v` 與索引的 remote 一致。
3. **讀入口**：讀該專案的 `AGENTS.md`（或入口指引所列文件）與 `README`，依其規則工作。
4. **操作**：在這個外部 checkout 中工作。修改沿用 AGENTS.md 的 single writer 與任務隔離；本框架不另設 Git 管理流程。

完成條件：每個動作都發生在已核對身分的外部 checkout；回存 Home Repo 的只有分析與指標。

## 缺少 checkout

owner 已授權 clone、目的地明確（索引路徑或外部專案根目錄下的子路徑）且通過位置檢查時，才 clone 到該處。任一條件不成立就停止並回報，目的地維持在 Home Repo 之外。

## 已在 Home Repo 內的外部專案

發現 Home Repo 內有外部專案（nested repository、submodule、vendor 副本、ignored clone、指向專案的 symlink 等）時：

1. **暫停**該專案的操作；無關且安全的任務照常進行。
2. **只讀盤點**：路徑、remote、tracked／untracked／ignored 檔案、未 commit 或未 push 的變更、進行中的工作，以及依賴它的設定或檔案。
3. **提案**：提出通過位置檢查的外部目的地，以及保留上述資料的搬移方案，依 [review.md](review.md) 取得獨立 review 與 owner 授權。

授權前保持原狀：不移動、不刪除、不 reset 或 clean。Home Repo 的 commit 只 stage 指定路徑，不帶入這些程式碼。
