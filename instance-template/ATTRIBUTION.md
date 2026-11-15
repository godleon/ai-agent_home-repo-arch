# 出處與授權

前三節是本框架作者以自己的文字整理的概念摘要，未複製原文；原文的授權與最新內容以各來源為準。查閱日期：2026-11-15。最後一節「預載第三方 skills」的檔案則保留上游原文與各自的授權。

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

## 預載第三方 skills

以下 skills 從上游固定 commit 取得，隨範本放在 `.agents/skills/`。它們**不**依本框架的 MIT 授權（[FRAMEWORK-LICENSE](FRAMEWORK-LICENSE)），而依各自的授權：

| Skill | 來源 | 固定 commit | 上游路徑 | 授權 |
|---|---|---|---|---|
| `grill-me` | <https://github.com/mattpocock/skills> | `5863806dd4e15c63174bee4e7515a4256792ecfa` | `skills/productivity/grill-me` | MIT，Copyright (c) 2026 Matt Pocock；全文見該目錄 `LICENSE` |
| `grilling` | 同上 | 同上 | `skills/productivity/grilling` | 同上 |
| `skill-creator` | <https://github.com/anthropics/skills> | `1780c5e5a552fd4d1c9d6a7b0cab26b0c2bef064` | `skills/skill-creator` | Apache-2.0；全文見該目錄 `LICENSE.txt` |

上游 `grill-me` 只是要求以 Skill 工具呼叫 `grilling` 的 wrapper，所以一併預載 `grilling`。mattpocock/skills 的授權在 repository 根目錄，所以把根目錄 `LICENSE` 原樣複製到 `grill-me/` 與 `grilling/`。`skill-creator` 上游目錄共 18 個檔案，全部原樣保留。

### 本地差異

只有 `grill-me` 經本地適配，其餘檔案與上游 blob 相同：

- `grill-me/SKILL.md`：改寫為繁體中文；description 加入「任務中遇到必要資訊或決策缺口」的自動觸發；移除 `disable-model-invocation: true`；把「Call the Skill tool with "grilling"」改為有原生 skill 工具時呼叫、沒有時直接讀 `../grilling/SKILL.md`，並以本實例路徑為準；加入「本實例範圍」（只問必要缺口、事實自己查、subagent 只在可用且已授權時使用、等待規則依 `AGENTS.md`）。
- `grill-me/agents/openai.yaml`：`policy.allow_implicit_invocation` 由 `false` 改為 `true`。

### 檔案清單

路徑相對於 `.agents/skills/`。「上游 blob」是上游固定 commit 中該檔的 Git blob id；原文檔可用 `git hash-object <路徑>` 核對是否相同。

| 路徑 | 上游 blob | 狀態 |
|---|---|---|
| `grill-me/SKILL.md` | `3947ff9c4ad980d14fc07fccbf659d47c114e81d` | 本地適配 |
| `grill-me/agents/openai.yaml` | `4d6fb0c746c5d21364dce5d0cf8c51eab9712e7b` | 本地適配 |
| `grill-me/LICENSE` | `f1dd2c09108dde1a5f56097cee8461b3ea834499` | 原文（上游根目錄 `LICENSE`） |
| `grilling/SKILL.md` | `d2f09a18e6a6c65c55f1271ba08dfeaf06f0bed3` | 原文 |
| `grilling/agents/openai.yaml` | `ddbdb96139c0c1dfe6bca698f39d0465674b8a39` | 原文 |
| `grilling/LICENSE` | `f1dd2c09108dde1a5f56097cee8461b3ea834499` | 原文（上游根目錄 `LICENSE`） |
| `skill-creator/LICENSE.txt` | `4f881c52d1f72f4cfb720e339e2d35c3058d01a9` | 原文 |
| `skill-creator/SKILL.md` | `65b3a402dbd09b8e83f9d637c6b553875189085c` | 原文 |
| `skill-creator/agents/analyzer.md` | `14e41d6068635f4dd3fb878fd1626312395dda63` | 原文 |
| `skill-creator/agents/comparator.md` | `80e00eb45db3ee53a132fc2ba97fd59a7339e563` | 原文 |
| `skill-creator/agents/grader.md` | `558ab05c0a9a8bb062ef4c51823d4d76c3acf7c4` | 原文 |
| `skill-creator/assets/eval_review.html` | `938ff32aed9bffabf723bd5492d720f4736c8e4d` | 原文 |
| `skill-creator/eval-viewer/generate_review.py` | `7fa5978631fed1ed545591dbb2b0eb21ce3f3d08` | 原文 |
| `skill-creator/eval-viewer/viewer.html` | `6d8e96348a02e66c3363d2ff3b3ae58ac11e6382` | 原文 |
| `skill-creator/references/schemas.md` | `b6eeaa2d4a34c1653069585c6c5603da39a5bdbe` | 原文 |
| `skill-creator/scripts/__init__.py` | `e69de29bb2d1d6434b8b29ae775ad8c2e48c5391` | 原文 |
| `skill-creator/scripts/aggregate_benchmark.py` | `3e66e8c105be9bab9f0e9c61f0d1482619401580` | 原文 |
| `skill-creator/scripts/generate_report.py` | `959e30a0014ec165c41a2bb7420b7dfe1416bbac` | 原文 |
| `skill-creator/scripts/improve_description.py` | `06bcec76122446986e3610c20a39c466de36f495` | 原文 |
| `skill-creator/scripts/package_skill.py` | `f48eac444656ddc41204aac1760a217951ce609e` | 原文 |
| `skill-creator/scripts/quick_validate.py` | `ed8e1dddce77b16af13c6f36b3fe86c4ac7c590c` | 原文 |
| `skill-creator/scripts/run_eval.py` | `e58c70bea39d5b252a1e819f242bbdcdf20e8b87` | 原文 |
| `skill-creator/scripts/run_loop.py` | `30a263d674ef19de11c756d6f7537f91a421909e` | 原文 |
| `skill-creator/scripts/utils.py` | `51b6a07dd57174197a937034b7eecebd5768ff8a` | 原文 |

`skill-creator/scripts/` 中除 `__init__.py`、`utils.py` 外的腳本，在上游為可執行檔（mode 100755），本地相同。
