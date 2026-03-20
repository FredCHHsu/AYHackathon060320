---
name: 確認修改
description: 詢問 git commit message 後，執行 git commit 記錄修改
---

# 確認修改 Skill 指南

當使用者調用此 skill（「確認修改」）時，請遵循以下步驟：

1. **詢問 Commit Message**
   主動向使用者詢問這次修改的內容，請他提供 `git commit message`。

2. **等待使用者輸入**
   暫停指令執行，等待使用者明確提供或同意草稿中的 commit message。

3. **執行 Git Commit**
   取得 commit message 後，使用 `run_command` 工具在終端機依序執行：
   ```bash
   git add .
   git commit -m "<使用者的 commit message>"
   ```
   > 提示：若使用者只想 commit 特定檔案，請依要求調整 `git add` 範圍。

4. **回報結果**
   終端機指令執行完畢後，告知使用者變更已經順利 commit 完成。您可以提醒使用者如果需要推播，可以呼叫「發布」skill。
