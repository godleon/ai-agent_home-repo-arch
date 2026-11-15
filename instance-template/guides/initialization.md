# 初始化

依序完成以下步驟，每步達到完成條件才進下一步。結果寫入 [SETUP.md](../SETUP.md)。某一步因能力缺失受阻時，記在 SETUP.md 的「未完成或受阻」，**只停止依賴它的步驟**，其他步驟繼續。

## 1. 確認位置

```sh
git rev-parse --show-toplevel    # 必須等於本實例根目錄
git remote -v                    # 新實例應無輸出
```

- top level 不等於實例根目錄（例如在公開框架的 `instance-template/` 中）：**停止**，不寫入任何檔案，請 owner 依框架的 `INITIALIZE.md` 在別處建立實例。
- 已有 remote：確認它是 owner 刻意設定的 private remote（見第 8 步）；否則停止並回報。
- `SETUP.md` 已是 `status: initialized`：這是重新設定，只處理 owner 指定的部分。

完成條件：位置與 remote 檢查結果已記在 SETUP.md「實例」，含框架來源 URL@commit（若 owner 有提供）。

## 2. 用途訪談

分輪提問；每題附推薦答案，讓 owner 可以直接同意。只問會改變設定的問題：

- 主要用途與主要任務；
- 可保存與不可保存的資料、no-save 類別；
- 操作邊界：哪些地方可修改、哪些外部寫入需要逐次確認；
- 成本與延遲偏好；
- 是否需要跨機同步；
- 是否需要定期維護，以及大致頻率。

完成條件：SETUP.md「用途訪談」每欄有 owner 確認的答案，或明確寫「未決定」。

## 3. 查證 harness 能力

依 owner 使用的 harness 讀對應 adapter：[Copilot CLI](../adapters/copilot-cli.md)、[Claude Code](../adapters/claude-code.md)、[Codex CLI](../adapters/codex-cli.md)、[Pi](../adapters/pi.md)。每個 harness 記錄：

- installed version 與 `--help` 中實際存在的相關 flags；
- instructions 與 skills 是否載入（用 adapter 的讀回指令確認）；
- native subagent 或 extension 是否存在；既有第三方擴充依 [extensions.md](extensions.md) 盤點登記；
- 可用 models，以及**每個 model** 的合法 reasoning／effort 值與其來源。

每項標證據層級：`docs`（官方文件）、`visible`（目前安裝與帳號可見）、`live`（實際執行成功）、`unknown`。官方 model catalog 只是 `docs`，不代表目前帳號可用；不用訓練知識補空格。

完成條件：SETUP.md「Harness 能力證據」每列填完，未知者寫 `unknown`。

## 4. 角色與設定提案

依 [roles.md](roles.md) 產出提案表，表後附具體設定 diff：要新增或修改的檔案、欄位與值，以及會保留的既有設定。Model 與 effort 只用第 3 步取得的值；不支援或未知時列出替代方案，讓 owner 選擇。替代方案需要安裝或升級第三方擴充時，該部分依 [extensions.md](extensions.md) 另走提案、review 與授權，不在本步直接套用。

完成條件：owner 對提案表與 diff 明確同意、修改或拒絕；未同意的部分不套用。

## 5. 套用

- 只套用 owner 同意、且 harness 官方支援的**局部**設定。
- 修改既有設定檔前先讀出現況；只改提案中列出的 keys，保留其他內容。新檔不覆寫同名既有檔。
- 不在輸出、log 或 commit 中顯示 tokens、credentials 或整份個人設定。
- 實例內的設定檔（例如 project agent profiles）可 commit；使用者層級設定只記錄改了哪些 keys。

完成條件：每項變更都記在 SETUP.md「設定變更」。

## 6. 讀回

用 adapter 的讀回方式確認 harness 實際看到的值（instructions、skills、profiles、model、effort）。讀回值與提案不同（例如被 fallback 或被更高優先設定遮蔽）時，記錄差異並請 owner 決定，不自行改用其他值。

完成條件：每項變更有讀回結果，或標 `unverified` 與原因。

## 7. 最小 smoke

只在 owner 同意（含可能的費用）後執行 adapter 列出的最小 smoke：先無工具的單次回應；有設定角色時，再由主 session 派一個最小 child 任務給該角色。記錄實際 model／effort 的證據；模型自我描述不算證據。

完成條件：SETUP.md「Smoke」記錄結果；未執行者寫 `not-run` 與原因。設定已寫入不等於 smoke 通過。

## 8. 同步（選用）

owner 想跨機同步時：

1. 由 owner 指定 remote 位置；先確認它是 private（例如 GitHub：`gh repo view <owner>/<repo> --json visibility`），結果記入 SETUP.md。
2. owner 同意後才 `git remote add`，並記錄可 push 的 branch 與時機。

完成條件：「核可 remote」與「同步授權」已記錄，或維持 `none`。

## 9. 定期維護授權

owner 需要定期維護時，依 [maintenance.md](maintenance.md) 逐欄取得同意並填入 [maintenance/state.md](../maintenance/state.md)，再依 adapter 設定排程（或選 `session-due-check`）。設定完成、人工 dry run、實際觸發分開記錄。owner 不同意或未決定：維持 `status: not-enabled`。

完成條件：`state.md` 為 `enabled`（授權欄位齊全）或 `not-enabled`。

## 10. 完成

1. 確認 SETUP.md 各段已填，受阻項目已列出。
2. 改為 `status: initialized`。
3. 本機 commit 初始化結果。
4. 向 owner 回報：已配置、已讀回、已實測、仍受阻或未驗證的項目，分開列出。

完成條件：commit 已建立，回報中沒有把「已配置」說成「已實測」。
