---
title: Verto 撞頭（head bonk）手感慣例
scope: projects/verto
project: verto
status: active
updated: 2026-10-31
---

# Verto 撞頭手感

PR #42（`9951242`）。完整調查表與來源見 repo 的 `Documentation/HeadBonk.md`。

## 慣例

- 撞到天花板時 `vy = 0`、`vx` 不動；Celeste 同樣（`Speed.Y = 0`、不碰 `Speed.X`、`UpwardCornerCorrection = 4`、`CeilingVarJumpGrace = .05`）。
- 使用者覺得「撞頭後直接掉下來、失去前進感」。解法：`BonkHang = 0.1`（約 6 幀）暫停重力，期間 `vx` 保持；撞頭一律結束 variable-jump hold（`varTime = 0`），否則早撞會黏在天花板。
- `hang` 必須在 `Jump()`、落地（grounded）、`EnterFrom`（換房）清零，否則走下平台邊緣或進下一房會漂浮。review 兩個 bot 各抓到其中一項。
- 只在 `vy < 0` 的撞擊才設 hang；jump 高度、wall slide/jump、R→B→G 規則都沒動。

## 測試與流程陷阱

- 測試用「早撞」那幀與 hit 到首次下落之間的幀數（4–8）才能在舊程式上失敗；mutation 驗證（改回舊行為看測試是否紅）。
- `vx` 在撞擊幀可能仍在加速，斷言用 `>=` 前一幀而不是 `==`。
- `Verification/BuildWeb.py` 的 ControllerTests 計數要同步（16 → 18）。
- 本機 `dotnet test Verification/ControllerTests.csproj` 需 `DOTNET_ROOT=/opt/homebrew/opt/dotnet/libexec`（預設 SDK 9 不能 target net10）。
- Swear Review 可能連續回「Review infrastructure/model failure」；兩次失敗後改 `/gemini review` 即可通過。
- `gh pr merge` 會被 auto-mode classifier 以「Merge Without Review」擋下，需使用者自己合併或加權限規則。
- Umbrella 路線測試（`UmbrellaLiveInput`）在 main 已失敗，無法用來判斷懸停影響；交由 route-tolerance 處理。
