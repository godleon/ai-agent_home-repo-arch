# 建立 Home Repo 實例

本檔只負責把範本複製成一個**新的獨立 Git repository**。用途訪談、harness 設定與維護授權在新實例內，依其 `guides/initialization.md` 進行。

## 1. 選擇目標目錄

條件全部成立才繼續：

- 目標目錄不在本框架 checkout 之內。
- 目標目錄不存在，或存在但為空。
- 目標的上層目錄不在任何 Git working tree 內（避免實例被其他 repository 當成未追蹤檔案）。

```sh
FRAMEWORK=/path/to/framework-checkout   # 本 repository 的 checkout
TARGET=/path/to/my-home-repo            # 新實例位置
PARENT=$(dirname "$TARGET")

test ! -e "$TARGET" || test -z "$(ls -A "$TARGET")"
! git -C "$PARENT" rev-parse --is-inside-work-tree 2>/dev/null
```

兩個檢查都需成功（exit 0）。任一失敗就換目標目錄。

## 2. 複製範本並建立獨立 Git root

```sh
mkdir -p "$TARGET"
cp -R "$FRAMEWORK/instance-template/." "$TARGET/"
git -C "$FRAMEWORK" rev-parse HEAD      # 記下框架 commit，稍後寫入 SETUP.md
cd "$TARGET"
git init
git rev-parse --show-toplevel           # 必須等於 $TARGET
git remote -v                           # 必須沒有任何輸出
```

`instance-template/.` 會一併複製隱藏的 `.agents/` 目錄（含 `home-*` 與預載的第三方 skills 及其授權檔），但不會帶入框架的 `.git`。預載只代表檔案已複製，是否被 harness 載入在新實例初始化的第 3 步確認。新實例預設**沒有 remote**；跨機同步要等初始化時另外決定 private remote（見新實例的 `guides/initialization.md`）。

完成條件：`git rev-parse --show-toplevel` 等於目標目錄，`git remote -v` 無輸出，`SETUP.md` 仍為 `status: uninitialized`。

## 3. 在新實例中初始化

在新目錄啟動你的 harness（GitHub Copilot CLI、Claude Code、Codex CLI 或 Pi），要求「初始化這個 Home Repo」。Agent 會讀新實例的 `AGENTS.md`，經 `home-init` skill 進入初始化 guide。Harness 沒有自動發現 skills 時，請它直接讀 `.agents/skills/home-init/SKILL.md`。

## 之後更新框架

框架有新版本時，不要整包覆蓋實例。以框架的 `git diff <舊 commit> <新 commit> -- instance-template/` 產生變更提案，依實例的 `guides/review.md` 審查後再套用；skills 的變更另依實例的 `guides/skills.md` 驗證。Memory、Wiki、sources 與紀錄不屬於框架檔案，不被更新覆寫。
