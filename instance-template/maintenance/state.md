# 定期維護狀態

status: not-enabled

> 定期維護未啟用：尚未取得 owner 同意。`not-enabled` 或 `paused` 時，任何排程或 session 都不執行定期維護；owner 當次明確要求的單次整理不受此限，但不改變本狀態。流程見 [../guides/maintenance.md](../guides/maintenance.md)。

## 授權

owner 同意後一次填寫；之後在範圍內執行不再逐次詢問。範圍改變需重新同意。

- 同意日期與方式：
- 頻率：
- 時區：
- Harness、model、effort（含證據層級）：
- 每次預算（檔案數、bytes、token 或費用上限）：
- 可處理的資料範圍：
- 排除範圍：
- 可自動修改的類型：
- 本機 commit：not-allowed
- 同步（push）：not-allowed
- 排程模式（native／os-scheduler／session-due-check）：
- 排程識別與設定位置：

## 驗證

三者分開記錄；設定檔存在不代表排程會觸發。

| 項目 | 結果 | 日期 | 證據 |
|---|---|---|---|
| 排程設定完成並讀回 | not-run | | |
| 人工 dry run | not-run | | |
| 排程實際觸發 | not-run | | |

## Checkpoint

- 已完整處理到的 commit（base）：none
- 下次到期：
- 上次結果：

## 剩餘清單

未完成的批次，下次先處理並沿用同一組 base／head。格式：批次 `base`／`head`，每個項目一行「類型｜讀取版本（`D` 讀 base，其餘讀 head）｜路徑｜blob id｜offset｜原因」。

（無）

## 執行紀錄

（無）
