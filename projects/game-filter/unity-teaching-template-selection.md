---
title: Unity 教學模板人工篩選：選用 Pomdap 3D Platformer
scope: project
project: game-filter
tags:
  - unity
  - teaching
  - game-template
  - platformer
status: active
created: 2026-08-26
updated: 2026-08-26
---

# Unity 教學模板人工篩選：選用 Pomdap 3D Platformer

## 最終決策

人工遊玩 GATEUnity、Pomdap 3D Platformer 與 Unity Asteroids Workshop 後，選用 **Pomdap 3D Platformer** 作為主要教學模板。它是三款中唯一同時具備舒服操作、完整核心循環、適中規模與低修改門檻的專案。

適合從現成底盤延伸關卡、平台機關、敵人、角色能力、收集物、UI 與音效；學生能先讀懂角色移動、二段跳、相機、金幣、掉落平台與手把支援，再完成一項明確改造，不必先處理大量相容性或架構債務。

## 未選用原因

- **GATEUnity**：系統較完整，但玩法引導、攻擊回饋與通關流程不完整，程式耦合與效能問題較多。它較適合進階的「讀懂並修復既有系統」課程，不適合作為本次主要底盤。
- **Unity Asteroids Workshop**：核心循環過於精簡，而且 repository 的程式與序列化場景／Prefab 已經不同步。匯入與 C# 編譯通過不代表能正常遊玩。

## Asteroids 執行期陷阱

實際進入 Play Mode 後才發現三個根因：

1. 場景保存的是 `UnityEngine.UI.Text`，`Game.cs` 卻宣告 `TMP_Text`，造成 `Serialized reference type mismatch`。
2. `PlayerPrefab` 保存的是 `engineAudioSource`，`Player.cs` 卻改成 `thrustSounder`，造成 `UnassignedReferenceException`。
3. `Game.cs` 新增 `canvas` 欄位後，`Asteroids.unity` 沒有重新綁定，造成 `UnassignedReferenceException`。

Unity 6.5 測試副本只修改 `Assets/Game.cs`、`Assets/Player.cs` 與 `Assets/Scenes/Asteroids.unity` 後即可正常顯示飛船、小行星、生命與分數；原始 Unity 2022 repository 保持未修改。因此不要把原始專案記為「可直接執行」。

## 後續驗證標準

Unity 教學候選不能只做匯入／編譯檢查。至少要用目標 Editor：

1. 載入正式入口場景。
2. 進入 Play Mode。
3. 實際操作核心循環。
4. 檢查 Console 的執行期例外與序列化引用。
5. 再評估手感、可教內容與學生修改成本。
