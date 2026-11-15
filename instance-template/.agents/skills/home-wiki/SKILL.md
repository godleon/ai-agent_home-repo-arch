---
name: home-wiki
description: 把來源 ingest 進 Home Repo 的 sources 與 Wiki、以 Wiki 回答問題，或 lint Wiki。Use when the owner shares an article, document, or URL to keep, asks a research question answerable from saved knowledge, or asks to check the Wiki for contradictions or broken links.
---

# Home Repo Wiki

1. 確認 [SETUP.md](../../../SETUP.md) 為 `status: initialized` 且 Git top level 是本實例根目錄；只讀查詢可在未初始化時進行，但不寫入。
2. 來源的保存、版本與摘要更正依 [sources.md](../../../guides/sources.md)。
3. Ingest、query、lint 依 [wiki.md](../../../guides/wiki.md)。
4. 有效結論的新增或改變、宣告矛盾已解決、刪除重要證據或異議，依 [review.md](../../../guides/review.md) 提案，不直接套用。

完成條件：每個新增主張可追到來源 ID，索引與 `wiki/log.md` 已更新，回報中列出待 review 的提案。
