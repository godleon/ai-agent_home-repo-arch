---
name: home-memory
description: 保存、更新或查詢 owner 已確認的背景、偏好、目標、決策與教訓。Use when the owner says remember/forget something, states a preference or decision worth keeping, or asks what is remembered.
---

# Home Repo Memory

1. 確認 [SETUP.md](../../../SETUP.md) 為 `status: initialized` 且 Git top level 是本實例根目錄；否則不寫入。
2. 依 [classification.md](../../../guides/classification.md) 判斷內容是否屬於 Memory；混合內容拆開。
3. 依 [review.md](../../../guides/review.md) 判斷能否直接保存；agent 推論與個人政策需 owner decision。
4. 依 [memory.md](../../../guides/memory.md) 的格式寫入或更新，必要時更新 `MEMORY.md` 的 Index，並做本機 commit。

完成條件：條目附 `source` 與日期、沒有秘密或 no-save 內容，回報寫入位置；未寫入時說明原因。
