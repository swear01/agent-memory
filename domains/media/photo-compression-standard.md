---
title: 照片與 PDF／PPTX 內嵌圖片的統一壓縮標準
scope: domain
status: user-confirmed
updated: 2026-10-12
tags: [photo, compression, HEIC, Q70, PDF, PPTX, quality]
---

# 照片與文件內嵌影像的品質校準

- 獨立照片使用已核准編碼器的 HEIC Q70（原生實測 `quality=0.7`），保留原解析度、日期、EXIF／ICC、HDR、透明度。動態照片是否保留依該案授權，普通的靜態重編不能保留 MP4／gainmap。從原始來源處理，避免對有損中間檔反覆轉碼。以實際 MIME／副檔名和輸出格式核對引用。
- PDF/PPTX 需依容器和閱讀器選擇格式。PDF 1.7 的標準 filters 不含 HEIC，不能把圖片改成 HEIC 再標 `/DCTDecode`。PowerPoint 可匯入不等於可以直接換 PPTX 封裝內部。對平坦 logo、線稿及透明圖以無損或保留原編碼，保護文字、向量、可編輯元素及活躍內容。
- 需要用 JPEG／JPEG2000 作替代時，曾以原生 HEIC Q70 reference 量化 PSNR：不比 reference 低超過 0.25 dB、最低 32 dB，禁止為此再降採樣。這是特定流程採用的 calibration，不等同跨編碼器感知品質；重新校驗實際工具和讀者後再用。教科書 PDF 的既有節省門檻是至少 2%，未達留原件。
- 全頁／投影片前後渲染，另核對實際容器與雲端完整讀回，成功後只依該批授權清來源。HDR gainmap 存在不等於 bytes 一致；手機原生 HDR、PowerPoint 動畫、PDF 內嵌播放均需個別驗證。 [PDF 1.7 規格](https://opensource.adobe.com/dc-acrobat-sdk-docs/pdfstandards/PDF32000_2008.pdf) · [Microsoft HEIF codec](https://learn.microsoft.com/en-us/windows/win32/wic/heif-codec)

[各批次 JPEG 試做、HEIC 重評與驗證收據](https://github.com/swear01/agent-memory/blob/f4e2dfbea776667f5b468d76c07353fb4e0977a4/domains/media/photo-compression-standard.md)
