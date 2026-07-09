---
name: my-commit-report
description: >-
  從 Git 儲存庫撈出「我」（目前 git config 使用者）在指定時間區間內的 commit，
  整理成簡潔的摘要報告列表。未指定時間時預設為過去兩週。
  只要使用者提到 commit 摘要、工作紀錄、這段時間做了什麼、我的 commit 報告、
  週報、個人開發紀錄、git log 整理，或想回顧某段期間的提交內容，就應使用此 skill。
license: MIT
metadata:
  author: maylogger
  version: "1.0.0"
---

# 我的 Commit 摘要報告

協助使用者回顧自己在一段時間內的 Git 提交，產出易讀的摘要列表。

## 適用範圍

- **儲存庫**：預設為目前工作區的 Git 根目錄；使用者若指定路徑，改以該路徑為準
- **作者**：以 `git config user.email` 為主；若 email 為空，改用 `git config user.name`
- **時間區間**：使用者未指定時，**預設 `--since="2 weeks ago"` 至今天**
- **語言**：報告與說明使用臺灣正體中文

## 執行流程

### 1. 確認環境

在目標儲存庫執行：

```bash
git rev-parse --is-inside-work-tree
git config user.email
git config user.name
```

若不在 Git 儲存庫內，告知使用者並停止。若 author 資訊為空，請使用者先設定 `git config user.name` / `user.email`。

### 2. 解析時間區間

依使用者描述轉成 `git log` 參數：

| 使用者說法               | git 參數                      |
| ------------------------ | ----------------------------- |
| （未提及）               | `--since="2 weeks ago"`       |
| 過去一週 / 上週          | `--since="1 week ago"`        |
| 過去兩週 / 兩個禮拜      | `--since="2 weeks ago"`       |
| 過去一個月               | `--since="1 month ago"`       |
| 今天                     | `--since="midnight"`          |
| 昨天                     | `--since="yesterday"`         |
| 自 YYYY-MM-DD 起         | `--since="YYYY-MM-DD"`        |
| YYYY-MM-DD 到 YYYY-MM-DD | `--since="..." --until="..."` |

`--until` 未指定時，等同到今天為止。

### 3. 收集 commit 資料

一律用 **git 指令**（跨 PC / Mac 通用），不要依賴 PowerShell 或額外腳本。

先取得作者與 repo 根目錄：

```bash
git -C "<repo>" rev-parse --show-toplevel
git config user.email
git config user.name
```

以 email 為 `--author` 查詢（email 為空時改用 name）：

```bash
git -C "<repo>" log \
  --author="<email 或 name>" \
  --since="<since>" \
  [--until="<until>"] \
  --pretty=format:"---COMMIT---%n%H|%h|%ad|%s" \
  --date=short \
  --numstat
```

**解析輸出：**

- `---COMMIT---` 後第一行：`full-hash|short-hash|date|subject`
- 接著數行 `additions\tdeletions\tfilepath`（binary 檔可能顯示 `-`）
- 加總每筆 commit 的 additions / deletions，得到該 commit 與整體變更規模

若 email 與 name 可能對應不同 author 字串，分別查詢後以 full hash 去重。

**輔助查詢（按需使用）：**

```bash
# 僅列出摘要（快速確認筆數與訊息）
git -C "<repo>" log --author="<author>" --since="<since>" [--until="<until>"] \
  --pretty=format:"%h %ad %s" --date=short

# 實際涵蓋的日期範圍（取最早與最新 commit 日期）
git -C "<repo>" log --author="<author>" --since="<since>" [--until="<until>"] \
  --pretty=format:"%ad" --date=short | tail -1   # 最早
git -C "<repo>" log --author="<author>" --since="<since>" [--until="<until>"] \
  --pretty=format:"%ad" --date=short -1          # 最新

# merge commit 過多時重查
git -C "<repo>" log --author="<author>" --since="<since>" --no-merges ...
```

### 4. 分析與分組

閱讀 commit message，依 **Conventional Commits 的 type**（若有的話）或主題分組，例如：

- `feat` / 新功能
- `fix` / 修正
- `refactor` / 重構
- `style` / 樣式
- `docs` / 文件
- `chore` / 維護
- 其他 / 無 type 前綴

每組用 1～2 句話概括做了什麼，避免逐字複製所有 message。

### 5. 產出報告

使用以下模板，保持簡潔：

```markdown
# Commit 摘要報告

**期間**：YYYY-MM-DD ～ YYYY-MM-DD
**作者**：<name>（<email>）
**儲存庫**：<repo 名稱或路徑>
**Commit 數**：N 筆｜**變更規模**：約 +XXX / -YYY 行

## 重點摘要

- （2～5 條，每條一句話，描述這段時間主要完成了什麼）

## 依主題分類

- 不用列出所有 commit 或 Hash 列表
- 以主題區分內容

## 列表格式

# {時間範圍}

## {主題1}

- {改動摘要}
- {改動摘要}

## {主題2}

- {改動摘要}
- {改動摘要}
```

**格式原則：**

- 「重點摘要」給忙的人快速掃描；「完整列表」保留可追溯細節
- commit 數為 0 時，仍輸出報告標頭並說明「此期間沒有找到你的 commit」，建議檢查時間區間或 author 設定
- 不要貼完整 diff，除非使用者明確要求某幾筆的詳細變更

### 6. 可選延伸

使用者若要求更細節，可針對特定 commit 補充：

```bash
git show <hash> --stat
```

或依檔案類型、目錄統計變更熱點。

## 注意事項

- 只報告 **目前 git config 對應的使用者**；若要查其他人，需使用者明確指定 author
- 不修改 git 設定、不執行 destructive 指令
- merge commit 可保留，但若數量過多且雜訊大，可在報告中註明並可選擇 `--no-merges` 重查
- 跨多個儲存庫不在預設範圍；若使用者指定多 repo，分別產報告後合併摘要

## 範例

**使用者**：「幫我看這兩週我做了什麼」

**行為**：在當前 repo 用 `--since="2 weeks ago"` 查 author=自己，產出上述模板報告。

**使用者**：「3/1 到 3/15 我在這個專案 commit 了什麼」

**行為**：`--since="2026-03-01" --until="2026-03-16"`（until 為次日以包含 3/15 整天），產出報告。
