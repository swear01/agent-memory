---
title: "空排除 regex 會把所有候選濾掉"
scope: tools/shell
status: active
updated: 2026-09-16
evidence_digest: 639f59d55684bd7402b6fb58a415aefebcc2ac8434c359deb5b2cb24d789eee0
---

# 空排除 regex 會把所有候選濾掉

助手承認腳本組出空的 grep 排除 pattern，讓全部輸入都被濾掉，原先零候選是腳本問題。後續貼出的歷史清單其實非空。

沒有排除條件時明確略過該過濾步驟，不把空字串交給反向匹配；以已知非空輸入檢查這個邊界。

後續反向 patch 套用比例只代表當時樣本，不作語意可行性或其他版本的保證；與空 pattern 根因分開。
