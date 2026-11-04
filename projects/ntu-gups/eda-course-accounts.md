---
title: GUPS3006 EDA course student accounts on gups (FreeIPA, created 2026-11-04)
scope: projects/ntu-gups
status: done
updated: 2026-11-04
---

# GUPS3006 學生 gups 帳號

## 2026-11-04 已建立並寄出

- 8 位修課學生（名單在 Drive「GUPS助教/gups_帳號待建立名單.md」，不放本庫）已建 FreeIPA 帳號，都在 `pdk_approved`（本機 GID 1024 `PDK_approved`），家目錄建在 ieda1（0700）。師大學生數字開頭帳號 `41375054h` IPA 直接接受，不用加前綴。
- 驗證：`id` 群組正確；`su - 41375054h` 讀得到 `/cad/tutorial/innovus_u18_tutorial`、PDK 目錄、`/cad/cadence/CIC/innovus.sh`，`cp -r` 教材成功。
- 信從 NTU 信箱（himalaya `-a ntu`，`--no-save` 所以寄件備份沒有）寄出，密碼不印不存；要重寄用 `gups_accounts.sh --reset`。

## 流程（腳本在 Drive GUPS助教，`gups_accounts.sh`）

- stdin 每行 `帳號 Email 姓名`（姓名可含空白）；腳本經 `ssh gups` 呼叫 ieda1 `/root/ipa_add_user.sh`（`ipa user-add` + 加 `pdk_approved` + 設密碼並延 90 天到期、不強制首次改密碼 + 建家目錄）。`--preview` 只印第一封信。
- 產生輸入：`grep -E '^\| [0-9a-z]+ \|' gups_帳號待建立名單.md | awk -F' *[|] *' '{split($4,n,"（"); split(n[1],m,"；"); print $2, $3, m[1]}'`。名單備註欄格式「寄信用名（中文名）」。
- 使用者定稿的信（英文）：主旨 `GUPS workstation account`；內文 `Dear <name>,` / `This is your GUPS workstation account for the EDA Final Project.` / Account 行 / Password 行 / `Login: ssh <帳號>@intl.gups.ntu.edu.tw`。
  - 只給 ieda1：ieda2（140.112.186.23）還沒設定好，不要寫「任選一台」。
  - 用網域不用 IP：`intl.gups.ntu.edu.tw` 公網 DNS 解析到 140.112.186.18（ieda1，PTR 相同）；`ieda1.gups.ntu.edu.tw` 公網查不到。
  - 本名是英文的國際生用英文名（來源：同資料夾 TSRI 使用者名單 xls 的 Name 欄），其他寫中文名。

相關：[[eda-environment]]、[[final-project-package]]
