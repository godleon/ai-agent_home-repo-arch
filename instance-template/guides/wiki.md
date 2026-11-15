# Wiki

Wiki 是由來源增量整理、互相連結的知識：事實、研究、比較與標明的推論。來源規則見 [sources.md](sources.md)；保存與 review 見 [review.md](review.md)。

## 頁面格式

`wiki/<topic>.md`，開頭：

```markdown
# 主題名稱

status: draft
sources: [source-id-1], [source-id-2]
updated: YYYY-MM-DD
review: （valid 時填採納的 review ID）
```

- 每個主張附來源 ID；推論以「推論：」開頭並說明依據。
- 矛盾或不同觀點放在 `## 異議與未決`，不刪除、不靜默改寫。
- `draft` 可直接新增或更新；`valid`（有效結論）的新增或改變需提案與獨立 review。

## Ingest

1. 確認來源已在 `sources/`（原檔或指標＋摘要）。
2. 讀摘要，必要時回到原始來源核對。
3. 更新或新增相關頁面（draft），補上互相連結；與既有內容矛盾時寫入 `## 異議與未決`。
4. 更新 [wiki/index.md](../wiki/index.md)，在 [wiki/log.md](../wiki/log.md) 追加一筆。

完成條件：每個新增主張都能追到來源 ID，索引與 log 已更新。

## Query

先讀 `wiki/index.md` 找相關頁，再讀頁面並附來源回答。有重用價值的答案可回存為 draft 頁，並在 log 追加一筆。

## Lint

- 壞連結、孤立頁、索引缺漏：機械修正，可直接處理。
- 無來源的主張、來源版本已更新、`needs-recheck` 頁：標記並列出，不自行改寫有效結論。
- 頁面間矛盾：記入 `## 異議與未決`；宣告已解決需提案。
- 重複頁：內容等價時合併並保留重新導向連結；不等價時提案。

每次 lint 在 log 追加一筆，列出已修正與待處理項目。
