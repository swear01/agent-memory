---
title: Google Drive 2026 年 10 月整理總結與保留邊界
scope: domain
status: verified-with-limits
updated: 2026-10-12
tags: [google-drive, cleanup, anime, mega, goodnotes, lda, sparse, motion-photo]
---

## 定位與偏好

這是 2026-10-08 至 10-12 已授權整理的結案索引，不能當作後續刪除授權或即時容量。工作區 `<workspace-root>` 為 `<user-documents>/Codex/2026-10-08/new-chat`；詳細 JSON 回條保留在該工作區 `outputs/`，不把私人逐檔清單、Drive ID 或憑證放入共用記憶。

- 保留原本簡潔的名稱及資料夾結構；壓縮後同 ID、同名、同父目錄取代，不在旁邊另放「新版」或加長括號名稱。
- 去重要兼顧程式、資料集分區及專案引用，不能只憑全域雜湊相同刪除。封存保留相對路徑，完整解壓與逐檔雜湊驗證、雲端讀回成功後才清來源。
- 移除採可恢復垃圾桶，未清空。使用者曾接受特定舊圖片包的遺失風險，不能延伸為其他檔案的一般授權。
- 本輪結案時保留背景圖、Git 歷史、Minecraft 歷史世界及缺乏精確下載證據的 JAR；這些是有意保留，不能當作待自動續刪佇列。
- **2026-10-12 最新影片標準：最高 1920×1080、不放大、保留原幀率與音訊封包／時間戳；使用 `hevc_videotoolbox` Q60、2 B-frames，停用軟體 fallback。此指示取代先前的 720p 上限。** 使用者明確要求從原始來源重做前批 68 支影片；不可放大已壓縮的 720p 版本冒充恢復細節。

## 媒體與備份

| 項目 | 已完成結果 | 驗證邊界 |
| --- | --- | --- |
| `Media/影片剪輯專案` | 61,357,996,336 → 1,898,032,239 bytes，1,831 檔、348 目錄；保留 9 份 OSP／DRP。精簡版沿用原名，舊資料夾進垃圾桶。 | 全部雲端大小／MD5 與結構已核對；OSP 改 52 條相對路徑，DRP 原件未改為相對路徑。原生編輯器未開啟，既有缺失來源仍列於回條。 |
| Anime `picture` | 46,097 張、10,578,429,907 bytes；移除 46,092 個 hash 後綴，16,971 個同名項目加編號避免覆蓋，保留檔名對照及所有圖片。 | 下載與 ZIP 內單檔 MD5、CRC 已驗證。使用者其後確認已手動上傳 MEGA，再授權將 Drive picture 與本機整理包移到各自垃圾桶；MEGA 未由 agent 獨立讀回。這取代舊「等待使用者上傳、Drive 尚未刪除」狀態。 |
| `好看的圖片.7z` | 40,078,651,355 bytes 已依明確授權進垃圾桶。 | 未找到密碼、未解密；86.17% 圖片有來源回條關聯，並非整包內容相等證明。使用者接受未對應內容的遺失風險。 |
| GoodNotes | 71 份外部備份與舊封存 410 份合併，保留 480 份主要筆記及 1 份舊版本；以 `GoodNotes 封存.7z` 取代舊 ZIP，外部來源進垃圾桶。 | 481 份筆記、483 個還原檔案及 61 目錄完整解壓／雜湊、封存 CRC 與雲端讀回通過；GoodNotes 原生匯入未測。本備份按不再持續更新的封存處理。 |
| Photos migration recovery | 舊 recovery 資料夾的 21 檔、1,734,932,542 bytes 已逐份確認進垃圾桶。 | 不重開退休的 Google Photos／Takeout 來源佇列。 |

GoodNotes 最終封存 2,181,076,835 bytes；使用 `7zz` 的 `-t7z -mx=9 -m0=lzma2:d=512m:fb=273:mf=bt4 -mmt=2 -ms=on -mqs=on -mmemuse=8g`。256／512 MiB 字典做過整包比較，512 MiB 較小；1 GiB／極高搜尋 cycles 的未完成試驗不是已驗證最優參數。筆記本身已是 ZIP，增加 RAM／字典不保證顯著縮小。

逐格動畫的大宗是 Canon CR2 RAW 原片與重複副本：634 份 CR2、317 個不同內容，約 17.64 GB；DRP 本體合計僅約 8.73 MB。不能把素材容量歸因於 Resolve 專案檔，也不能把保留 DRP 宣稱為已驗證可直接開啟／所有路徑相對化。

## 學校、Minecraft 與 CyberpunK

- [學校講義與學習紀錄整理](google-drive-school-cleanup.md)：去重、環境重建資訊、文字 7z、同 ID 的簡報／PDF／影片替換、最新 hw2 ZIP 及動態照片結果。
- [Minecraft 最終交付與保留範圍](../../projects/minecraft-prism-recovery/final-delivery.md)：以精確版本、下載位元組雜湊重建 JAR，保留世界與設定；2026-10-11 再清完已刪祖先下的 9,654 個檔案級非垃圾桶殘留。
- [CyberpunK 歷史與十月續作](../../projects/cyberpunk-drive-cleanup/status.md)：影片／簡報壓縮、受保護重複群與最終 15,421,731,174 bytes 快照；不要拿九月容量當十月現況。

## LDA 模型：無損改存法並同步程式

兩個模型合計 451,755,366 → 17,426,832 bytes。使用 PyData/Sparse COO 的逐列 exact fill、值與座標一起排序、int32 索引、未壓縮 NPZ payload，再把完整 state 以 LZMA2 preset 9、lc=1、lp=3、pb=0 壓縮。LDA 浮點矩陣並非一般零值稀疏矩陣；直接用零 fill COO／各 NPZ 項目各自壓縮可能更大，應實測整個狀態與實際讀回，不做浮點量化。

原 `.pickle` 檔名保留，程式改走 `lda_storage.save_model`／`load_model`；載入時還原 NumPy 陣列。NPZ 為了壓縮而排序後，重建 COO 要用 `sorted=False`；full-row fill 及截斷／損毀 footer 有回歸檢查。所有 state、RNG、float64 位元與 8 文件推論在 scikit-learn 1.2.2／1.9.1 比對通過，雲端模型完整讀回也通過。

`comments_classifier` 的 PR #2 已合併，master 為 `7c263d091d52a1c308547d5144ae2d1fcbcd2694`，repo 已重新封存，工作樹／任務分支已清。不要再次解除封存或重開審查。原 scikit-learn 0.22.1 與完整分析 pipeline 未測；`lda_abstract.py` 原本指定缺失的 `lda_model20.pickle`，不能默換成 10-topic 模型。

## 2026-10-12 最後一批瘦身結案

以下是原批次已完成的歷史結果。使用者隨後另授權將其中 68 支影片從原始來源改按 1080p 上限重做，修復完成前不得稱為已恢復；其他已完成項目不重開。329 個上傳檔案加 1 個捷徑全部核對，選定檔案由 40,020,825,685 → 11,907,706,461 bytes；扣除 3 個引用檔增加的 12 bytes，淨縮小 **28,113,119,212 bytes（28.11 GB）**。這是本批檔案大小差，不能與歷史批次容量直接相加，也不等於清空垃圾桶後的帳戶配額釋放。

| 類別 | 結果 | 減少 bytes |
| --- | --- | ---: |
| 近代物理錄影 | 32／32 原位替換 | 24,503,395,438 |
| 音遊譜面影片 | 36／36 原位替換 | 3,077,777,572 |
| 教科書 PDF | 4 份試壓，採用 1 份；其餘 3 份縮小不足而保留 | 139,516,344 |
| PPTX | 224 個上傳項目採用 184 份，保留 40；另 1 個 Drive 捷徑保留 | 289,365,790 |
| WAV | 33 份試轉，採用 10 份無損 FLAC，保留 23 份 WAV | 103,064,080 |

- 68 支影片當時依「每一支都壓到720P」指示，使用 `hevc_videotoolbox` Q60、2 B-frames、停用軟體 fallback，最高 1280×720、不放大，保留原幀率及原 AAC 封包／時間戳。完整解碼、幀數／時間戳、音訊雜湊及品質抽樣通過。**720p 是歷史結果，已被上述最新 1080p 指示取代。** 快速 seek 的不同 B-frame 不能當畫質退化證據，改用同一解碼幀索引比對。
- PDF 採用的是 117 頁掃描解答，182,092,264 → 42,575,920 bytes；頁面、文字、內容串流、附件與渲染檢查通過。其餘 3 份試壓不足 2% 而保留；未採用候選不宣稱已完整渲染。教材與簡報採 [Q70 實測品質標準](../media/photo-compression-standard.md)，不是任意指定相同 JPEG Q 值。
- 184 份採用簡報的所有投影片均以相同 LibreOffice 環境前後渲染，最低 PSNR ≥32 dB；非圖片部件保持位元組一致。平坦 logo／線稿及透明圖優先無損或保留原圖，某張 sponsor logo 因初次渲染低於門檻而恢復原 JPEG 後通過。Drive 檔名中的斜線僅在本機輸出路徑轉義，雲端名稱未改。
- PPTX 保留 40 個項目包含：35 個未顯著縮小、3 個既有 ZIP／CRC 錯誤來源、1 個 Office owner lock、1 個相對路徑文字指標；不是 40 份都因壓縮無效。損毀來源雜湊與雲端一致，保留原件；Office 能否修復未測。
- 33 個 WAV 都完成 FLAC level 12 試轉與解碼 PCM 雜湊比對。採用的 10 份同時更新 `DampTest.py`、`RevealVideo.tsx` 及素材對照表的相關引用；5 個網頁音效另以 Chromium 實際解碼，樣本差為 0。
- 23 個 WAV 的準確狀態是「尚未確認替換後仍能正常使用，因此保留原格式」：17 個 Resolve／DRP 素材、5 個 pygame 1.9.6 音效、1 個譜面固定命名 `out.wav`。不能寫成 FLAC 已證實不相容，也不能僅因 codec 支援就改 DRP 的序列化格式資訊。
- 每份採用檔案及 3 個引用檔均完整雲端讀回比對；全部目標另有最終 metadata 核對。原始來源保留本機；沒有新增清空垃圾桶或清除本機原件的授權。

回條：`<workspace-root>/outputs/剩餘檔案壓縮完成_20261012.json`；逐檔證據在 `<workspace-root>/work/remaining-compression-20261012/`。原生 PowerPoint 動畫、Resolve 開啟／重連、上述保留 WAV 的實際消費端未測。

### 1080p 修復授權與待驗證事項

範圍限 `Share/近代物理2024` 的 32 支及 `Share/13puta` 的 36 支。其餘 Share 與近期 CVSD 錄影仍排除。先核對保留原件的雜湊與解析度，再硬體重轉；保留原 Drive ID／名稱／位置，逐檔完整解碼、幀數與時間戳、音訊封包、同一解碼幀畫質及雲端完整讀回驗證。1080p 候選若比原始檔更大，恢復原始檔而非強制降畫質。此段是已授權計畫，尚非完成證據。另一次 CyberpunK 舊影片 720p 試壓僅發生在本機，雲端原件未替換，不需從該候選放大還原。

## 最新容量與後續驗證

最新已驗證快照為 **2026-10-12 02:19:49（臺灣）**，官方 `usageInDrive - usageInDriveTrash` 為 **79,318,268,341 bytes（79.32 GB）**。這是結案回條的量測時間，不是日後查詢當下的即時容量，也不代表清空垃圾桶或總配額釋放。先前 2026-10-11 18:17 的 108,106,120,357 bytes 已被本快照取代；檔案 `size` 差與 `quotaBytesUsed` 差可能不完全一致，不能硬湊成相同精確值。

盤點須依檔案自己的 `ownedByMe`／`trashed`，不能因祖先已刪就排除其非垃圾桶子檔。全量基線加 Drive changes token 可續接盤點；批次 API 要逐份核對內層 HTTP 狀態及操作後 metadata，外層 HTTP 200 不等於全部成功。

除前述特定限制，原生 PowerPoint 動畫／PDF 內嵌影片、Resolve／OpenShot、GoodNotes 匯入、手機 HDR／動態照片與 Minecraft 遊戲內行為均不可由檔案雜湊或渲染結果推論為已實測。原照片、PDF 與模型保留本機來源作回復；再次清理前需核對最新位置與範圍。

## 證據入口

`<workspace-root>/outputs/`：`compact-editing-archive-20261008.json`、`anime-picture-migration-result-20261008.json`、`anime-picture-trash-result-20261009.json`、`anime-local-trash-result-20261009.json`、`picture-archive-trash-result-20261008.json`、`goodnotes-archive-result-20261009.json`、`photos-recovery-trash-result-20261008.json`、`pdf-compression-completed-20261010.json`、`昆蟲與簡報壓縮完成_20261011.json`、`LDA_PyData調整與替換_20261011.json`、`雲端硬碟整理結果_20261011.json`。不同批次及舊快照不可直接相加成總節省。
