---
title: DVLab SpyGlass Linux 7、NSS 與 snpsmem 相容性
scope: machines/hapi-fleet
project: dvlab-mis
status: active
confidence: high
created: 2026-09-05
updated: 2026-10-12
tags:
  - synopsys
  - spyglass
  - ubuntu
  - linux
  - eda
  - nss
  - snpsmem
---

## 2026-10-12 現況

SpyGlass Y-2026.03 在 Mazu、Cthulhu、Valkyrie、Zeus 已保留原廠 `OPTIMIZE_PERF = snpsmem`，修正主機 NSS 查詢順序後通過原生啟動、GUI、RTL 編譯與 lint。這次沒有把 2026.03 改成 `runtime`，也沒有修改共享 vendor binary、授權、compiler 或重啟 HAPI。

| 主機 | passwd／group 的來源順序 | 狀態 |
|---|---|---|
| Mazu、Cthulhu、Valkyrie | `files nis sss systemd` | 已部署並驗證 |
| Zeus | `files nis systemd` | 已部署並驗證，保留 NIS-only |
| Athena | 尚未核對回機後設定 | 送修紀錄仍有效，live HAPI 離線；待回機同步，不能宣稱五台完成 |

Oracle 的帳號來源只有 `files`，且沒有這套 SpyGlass 安裝；Mac／Windows 不適用 Linux NSS 修正。同步的是讓 `systemd` 排在原有 NIS／SSSD 後方的規則，不能把有 `sss` 的完整檔案盲目複製到 Zeus。

## 根因與獨立重現

Mazu 使用 Ubuntu 26.04、kernel 7、glibc 2.43、libnss-systemd 259.5。GDB 在實際 `check.Linux4` 中抓到：

```text
SML::SMLUtil::sml_get_user_name()
  -> __getpwuid_r()
  -> _nss_systemd_getpwuid_r()
  -> glibc __malloc_usable_size() / musable()
  -> SIGSEGV（SpyGlass exit 255）
```

`snpsmem` 的 `libreplacemalloc.so` 替換 `malloc/free/calloc/realloc`，但不提供相符的 `malloc_usable_size`。NSS 的 systemd 模組呼叫該函式時，落到 glibc 實作並解讀非 glibc heap 的記憶體，因此崩潰。這是 allocator 介面缺口與系統函式庫互動，不能只籠統歸因為新版 glibc 或 shell 污染。

- 獨立 C 程式加上這個 vendor `LD_PRELOAD` 後，查詢本機 root 正常；查詢網路帳號可重現 SIGSEGV。64-byte `malloc` 搭配 glibc `malloc_usable_size` 得到 0，也不符合正確配置器應有的結果。
- 清空登入環境仍會 exit 255；`SYSTEMD_NSS_DYNAMIC_BYPASS=1`、`SYSTEMD_BYPASS_USERDB=1` 與兩者合用均沒有修好這次啟動。
- 隔離測試使用 Mazu 原本的全部 host libraries 與 glibc 2.43，只將 NSS 的 systemd 排到後方即可保留 snpsmem 跑通，之後才部署到主機。
- 現有 Rocky 8 `docker-shell` 亦保留 snpsmem 通過 GUI 與兩個 lint goals，可作為已驗證的環境替代路徑；不用降級主機 glibc。

## 部署與還原原則

- 使用既有 `su_*` SSH 管理別名與 sudo；先備份 `/etc/nsswitch.conf`，保存已移除密碼欄位的帳號／群組解析基準。
- 只原子更新 `passwd`、`group` 兩行，把 `systemd` 移到該行最後，保留 `files` 及原本 NIS／SSSD 的相對順序；`shadow`、`gshadow`、DNS 與 PAM 均未修改。
- 有啟用 nscd 的主機，在驗證前後只 invalidate passwd／group cache，避免舊快取掩蓋行為。保留備份與暫時還原計時，等帳號解析、原生工具及新的 SSH／sudo 都通過才取消計時。
- 本次備份在各主機 `/var/backups/spyglass-nss*-20261012-*/nsswitch.conf.before`。還原時先核對該主機實際備份，使用單一確定路徑，不能直接將萬用字元當作還原來源。

```bash
sudo cp -a <verified-backup>/nsswitch.conf.before /etc/nsswitch.conf
# 僅在 nscd 已啟用時刷新這兩個 cache。
sudo nscd -i passwd
sudo nscd -i group
```

## 已驗證範圍與限制

- Mazu 690、Cthulhu 698、Valkyrie 689、Zeus 676 項帳號／群組解析檢查，總計 2,753 項，修改前後一致；新的使用者 public-key SSH、管理員 SSH 與 sudo 均通過。
- 四台分別在乾淨的新 SpyGlass 2026.03 程序中讀取自己的 `/proc/<pid>/maps`，確認實際載入 `libreplacemalloc.so`，再跑 RTL `compile_design` 與 `lint/lint_rtl`；均為 0 fatal、0 errors、0 warnings。
- 另一次 `lint/lint_rtl_enhanced` 四台均為 0 errors、1 個 `CMD_overloadrule01` 規則設定 warning，指向 W339 的 overload 被忽略。不能宣稱 enhanced lint 零 warning。
- 原生 GUI 視窗均驗證成功：Mazu 使用 Xvfb，另外三台透過既有 SSH X11 forwarding 顯示到 Mazu 的 Xvfb。其他主機缺少 `xvfb-run` 時不必安裝新套件；先確認既有 X11 路徑，轉送認證失敗須重試確認，不能把顯示問題當成 allocator 問題。
- 驗證後沒有殘留測試 SpyGlass 程序或啟用中的還原 timer。
- 未測完整大型專案、password-based PAM 登入、重開機或使用者的實際遠端桌面。NSS 順序調整只避開已驗證帳號的觸發點，沒有補齊 vendor allocator API；其他仍經過 systemd 的 NSS 查詢仍可能觸發同一缺口。

## 現行使用方式

於新的 Bash shell 手動選版本，環境腳本會載入 vendor 授權設定：

```bash
source /apps/eda/synopsys/spyglass.sh 2026.03
spyglass
# 批次使用 sg_shell。
```

GUI 需要既有桌面、VNC 或 X11 forwarding。工具入口、其他版本與安裝狀態參考 [EDA 安裝盤點](eda-install-inventory.md)。

## 2025.06 的歷史修補（2026-09-05 至 09-16）

以下保留舊版決策，不能當成 2026.03 的目前 allocator 設定：

- 五台 Ubuntu 26.04／Linux 7 的 SpyGlass X-2025.06 platform selector 與 bundled Perl 原本只辨識 Linux 2–6，出現 `Unknown platform: Linux-7...`；共享 standard environment、主要與選配 SpyGlass-VCS bundled Perl 加入 Linux 7 判斷。
- 當時 snpsmem 會 SIGSEGV，`runtime`、`memory`、`tcmalloc` 的 smoke test 可運作，2025.06 部署選擇 `runtime`；這次沒有改回該版本的 allocator。
- Vendor `sg_shell` 使用 Bash 專屬 `${PIPESTATUS[0]}`，卻由 dash 執行而報 `Bad substitution`；舊版入口改由 Bash 執行。被修改的共用檔案保留於 `<shared-spyglass-root>/.dvlab-backups/20260905-spyglass-linux7/`。
- 當時五台 shell command resolution、實際 Tcl `compile_design` 與 help smoke 均通過。舊 launcher 自動 source CIC 的學生操作描述已被手動 `source /apps/eda/<vendor>/<tool>.sh <version>` 流程取代，不能恢復登入時固定載入整套環境。
