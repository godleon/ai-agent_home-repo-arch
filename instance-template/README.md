# Home Repo

這是由 Home Repo Framework 範本建立的個人 Home Repo：讓 AI coding agents 在同一個你擁有的 Git repository 中，共用指示、skills、Memory、Wiki 與來源。

> 如果你看到這份檔案的位置是公開框架裡的 `instance-template/`，這裡只是範本。請依框架根目錄的 `INITIALIZE.md` 在別處建立實例。

## 狀態

初始化狀態與決定記錄在 [SETUP.md](SETUP.md)；定期維護是否啟用記錄在 [maintenance/state.md](maintenance/state.md)。剛建立的實例兩者都是「未啟用」，直到你在初始化中逐項同意。

## 開始使用

在本目錄啟動 GitHub Copilot CLI、Claude Code、Codex CLI、Pi 或 OpenCode，要求「初始化這個 Home Repo」。Agent 會先訪談你的用途、資料邊界、主要任務、成本／延遲偏好、同步與維護需求，再提出角色與設定變更，經你同意後才套用並驗證；初始化不需要另找 reviewer。流程見 [guides/initialization.md](guides/initialization.md)。

## 目錄

| 路徑 | 內容 |
|---|---|
| `AGENTS.md`、`CLAUDE.md` | agent 入口；`CLAUDE.md` 以 `@AGENTS.md` 引用同一入口 |
| `MEMORY.md`、`memory/` | 你確認過的背景、偏好、目標、決策與教訓 |
| `wiki/` | 由來源整理出的事實、研究、比較與推論 |
| `sources/` | 原始來源或其指標，以及 LLM 摘要 |
| `reflections/` | 待確認的反思與改善候選 |
| `reviews/` | 提案與獨立 review 紀錄 |
| `maintenance/` | 定期維護授權、checkpoint 與執行紀錄 |
| `REPOS.md` | 外部專案索引：只放導航指標，專案本身在 Home Repo 之外（見 [guides/projects.md](guides/projects.md)） |
| `EXTENSIONS.md` | 預載與已安裝的第三方 skills、extensions、plugins、packages 清單 |
| `guides/`、`adapters/`、`.agents/skills/` | 框架提供的規則、harness adapter 與 skills（含預載的 `grill-me`、`grilling`、`skill-creator`、`find-skills`，見 [guides/skills.md](guides/skills.md)） |

## 授權與出處

框架衍生的檔案（guides、adapters、skills、範本）依 [FRAMEWORK-LICENSE](FRAMEWORK-LICENSE)（MIT）授權；預載的第三方 skills 除外，依各自授權（`skill-creator` 為 Apache-2.0，`grill-me`、`grilling`、`find-skills` 為 MIT，全文在各 skill 目錄）。你自行加入的 Memory、Wiki、來源與紀錄屬於你，不受該授權影響。概念與預載 skills 的出處見 [ATTRIBUTION.md](ATTRIBUTION.md)。
