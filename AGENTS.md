# AGENTS.md

本儲存庫是公開的 Agent Skill 套件，遵循 [Agent Skills](https://agentskills.io/) 開放格式，並以 `.agents/skills/` 作為跨工具通用安裝路徑。

## 專案目的

提供 `my-commit-report` skill：讓 AI agent 依目前 `git config` 使用者，整理指定時間區間內的 commit 摘要報告（預設過去兩週）。

## 目錄約定

```text
.agents/skills/my-commit-report/SKILL.md   # skill 本體（唯一必要檔）
AGENTS.md                                  # 本檔：給 agent / 貢獻者的常駐說明
README.md                                  # 給人類的安裝與使用說明
LICENSE                                    # MIT
```

- Skill 目錄名稱必須與 `SKILL.md` frontmatter 的 `name` 一致。
- 不要把 skill 放到 `.cursor/skills-cursor/`（Cursor 內建保留路徑）。
- 若新增其他 skill，放在 `.agents/skills/<skill-name>/SKILL.md`。

## 修改 Skill 時

1. 先讀 `.agents/skills/my-commit-report/SKILL.md`。
2. 保持 frontmatter 的 `name`、`description` 正確；`description` 需同時說明「做什麼」與「何時觸發」。
3. 流程仍以跨平台 `git` 指令為準，不要改成依賴特定 shell（例如只在 PowerShell 能跑的腳本）。
4. 報告輸出維持臺灣正體中文。
5. 不修改使用者的 git config，不執行 destructive git 指令。

## 安全

- 本 skill 僅讀取 git 歷史並產出摘要，不應新增會寫入遠端、改寫歷史、或略過 hook 的步驟。
- 若未來加入 `scripts/`，必須在 README 與 skill 內標明會執行哪些指令。
