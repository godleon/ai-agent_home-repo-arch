---
name: grill-me
description: 釐清缺口：任務開始前或進行中，遇到無法自行查明、後續工作需要的資訊或 owner 決策時，分輪提問並等待回答後再繼續。也用於 owner 要求 grill 或壓力測試計畫、決策或想法。
---

# Grill me（本實例適配版）

改寫自上游 `grill-me` wrapper；來源與差異見 [ATTRIBUTION.md](../../../ATTRIBUTION.md)。

## 載入 grilling

照本實例的 [grilling](../grilling/SKILL.md) 進行訪談。有原生 skill 工具、且確定載入的是本實例 `.agents/skills/grilling/` 時，可用工具呼叫 `grilling`；沒有工具，或可能載入同名的使用者／全域版本時，直接讀 `../grilling/SKILL.md` 並照做。

## 本實例範圍

訪談方法依 grilling 原文；以下範圍優先於原文：

- **必要缺口**：因釐清而使用時，只問本任務往下做所需、無法自行查明的資訊或 owner 決策。已確認的答案與已授權範圍沿用，不擴充需求。沒有需要 owner 回答的缺口就回到原任務。owner 明確要求 grill 整個計畫時，才依原文走完整個 design tree。
- **事實自己查**：先查目前會話、repository 與其他可取得的事實。原文的 sub-agent 只在 harness 有此能力、且已在授權範圍內時使用；否則由主 agent 自己查。
- **等待與恢復**：依 [AGENTS.md](../../../AGENTS.md)「釐清」：等 owner 回答並確認共同理解後，才繼續相依的工作。
