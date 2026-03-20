---
name: 發布
description: 執行 git push 後，將遠端當前分支自動 merge 進遠端 main 分支
---

# 發布 Skill 指南

當使用者調用此 skill（「發布」）時，請遵循以下步驟來完成推播與遠端 merge 流程：

1. **執行 Git Push**
   使用 `run_command` 工具在終端機執行推播指令，確保本地變更推播到遠端對應的分支：
   ```bash
   git push -u origin HEAD
   ```

2. **遠端 Merge 至 main 分支**
   在確定推播成功後，使用 GitHub CLI (`gh`) 透過建立 Pull Request 並立即合併的方式，將當下 branch 的變更合併進遠端的 `main` 分支。請依序執行以下指令：
   ```bash
   gh pr create --base main --head HEAD --fill || true
   gh pr merge --merge
   ```
   > **註**：第一行 `|| true` 是為了防止 PR 已經存在時指令報錯中斷。第二行 `gh pr merge --merge` 則會將該 PR 正式合併至 `main`。若使用者的 GitHub 設定要求使用 squash 可以改用 `--squash`。

3. **回報結果**
   指令順利完成後，告知使用者分支已經成功推播，並且也已經順利在遠端合併進入 `main` 分支（即發布流程已全部完成）。
