# Home Repo Framework

一套公開、通用的 Home Repo 框架：在**你自己擁有的獨立 Git repository** 中，讓 AI coding agents 共用同一份指示、skills、Memory、Wiki 與來源，並以明確的 review 與授權邊界持續維護。

> **這個 repository 是框架，不是 Home Repo 實例。** 請不要在這裡（包含 `instance-template/`）寫入個人資料；依 [INITIALIZE.md](INITIALIZE.md) 在另一個新目錄建立你的實例。

## 核心想法

- **Home Repo**：一個由你擁有的 repository 作為 agent 的工作基地，集中 `AGENTS.md`、skills 與共用文件，並指向你的其他 repositories。
- **Agent Memory Repo**：以 `MEMORY.md` 精簡索引加上互相連結的 Markdown 條目，跨 session 保存已確認的使用者背景、偏好與決策。
- **LLM Wiki**：原始來源保持不變，由 LLM 增量整理成互相連結的 Wiki，並以 ingest／query／lint 維護。

本框架在三者之上加入：保存分類、提案與獨立 review、owner decision、來源版本追蹤、有預算的定期整理，以及四個 harness 的 adapter 指南。來源與差異說明見 [instance-template/ATTRIBUTION.md](instance-template/ATTRIBUTION.md)。

## 結構

| 路徑 | 用途 |
|---|---|
| [AGENTS.md](AGENTS.md)、[CLAUDE.md](CLAUDE.md) | 本框架 repository 的 agent 入口；`CLAUDE.md` 以 `@AGENTS.md` 引用同一入口 |
| [INITIALIZE.md](INITIALIZE.md) | 從範本建立新實例的步驟 |
| [instance-template/](instance-template/README.md) | 自包含的實例骨架：入口、guides、skills、adapters、空索引與「尚未初始化」紀錄 |
| [LICENSE](LICENSE) | MIT |

## 支援的 harness

| Harness | Adapter | 狀態 |
|---|---|---|
| GitHub Copilot CLI | [copilot-cli.md](instance-template/adapters/copilot-cli.md) | 官方文件查核（2026-11-15）；未做 live smoke |
| Claude Code | [claude-code.md](instance-template/adapters/claude-code.md) | 官方文件查核（2026-11-15）；未做 live smoke |
| Codex CLI | [codex-cli.md](instance-template/adapters/codex-cli.md) | 官方文件查核（2026-11-15）；未做 live smoke |
| Pi | [pi.md](instance-template/adapters/pi.md) | 官方文件查核（2026-11-15）；未做 live smoke |

「官方文件查核」只代表文件當日描述的能力；你的 installed version、帳號可用 models 與實際行為，要在初始化時查證並記錄。本框架不提供 installer、daemon 或跨 harness runner。

## 狀態

範本為**尚未初始化**、**定期維護未啟用**。初始化與維護只在你的實例中、經你同意後進行。

## 授權

[MIT License](LICENSE)，Copyright (c) 2026 godleon。
