---
title: swairM5 DVLab CRA parity 本機 preview 路徑
scope: machines/swairM5
tool: local-preview
machine: swairM5
status: active
updated: 2026-09-30
---

- Machine name 可寫作 `swairM5` 或 `SwairM5`；以下使用 `<remote-home>` placeholder，不保存 raw user-home path。
- Astro PR101 checkout：`<remote-home>/dvlab-ntu.github.io-pr101`；執行 `npm run dev -- --host 127.0.0.1 --port 4321`，預覽 `http://127.0.0.1:4321/`。
- Old CRA checkout（detached `6e33b72`）：`<remote-home>/dvlab-ntu.github.io`；frontend build 為 `<remote-home>/dvlab-ntu.github.io/frontend/build`，歷史上以 `:4173` 提供。
- Full offline CRA static mirror：`<remote-home>/dvlab-cra-static-mirror/`，zip 為 `<remote-home>/dvlab-cra-static-mirror.zip`（約 88MB），以 `http://127.0.0.1:4174/` 提供；包含 home、host、members（含 details）、publications、courses、about 與 frozen JSON。
- Cloud agent 使用的 slim pack 在 box：`/workspace/dvlab-cra-static-ref.tar.gz`（已移除 member photos）；CRA screenshot refs 在 `<remote-home>/dvlab-cra-ref/`。
