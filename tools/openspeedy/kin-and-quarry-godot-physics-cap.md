---
title: Kin and Quarry 的 OpenSpeedy 倍率、QPC 與採礦驗證陷阱
scope: tools/openspeedy
status: active
updated: 2026-11-28
---

## 加速不符設定或降速後卡住

- **DLL 已正確載入，10× 卻只像 3–4×。** 用同一天的採礦倒數比未掛鉤的牆鐘。Kin and Quarry 使用 Godot 4.5.1，原設定為 60 FPS、60 Hz 物理與每幀最多 8 步。物理補算撞上上限時，引擎截短遊戲時間。按預設 10× 需要每牆鐘秒 75 個外部畫面，低幀率會進一步限速。[Godot 4.5.1 主迴圈](https://github.com/godotengine/godot/blob/4.5.1-stable/main/main.cpp#L4345-L4378)
- **關 VSync 且高倍率時，FPS 上限失去預期效果。** 舊版 OpenSpeedy 同時加速 QPC 和 sleep，名義 60 FPS 並非 60 真實 FPS。用未掛鉤時鐘量畫面與採礦，軟體幀率限制須依倍率補償，回 1× 還原使用者設定。調降物理頻率能提升倒數倍率，也會放大碰撞步距。礦工在二十多倍曾因粗步跳過 Area3D 地面接觸而跌出地圖，傷害計時器和動畫跨 idle／physics 時計也曾取消尚未命中的揮擊。先用真實礦工位置、接觸與傷害驗證，才調每幀步數。
- **從高倍率切回 10× 後畫面卡住，或倒數繼續但所有礦工停住。** 舊版切倍率的 QPC 倒退可污染 Godot 已計算的物理時間，GDScript 在事後攔異常幀無法修正。升級到已驗證的官方 3.3.20 整套程式，重新啟動遊戲並確認新 DLL 實際載入。八讀者同一壓測的 3.3.19 約 8,043 萬次有 12,434 次倒退，3.3.20 約 1.11 億次零倒退；此結論限 QPC 壓測，採礦要獨立驗證。[上游 issue #320](https://github.com/game1024/OpenSpeedy/issues/320) [3.3.20 發行](https://github.com/game1024/OpenSpeedy/releases/tag/3.3.20)
- **升級了 Start menu 的 OpenSpeedy，Quarry 還在用舊 DLL。** `OpenSpeedy (Quarry clock)` 是 %LOCALAPPDATA%/OpenSpeedy-Quarry 下的平行套件，`OpenSpeedy` 是 %ProgramFiles%/OpenSpeedy 下的 MSI 安裝。兩套共用設定、名稱登錄和 bridge pipe，只留一套執行。新 bridge 需要 `SP_GetHookFailures`，不能混用 3.3.8 私有 DLL；停止舊 bridge 並用新的遊戲程序驗證實際 DLL 路徑。WinGet 的 `Game1024.OpenSpeedy` 3.3.20 提供的是 portable zip；要原位更新 MSI，使用官方簽署 MSI，並用該 MSI 的管理抽取比對安裝檔案。歷史研究若鎖定 3.3.8 雜湊，指向保留的整套備份。

## 高速原型驗收

- **時鐘、畫面和挖礦量看似達標，卻沒有保住原版遊戲結果。** 同一 Day 9、50 單位的存檔比對真實按下開始到倒數歸零的牆鐘、原生加輕量攻擊的有效發射件數，以及歸零後已入帳的各種材料。重複配對並對齊開局地形與角色，保留一般亂數變異。較好的私有樣本在實際約 50× 的 OpenSpeedy 時鐘下，80 遊戲秒仍花 2.130／2.252 牆鐘秒（37.56×／35.52×）。08:08 研究版本即使入帳方塊更多，兩組 Hg 仍少 18.2%／16.8%。45–55× 且保住攻擊與產出的目標尚未達成，這些二進位檔是私有研究版。
- **隔離 fixture 修好了個別機制，整天仍出現少打、少領。** 保留 native 接觸、攻擊與傷害 FIFO；依原生事件核對 FIND、實際接觸、首擊及入帳。Supervisor 隔離測試保留 child ground、inbound area 和 buff，一個完整日配對的 Nb 仍少約 9%。Exploder 的 Timer 防護讓真實開局 ACT 繼續，但全日有效齊射的兩組方向相反（96→101／96→89）。只把兩者當局部診斷，整日收益與速度要單獨通過。
- **黑幕還在，容易把揭幕當作已開工。** 真實換日分別量倒數歸零、畫面揭幕與下一天開始，保留原版 autosave、地形完成與最新玩家狀態。既有交付在隔離實際加速下約 0.384–0.409 秒揭幕、0.481–0.521 秒開工；0.2 秒未達成。用途為正式採礦時，在 PowerShell 直接執行已交付的 `Auto Mining.ps1`，可留在同一視窗用一次 Ctrl+C 停止。

[完整原始紀錄、原型結果與安裝驗證](https://github.com/swear01/agent-memory/blob/8696a2520bf54c862500a0fbea6b035a4db2ffd0/tools/openspeedy/kin-and-quarry-godot-physics-cap.md)
