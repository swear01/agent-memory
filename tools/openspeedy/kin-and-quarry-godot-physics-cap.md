---
title: Kin and Quarry：高倍數 OpenSpeedy 會碰到 Godot 的物理步數上限
scope: tools/openspeedy
status: active
updated: 2026-11-15
---

Steam 版 Kin and Quarry（App 3505770）的遊戲 log 確認使用 Godot 4.5.1。OpenSpeedy 選對程序且 speedpatch64.dll 已載入。用同一天挖礦的 TIME REMAINING 與牆鐘比對：2× 設定在大小視窗都實測約 2.0×；10× 設定在最大化約 3.7×、縮小後約 3.5×。短樣本只代表當時的遊戲場景。

遊戲 PCK 的 project.binary 設定 application/run/max_fps=60，未覆寫預設的 60 Hz 物理頻率或每幀最多 8 步。OpenSpeedy 加快 QPC，而 Godot 用 QPC 決定經過的時間；要補算的物理步超過上限時，Godot 會截短遊戲時間。按預設步數，外部 10× 需要 75 個實際畫面每秒；60 FPS 上限最多容許 8×，實際幀率更低時上限也更低。這是由設定、原始碼與實測推定，沒有單獨量到實際幀率或物理步數。

修正方式：在遊戲安裝目錄新增 Godot 的 override.cfg，透過 [autoload] 載入相容腳本。Steam 正常 Play 就會載入，不需改啟動選項、原本 PCK、遊戲執行檔或存檔。以 Godot 的 Time.get_ticks_usec（受 OpenSpeedy QPC 掛鉤）除以 Time.get_unix_time_from_system（Windows GetSystemTime，實測不受 OpenSpeedy 3.3.8 影響），每 0.35 真實秒估計使用者倍率。調整每個遊戲秒的物理步數到 round(120/倍率)，限制在 1 至原本的 60，適當提高單幀的步數上限，仍由 OpenSpeedy 決定遊戲時計。回 1× 時恢復原本物理步數。以 PID 掛鉤需在重開後重新勾選；OpenSpeedy 的 NAME 分頁可登記自動套用。

原版遊戲和 OpenSpeedy 3.3.8 的隔離實測，在 Level 內至少確認 0.5×、1×、2.5×、10×、25×、50×、100× 的引擎時間接近設定，10× 約 10.02×、100× 約 99.93×。100× 時每個物理步長達 0.5 遊戲秒，不保證碰撞、動畫和挖礦結果與正常速度等價；別宣稱 1000× 或以遊戲引擎時間等同挖礦效率。用同一局挖礦倒數驗證實際玩法，與引擎時間分開報告。

2026-11-15 修正畫面卡頓：OpenSpeedy 3.3.8 也縮短 sleep，使用 QPC 的 Godot 軟體幀率上限跟著放大；關 VSync 時，60 的上限在 10× 不再等於 60 真實 FPS。相容腳本在關 VSync 時以 round(使用者原本 FPS 上限/倍率) 補償，也尊重無上限、VSync 與遊戲選單更改；到 1× 還原。10× 名義 60 FPS 的引擎內部設 6，目標約 60 真實 FPS。Windows 個別遊戲或驅動設定不用改。

同一隔離礦場比較，限制約 60 真實 FPS 時，240 次物理更新/秒只跑 47–52 FPS，P95 25–34 ms；改 120 次後約 60 FPS、P95 約 18 ms，遊戲倒數仍為 10×。另用原存檔副本確認 10×/120 可走完一天，5.84 真實秒內倒數從 60 到 1.3，挖了 12 塊，角色有移動。全域插值在受控測試反而使 P95 變差，因此保留原本的關閉狀態。物理步減少會改變碰撞精度；不保證高倍率與原速相同。


- [Godot 4.5.1 frame loop](https://github.com/godotengine/godot/blob/4.5.1-stable/main/main.cpp#L4345-L4378)
- [Godot physics steps and time scale](https://docs.godotengine.org/en/stable/classes/class_engine.html#class-engine-property-max-physics-steps-per-frame)
