# Home Repo Framework — agent 入口

這個 checkout 是**公開框架原始碼**，不是任何人的 Home Repo 實例；`instance-template/` 也只是範本。

- 在這個 checkout 內（含 `instance-template/`），不寫入任何使用者的 Memory、Wiki、來源、harness profile、維護設定或紀錄。即使 agent 從 `instance-template/` 啟動並讀到其 `AGENTS.md`，也以本檔為準。
- 使用者要建立或初始化自己的 Home Repo：讀 [INITIALIZE.md](INITIALIZE.md)，在此 checkout 之外建立新的獨立 Git repository，再到新目錄繼續。
- 使用者要修改框架本身（guides、skills、adapters、templates、分類或目錄結構）：先寫完整變更提案並取得獨立 review，流程依 [instance-template/guides/review.md](instance-template/guides/review.md)。
- `instance-template/adapters/` 的內容是有查核日期的官方文件摘要；修改前重新查官方來源，並更新查核日期與「未驗證」清單。
- 公開內容只放通用指南與虛構示例：不放個人 email、主機路徑、私人 repository 名稱、tokens、真實 profiles 或對話紀錄。
