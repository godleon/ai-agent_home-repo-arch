# 第三方擴充清單

本實例使用的第三方 skill、extension、plugin、package。流程見 [guides/extensions.md](guides/extensions.md)。內建的 `home-*` skills 與 owner 自寫的 skills 不在此列。

## 預載（隨範本 bundled）

框架隨範本提供、位於 `.agents/skills/` 的第三方 skills。這是預載清單，不是安裝觀測：各 harness 是否載入記在 [SETUP.md](SETUP.md)「Harness 能力證據」。變更依 [guides/skills.md](guides/skills.md)；逐檔 checksum 與授權見 [ATTRIBUTION.md](ATTRIBUTION.md)。

| 名稱 | 類型 | 來源 | 版本／ref | 用途 | 授權 | 本地差異 |
|---|---|---|---|---|---|---|
| `grill-me` | skill | <https://github.com/mattpocock/skills> | `5863806dd4e15c63174bee4e7515a4256792ecfa` | 必要資訊或決策缺口的釐清入口 | MIT | 本地適配（見 ATTRIBUTION.md） |
| `grilling` | skill | <https://github.com/mattpocock/skills> | `5863806dd4e15c63174bee4e7515a4256792ecfa` | `grill-me` 依賴的訪談方法 | MIT | 無；另附上游根目錄 `LICENSE` |
| `skill-creator` | skill | <https://github.com/anthropics/skills> | `1780c5e5a552fd4d1c9d6a7b0cab26b0c2bef064` | 新增、修改與驗證 skill | Apache-2.0 | 無 |

## 實例安裝觀測

本實例經 harness 原生機制安裝的第三方擴充。

| 名稱 | 類型 | Harness | Scope | 來源 | 版本／ref | 用途 | 授權／review | 安裝 | 載入 | Smoke | 核對日期 | 升級／回復 |
|---|---|---|---|---|---|---|---|---|---|---|---|---|

（尚無條目）

## 安裝觀測欄位

- **類型**：`skill`、`extension`、`plugin`、`package`。
- **Scope**：`project` 或 `user`；寫明設定位置的類別，不寫個人路徑。
- **來源**：canonical 官方來源 URL。
- **版本／ref**：實際安裝的版本、tag 或 commit；浮動名稱（例如 latest、main）不算。
- **授權／review**：owner 授權日期與範圍、`reviews/log.md` 的提案 ID；owner 自行安裝而無紀錄者寫 `unknown`。
- **安裝、載入、Smoke**：分開填 `ok`、`failed`、`unverified` 或 `not-run`，附簡短原因。
- **升級／回復**：前一版本或 ref、回復方式與限制；沒有可回復的舊版時寫明。

查不到的值寫 `unknown`。不寫 tokens、credentials 或個人路徑。
