---
title: "Codex 語音 WebRTC 斷線與更換網路後改善的驗證界線"
scope: tools/codex
status: observed-improvement-root-cause-unconfirmed
updated: 2026-10-15
---

# 已確認的桌面語音斷線

2026-10-15 檢查 macOS ChatGPT/Codex 桌面 App 的保留紀錄，找到八次成功建立通話後的 `Realtime voice connection failed`，來源均為 `source=webrtc`。建立通話到失敗約為 25.5、30.8、44.1、418.4、16.5、57.4、76.0、36.1 秒，沒有固定一分鐘截止的規律。

失敗通話屬於舊程序 build 20156。App 在台北時間 03:21 重啟後，程序紀錄為 build 20398；安裝版本為 26.1007.71326，當次 updater 檢查顯示 `up_to_date`。未驗證新版是否修復語音，不要把目前安裝版本套用到重啟前的失敗紀錄。

最近三次失敗附近的五分鐘 Wi-Fi 紀錄共有 62 筆品質樣本，RSSI 約 -48 至 -42 dBm，樣本中的 `txFail` 均為零，未找到實際 Wi-Fi 離線事件。當次系統 HTTP/HTTPS/SOCKS 代理未啟用，列出的 VPN 未連線。這些只描述本機無線鏈路與設定，不能排除路由器、外網、封包路徑或服務端故障。

# 使用者回報的改善

同日使用者表示：換到實驗室網路後，語音「好上不少」。把這記為使用者觀察到的改善；原本網路到語音服務的路徑值得優先比較，但根因尚未確定。

沒有受控的相同版本、相同裝置跨網路比較，也沒有量測改善後的連續通話時長。這段期間 App 也曾重啟並切換 build，不能把改善完全歸因於網路，或宣稱已永久修復。

# 手機遠端 Codex 的範圍

使用者詢問的是手機透過 Codex/Remote 連回電腦，討論電腦上的同一個工作；不是另外開一般 ChatGPT 語音聊天。

官方 Remote 說明支持繼續既有工作，使用連線電腦上的檔案、工具與權限，指令仍在主機執行。主機必須保持清醒、在線且桌面 App 開啟。

換手機可能改善手機端與桌面端語音客戶端差異造成的問題，但不會排除主機、遠端 relay、共用網路或服務端的問題。本次沒有實測手機遠端語音，不能記成有效解法。公開回報 `openai/codex#45541` 也描述 Remote Voice 中途斷線；屬於其他使用者案例，不是本次根因證明。

# 後續重查的重點

- 區分一般 ChatGPT Voice、Codex 桌面 Voice、手機 Codex Remote Voice 與 Dot 通話，不把相近錯誤視為同一故障。
- 把每次失敗與同一通話的建立時間配對，保留失敗程序的 build；`source=webrtc` 能定位通話傳輸層，不能單獨判斷本機或遠端責任。
- 強 Wi-Fi 訊號不等於對外即時音訊連線穩定；換網路改善是線索，仍須同條件比較與持續通話驗證。
- 官方參考入口：`learn.chatgpt.com/docs/remote-connections`、`learn.chatgpt.com/docs/features/voice`、`help.openai.com/en/articles/9247338-network-recommendations-for-chatgpt-errors-on-web-and-apps`。版本、公告與功能可用性必須重新查驗。
