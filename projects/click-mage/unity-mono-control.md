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

Earlier OpenSpeedy saves can carry future in-game timestamps. The game sorted the save written at 19:47 before the saves written at 19:51 and 19:53 because its recorded time was 21:35. Continue opened the older campaign. The main-menu Load picker selected the intended save by its counter, and the bridge verified the portal and storage progress after loading.

The visible game ran in Unity FullScreenWindow mode at the monitor's native 3840×2160 even though Windows reported a 1920×1080 logical desktop. Unity VSync was disabled and the frame cap set to 60. With the stock render window open and native timeScale set to 10, the game clock advanced about 9.7 times faster than the wall clock in a live sample.

A stock StorageBuilding holds 100 matching items. A new, empty chest can report wood because ResourceType.wood is the zero-valued enum member. An emptied chest can also retain its previous type label. Use the saved item count to find an empty chest. Feed one existing item through the normal drop path and check its accepted count and resulting type before filling it. Favor chests of the same type first.

Use one serial production actor per game mailbox. Cap loose buffers and take input from chests up to the receiver's actual free capacity. In a live cleanup, the warehouse took all 521 copper ore and 192 iron ore off the ground; later production consumed the same boxed ore. A client scan timeout can precede a successful response for the same request. Inspect the process and request identity before resuming. Re-read actual payment slots before continuing an interrupted purchase.
