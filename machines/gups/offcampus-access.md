---
title: Reaching gups and the 140.112.171.* lab fleet from off campus (jump via oracle)
scope: machines/gups
machine: gups
status: active
updated: 2026-11-24
---

# 校外連 gups 與實驗室機群

2026-11-24 實測，Mac 在家用網路上：

- 直連 gups（140.112.186.18）以及 cthulhu、mazu、zeus、valkyrie（140.112.171.*）的 ssh 22 埠都 timeout；一般網路與 Tailscale 正常。
- oracle（138.2.52.9）連得到 gups:22，但連不到 cthulhu。gups 連得到 cthulhu:22。
- 可用路徑：`ssh -J oracle gups`、`ssh -J oracle,gups cthulhu`。
- 要讓寫死 `ssh gups`／`ssh cthulhu` 的腳本不改就能用，做一個 `ssh` 包裝放在 PATH 最前面：`exec /usr/bin/ssh -F <cfg> "$@"`。cfg 裡替兩台加 ProxyJump，並開 ControlMaster；最後一行 `Include ~/.ssh/config` 承接原本的 User 與金鑰。
- macOS 的 ControlPath 上限是 104 bytes，socket 放在 scratch 這類長路徑底下會報 `ControlPath too long`，改放 `~/.ssh/<短目錄>/%C`。
- 從非 login 的 ssh shell 在 gups 上啟動 Innovus，PATH 裡沒有 innovus：`xvfb-run innovus` 會回報 `innovus: command not found` 並直接結束。啟動前要 export：
  - `INNOVUS_HOME=/cad/cadence/DDI/cur/INNOVUS251`
  - `CDS_LIC_FILE=5280@lshc`、`LM_LICENSE_FILE=5280@lshc`
  - `PATH=/cad/cadence/DDI/cur/bin:/cad/cadence/DDI/cur/INNOVUS251/bin:$PATH`
  - `TMPDIR=/tmp`

  啟動約 30 秒後看一次 run.out，確認真的跑起來了。

相關：[[eda-environment]]、[[freeipa-ieda-servers]]
