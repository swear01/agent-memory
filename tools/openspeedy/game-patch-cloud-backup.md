---
title: 遊戲補丁雲端備份：SomethingWouldDisappear / Game Patches
scope: tools/openspeedy
status: active
updated: 2026-11-26
---

## 保存位置與偏好

使用者要求簡單保存到自己的 Google Drive，並明確指定放在
`SomethingWouldDisappear` 底下。遊戲補丁後續備份應沿用此位置：

- [Game Patches 資料夾](https://drive.google.com/drive/folders/1zi-3uqNduhSH0KcX_Hiz-_T8-GP7ZjFv)
- 上層 `SomethingWouldDisappear` ID：`19-_XhXqxJtBpb7CaT92l0118bKfKkmm6`
- [2026-11-18 備份 ZIP](https://drive.google.com/file/d/1i6T5eJCEH1Pj_wljvNv_u6OwRhPZtaAE/view)
- [README 遊戲清單](https://drive.google.com/file/d/1xxt2xTBduBSmPbxrHY27stkPrHB2-Nxf/view)

這是私人保存快照，未公開發布。移動後已列出上層目錄核對位置；
備份 ZIP 與 README 仍在原資料夾，Drive 顯示 `shared=false`。
後續公開搜尋網站或 GitHub 發行版屬另一項工作，不是本次保存方式。

## 備份範圍與驗證

檔名 `Game-Patches-Backup-2026-11-18.zip`，86,087,243 bytes，
SHA-256 `d59daada3a165c809177ee3471af2d251631da2f703b33556021a0aa56e2e08d`。
ZIP 內 131 個檔案已逐一核對 `SHA256SUMS.txt`；另有來源及雜湊清單。
上傳後讀回 Drive metadata，大小一致。Connector 沒回傳雜湊，
未另做雲端下載驗證；勿把這次核對說成全部遊戲補丁重新實測。

範圍：Minutescape、Idle Spiral、Kin and Quarry、Sixty Four、
Particul、Max Manos、Unfair Flips。包含可安裝版本、原始修改工具、
實驗 diff 與必要的私人還原素材，不能整包當作公開發行版本。

## 查回時要保留的區別

- **Max Manos 確實有補丁產物**：本機工作區相對路徑
  `work/draw-test/data.win`，並有 `patch_draw.csx`、
  `draw_end_step.gml`、依賴的
  `work/draw-audit/gml_Object_objCTRLgame_Draw_75.gml`。
  補丁產物和原版還原檔均已備份。屬跳幀實驗，當時測完還原原版；
  不要再只說是設定指南。
- **Unfair Flips**：目前找回的證據是 Steam
  `-screen-quality Low` 與 NVIDIA 個別設定，未找到獨立遊戲補丁包。
  已查主、子聊天均為設定調整，不能因此斷言從未做過其他補丁。
  條件與還原方式見 `unfair-flips-unity-vsync.md`。
- **Minutescape**：三版 ZIP 均以原版 build 25433331 在獨立目錄重建，
  結果與當時現用封包及兩份舊版備份雜湊吻合。
- **Kin and Quarry**：11 個安裝中的 GDScript 與留存副本一致；
  備份另收載入設定和未提交 Godot 引擎實驗 diff。引擎實驗未驗證為穩定版。
- **Sixty Four**：備份分別保留輸出 source 與 installed 模組；
  其中三個模組不同，尚未合成單一已驗證安裝包。
- **Idle Spiral**：已有公開歸檔 Release；備份另收早期版本與舊工作樹暫存 diff。
- **Particul**：收錄歷史修改腳本與說明，未重跑完整安裝或還原。

本機原始來源可從 ZIP 內 `sources-and-sha256.json` 查回，
不依賴 Codex 工作目錄仍存在才能找出雲端備份。

## 2026-11-26 可重建的 Game Patch

使用者指定兩邊都要保存，GitHub 專案公開。輕量版本名為 Game Patch，收錄 11 款遊戲的補丁原始碼、依賴版本、安裝或重建說明、驗證紀錄。以 [swear01/game-patch](https://github.com/swear01/game-patch) 為公開版本，Drive 在 `SomethingWouldDisappear/Game Patch`。Drive 的檔名與大小已讀回核對，本機同步副本與 Git 內容逐檔核對。原先複數的 Game Patches 及舊 ZIP 是私人歷史快照。

遊戲安裝檔、生成的完整遊戲容器、私人存檔與機器憑證保留在私人來源；公開版本以安裝端原版檔案、輸入雜湊及可執行的組建工具重建。Idle Spiral 的既有歸檔 Release 由索引連結。Max Manos 和 Particul 的公開來源已由原版離線重建並對到歷史雜湊。

某些封裝器會將 JavaScript 片段逐位元寫進遊戲容器。發行後應從乾淨 Git 封存再核對一次輸出，特別是 Windows 存檔、ZIP 和 Git 的 CRLF/LF 轉換可能改變結果。
