---
title: DVLab SpyGlass Linux 7、NSS 與 snpsmem 相容性
scope: machines/hapi-fleet
project: dvlab-mis
status: active
confidence: high
created: 2026-09-05
updated: 2026-11-19
tags:
  - synopsys
  - spyglass
  - ubuntu
  - linux
  - eda
  - nss
  - snpsmem
---

# SpyGlass exit 255／SIGSEGV：版本、NSS 與 allocator

- 新 Bash 選 `source /apps/eda/synopsys/spyglass.sh 2026.03`；GUI 用既有桌面／VNC／X11，批次用 `sg_shell`。Y-2026.03 的 `check.Linux4` 若在 `sml_get_user_name → __getpwuid_r → _nss_systemd_getpwuid_r → malloc_usable_size` 崩潰，先查 NSS 與 `/proc/<pid>/maps`。Ubuntu 26.04／glibc 2.43 上已重現 `snpsmem` 的 `libreplacemalloc.so` 替換 malloc/free 等卻沒提供相符 `malloc_usable_size`。本機 root 能查詢而 NIS 帳號出錯符合這個觸發差異。新 shell 與 `SYSTEMD_NSS_DYNAMIC_BYPASS`／`SYSTEMD_BYPASS_USERDB` 未解決。
- 保留 2026.03 的原廠 `OPTIMIZE_PERF = snpsmem`。避免網路帳號走到 NSS systemd 路徑。Zeus 只用 `files nis systemd`。其他 DVLab client 有 SSSD，2026-11-19 SSSD-first `files sss systemd` 已通過 native lint，但四台 GPU 的密碼登入後續改為 `files sss systemd nis` 並需專用 PAM 修補，照 [目前 NIS 方案](ubuntu-upgrade-recovery-runbook.md) 核對。不要以 SpyGlass 成功就改回舊 `files nis sss systemd`，Zeus 失聯時直接 NIS 會拖慢 SSH。不要把 client 配置套到 Zeus 或 Oracle。
- 變更 NSS 前保留管理登入、備份、暫時還原和已遮蔽 hash 的基線。依版本／主機對齊正式修補，只有 passwd/group 要改時勿動 shadow、PAM 或 DNS。只在 nscd 已啟用時 invalidate passwd/group，並以新 SSH/sudo、帳號解析、實際 snpsmem load、RTL `compile_design` 與 `lint/lint_rtl` 核對。10/12 還原原檔在各機 `/var/backups/spyglass-nss*-20261012-*/nsswitch.conf.before`，選確認過的精確路徑且不要蓋掉後續 NIS/PAM 變更。
- 10/12 的原生 GUI 及 lint 為當時版本證據，enhanced lint 有 `CMD_overloadrule01` 的 W339 warning；11/19 rerun 為 batch。原生桌面、完整大型專案和後續 PAM 變化的效果要另驗。沒有完整補齊 allocator API，走到 systemd 的其他查詢仍可能觸發。Rocky 8 `docker-shell` 已驗 snpsmem GUI／lint，可作相容環境。
- 舊 SpyGlass X-2025.06 報 `Unknown platform: Linux-7...` 是 platform selector 和 bundled Perl 只辨識 Linux 2–6，曾在共享入口／相關 Perl 修補；舊版 `snpsmem` 另選 `runtime`，不要套到 2026.03。舊 `sg_shell` 的 dash `Bad substitution` 來自 Bash-only `PIPESTATUS`。原檔備份為 `<shared-spyglass-root>/.dvlab-backups/20260905-spyglass-linux7/`。工具相容性和掛載基線見 [EDA 安裝盤點](eda-install-inventory.md)。

[SpyGlass 各階段相容性重現、部署、還原與精確收據](https://github.com/swear01/agent-memory/blob/9b2d556e370f3a71736d5dc0a63f3b1e1c8c3a0e/machines/hapi-fleet/synopsys-spyglass-linux7-compatibility.md)
