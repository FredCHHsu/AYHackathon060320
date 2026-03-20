---
name: new-feature
description: 建立並切換到以 feat/ 為開頭的新 git 分支
---

# new-feature Skill 指南

當使用者調用此 skill（例如輸入「執行 new-feature xxxx」或「new-feature xxxx」）時，請遵循以下步驟：

1. **辨識分支名稱**
   從使用者的指令中擷取想要建立的功能名稱（亦即 `xxxx` 的部分）。如果使用者沒有提供確切名稱，請先詢問：「請問您想要建立的新功能名稱是什麼？」

2. **執行 Git Checkout**
   取得名稱後，組合出 `feat/<名稱>` 作為新的分支名稱。接著使用 `run_command` 工具在終端機執行以下指令，以建立並切換至該分支：
   ```bash
   git checkout -b feat/<名稱>
   ```

3. **回報結果**
   指令執行完畢後，回報給使用者是否成功建立且切換到了新的 `feat/xxxx` 分支。如果因為分支名稱已存在或含有無效字元而導致錯誤，請將錯誤訊息告知使用者。
