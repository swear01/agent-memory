---
title: Athena 送修歸還與回機後狀態
scope: machines/athena
status: active
updated: 2026-11-24
---

# Athena 回機後的 Runner 本機狀態復原

- 2026-11-12 歸還後恢復一般 fleet 作業範圍。這不等於硬體故障已修復；部分核心負載仍可 freeze，操作與限制見 [Athena freeze](recurring-freeze-2026-11.md)。HAPI 共用家目錄的檔案從其他機器可見，不代表 Athena runner 本機正常。
- 若返修或清理後 `hapi-runner.service` 出現 `Failed to load environment files` restart loop，先檢查 Athena 自己的 `/var/tmp/hapi-runner/runner.env`、`/var/tmp/hapi-athena/settings.json` 與本機 Agent state。NFS home 的 unit/binary 可全數完好，而 `/var/tmp` 已被清掉。
- 從已確認的 fleet source／備份重建 `runner.env`，確保 `HAPI_HOME=/var/tmp/hapi-athena` 與該機原 machine ID，避免複製別台身分或讓 Hub 新增重複 machine。只在正確的 local state target 從既有 `.nfs-seed` 種回。重啟前查現役 session；從 Athena 直接驗 systemd active、`KillMode=process`、MainPID 等於 runner state PID、Hub 仍是原記錄與 agent canary。管理操作用 `su_athena`，實際 live 設定與身分查 HAPI skill 及 Hub。
- 登入延遲看 [earlyoom 與 systemd-oomd](../hapi-fleet/earlyoom-nis-dynamicuser.md)，NSS/PAM 以 [後續 NIS 復原](../hapi-fleet/ubuntu-upgrade-recovery-runbook.md) 為準。不要從回機時的套件、NVIDIA 或 NSS 快照推目前版本。

[Athena 送修／回機原始檢查和修復收據](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/machines/athena/repair-status.md)
