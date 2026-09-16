---
title: "Resume 後沿回傳的新 session 身份處理復原"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: c41573fd667adeef8e42060ca99c8347a09a694ff1e5c132e64219c2a5b1413f
---

# Resume 後沿回傳的新 session 身份處理復原

助手曾一直輪詢被合併的舊 session，將預期的 404 誤報成 reopen bug；後來按 response 新 ID 驗證成功。另一個 source 指出 resume 後 send 失敗卻把可重試 transcript 存回舊 ID。

沿 resume 的有效回傳身份更新導覽、查詢與失敗復原的 key，分開檢查舊 ID 的退休與新 ID 的可用性。不能把舊 ID 不存在當整個 resume 失敗。

舊 ID 誤判的更正是歷史自述，send recovery 只有定位。Service tier 是否完整保留屬另一欄位契約，沒有本筆中的修復完成證據。
