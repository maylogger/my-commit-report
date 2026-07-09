# my-commit-report

開會前整理報告神器：從目前 Git 使用者的 commit 歷史，整理成簡潔的摘要報告（預設過去兩週）。

適用於 Cursor、Claude Code、GitHub Copilot、Codex 等支援 [Agent Skills](https://agentskills.io/) 的工具。

## 使用方式

安裝後，直接用自然語言請 agent 整理即可，例如：

```text
幫我看這兩週我做了什麼
```

```text
過去一週的 commit 摘要
```

```text
3/1 到 3/15 我在這個專案 commit 了什麼
```

```text
幫我寫這段時間的週報（依我的 git commit）
```

Agent 會讀取本 skill，用 git log 收集資料後產出報告。

## 報告範例結構

```markdown
# Commit 摘要報告

**期間**：YYYY-MM-DD ～ YYYY-MM-DD
**作者**：name（email）
**儲存庫**：repo
**Commit 數**：N 筆｜**變更規模**：約 +XXX / -YYY 行

## 重點摘要

- ...

## 依主題分類

### feat / 新功能

- ...
```

## 功能

- 依 `git config user.email`（或 `user.name`）篩選「我的」commit
- 支援常見時間說法：過去一週、兩週、一個月、今天、昨天、自訂日期區間
- 依 Conventional Commits type / 主題分組，產出正體中文摘要
- 統計 commit 數與變更規模（+/- 行數）
- 只讀取 git 歷史，不修改設定、不執行破壞性指令

## 安裝

### 用 skills CLI（建議）

```bash
npx skills add maylogger/my-commit-report
```

全域安裝：

```bash
npx skills add -g maylogger/my-commit-report
```

儲存庫：https://github.com/maylogger/my-commit-report

### 手動安裝

把 skill 目錄複製（或 symlink）到工具會掃描的位置：

| 範圍             | 路徑                                 |
| ---------------- | ------------------------------------ |
| 專案             | `.agents/skills/my-commit-report/`   |
| 使用者（Cursor） | `~/.cursor/skills/my-commit-report/` |
| 使用者（通用）   | `~/.agents/skills/my-commit-report/` |

例如：

```bash
# 專案內
git clone https://github.com/maylogger/my-commit-report.git
mkdir -p .agents/skills
cp -R my-commit-report/.agents/skills/my-commit-report .agents/skills/

# 或使用者層級（Cursor）
mkdir -p ~/.cursor/skills
cp -R my-commit-report/.agents/skills/my-commit-report ~/.cursor/skills/
```

## 專案結構

```text
.
├── .agents/
│   └── skills/
│       └── my-commit-report/
│           └── SKILL.md
├── AGENTS.md
├── LICENSE
└── README.md
```

符合跨工具通用的 `.agents/skills/` 約定。

## 需求

- 目標目錄必須是 Git 儲存庫
- 已設定 `git config user.name` / `user.email`

## License

[MIT](./LICENSE)
