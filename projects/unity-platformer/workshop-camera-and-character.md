---
title: Unity Platformer 工作坊：手動換皮流程與保留入場效果
scope: projects/Starter-Kit-3D-Platformer-for-Unity
status: active
updated: 2026-09-29T03:05:19+08:00
tags: [unity, platformer, camera, character, prefab, manual-import]
---

# Unity Platformer 工作坊

## 使用者已確認的做法

- 角色換皮示範使用 Unity 原生 Inspector 與手動操作；不要建立自訂 Editor 選單、匯入工具或一鍵準備工具。先前的 `Tools > Character Workshop > 1 Prepare Zombie Prefab`／`2 Apply Prepared Zombie To Player` 已刪除，不要再推薦或重建。
- 要教的是：下載模型／Prefab → 匯入設定 → 尺寸與朝向 → 材質與骨架／動畫 → 保存可用的角色 Prefab → 接到原 Player → Play 驗證。
- Game 畫面只保留原 HUD，F1 原版／F2 改良版 camera 是隱藏示範功能。
- 開場旋轉及跳躍效果是使用者要保留的入場表現，不要當作控制 bug 刪除，也不需要另做開始／結束階段系統。舊紀錄的「移除開場角度漂移」已被這個決定取代。`View.cs` 現用 `m_OpeningRotationOffset` 保留開場旋轉，同時讓 mouse look 立即響應；切換模式不重播。
- 持續在正式原專案 `<user-home>/Documents/project` 操作，不另外建立獨立 Unity checkout/worktree。保存 unrelated dirty/untracked 內容，只提交本次檔案。
- `origin` 是使用者 fork `swear01/Starter-Kit-3D-Platformer-for-Unity`；`upstream` 是 `Pomdap/Starter-Kit-3D-Platformer-for-Unity`。原作者 push 曾 403，後續走使用者 fork 的 branch／PR 流程。

## 手動換皮的實例與限制

- 範例素材：Kenney Animated Characters Survivors，使用 `Model/characterMedium.fbx`、`Animations/idle.fbx`／`run.fbx`／`jump.fbx`、`Skins/zombieA.png`，保留 License.txt。可一次拖入全部檔案；FBX 匯入設定各自保存。
- 這包素材的 Model 設定：Scale Factor `0.28`、Convert Units 開啟、Bake Axis Conversion 開啟。先 Apply Model，再於 Rig 建立 Humanoid／Create From This Model；不是其他素材通用的縮放數字。
- 三個動畫留下真正的 Idle／Run／Jump Clip，移除 Targeting Pose；Idle／Run 循環、Jump 不循環，Root Transform Bake Into Pose，Animator 關閉 Apply Root Motion，由 CharacterController 控制移動。
- 綠色 Avatar 標記不能證明尺寸正常。先前改單位／縮放後舊 Avatar 映射曾造成動畫骨架放大；手動可用 Configure → Mapping Clear → Pose Sample Bind-pose → Mapping Automap → Pose Enforce T-Pose → Apply／Done，然後預覽確認。仍有問題則把原始 FBX 匯入新資料夾，先套尺寸再建立 Rig；不要假稱單純 Reimport 就一定清除舊映射。
- `ZombieCharacter` 外層 Transform 為 0／0／1；內部模型 Y 旋轉 `180°`，讓這個素材朝 +Z。保存在 Prefab，不在 Player.cs 以固定補償強行修正所有模型。不同模型要實查朝向。
- URP Lit 材質使用 zombieA 貼圖、白色 Base Color。角色使用複製的專用 Animator Controller；保留 Speed（Float）／Grounded（Bool），Locomotion Motion 為 Idle／Run／Run，Threshold `0`／`0.5`／`0.7`，第三格速度 `1.2`。Jump 使用 Jump，兩條 Grounded 轉場關閉 Has Exit Time。
- Player 的 Model 指向角色外層，Animator 指向內部骨架 Animator。保留 Player 移動、碰撞、Input Actions、二段跳與效果引用；`Player.cs` 以視覺初始大小做跳躍／落地變形及恢復，不覆寫素材朝向。
- 現成 Unity Prefab 必須帶齊模型、材質、貼圖、動畫與 .meta 引用。需移除來源控制／碰撞元件時，先在 Hierarchy 對實例用 Prefab → Unpack Completely，再整理並保存自己的 Prefab；不要依賴不能任意移除 inherited components 的 Variant。
- 正式教學文件是 `<project-root>/Docs/CharacterWorkshop.md`，README 有入口。文件包含 Avatar 重建、現成 Prefab 匯入與手動 Play 檢查，不要求執行匯入工具或 CharacterSmoke。

## 收尾證據：2026-09-29 快照

- Unity 為 `6000.6.0f1`。PR #3 恢復開場 camera 旋轉；PR #4 刪除 `Assets/Editor/SwapCharacter.cs`、script meta、空 Editor folder meta，更新手動教學與 README。
- PR #4 已 MERGED 至使用者 fork main；head `02abfbe6ff2ebd501c78a59f9fa197dbdd0bc7bd`，merge `4378e9d0c0fef4375443c4e58c27a0fc0680cf88`。本機 main／origin/main／GitHub main 相同，僅原專案工作樹。這些是快照，後續修改前重新實查。
- 移除工具後 AssetDatabase.Refresh、recompile／recompile_status 無編譯錯誤；loaded assemblies 找不到 SwapCharacter，現有 Player Model／Animator／Humanoid Avatar／專用 Controller 引用有效。29 個原有 dirty/untracked 檔案 SHA-256 保持相同。
- 本次記憶更新再次實查成品：Avatar valid／human=true，humanScale `0.4499078`，外層 Scale `1`，內部模型 Euler Y `180°`。
- PR #4 本次只做編譯、引用、來源文件與匯入設定檢查，沒有從頭重跑完整手動匯入；不要把先前工具準備成品說成已驗證完整手動操作。
- Swear Review 因 no items selected 跳過，不能把 execution success 當 substantive clean review。Gemini 後續對精確 head 給出無意見終態；依 OR 規則完成合併。沒有 required CI checks，不宣稱遠端 build 通過。
