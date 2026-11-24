# Machines

在 `machines/<machine>/` 儲存只對單一 machine 或 environment 有效的設定、硬體、相容性與操作注意事項。

## 目前可用性

- Athena：[2026-11-12 回機補齊](athena/repair-status.md)，但 [2026-11-24 確認部分核心高頻負載仍會反覆 freeze](athena/recurring-freeze-2026-11.md)（目前在線，已啟用 iTCO 硬體 watchdog 與 P-Core 0 `CPU 0–1` 隔離／`3.0 GHz` 基頻鎖定）。
- [實驗室四台 degraded 的常駐原因與修法（logrotate、sssd 輪替、apport、gdm 殘骸）](hapi-fleet/degraded-units-cleanup-20261112.md)。
