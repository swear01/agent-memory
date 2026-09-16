---
title: "TeX 控制字與後續文字需要明確邊界"
scope: tools/latex
status: active
updated: 2026-09-16
evidence_digest: 1f815fd85f238a1a4341b75e09d8a1ec32da0e6d106fc8df1fbad8df14e12ee3
---

# TeX 控制字與後續文字需要明確邊界

來源 frame 含 footnotesize 與 RTL、wordlevel、reset 直接串接的控制字；助手先改 include，後來才指出字型命令與文字間的空格遺失。

遇到像既有控制字加上正文的錯誤名稱，直接檢查 token 邊界，以空格或空群組分開，再做最小編譯。Include 選擇與此語法錯誤分開確認；來源沒有修正後編譯結果，frame 內研究效能宣稱也未在本次核驗。
