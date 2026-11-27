---
title: Grok Bot GIF 頭像的原生設定入口與 Mac 播放驗證
scope: tools/grok-bot
status: verified
created: 2026-10-09
updated: 2026-11-19
tags: [grok-bot, gif, avatar, macos]
---

## 匯入與驗證

1. 先完成循環 GIF。遇到 `The image must be non-empty and at most 5 MB.` 時，以小於 5,000,000 bytes 為保守目標，輸出後再實際解碼檢查。直接提供 Mac 檔案路徑曾回 `Could not read`。
2. 請目標 Bot 將原 GIF 原封不動複製到它的雲端 `/workspace`，再用內建的檔案路徑功能設成自己的頭像。在 0.68.1、0.78.1 Mac App 驗證播放；其他平台與工作狀態切換未驗證。
3. 核對目標的 live avatarVersion 與 Mac `roster-avatars` 快取的完整 SHA-256，解碼檢查 format、畫格、時長與循環。快取可能晚於 Bot 回覆出現。保存檔可能叫 `avatar.png`，內容仍是 `GIF89a`。
4. 用實際 App 側欄連續取樣，至少覆蓋最長循環。單張截圖、工具成功、快取一致及 headless 瀏覽器播放各有自己的證據範圍。Mail 大動作版當時只有保存與瀏覽器播放證據。

桌面裁切編輯器在 Mac 0.68.1 以 256×256 canvas 的 `toDataURL("image/png")` 經 `setAgentAvatarBytes` 保存。貼上 GIF 可顯示候選，但無法避開該裁切入口的 PNG 轉檔。

## 角色與動畫

- 使用者偏好同系列大頭、極小身體。先看 48／72 px 圓形預覽，確保臉、手或道具可辨認，動作在該尺寸有足夠幅度。頭可平順晃動。逐格重畫會使臉、手與道具的位置或比例漂移，同一原畫分層變換較容易控制。
- 使用者要求「十六張」時是 16 張循環 GIF 畫格。讀書鯨左接書、翻頁、右送出、回到空手，閉合點應放在空手姿勢，不能倒播讓書從右邊回來。檢查最後一格到第一格、時長與一般相鄰格。
- 信封要和手一起前推，維持同一隻手揮動，麥克風與握持手一起移動。多個前景遮罩先各自 source-over 形成 alpha 聯集，避免反向 compound path 在交界挖出透明洞。用最終 GIF 解碼確認顏色、遮罩、時長與圓形裁切。
- 好康 Bot 大頭版已由使用者確認。它沿用同一張頭部底圖穩定帽子與臉，真正分格開禮物盒、冒星星，再蓋回去。512×512、16 張不同畫格、2.8 秒無限循環；Mac 快取比對與實際側欄播放皆通過。

## 建立與送出

`grok-bot-cli` 0.12.4 的 `doctor` 曾找不到 `gateway-descriptor.json`。App 已登入與 CLI 已安裝不代表 gateway session 可用。原生介面可建立 Bot 並交接摘要及教材；Bot 自述雲端 hash 不能代替獨立讀回。

AXPress 或 `NSRunningApplication.activate` 回報成功後仍要讀回實際視窗。啟用後保留 run loop，確認新前景與角色名稱。在同一程序檢查前景 App、精確角色標題及空白輸入框，輸入後再查一次才送出。若 OCR 把聊天文字連到標題，縮小標題區域；避免改用任意子字串命中。更新時也可比對其他角色的 version/hash，確認操作只落在目標。

歷代成品路徑、各角色規格、播放樣本與教材交接見 `git show 54da0f0adfdb3dacb3bca59a729b12975d44ec9b:tools/grok-bot/animated-avatar-file-import.md`。
