# Skills：預載與變更

本實例的 skills 都在 `.agents/skills/<name>/`。本檔說明預載的第三方 skills、新增或修改任何 skill 時的驗證，以及 `skill-creator` 在各 harness 的使用。

## 預載 skills

| Skill | 用途 | 來源與授權 |
|---|---|---|
| `grill-me` | 必要資訊或 owner 決策缺口的釐清入口（本地適配版） | mattpocock/skills，MIT |
| `grilling` | `grill-me` 依賴的分輪訪談方法（上游原文） | mattpocock/skills，MIT |
| `skill-creator` | 新增、修改與驗證 skill（上游原文） | anthropics/skills，Apache-2.0 |

固定 ref 與本地差異列在 [EXTENSIONS.md](../EXTENSIONS.md)「預載」；逐檔 checksum 與授權歸屬見 [ATTRIBUTION.md](../ATTRIBUTION.md)。

- **預載＝隨範本 bundled**：初始化時跟著範本複製。它不代表任何 harness 已安裝、已載入或已實測；是否載入以 [initialization.md](initialization.md) 第 3 步的讀回為準。
- **載入方式**：依 [adapters](../adapters/) 能原生發現 `.agents/skills/` 的 harness 用原生方式；沒有原生 skill 工具或未發現時，直接讀 `.agents/skills/<name>/SKILL.md` 執行其中的文字工作流。Skill 被 harness 權限或 policy 拒絕、停用時（有些 harness 會因此不列出它，見各 adapter），記為受阻並回報，不以直接讀取繞過。腳本需要的工具仍要實際檢查（見「skill-creator 的依賴」）。
- **同名衝突**：harness 同時發現同名的使用者／全域 skill 時，以本實例路徑為準。初始化不覆寫既有的同名 skill：先列出兩邊的版本與差異，依 [review.md](review.md) 提案。
- **權限**：skill 文字（含上游原文）不擴大 owner 或 harness 給的權限。預載不等於授權執行其中的腳本、使用網路、做外部寫入或派 subagent；這些照當前授權處理。

## 新增或修改 skill

適用任何 agent skill 的新增或修改：自寫、第三方匯入或升級、本地適配，以及 `SKILL.md`、description、`agents/`、`scripts/`、`assets/` 的任何變更；`skill-creator` 本身也一樣。每步達到完成條件才進下一步。

1. **讀 skill-creator**：讀 [skill-creator](../.agents/skills/skill-creator/SKILL.md)，依其 intent → 撰寫或適配 → 驗證 → review 的順序進行。以變更前已核對的版本（例如 `git show <base>:<path>` 取出到暫存目錄）作為比較基準與檢查工具；修改 skill-creator 自己時，也用變更前的版本檢查修改後的版本。完成條件：意圖、範圍與要驗證的行為已寫下。
2. **驗證**：依變更內容選擇，至少包含：
   - **結構與 frontmatter**：`SKILL.md` 存在，`name` 與目錄名一致，`description` 寫明用途與觸發情境。有 Python 3 與 PyYAML 時可跑 `quick_validate.py`（先讀過腳本；它只讀檔案）。它允許的 frontmatter keys 依 Anthropic 規格，harness 專用 key 被拒時記錄原因與處理方式。
   - **引用與依賴**：相對連結、引用的檔案、scripts、assets 與依賴的其他 skill 都存在。
   - **行為情境**：挑與變更相符的少數情境，寫下預期行為，追蹤 skill 文字是否導向它。靜態追蹤要標明不是 live eval。
   - **腳本或邏輯**：有變更時加跑相關的可執行檢查（例如語法檢查、離線執行或既有測試）。

   完成條件：每項有結果，或標 `unverified` 與原因。
3. **記錄**：在提案（`reviews/<id>.md`）寫下版本或 ref、驗證方法、結果與未驗證項目。
4. **處理結果**：失敗就修正後重驗。必要項目無法驗證時列為 blocker；未通過時回報實際狀態，不宣稱完成或已載入。
5. **Review**：依 [review.md](review.md) 提案並取得獨立 review。驗證不代替 review 或 owner decision。

動態 eval、benchmark 與 description optimization 會呼叫 model 而產生費用：只在 owner 同意範圍與費用後執行，一般變更不需要。

## skill-creator 的依賴

`skill-creator` 原樣保留，內容以 Claude 產品為主要對象。撰寫、檢查與 review 的部分可直接照文字遵循，不需要先跑動態 eval。下表依實際腳本列出需要的工具；使用前查證是否存在，不自動安裝，也不假定已有：

| 部分 | 需要 | 注意 |
|---|---|---|
| `scripts/quick_validate.py` | Python 3、PyYAML | 只讀檔案 |
| `scripts/package_skill.py` | Python 3、PyYAML | 寫出 `.skill` 壓縮檔 |
| `scripts/aggregate_benchmark.py`、`scripts/generate_report.py` | Python 3 標準函式庫 | |
| `eval-viewer/generate_review.py` | Python 3 標準函式庫 | 預設先結束佔用該 port 的 process（用 `lsof`）、啟動 localhost server 並開瀏覽器；`--static <path>` 只寫出 HTML |
| `scripts/run_eval.py`、`scripts/improve_description.py`、`scripts/run_loop.py` | `claude` CLI（`claude -p`） | 呼叫 model，產生費用。`run_eval.py` 在最近一個含 `.claude/` 的上層目錄（找不到時為目前目錄）建立暫時 command 檔，要在可丟棄的副本中執行 |

- 部分腳本的型別語法需要 Python 3.10 以上；匯入 `scripts.` 的腳本以 `python -m scripts.<name>` 在 skill-creator 目錄執行。
- 上游流程的 subagent 平行執行，只在 harness 有此能力且已授權時使用；否則依上游 Claude.ai 段落，由主 agent 依序執行。
- 非 Claude 的 harness：不要把目前 provider 的 model ID 傳給 `claude`；只用 `claude` CLI 可用、owner 已同意的相容 model 與範圍。缺 `claude` CLI 或其他依賴時如實回報，改用不需動態 eval 的部分。
