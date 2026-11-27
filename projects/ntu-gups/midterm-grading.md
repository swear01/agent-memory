---
title: GUPS3006 批改：學生對應、COOL 發布與 ABC 工具差異
scope: projects/ntu-gups
status: done
updated: 2026-11-10
tags:
  - ntu-gups
  - grading
  - ntu-cool
---

- **手寫學號可能對到其他人。** 依姓名對 COOL `/api/v1/courses/<c>/users?enrollment_type[]=student` 的 `sis_user_id`／`login_id`，再修本機 csv；師大交換修課者可能沒有 sis_user_id。旁聽生與未繳交分開。個別評分、附件與名單留私人課程資料。調分頁的 localStorage 不會自動改 csv；先備份頁面值，定稿寫回並讀回總分，再清舊覆寫。
- **用錯扣分標準或把批改存檔直接發布。** 按當次助教核定 rubric；核心答案對可給分，嘗試分限方向對，細節按該小題扣分單位。先用卷面證據解釋落差；COOL 留言、逐題明細與附件按使用者指定，內部備註不發。寫入後查數值、附件和 `posted_at`；隱藏與公布分開，須依課程授權。[COOL API 說明](../../domains/other/ntu-cool-api-access.md)
- **ABC 新舊版結果差異被當成學生流程錯誤。** HW2 評分看指定流程與 `print_stats`，該次只缺輸出扣 2、以有效資料確認實作，可接受工具版本差異。c432 以 strash 後 and 數及等價為準，該次 ≤119 滿分、121 → 80%、137 → 50%、其間線性，整數四捨五入。新版用 `read_lib`，`read_verilog -m` 前讀庫，CEC 避免 black boxes。AIGER roundtrip 若少 PI 名稱，跟已驗證 AIG 比。36-input c432 的 `&rewire` 可能爆真值表，舊版 `abc10216.exe` 可產不同 gate/area，必要先轉 BLIF；當次分數與發佈收條見各私人正式資料。

[原始紀錄](https://github.com/swear01/agent-memory/blob/2505b5e7b1170ecc3a61858e10894b2af847b5b7/projects/ntu-gups/midterm-grading.md) · [HW2 ABC 批改與工具驗證](https://github.com/swear01/agent-memory/blob/2505b5e7b1170ecc3a61858e10894b2af847b5b7/projects/ntu-gups/hw2-abc-grading.md)
