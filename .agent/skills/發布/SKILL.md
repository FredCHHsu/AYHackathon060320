---
name: 發布
description: 執行 git push 後，建立 Pull Request 並等待人員 review
---

# 發布 Skill 指南

當使用者調用此 skill（「發布」）時，請遵循以下步驟來完成推播與建立 PR 流程：

1. **執行 Git Push**
   使用 `run_command` 工具在終端機執行推播指令，確保本地變更推播到遠端對應的分支：
   ```bash
   git push -u origin HEAD
   ```

2. **建立 Pull Request (等待 Review)**
   在確定推播成功後，使用 GitHub CLI (`gh`) 建立一個 Pull Request 至 `main` 分支，交由團隊人員 review：
   ```bash
   gh pr create --base main --fill
   ```

3. **回報結果與連結**
   指令順利完成後，從終端機擷取生成的 PR 連結。告知使用者分支已經成功推播，並且 PR 已經建立，可以將該連結交給負責的 reviewer 進行審核。
