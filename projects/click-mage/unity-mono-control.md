---
title: "Click Mage Unity Mono control and pickup capacity"
scope: projects/click-mage
status: active
updated: 2026-11-26
---

# Click Mage Unity Mono control and pickup capacity

Click Mage v1.5.1 uses Unity 6000.2.7f2 and the Mono game assembly `ClickMage_Data/Managed/_GameAssembly.dll`. The source and restore script are in the private Game Patches collection under `Click-Mage/`. The control bridge runs from `PlayerInputManager.Update` and substitutes world coordinates at the stock click, collect, and drop cursor lookups. It keeps the stock node level, item registry, hand and storage rules.

When automating held pickup, check capacity before each item enters the hand registry. In `PlayerCollect.CollectResources`, stock code calls `SpawnedResourceManager.MoveResourceToHand` before `AddToHand` tests the capacity. A guard at method entry alone allows a later item in the same overlap loop to be moved in the registry after the hand fills. The bridge checks immediately before the registry call, and its check allows the stock manual path through unchanged.

OpenSpeedy and Unity `Time.timeScale` can both accelerate the same gameplay. Disable the per-game OpenSpeedy toggle before measuring Unity-only acceleration and compare in-game clock advance against an independent wall clock. The rendered v1.5.1 test measured about 5.9× at a 6× setting. The stock pause stopped the clock, blocked pickups, and restored the requested multiplier after resume. The active save accepted stock stone-block deliveries up to the portal limit, and level 2 click did not produce copper ore from the level 3 node.
