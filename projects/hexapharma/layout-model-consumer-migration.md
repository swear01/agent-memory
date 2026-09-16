---
title: "Layout 改成 machines 陣列時要同步所有 consumer"
scope: projects/hexapharma
status: active
updated: 2026-09-16
evidence_digest: 146722c7cd89b114801a3f8ec6c37556017306636799c6bb18da5af3f432abd6
---

# Layout 改成 machines 陣列時要同步所有 consumer

原始 TypeScript errors 顯示 renderer 與 UI 仍把 machine 當 tile、建立缺 machines 的 layout，且 analyzeThroughput 仍傳舊參數；來源另外指出 bottleneck 已是實例數字 ID，renderer 卻當 type ID。

沿模型建立、分析、渲染與互動的所有 caller 完成契約變更，區分實例身份與種類，不以 cast 或補舊欄位掩蓋分歧。來源只到盤點與準備修改，沒有 typecheck 或遊戲驗收通過，不能把理解新模型稱為遷移完成。
