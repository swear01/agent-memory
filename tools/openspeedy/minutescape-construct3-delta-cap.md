---
title: Minutescape：OpenSpeedy 的兩種影格時鐘與 Construct 3 步進修正
scope: tools/openspeedy
status: active
updated: 2026-11-18
---

Steam Minutescape（App 3327170，build 25433331，NW.js 0.100.1／Construct 3）搭配 OpenSpeedy 3.3.20。已驗證 stock package.nw SHA-256 BE9F864E19141B337B2F43CD1E4F54A7632BACDA5BB310E86850A986AAF43119。DLL 載入和 OpenSpeedy.<pid> mapping byte 1 只證明原生加速開啟；到 OpenSpeedy 安裝目錄執行 bridge64.exe GETSPEED 的 stdout 可確認共享倍率，不能代替玩法量測。CLI ISENABLED／STATUS 不是可靠的程序啟用查詢。

## 計時根因及實測

安裝版 package.nw 的 data.json project[37] 是 vsync。scripts/c3runtime.js 將 requestAnimationFrame (rAF) 的 timestamp 送給 Tick 和 _MeasureDt。非影片匯出模式把超過 0.5 秒的差值歸零，其餘夾在 0 與 1/30 秒之間，再乘遊戲的 _timeScale。Dodge 事件 #34 把 System.Exps.dt 累加到 levelTime，#82 存入 recordTime。遊戲本身的閃避慢動作及子彈碰撞需要保留，不能只把 dt 上限放大。

- renderer-only：獨立 NW.js clock probe 用外部 Stopwatch 驗證 10× 時 rAF 仍是約 1×，performance.now()／Date.now() 約 10×；實際完整隔離 Dodge 也證實 stock levelTime 約 1×。初版相容排程用 performance.now() 加短步，在這條路徑得到 9.994×。
- 六程序 NAME：正式 Steam 試用有進 Dodge 後明顯卡頓；隔離完整遊戲重現停在接近首幀。兩個時鐘每回呼的中位差值都約 156 ms，而初版相容排程錯把原始 rAF 差值大於 125 ms 當成失焦中斷，持續歸零。保持隔離 fixture 其餘五程序不變，只停用主程序，rAF 立即回到 16.7 ms，關卡恢復。主程序使用 in-process GPU；未有證據把 Minutescape 的這次卡頓歸因於 Steam 原生等待。單一隱藏 stock fixture 在每真實秒約 539 影格的另一場景也量到約 9.44×；不能把 stock 有效倍率當成固定。
- OpenSpeedy NAME 每秒重套同名程序。直接對各 PID 執行 DISABLE 後，保存的 NAME 規則會重新啟用，這是設計行為。單以 OpenSpeedy 設定倍率作為遊戲控制（不使用原生注入）雖可行，但使用者明確要求由真實 OpenSpeedy 的程序開關套用，不能拿純倍率模式交付。

## 實裝

最終直接套用補丁只修改 c3runtime.js：在 Dodge 對已變速的原生時間差做序列 Tick，每步不超過 25 ms、每幀最多 24 步且盡量在 8 ms 真實時間的工作預算內，最後只繪製一次。分辨 renderer-only 與兩個 clock 同加速；有可用的 WebAudio currentTime 時用它驗證真實間隔與預算。用原始倍率時間一次，不再乘全域值。保留原 _timeScale、原版 delta cap、關卡切換與暫停時的中斷。超出預算的時間不排進積壓。

最終 SHA-256 29E2993AD2166413A58C905961B727955CE8DDADB73A40FFDBBEF7312B572593。無 Steam 整合的 NW.js 完整隔離遊戲，在約 63–64 個真實 gameplay callback／秒時，六程序實際注入啟用、關閉、再啟用皆成功。10× 兩輪有效 levelTime 量到 7.177×／8.122×，關閉時 1.000×；未停住或出現未處理 JS 錯誤。不要宣稱正式存檔或重場景已量到完整 10×。正式 Steam 重開驗證：載入相同修改 runtime、六程序均實際注入啟用、NAME 恢復、設定仍是 10×，視窗回應。只有 package.nw 被修改，保留原版備份；未修改存檔。
