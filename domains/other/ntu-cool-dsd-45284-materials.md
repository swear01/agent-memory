---
title: "NTU COOL 數位系統設計（course 45284）講義下載與存放"
scope: "domains/other"
status: active
confidence: high
updated: 2026-09-29
tags:
  - ntu-cool
  - course-45284
  - digital-system-design
---

# NTU COOL 數位系統設計（course 45284）講義下載與存放

## 帳號與課程

- 帳號：`<redacted-id>`（Canvas user id `<redacted-id>`）
- 課程：`數位系統設計 Digital System Design`，course id `45284`
- API token（用途名 `NTU COOL bot <redacted-id>`，不過期）存於 `<home>/.secrets/ntu_cool_token_b11901015`（mode 600）；勿寫入本檔

## 下載結果（2026-09-29）

- 約 31 個教學檔（投影片 PDF + Homeworks zip/tcl），約 117MB
- 含 Basics of CA、Week 1–6 / 8–9 / 11–13、Homeworks（HW0–3、Exercise、Final）
- 跳過：`Midterm_seats.xlsx`（名冊）；串流 ExternalTool 講義影片
- 拿不到：舊版 `W8_Synthesis.pdf`（file id `6938576`，`hidden_for_user` / API 404）

## 本機最終路徑

`<mac-home>/Library/CloudStorage/<redacted-email>/我的雲端硬碟/document/學校講義/大二/數位系統設計`

對應 rclone：`gdrive:document/學校講義/大二/數位系統設計`（遠端名 `gdrive:`）

## 傳檔備註

- `CopyFromBox` 曾對 SwairM5 回 temporarily unreachable；改以 Drive 中繼 + Mac 端 `rclone copy` 可靠
- 單檔 `upload_file` 上限 50MB、`CopyFromBox` 約 100MB；大包需拆分

相關：`domains/other/ntu-cool-api-access.md`
