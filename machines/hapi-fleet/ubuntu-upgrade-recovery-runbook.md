---
title: Ubuntu release-upgrade recovery and authentication evidence
scope: machines/hapi-fleet
project: hapi
status: active
confidence: high
created: 2026-08-23
updated: 2026-08-23
tags:
  - ubuntu
  - release-upgrade
  - ssh
  - nis
  - nfs
  - recovery
---

# 已確認的證據界線

- 最近一次 fleet upgrade 記錄是多個獨立 worker 同時執行：四台主機做
  `22.04 → 24.04`，另一台做 `24.04 → 26.04`。因此後續同時出現的登入與
  control-plane 問題不能直接歸因於 Zeus 單一主機。
- 該記錄在 `24.04 → 26.04` 使用過 `do-release-upgrade -d`；`-d` 是 development
  release 選項，穩定版本維護窗口不得使用。
- 新的明確 TCP/22 探測顯示 Valkyrie、Athena、Cthulhu 可達；Mazu、Zeus
  探測逾時。逾時只能證明該次探測失敗，不能證明主機斷電或整機 freeze。
- 本機帳號互動式登入在 Valkyrie、Athena、Cthulhu 成功，且 `id -un` 與主機
  帳號一致；同時中央管理帳號的 SSH 公鑰流程在這三台到達 authentication
  boundary 後被拒絕。這把 host/network/SSH/local-auth 與中央 NIS/NFS/key path
  問題分開，但尚未單靠遠端證據證明具體 host-side 根因。
- Mazu 是 HAPI hub/tunnel 所在主機，Zeus 是 NIS master。兩者在同一事件窗口
  不可達時，會同時影響遠端控制與中央登入依賴；這是架構上的共同依賴，不是
  主機已關機的證明。

# 下次升級的最小流程

1. 每台主機先做 read-only preflight：OS/kernel、`dpkg --audit`、pending reboot、
   SSH enabled、network-online、`findmnt <remote-home>`、NIS/NFS readiness、GPU/DKMS、
   HAPI runner 與 workload。不要把 active SQLite/WAL 或事件輸出寫回 NFS home。
2. 對每台分別測試 TCP、SSH handshake、公鑰帳號與主機本機帳號；記錄邊界，不用
   單一 ping 或 HAPI 狀態代替全部證據。
3. Release upgrade 一次只跨一個受支援的 Ubuntu release；穩定窗口不用 `-d`。
   Package transaction 完成後仍必須 reboot，並在 reboot barrier 後重做全部檢查。
4. 若新 kernel panic、root filesystem mount 失敗或 boot loop，從 GRUB 選最後一個
   known-good kernel，先保留它，收集 failed-boot journal、initramfs、DKMS 與
   `dpkg` 證據，再修復；不要先 bulk remove kernel/driver。
5. 若本機帳號成功而中央帳號失敗，優先檢查 NIS binding/server、NFS `<remote-home>`、
   authorized-key 路徑與 SSH auth log；不要把它誤判成 GPU freeze 或斷電。

# 維運結論

升級的 package success、主機有電、TCP/22、SSH 認證、NIS/NFS、HAPI control
plane 是不同 gate。只有 post-reboot 的全部必要 gate 都通過，才把該台標成完成。
任何尚未在主機端 log 證明的根因都必須標為 hypothesis，並保留獨立 console/LAN/
BMC/PDU 的下一步驗證。
