---
title: "kpsewhich 命中殘留檔不代表 LaTeX class 可用"
scope: tools/latex
status: active
updated: 2026-09-16
evidence_digest: 0028f1b39a6ba682cc337ec4b4a7f783e55fd1a2a85ee8420fe8819dc583e44f
---

# kpsewhich 命中殘留檔不代表 LaTeX class 可用

下載 class 的回應實際是少量 HTML，tlmgr 又因缺少資料庫路徑失敗；kpsewhich 後來命中工作目錄中的同名檔，助手才指出可能是失敗下載殘留。

檢查 payload 型別、内容與實際解析路徑，再做最小編譯；在乾淨目錄辨識是否僅由当前目錄遮蔽。檔案存在、名稱正確或搜尋命中都不等於安裝成功。來源沒有正確 class 取得或編譯通過證據。
