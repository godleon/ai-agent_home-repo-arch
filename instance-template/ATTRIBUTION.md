# 概念出處

以下為本框架作者以自己的文字整理的概念摘要；未複製原文。原文的授權與最新內容以各來源為準。查閱日期：2026-11-15。

## The Home Repo Pattern

來源：<https://www.aihero.dev/the-home-repo-pattern>

一個 repository 集中保存 skills、`AGENTS.md` 與共用文件，agent 從這裡啟動；`AGENTS.md` 為其他 repositories 提供導覽指標，讓 agent 在跨 repository 工作時有完整視野，並避免各 repository 各自複製 agent 設定。

本框架採用：以實例作為 agent 工作基地，並以 [REPOS.md](REPOS.md) 保存其他 repositories 的指標。

## Agent Memory Repo

來源：<https://github.com/AgentMemoryRepo/agentmemoryrepo>

把 agent 從使用者身上學到的事保存在獨立的 Git repository：`MEMORY.md` 是每個 session 先讀的精簡入口與索引，條目是一行一則、可附 `[key: value]` metadata（例如 `source`、`added`），並以 `[[path]]` 互相連結；另有週期性的 Dreaming 負責新增 pattern、合併重複、移除過時內容與依來源化解矛盾。

本框架的調整：只有 owner 已確認的內容才直接寫入 Memory；本機 commit 不自動 push，同步需另外授權；Dreaming 的語意變更需提案與 review（見 [guides/review.md](guides/review.md)）。

## LLM Wiki

來源：<https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f>

原始來源保持不變；LLM 把來源增量整理成互相連結的 Markdown Wiki，並以一份 schema 文件（例如 `AGENTS.md`）規範結構與流程。主要操作是 ingest（整合新來源）、query（依索引查詢並引用，好的答案可回存）與 lint（找矛盾、過時主張、孤立頁與缺漏）；`index.md` 依內容導覽，`log.md` 依時間記錄。

本框架的調整：LLM 摘要不等於原始事實，需標示來源版本；Wiki 有效結論的新增或改變需提案與獨立 review；矛盾與異議保留而不靜默抹除（見 [guides/wiki.md](guides/wiki.md)、[guides/sources.md](guides/sources.md)）。
