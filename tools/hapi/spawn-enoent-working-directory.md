---
title: "spawn ENOENT 也要檢查 session 工作目錄"
scope: tools/hapi
status: active
updated: 2026-09-16
evidence_digest: 602f011266308bd360e78a40f0d9db4dfba0cfee9553879a8d63c8fcbafc3b92
---

# spawn ENOENT 也要檢查 session 工作目錄

歷史 HAPI 診斷回報中，runner 的 PATH 存在，git 本身也可執行，但 session 指向已不存在的工作目錄；取得 Git 狀態時卻出現看似找不到 git 的 posix_spawn ENOENT。

遇到子程序啟動的 ENOENT，要分別檢查可執行檔與該 session 的 cwd。runner 在線或其他目錄能執行 git，不代表這個 session 的啟動條件成立。先確認正確工作目錄，再依介面支援方式修正或重開該 session，避免無依據地重啟整個 runner。

根因與建議來自歷史助手報告，來源没有修正後成功啟動的紀錄；不把它當成所有 ENOENT 的唯一原因。
