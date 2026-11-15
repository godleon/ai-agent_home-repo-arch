# 擴充管理

管理**第三方**擴充：別人發布、經 harness 原生機制安裝的 skill、extension、plugin 或 package。清單在 [EXTENSIONS.md](../EXTENSIONS.md)。本實例內建的 `home-*` skills 是框架檔案，owner 自己編寫的 skill 屬 [classification.md](classification.md) 的 Skill 類別；兩者都走 [review.md](review.md)，不登記在清單中。隨範本預載的第三方 skills 列在清單「預載」，不是原生安裝；升級或修改依 [skills.md](skills.md) 與 review.md。

## 第三方擴充與自寫 skill

- 現成擴充符合需求就用它：以該 harness 的原生安裝與載入機制處理，不要求 owner 先自行開發，也不改寫成 guide 或 wrapper skill。
- 能力不互換：一個 harness 的擴充不代表其他 harness 有同等能力。缺口依 [roles.md](roles.md)「能力不足時」列替代方案；不新增跨 harness 的 package manager 或 installer。
- 自寫 skill 只在方法經多次實際使用驗證後提議，放 `.agents/skills/`，依 review.md 提案。
- 安裝與載入方式查對應 adapter（[Copilot CLI](../adapters/copilot-cli.md)、[Claude Code](../adapters/claude-code.md)、[Codex CLI](../adapters/codex-cli.md)、[Pi](../adapters/pi.md)）。Adapter 沒寫的指令，查 installed `--help` 或與 installed version 相符的官方文件；兩者都查不到就記為受阻，不猜指令。

## 盤點既有擴充

owner 自行安裝的擴充，只依可觀察證據登記：harness 的已安裝清單或讀回、設定檔中相關 keys、套件 metadata。看不到的欄位寫 `unknown`；授權與 review 沒有紀錄時寫 `unknown`，不寫成已批准或已審查。只擷取需要的欄位，不輸出整份設定或秘密。

未登記或不在預設中的擴充，不代表可以刪除。盤點只記錄現況、不改變任何能力，在 owner 已授權的任務範圍內可直接保存並本機 commit。

## 新增、升級或移除

每步達到完成條件才進下一步。

1. **盤點**：列出相關的既有擴充、同名項目、功能重疊與會碰到的設定 keys。完成條件：衝突與會保留的項目已列出。
2. **查證**：取得 canonical 官方來源、與 installed harness 相容的實際版本或不可變 ref、原生安裝 scope（project 或 user／global）。擴充可能在本機執行程式碼：依來源文件、manifest 與設定宣告等實際證據，判斷它需要的工具、檔案、網路與資料存取。這是有界的權限判斷，不是完整原始碼稽核；adapter 要求審閱原始碼時，範圍也以此為限。升級另核對版本差異、相容性與權限變動。完成條件：每項有證據或標 `unknown`。
3. **提案**：把有界計畫寫成 `reviews/<id>.md`，包含名稱、版本／ref、scope、會新增或修改的檔案與 keys、會保留的既有擴充與設定、同名衝突處理、驗證方式與可能費用、回復方式。移除另列依賴：[roles.md](roles.md) 角色設定、skills、[maintenance/state.md](../maintenance/state.md) 的排程，以及要保留的使用者資料。
4. **Review 與授權**：依 [review.md](review.md) 提案流程取得獨立 review，並另外取得 owner 授權；review 通過不等於授權，owner 授權也不代替 review。user／global scope 與有費用的 smoke 都要明確包含在授權中。已授權的範圍不重問。沒有 reviewer 時保持 `pending`，不先安裝再補審。
5. **套用**：只改計畫列出的檔案與 keys，保留既有擴充、設定與同名項目。實際變更超出計畫（例如原生指令會改其他設定或覆寫同名項目）時停止，回報並重新提案。只移除已授權項目，保留使用者資料。
6. **驗證**：安裝、載入（harness 讀回看得到）、最小 smoke（實際使用一次）三者分開記錄，前者成功不代表後者成功。無法測試寫 `unverified` 與原因。類型是 skill 時，另依 [skills.md](skills.md)「新增或修改 skill」用 `skill-creator` 驗證並記錄。
7. **登記**：更新 EXTENSIONS.md 該列並本機 commit。完成條件：清單反映實際狀態，包括失敗與未驗證項目。

## 回復

變更前在計畫中記下原版本或 ref、被修改 keys 的原值（不含秘密）。回復取決於舊版本與設定是否仍可取得，不保證能 rollback。回復失敗時記錄實際狀態，不宣稱已還原。

## 定期核對

定期維護的授權資料範圍包含 EXTENSIONS.md 時，[maintenance.md](maintenance.md) 每批把它固定列為一個項目（即使沒有變更），核對清單與可觀察的安裝狀態；授權允許網路存取時，也可核對官方來源是否有新版本。發現更新、失效或不一致時，在授權的修改類型內更新清單的觀察結果，並提出變更提案；不自動安裝、升級或移除。
