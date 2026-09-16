---
title: "修 TTS 發音前先找真正被讀取的講稿"
scope: tools/media
status: active
updated: 2026-09-16
evidence_digest: b5a1fd85999679a92a545f57ee5e464cdbc77f448f9c697d9b7326651dd8ba47
---

# 修 TTS 發音前先找真正被讀取的講稿

使用者回報中英混合詞發音怪並追問講稿來源；歷史助手說投影片順序依 main.tex、內容參考每頁 note，實際合成卻讀 narration_pipeline 的逐頁文字檔。

保留正式講稿，另在實際合成輸入處調整縮寫、符號及專有名詞讀法，維持頁序與來源對應。只改參考稿未必影響下一次合成；來源只是建議與路徑說明，沒有重新合成或聽測成功，不能保證列出的拼讀適合所有聲音模型。
