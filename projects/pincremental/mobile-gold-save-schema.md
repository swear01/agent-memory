---
title: Pincremental Steam and mobile save schema
scope: project
project: Pincremental
status: active
created: 2026-09-05
updated: 2026-09-05
---

- **Export Save includes an amount Android/Steam display but Web ignores.** Android 1.10.79 and Steam 1.05.0 both reported internal v1.107; Gold and purchase bookkeeping are in base64 JSON `localStorage.dAustin`. Preserve cross-platform unknown keys, since older Web has no Gold UI. Restore Purchases recomputes from recognized `gold*` transactions minus WarpSpend; its value can differ from current local balance.
- **Import Successful but Lifetime Mana is unchanged.** Import only validates outer decoding; native LoadGame ignores unknown keys. Decode every inner base64 JSON, verify no U+FFFD and read back each intended field in the game. Real damage from `Lifet\uFFFDMana` left normal Mana and TotalMana visible but made LifetimeMana stale. Back up before recovery and work with the game closed.
- **Mana button has a positive label but is disabled.** `AvailableManaNow()` reports investor-derived amount. `WizardReady` separately requires >=4 and all direct investor upgrades (except `AutoCompanyOn` and the `PermaBuyers` container), plus each permanent buyer. Check AutoShares and AutoArcade; both blocked a real case.
- **Normal achievement event runs but Steam cache stays unchanged.** Use `Start-Process "steam://rungameid/1369470"` or `steam.exe -applaunch 1369470`, then perform the original in-game action and check Steam. Directly running Pincremental.exe did not trigger Sacrifice achievement in the observed case.

[完整原文](https://github.com/swear01/agent-memory/blob/06f50e3a42e850b37cffb6b6a4a446ef670e9625/projects/pincremental/mobile-gold-save-schema.md)
