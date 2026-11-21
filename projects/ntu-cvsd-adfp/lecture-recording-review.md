---
title: CVSD 上課錄影：音訊轉逐字稿，再交給 Antigravity Gemini 逐段複習
scope: projects/ntu-cvsd-adfp
status: active
updated: 2026-11-21
---

# CVSD 上課錄影複習流程

## 轉逐字稿（2026-11-18 驗證）

- 錄影在 `<GoogleDrive-root>/Media/上課錄影/CVSD/CVSD_<date>.mp4`，課程由 TA 用英文講。Google Drive MCP connector 未授權時，直接讀 DriveFS 本機同步資料夾。
- 使用者要的是**聲音轉文字**；只用每幾分鐘抽一張畫面來摘要，會被退回。
- 做法：先用 `ffmpeg` 抽出 mono 16 kHz 音訊，再用 `-f segment -segment_time 1200` 切成 20 分鐘、48 kbps 的 mp3（約 7 MB，API 上限 25 MB）。各段平行送 OpenAI `/v1/audio/transcriptions`，參數 `model=gpt-transcribe`、`response_format=text`，用 `--form-string prompt=` 帶入專有名詞。key 用 `vault run OPENAI_API_KEY=<openai item> -- curl ...` 取得。
- 成本：2.7 小時錄影約 0.73 USD，wall time 約 2 分鐘。合併後存成影片旁的 `CVSD_<date>_transcript.txt`，每 20 分鐘加一個時間標記。
- 本機沒有 whisper model。vault 另有 groq、elevenlabs key 可當備援。

## 交給 Antigravity App 複習（2026-11-21）

- `open -a Antigravity <folder>` 不會切換 workspace。要在 workspace 下拉選單選 New Project，按 Add Folder，在 macOS 開檔視窗按 cmd+shift+g 輸入路徑，再按 Next。建立專案後 Gemini 就能直接讀資料夾裡的逐字稿。
- Antigravity 視窗在另一個全螢幕 Space 時，`app_*` 背景點擊會被拒，`app_bring_to_current_space` 也會失敗；要改用 full-screen `computer_batch`。
- 給 Gemini 的指令：先列整堂課大綱，再用中文逐段講解，專有名詞保留英文；每段講完停下，等使用者說「繼續」才往下；逐字稿的聽錯字由它自行修正；不准改檔。
- 送出後若使用者已經在輸入框打字或語音輸入，就不要再操作那個 App。
- 講義 PDF 存在 `學校講義/.../電腦輔助積體電路系統設計/`（見 `lecture-storage.md`），不在錄影資料夾。要 Gemini 對照講義，需另外加進專案或複製過去。
