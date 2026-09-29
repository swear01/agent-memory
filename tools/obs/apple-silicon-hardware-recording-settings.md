---
title: "Apple Silicon (M1/M5) OBS 螢幕錄影最佳化與硬體編碼避坑設定"
scope: tools/obs
status: active
updated: 2026-09-29
---

# Apple Silicon (M1/M5) OBS 螢幕錄影最佳化與硬體編碼避坑設定

## 核心問題與掉幀根因

在 Apple Silicon（包括無風扇的 MacBook Air 與單媒體引擎的基礎款 M1 Mac）上，使用 OBS Studio 進行長時間螢幕錄影時若發生嚴重卡頓或大量掉格（例如 OBS 日誌顯示 `Video stopped, number of skipped frames due to encoding lag: 43.9%` 與 `rendering lag: 9.8%`），其核心根因如下：

1. **色彩格式誤選 P010 (HDR 10-bit)**：
   OBS 在 macOS Metal 管線下若未提供原生 P010 紋理支援，會記錄 `P010 texture support not available`，迫使 CPU/GPU 每一幀做軟體格式轉換，且 VideoToolbox 被迫套用 `main42210`（10-bit 4:2:2）Profile，無法走 Apple Silicon 原生零拷貝（Zero-Copy）硬體編碼通道。
2. **4K (3840×2160) 與 60 FPS 負載超載**：
   基礎款 M1 僅具備單組視訊編碼引擎（AVE），無風扇 MacBook Air（如 M5）則依賴機身被動散熱。高壓 4K 60fps 會在 10～15 分鐘後引發機身過熱降頻（Thermal Throttling），導致編碼速度跌破實時吞吐量而瘋狂掉格。
3. **螢幕比例變形**：
   MacBook Air 視網膜渲染緩衝多為 2940×1912 或面板原生 2560×1664（約 16:10.4），強制縮放填滿 4K 16:9 會造成非等比拉伸。

---

## 驗證通過之最佳配置（M1～M5 通用）

為最大化利用 Mac 專屬硬體編碼器、降低檔案體積並徹底杜絕掉幀，經實機吞吐量測試驗證之標準參數：

### 1. 輸出設定 (Settings $\rightarrow$ Output $\rightarrow$ Recording)
- **輸出模式**：`進階 (Advanced)`
- **錄影格式**：`Matroska 影片 (.mkv)`
  - 避免 MP4 在錄影中途當機、斷電時整個檔案損毀；單檔錄製時關閉「自動分割檔案」。
- **視訊編碼器**：`Apple VT HEVC 硬體編碼 (Apple VT HEVC Hardware Encoder)`
  - HEVC (H.265) 相較 H.264 在同畫質下節省 30%～50% 容量，投影片與代碼文字邊緣更清晰。
- **音訊編碼器**：`CoreAudio AAC`（160 kbps）
- **碼率控制**：`CRF`（品質因子設為 `55` ～ `60`）
  - 靜態投影片碼率自動降至數百 Kbps，換頁動態自動調高；若特定版本 OBS 之 CRF 異常，改用 `CBR 3500～5000 Kbps`。
- **主要畫面格間隔 (Keyframe Interval)**：`2 秒`

### 2. 視訊設定 (Settings $\rightarrow$ Video)
- **基礎畫布解析度**：`1920×1080`（或螢幕原生解析度）
- **輸出縮放解析度**：`1920×1080` (1080p Full HD)；**錄 Google Meet／需貼合 16:10 螢幕時改用 `1920×1200`**（見下方 M1 Meet 段）。
  - 1080p／1200p 像素量遠低於 4K，字體點對點清晰，且避免 M1 單編碼引擎超載。
- **常用 FPS**：`30 FPS`
  - 課程、會議、桌面錄影勿用 60 FPS，負載與發熱直接減半，徹底消除被動散熱降頻風險。

### 3. 進階設定 (Settings $\rightarrow$ Advanced)
- **視訊 $\rightarrow$ 色彩格式**：`NV12`
  - **最關鍵項**：Apple Silicon AVE 硬體專用直通格式，CPU 佔用率低於 1%，零掉幀。
- **色彩空間**：`Rec. 709`（SDR 標準空間）
- **色彩範圍**：`限制 (Limited / Partial)`
- **錄影 $\rightarrow$ 自動重混為 MP4**：`勾選`
  - 錄製時以容錯性最高的 MKV 安全寫入，按下停止錄影時 OBS 自動於 1 秒內無損封裝為 MP4。

---

## 既有分段影片無損合併工作流程

若因先前設定產生了多個相同編碼參數（HEVC/NV12/AAC）的分段 `.mkv` 檔案，可直接使用 FFmpeg 串流複製無損拼接，不失真且耗時僅數秒：

```bash
cat << 'CONCAT_EOF' > /tmp/concat_list.txt
file 'segment1.mkv'
file 'segment2.mkv'
CONCAT_EOF

ffmpeg -f concat -safe 0 -i /tmp/concat_list.txt -c copy "merged_output.mkv"
rm -f /tmp/concat_list.txt
```


---

## M1 雙螢幕 Google Meet 錄影（Swair-M1）

針對 `Swair-M1.local`（基礎款 Apple M1、OBS Studio 32.2.2）錄製 Brave 裡的 Google Meet 時，在通用設定之外再套用以下規則：

### 輸出比例用 16:10
- **畫布與輸出**：`1920×1200`（不是通用段落的 1920×1080）。
- 原因：M1 MacBook 原生約 16:10；Meet 視窗／外接螢幕內容用 16:10 可避免黑邊過多或非等比拉伸。

### 擷取來源：Brave 視窗，不要用舊版螢幕擷取
- 刪除 legacy `display_capture`（介面名稱常為「显示器采集」）。OBS 32 + 新版 macOS 會在啟動時 `Failed to create source`，且舊的 display UUID 常對不上目前連接的螢幕。
- 主要來源用 ScreenCaptureKit **視窗擷取**（OBS id `screen_capture`，`type=1`），應用程式 `com.brave.Browser`，並開啟來源音訊以錄到 Meet 對方聲音。
- 優點：Meet 視窗拖到哪個螢幕都跟得上，不必等外接螢幕接好再綁定。
- 缺點（OBS 32）：視窗擷取**只記住 window ID**，沒有依標題／應用自動重綁。Brave 重開或 Meet 換視窗後，必須在來源屬性重新選一次視窗。
- 可加一個隱藏的 Brave **應用程式擷取**（`type=2`）當備用；它不需重選視窗，但只顯示綁定那塊螢幕上的 Brave 視窗，且音訊應靜音以免與視窗擷取雙軌重疊。

### 音訊
- 麥克風／Aux **維持靜音**時，只錄 Meet 裡的聲音（對方與會議播放），不錄本機麥克風。
- 需要自己的聲音時再打開麥克風軌。

### 操作注意
- 快捷鍵：`U` 開始錄影、`I` 停止錄影。誤按 `U` 會再開一段新錄影。
- 設定檔位於 OBS Application Support 下的 `basic/profiles/<profile>/basic.ini` 與 `recordEncoder.json`；場景在 `basic/scenes/<collection>.json`。OBS 開啟時會覆寫設定檔，**必須先完全退出 OBS 再改檔**。
- 遠端用腳本送快捷鍵需要「輔助使用」權限；沒有權限時可用結束 OBS 程序來停錄（MKV 通常可收尾，但可能沒有自動 remux 成 MP4）。
- 驗證順序：預覽有畫面 → 會議有聲音 → 短錄 → 停止後檢查 remux 出的 MP4 解析度為 1920×1200、非全黑、音訊峰值不是靜音。

### 已驗證狀態（2026-09-29）
- Profile「未命名」、scene collection「未命名」：1920×1200、30 FPS、NV12、Rec.709 Partial、Apple VT HEVC Hardware、CRF 60、keyint 2s、MKV + AutoRemux、不分割。
- 來源：可見的「Brave 窗口采集」（視窗擷取）、隱藏的「Brave 应用备用」（應用擷取、靜音）、麥克風靜音。
- Screen Recording／麥克風／相機權限已授予 OBS。
