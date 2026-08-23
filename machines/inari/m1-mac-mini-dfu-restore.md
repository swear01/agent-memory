---
title: M1 Mac mini DFU restore and macOS 26 reinstall
scope: machines/inari
machine: inari
status: active
confidence: high
created: 2026-08-23
updated: 2026-08-23
tags:
  - macos
  - apple-silicon
  - m1
  - dfu
  - apple-configurator
  - restore
  - nfs
  - autofs
source_refs:
  - Apple Support 108900
  - Apple Support 120694
---

# 結果

- 目標機是 M1 Mac mini；主機是 M5 MacBook Air，當時為 macOS `26.5.2` 與 Apple Configurator `2.20`。
- 最終透過真正的 USB DFU 完成抹除與重裝。使用的 restore image 是 `UniversalMac_26.6.2_25G83_Restore.ipsw`；目標機成功重新啟動到 macOS 26 設定流程，主機與本機帳號命名為 `inari`。
- 維護紀錄只保存帳號名稱，不保存或重述任何密碼。

# Gaia NFS 整體 `<remote-home>` 掛載

- Inari 使用 macOS 內建 autofs 的 `-static` map，把整個 `192.168.1.200:/volume1/nfs-home` 掛到 `<remote-home>`；不要使用 `auto_home` wildcard 只掛 `<home>`。
- `/etc/auto_master` 保留預設的 `/- -static`，停用原本的 `<remote-home> auto_home` 項目；`/etc/fstab` 使用：

  ```fstab
  192.168.1.200:/volume1/nfs-home <remote-home> nfs rw,hard,intr,resvport,vers=4 0 0
  ```

- 套用前必須先正常卸載 `/System/Volumes/Data/<remote-home>/<entry>` 的子 NFS 掛載與 `/System/Volumes/Data/<remote-home>` 的舊 `auto_home` autofs 掛載，再執行 `sudo automount -vc`。Finder 若正開著 `<remote-home>`，要先關閉或重新啟動 Finder，否則子掛載會保持忙碌。
- `mount` 輸出的來源可能含空格，例如 `map auto_home on ...`，不可假設掛載點永遠是固定欄位；應找出文字 `on` 後面的欄位。
- 2026-08-23 即時驗證結果：

  ```text
  map -static on /System/Volumes/Data/<remote-home> (autofs, automounted, nobrowse)
  192.168.1.200:/volume1/nfs-home on /System/Volumes/Data/<remote-home> (nfs, nodev, nosuid, automounted, nobrowse)
  ```

  本機帳號 `inari` 可在 Finder 以 `Command-Shift-G` 開啟 `<remote-home>`，列出 Gaia NFS 根目錄並讀取 `<home>`。寫入權限仍由 NFS 上的 UID/GID 與 mode 決定。

# 根因與判斷證據

- M1 Mac mini 螢幕上的啟動選項、`Options`、macOS Recovery 與「啟用 Mac」都是目標機的本機 recoveryOS；它們不等於另一台 Mac 可用來執行外部 firmware restore 的 USB DFU。
- 失敗時主機曾看到 Apple USB `05ac:1901`、產品名 `Mac mini` 與 NCM transport。Configurator log 同時出現 `Unexpected backing device for booted OS proxy`，並找不到 `com.apple.RestoreRemoteServices.restoreserviced`。這是普通／recovery transport，不是 DFU。
- 真正成功時主機的 `ioreg` 明確顯示 `Apple Mobile Device (DFU Mode)`、Apple USB `05ac:1227`（十進位 `idProduct = 4647`）、`CPID:8103`。這才是可開始 Restore 的 gate；目標機畫面保持黑色。
- 主機跳出「允許配件連接」只證明 USB data 裝置曾被偵測，不足以證明已進 DFU。Finder 或 Configurator 必須實際顯示 `Mac DFU Mode`，或由 USB `05ac:1227` 證明。

# 可重複流程

1. 主機 Mac 開機、解鎖、接電與連網。USB-C 線直接連接兩台 Mac，不經 hub/dock；線材必須支援 data 與 charging，依 Apple 文件不要使用 Thunderbolt 3 cable。
2. 面對 M1 Mac mini 機背時，目標端使用最左邊的 USB-C 埠，也就是最靠近 Ethernet 網路孔的 USB-C 埠。主機端可用任一 USB-C 埠。
3. 拔除 M1 Mac mini 其他 USB 裝置。從 Recovery／啟用畫面正常關機，確認畫面全黑後拔掉 AC 電源。
4. 先按住 M1 Mac mini 背面的 Power，再在 Power 保持按住時插回 AC；最多持續約 10 秒，但看到主機出現 DFU 或配件允許提示就立即放開，不必硬按滿 10 秒。
5. 若出現配件允許提示，放開 Power 並在主機按「允許」。不要在目標機按 Option，也不要把本機 Recovery 畫面當成成功。
6. 在主機用下列唯讀檢查確認真正 DFU：

   ```sh
   ioreg -p IOUSB -l -w0 | rg 'Apple Mobile Device \(DFU Mode\)|"idProduct" = 4647'
   ```

   `0x1227`／`4647` 是成功；`0x1901`／`6401` 不是。

7. Apple Configurator 出現 DFU 方塊後選 `Actions > Restore`。本次 Configurator 已在主機快取約 18 GB 的 `UniversalMac_26.6.2_25G83_Restore.ipsw`，位置在：

   ```text
   ~/Library/Group Containers/K36BKF7T3D.group.com.apple.configurator/Library/Caches/Firmware/
   ```

8. Restore 完成後目標機自動重開，完成「啟用 Mac」與 macOS Setup Assistant。

# 容易誤判的地方

- Configurator 顯示 `Mac mini` 型號、開始下載 IPSW、或主機發出 USB 提示音，都不能取代 DFU gate；本次就是下載完成後才因模式錯誤遭拒。
- M1 Mac mini 前燈白色只表示已開機或睡眠。Firmware recovery 時可能是快速閃爍或常亮琥珀色，但仍以主機看到 `05ac:1227` 為最終判定。
- Brave、Codex 或一般瀏覽器不是這次的阻擋者；關閉它們沒有改變 USB mode。真正問題是目標機先前只進到 `05ac:1901`，尚未進入 BootROM DFU。
- 若畫面已出現本機 Recovery，又希望使用主機已下載的 IPSW，不要在目標機選「重新安裝 macOS」重新下載；先關機，再進真正 DFU，從 Configurator Restore。
