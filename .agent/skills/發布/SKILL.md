---
name: 發布
description: 執行 git push 並自動開啟 merge 回 main 分支的 Pull Request
---

# 發布 Skill 指南

當使用者調用此 skill（「發布」）時，請遵循以下步驟來完成從發布到開啟 PR 的完整流程：

1. **執行 Git Push**
   使用 `run_command` 工具在終端機執行推播指令，為了防範尚未設定 upstream 的問題，可以直接使用：
   ```bash
   git push -u origin HEAD
   ```

2. **建立 Pull Request (PR)**
   在確定推播成功後，使用 GitHub CLI (`gh`) 建立合併進 `main` 分支的 Pull Request：
   ```bash
   gh pr create --base main --head HEAD --fill
   ```
   > **註**：`--fill` 會自動拿標題與 commit message 作為 PR 的內容。如果系統因為缺少 `gh` 指令而失敗，請主動說明錯誤，並告知使用者可能需要安裝 `gh` 或登入。

3. **提供 PR 連結**
   `gh pr create` 指令成功執行後，終端機會輸出一條 PR 的網址（例如 `https://github.com/User/Repo/pull/123`）。
   請將該連結整理並以 Markdown 格式顯示給使用者（如：`[點此查看 Pull Request](網址)`），並告知發布流程已全部完成！
