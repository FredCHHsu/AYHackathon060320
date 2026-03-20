---
name: 確認修改並且發布
description: 詢問 git commit message 後，執行 git commit 與 git push
---

# 確認修改並且發布 Skill 指南

當使用者調用此 skill（「確認修改並且發布」）時，請遵循以下步驟來協助使用者進行版本控制與部署：

1. **詢問 Commit Message**
   向使用者詢問這次修改的內容，請他提供 git commit message。
   範例：「請提供這次 git commit 的 message，或是讓我幫你根據近期修改產生一個草稿？」

2. **等待使用者輸入**
   一定要等待使用者確實提供 commit message（或同意由你產生的 message）後，才能進行下一步。

3. **執行 Commit 與 Push**
   取得 commit message 後，使用 `run_command` 工具在終端機依序執行：
   ```bash
   git add .
   git commit -m "<使用者的 commit message>"
   git push
   ```
   > 提示：如果使用者有特定檔案或分支的要求，請視情況調整為特定的 `git add` 範圍或 `git push origin <branch>`。

4. **回報結果**
   終端機指令執行完畢後，告知使用者變更已經順利 commit 並 push 上去了。
