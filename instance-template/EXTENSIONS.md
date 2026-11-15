# 第三方擴充清單

本實例使用的第三方 skill、extension、plugin、package。流程見 [guides/extensions.md](guides/extensions.md)。內建的 `home-*` skills 與 owner 自寫的 skills 不在此列。

| 名稱 | 類型 | Harness | Scope | 來源 | 版本／ref | 用途 | 授權／review | 安裝 | 載入 | Smoke | 核對日期 | 升級／回復 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|

（尚無條目）

## 欄位

- **類型**：`skill`、`extension`、`plugin`、`package`。
- **Scope**：`project` 或 `user`；寫明設定位置的類別，不寫個人路徑。
- **來源**：canonical 官方來源 URL。
- **版本／ref**：實際安裝的版本、tag 或 commit；浮動名稱（例如 latest、main）不算。
- **授權／review**：owner 授權日期與範圍、`reviews/log.md` 的提案 ID；owner 自行安裝而無紀錄者寫 `unknown`。
- **安裝、載入、Smoke**：分開填 `ok`、`failed`、`unverified` 或 `not-run`，附簡短原因。
- **升級／回復**：前一版本或 ref、回復方式與限制；沒有可回復的舊版時寫明。

查不到的值寫 `unknown`。不寫 tokens、credentials 或個人路徑。
