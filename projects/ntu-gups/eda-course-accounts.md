---
title: GUPS3006 EDA course student accounts on gups (pending until tutorial release)
scope: projects/ntu-gups
status: pending
updated: 2026-10-28
---

# GUPS3006 學生 gups 帳號：教材發布時再建立

## 2026-10-28 狀態：一切就緒，老師說再等一兩週（約 11 月上中旬）再建

- 腳本 `gups_accounts.sh` 放在 Drive GUPS助教 資料夾（名單旁）。stdin 每行 `帳號 Email 姓名`；逐人 `useradd -m -G PDK_approved`、16 碼亂數密碼（去掉 0/O/1/l/I）經 stdin `chpasswd`、`himalaya message compose -a ntu ... --no-save --send` 從學校信箱寄出。密碼不印、不存。`--preview` 只印信不動作；`--reset` 對已存在帳號重設並重寄（寄信失敗時用）。數字開頭帳號被拒時自動改 `ntnu` 前綴。
- 從名單產生輸入：`grep -E '^\| [0-9a-z]+ \|' gups_帳號待建立名單.md | awk -F' *[|] *' '{split($4,n,"（"); print $2, $3, n[1]}' | ./gups_accounts.sh`
- 使用者定稿的信（不要加任何注意事項、登入說明或署名）：主旨「GUPS 帳號」，內文「給 <姓名> 同學：／您的 GUPS 帳號是 <帳號>，密碼是 <密碼>。／GUPS IP：140.112.186.18」。姓名有中文寫中文、只有英文寫英文。
- 10-28 已確認 8 個帳號（含 `ntnu41375054h`）在 gups 都不存在、`PDK_approved` 仍只有 `ICD001,joinet`；執行前再查一次。寄出前仍要給使用者最後確認。
- 執行後：`id <帳號>` 確認群組，用一個學生帳號實測讀得到 `/cad/tutorial` 與 `/cad/cadence/CIC/innovus.sh`，再把本筆改成 done。


- 2026-10-22 使用者決定：8 位修課學生的 gups 帳號**先不要建**，等 Innovus U18 教材正式發布時再一起建立。
- 名單（帳號＝NTU COOL Login ID 的 @ 前段，另含一位師大學生）與寄送 Email 存在使用者 Google Drive 的 GUPS助教資料夾 `gups_帳號待建立名單.md`；不放進本記憶庫。
- 建立方式：`useradd -m -G PDK_approved <帳號>`，亂數密碼，建立後用 Email 把密碼寄給本人；不強制第一次登入改密碼（使用者認為在意的人會自己改）。寄信前照慣例先列出收件人與內容確認。
- 2026-10-21 時 gups 上沒有任何一位學生的帳號（現有帳號都是前屆使用者），`PDK_approved` 只有老師帳號；建帳號後才能讀 `<remote-home>/PDK`、`/cad/cadence/{DDI,XCELIUM,VERISIUM}` 與 `/cad/tutorial`。
- 師大學生帳號開頭是數字，Rocky 8 `useradd` 可能拒絕，屆時改加前綴。

- 2026-10-22 使用者偏好：gups 上的教材（`/cad/tutorial/innovus_u18_tutorial*`）直接修改即可，不要另外做備份。

相關：[[eda-environment]]
