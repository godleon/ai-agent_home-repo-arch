# 保存、提案與 review

決定一項變更能直接保存、需要提案與獨立 review，或需要 owner decision。分類先依 [classification.md](classification.md)。

## 可直接保存

以下條件**全部**成立時，writer 可直接寫入並做本機 commit，不需另外詢問：

- 任務在 owner 已授權的範圍內；
- 內容可重用，來源已核對；
- 不是 no-save，也不含秘密。

適用的變更：

- 新增 source 摘要（`sources/`）；
- Wiki 草稿（`status: draft`）；
- owner 明確確認的 Memory 條目；
- 機械修正：索引、連結、格式；
- [EXTENSIONS.md](../EXTENSIONS.md) 的盤點：只記錄可觀察的現況，不改變任何能力；
- 上述變更的本機 commit。

本機 commit 不等於 push 或 merge。只有 [SETUP.md](../SETUP.md) 的同步授權（或當前任務明確授予的同步範圍）涵蓋時才 push。授權仍有效時不重複詢問。

## 需完整提案與獨立 review

- 新增或改變 Wiki 的有效結論，包括第一次把 draft 改為 `valid`；
- 刪除重要證據或異議；
- 宣告矛盾已解決；
- 修改 skills、guides、adapters、分類、範本或目錄結構；skill 的新增或修改另依 [skills.md](skills.md) 驗證，結果寫在提案中；
- 新增、升級或移除第三方擴充（依 [extensions.md](extensions.md)，另需 owner 授權）。

## 需 owner decision

- 個人政策：保存範圍、同步、維護授權、資料邊界；
- 擴充的安裝、升級、移除，以及 user／global scope 的設定；
- agent 推論出的 Memory；
- 重大取捨。

owner 已在當前對話明確決定的事項不重問；把決定與日期記在對應條目的 `source`。

## 提案流程

1. **準備**：writer 在 `proposal/<id>` branch 完成變更並本機 commit。在 [reviews/log.md](../reviews/log.md) 新增一列：ID、摘要、base、head、來源、`pending`。
2. **審查**：reviewer 必須獨立於 writer（另一個 session 或 agent 角色，或 owner 本人），只讀不改，審 `git diff <base> <head>` 與所列來源，回報 findings 並把狀態改為 `reviewed`（通過或不通過），綁定該 head。
3. **變更**：審查後 head 有任何改動，狀態回到 `pending`，需重新審查。
4. **採納**：只能套用與 reviewed head 完全相同的變更（例如 fast-forward merge 該 commit）。記錄 `adopted` 或 `rejected` 與理由。

沒有可用的獨立 reviewer 時，提案保持 `pending`，不採納。`pending` 與 `reviewed` 都不是 `adopted`；Git merge 成功也不代表語意已一致。
