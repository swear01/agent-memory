---
title: XLSX 匯入空白值異常時保留原模板
scope: tools/documents
status: active
updated: 2026-11-04
---

- 在文件 runtime 26.1029.11837 的實測中，artifact-tool 匯入既有 XLSX 後，部分原本空白的儲存格變成字串 `7`；原檔以獨立讀取器檢查仍是空白。不能只信匯入後的預覽或成功匯出。
- 先用唯讀來源值還原預覽；需要嚴格保留原模板時，讓 artifact-tool 產生要改的欄位，再只把這些儲存格寫回原 XLSX ZIP，其餘成員逐位元組保留。
- 不可直接沿用另一活頁簿的 shared-string 索引或 style ID；轉成自含字串並保留原儲存格樣式。不要重序列化整份 worksheet 而遺失 `mc:Ignorable` 對應的 namespace 宣告。
- 以獨立讀取器比對所有未修改值與樣式，確認學號唯一對應、分數範圍及筆數、空白與零的差異；同時核對產出 MIME 中附件位元組。
