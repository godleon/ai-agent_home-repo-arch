# 角色設計

角色由 owner 的用途推導，從最少開始；不是固定四角色，也不是固定模型。初始化時依本檔產出提案表，經 owner 同意才寫入設定。

## 從最少開始

1. 預設只有**主 session**：同時是 supervisor 與唯一 writer。
2. 需要提案 review 時加上**獨立 reviewer**；沒有它，提案只能保持 `pending`。
3. 只有在有理由時再加：
   - **reader**：大量閱讀或搜尋，想用較低成本的設定、或與主 session 平行；
   - **獨立 writer**：主 session 只負責協調與驗收、不直接修改時。

## 職責與權限

| 角色 | 職責 | 權限 |
|---|---|---|
| supervisor | 與 owner 溝通、拆分工作、整合與驗收結果；對外寫入（push、發布）只在 owner 授權範圍內 | 依主 session 設定 |
| writer | 唯一修改檔案與做本機 commit 的角色 | 讀寫實例；不 push、不發布 |
| reader | 閱讀、搜尋、回報證據與未知 | 只讀工具；無 shell 寫入 |
| reviewer | 審查指定 base／head 的 diff 與來源，回報 findings | 只讀工具；不修改受審內容 |

所有 child 角色：不再委派其他 agent、不 push、不發布、不建立排程。工具 allowlist 與文字指示都**不是 OS sandbox**；需要真正隔離時由 container、VM 或 OS 權限提供，並如實說明目前沒有。

## Model 與 effort

只從當前 harness 查到的證據挑選（[initialization.md](initialization.md) 第 3 步），不寫死「最新」或「最大」：

| 角色 | 建議方向 |
|---|---|
| reader | 帳號可用者中成本與延遲較低的 model；較低 effort |
| writer | owner 平常使用、品質足夠的 model；中等 effort |
| reviewer | 可用者中判斷力最強的 model；較高 effort。可用時選與 writer 不同的 model 以增加獨立性 |
| supervisor | 與主 session 相同 |

成本或價格查不到時寫 `unknown`，不估算成另一個數字。

## 提案表

```markdown
| 用途 | 角色 | 權限 | Model／effort | 理由與成本 | 證據層級 |
|---|---|---|---|---|---|
| 審查 Wiki 有效結論提案 | reviewer | 只讀：read, search | <MODEL_ID>／<EFFORT> | 需獨立判斷；成本 unknown | visible |
```

表後附**具體設定 diff**（要新增或修改的檔案與欄位，見各 adapter），並標出既有設定中會被保留的部分。

## 能力不足時

harness 缺少 native subagent、per-role model 或 reasoning 設定時，列出替代方案供 owner 選擇，例如：

- reviewer 改為另開一個獨立 session 手動執行；
- 不設定 per-role model，全部沿用主 session；
- 安裝官方或第三方 extension（需 owner 同意，並記錄來源與版本）。

owner 選定前不套用；不靜默更換 model、effort 或角色，也不宣稱未生效的設定已生效。
