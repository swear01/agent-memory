---
title: "Unity Web 版按 Esc 後重鎖滑鼠跳 alert"
scope: tools/games
status: active
updated: 2026-11-25
---

# Unity Web 版按 Esc 後重鎖滑鼠跳 alert

症狀：Unity 6 WebGL／WebGPU 版用 `Cursor.lockState = Locked`，玩家按 Esc 後馬上點畫面，跳出 Unity 的
「An error occurred running the Unity content」alert，內容是
`SecurityError: Pointer lock cannot be acquired immediately after the user has exited the lock.`

原因：瀏覽器在離開 pointer lock 後短時間內拒絕 `requestPointerLock()`，promise reject 被 Unity loader
當成致命錯誤。遊戲本身沒當，稍等再點就能鎖。loader 只內建忽略 "fullscreen error"。

修法：在 `index.html` 的 `createUnityInstance` 之前加

```js
config.errorHandler = (msg) => /pointer lock/i.test(msg);
```

回傳 true 代表已處理，Unity 就不跳 alert。用 build 後處理插入時，找不到插入點要讓 build 失敗，不然模板改版會悄悄失效。

驗證：headful Chromium（Playwright）點擊鎖定 → 連按 Esc 四次 → 再點，記錄 `dialog` 事件和
`document.pointerLockElement`。headless 或視窗不在前景時鎖不到，要先 `bring_to_front` 並重試點擊。
