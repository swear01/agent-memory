---
title: Idle Spiral 3.0.82 的外部倍速被模擬預算轉入離線記憶
scope: tools/openspeedy
status: active
updated: 2026-10-24
---

Idle Spiral 3.0.82 build 117，Unity 6000.5.7f1／Mono，搭配 OpenSpeedy 3.3.8。唯讀檢查加速 DLL、每程序啟用旗標與 shared 段倍率可證明注入設定，不能單獨證明玩法達完整倍速。

本機反編譯確認 `GameSystem.Update()` 以 `Time.realtimeSinceStartupAsDouble` 差值餵入 `FixedStepSimulationClock`。固定步長 0.02 秒，每個 rendered frame 最多 4 步；多餘完整步長由 `DrainRenderedFrame()` 轉為 offline seconds，`OfflineBoost.ApplyOfflineGap()` 每秒增加 10 單位記憶，不立即模擬戰鬥或生產。單次大於 2 秒的 elapsed gap 亦直接走此路徑。

外部獨立時鐘與自動存檔交叉量測：35.0394 真實秒，存檔程序時間戳增加 700.9469 秒（20.0045×），`dto.totalTime` 增加 195 秒（5.5652×），`offlineBoostAmount` 增加 5061.1495。`195 + 5061.1495 / 10` 與程序時鐘差約 0.168 秒，支持超額時間被轉存。第二段約 20.24 秒亦量得程序時鐘約 19.87×、遊戲時間約 4.55×。這些是場景與負載相關量測，不能當固定上限。內建 Offline Boost 在兩段皆關閉。

主資源、Battle Spiral、Equation、Reactor、Theory 等由 `IdleSpiralSystem.Advance(gameDelta, baseDelta)` 分派；戰鬥道具持續時間與部分 logger 使用 baseDelta。Equation 只有 `IsUnlocked && IsSelected` 才增加研究；沒有選取不等於加速失敗。自動購買／轉生每程序時鐘秒排程，並受每幀檢查限制。Daily Spiral 用 DateTime.Now 判斷日期，不能和主模擬時間混為一談。

內建倍率為 1 時，每個 rendered frame 最多直接模擬 0.08 遊戲秒；完整外部 20× 在正常路徑需約 250 個實際 rendered frames／真實秒。這是推算，非獨立 FPS 量測，不能把保存的 targetFrameRate 或被掛鉤時鐘顯示的 FPS 當真實吞吐量。

唯讀檢測可解析 `<persistentDataPath>/IdleSpiralSaveV2/shared-v1/primary.idlespiral-save`：外層 JSON 的 entries 各項以 key=dto/save/saveR 儲存內層 JSON 字串。用外部 Stopwatch、連續自動存檔及記憶量增量驗證，不需 UI。注意自動存檔延遲、totalTime 整秒量化與存檔重載，不能對沒有更新的檔案推斷零速度。

OpenSpeedy 3.3.8 的 `src-bridge/src/main.rs` 中 STATUS 直接回 OK，ISENABLED 也有 always-true 路徑；不能以該版 bridge 狀態回應代替唯讀啟用映射與進度量測。GETSPEED 可讀倍率，shared 段也可在確認 PE 段表後以 PROCESS_VM_READ 讀 double。勿把新版 bridge 行為外推到 3.3.8。

僅確認內部 C# 方法（Advance、TimeWarp、AdmittedSimulationDelta 等），沒有因此取得外部公開控制 API。TimeWarp 不涵蓋完整戰鬥等所有系統，不能當等價快轉。這次未做調倍率 A/B、FPS 優化或修改遊戲；改善方案仍須另測。

## 後續補丁與量測

歷史 20× 補丁把每幀模擬預算從 4 步按有效倍率放大，並限制單幀追算時間與積壓，以免高倍率讓主執行緒餓死。Harmony 直接修改 `GameSystem.Update` 呼叫以避開小方法 JIT inline，核對核心 DLL 雜湊與預期 IL。載入新版本前先卸除舊 Harmony owner，使用不同 assembly identity，避免 Mono 重用舊版。20× 實測遊戲約 19.9–20.0× 且沒有增加離線記憶，Steam 重開後自動載入也曾驗證。這是當時負載，不是現在所有倍率的保證。

普通 Stopwatch 會受 OpenSpeedy QPC hook 影響。`QueryUnbiasedInterruptTimePrecise` 在該 Windows build 實際出現在 KernelBase.dll，經 Kernel32.dll 呼叫曾回 `EntryPointNotFoundException`。新版方案以原生時鐘隔離與宏步取代舊的 8 ms／128 步守衛，見 [100–1000× 時鐘隔離](idle-spiral-1000x-clock-isolation.md)。

## 各系統後續檢查：BigDTO 與日曆時鐘

讀取此版本存檔時，不能僅看 dto.r、dto.theoryDatas 等舊欄位：即使 IsBigDTOApplied=false，執行中的 Theory/Reactor 等原始碼直接使用 dto.bigDTO；實測 Theory 0 的 bigDTO.canProgress=true、exp +1639.899965 對應 +1640 遊戲秒，舊 dto.theoryDatas 卻全為 canProgress=false。從舊欄位推論「全部未研究」會錯誤。Equation 的 selected flags 與 Theory 研究不是同一系統。

## 主螺旋顯示與 Theory 計算

遊戲已有 `CreateSpirals.SetRenderingSuppressed(object owner, bool)`，以 owner 集合管理主螺旋渲染抑制。用單獨 owner、在 GameSystem.Update postfix 的 Unity 主執行緒呼叫，移除時只釋放自己的 owner。保留 LineDraw 的角度與歷史更新，不能為了關 3D 而停用整個 LineDraw／Spiral：A_TotalTheta.Start 訂閱 Spiral.ThetaChanged，NotifyAchievements 依角度更新成就。此接口只隱藏主螺旋，不能宣稱所有戰鬥／方程式預覽都被關閉。當次場景沒有驗證到 FPS 收益。Install 必須先設定 requested 再安裝 postfix，Remove 一律排到主執行緒釋放並 unpatch，避免安裝／移除競態。

TheoryContainer.UpdateTheories 內原版逐項更新 EXP，但同次更新的兩個共用進度參數不變。只改寫 TheoryContainer 外層編譯器產生的 `<ApplyProduceLogic>b__19_1`／`b__19_2`，以 ThreadStatic cache 在該次 UpdateTheories 內惰性計算 study_progress／study_progress_add；finalizer 清理，巢狀呼叫回原 getter，下一模擬步重新求值。不可全域快取 Parameter.Number，生產參數包含週期及即時狀態。

隔離測試 10,000 步／100,000 筆 EXP 的 mantissa bits 與 exponent 均與原版一致，涵蓋變動共用參數、等級、材料、停用第一項研究、混合步長與例外清理。當次 live 抽樣 Theory 每步約 56.60→10.98 µs，整步約 212.37→173.17 µs；這是有觀測成本的場景結果，不保證全場同等提升。以每 127 步取樣避免與 CurrencyUpdateStepCount=4 同相；128 步取樣會產生嚴重偏差。巢狀階段耗時為 inclusive，禁止直接相加。

純 Formula cache 隔離測試成功，但 live cache-eligible hit／miss 都是零，沒有可驗證收益，已移除且不納入 payload／autoload。便宜分支繞過計數，不能說「函式呼叫為零」。保留實驗失敗證據，不應為有寫程式就上線。

## 日曆、儲存與 Steam 的時間來源

主模擬、日曆、Steam 認證與雲端儲存須各自核對。遊戲時鐘超前可能使 `dailyLastResetDate` 落在未來，Steam／Google 認證仍應用真實時間。存檔執行中即使舊 dto 欄位停滯，也要核對實際 bigDTO 與事件時間；連續自動存檔才適合量速度。若觀測原生長停頓，要把 managed Update 耗時和 Steam 原生等待分開。ProcDump 的 `target no longer running` 曾在 PID 仍存活時出現，應另查程序狀態；dump 可能含登入資料，僅留本機。

完整 A/B、各版原始碼與掛鉤清理、成就、Google 認證與 Steam 重開證據見 `git show 54da0f0adfdb3dacb3bca59a729b12975d44ec9b:tools/openspeedy/idle-spiral-simulation-budget.md`。可重建版本見 [補丁保存來源](game-patch-cloud-backup.md)。
