# Home Repo Framework — agent 入口

這個 checkout 是**公開框架原始碼**，不是任何人的 Home Repo 實例；`instance-template/` 也只是範本。

- 在這個 checkout 內（含 `instance-template/`），不寫入任何使用者的 Memory、Wiki、來源、harness profile、維護設定或紀錄。即使 agent 從 `instance-template/` 啟動並讀到其 `AGENTS.md`，也以本檔為準。
- 使用者要建立或初始化自己的 Home Repo：讀 [INITIALIZE.md](INITIALIZE.md)，在此 checkout 之外建立新的獨立 Git repository，再到新目錄繼續。建立與初始化實例不需獨立 review（界線見 [instance-template/guides/review.md](instance-template/guides/review.md)「初始化不需獨立 review」）；下一條的框架修改不適用此例外。
- 使用者要修改框架本身（guides、skills、adapters、templates、分類或目錄結構）：先寫完整變更提案並取得獨立 review，流程依 [instance-template/guides/review.md](instance-template/guides/review.md)。
- 任務開始前或進行中遇到尚未釐清的不確定、疑慮或資訊不足：先查目前會話與可取得的事實；仍無法自行查明、且後續工作需要的資訊或使用者決策，用 [grill-me](instance-template/.agents/skills/grill-me/SKILL.md) 分輪提問（列出必要問題與建議答案），等使用者回答並確認共同理解後才繼續相依的工作。沉默或逾時不算回答，也不預設選項或以自訂假設推進；等待時可繼續無關的唯讀調查。這適用任何必要缺口；已確認的答案與已授權範圍不重問、不擴需求。
- 新增或修改任何 agent skill（含 `instance-template/.agents/skills/` 的預載 skills 與 `skill-creator` 本身）：先讀 [skill-creator](instance-template/.agents/skills/skill-creator/SKILL.md)，依 [instance-template/guides/skills.md](instance-template/guides/skills.md) 驗證並記錄結果，再走上一條的提案與 review。
- 預載的第三方 skills 保留上游原文與各自授權（見 [instance-template/ATTRIBUTION.md](instance-template/ATTRIBUTION.md)）；升級時從上游固定 commit 取得檔案，並同步更新 ATTRIBUTION.md 與 `instance-template/EXTENSIONS.md` 的 ref、逐檔 checksum 與本地差異。
- `instance-template/adapters/` 的內容是有查核日期的官方文件摘要；修改前重新查官方來源，並更新查核日期與「未驗證」清單。
- 公開內容只放通用指南與虛構示例：不放個人 email、主機路徑、私人 repository 名稱、tokens、真實 profiles 或對話紀錄。
