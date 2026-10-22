---
title: GUPS3006 EDA course student accounts on gups (pending until tutorial release)
scope: projects/ntu-gups
status: pending
updated: 2026-10-22
---

# GUPS3006 學生 gups 帳號：教材發布時再建立

- 2026-10-22 使用者決定：8 位修課學生的 gups 帳號**先不要建**，等 Innovus U18 教材正式發布時再一起建立。
- 名單（帳號＝NTU COOL Login ID 的 @ 前段，另含一位師大學生）與寄送 Email 存在使用者 Google Drive 的 GUPS助教資料夾 `gups_帳號待建立名單.md`；不放進本記憶庫。
- 建立方式：`useradd -m -G PDK_approved <帳號>`，亂數密碼，建立後用 Email 把密碼寄給本人；不強制第一次登入改密碼（使用者認為在意的人會自己改）。寄信前照慣例先列出收件人與內容確認。
- 2026-10-21 時 gups 上沒有任何一位學生的帳號（現有帳號都是前屆使用者），`PDK_approved` 只有老師帳號；建帳號後才能讀 `<remote-home>/PDK`、`/cad/cadence/{DDI,XCELIUM,VERISIUM}` 與 `/cad/tutorial`。
- 師大學生帳號開頭是數字，Rocky 8 `useradd` 可能拒絕，屆時改加前綴。

- 2026-10-22 使用者偏好：gups 上的教材（`/cad/tutorial/innovus_u18_tutorial*`）直接修改即可，不要另外做備份。

相關：[[eda-environment]]
