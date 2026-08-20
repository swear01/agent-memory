---
title: Auto Storage Craftable catalogs and station geometry stay server-owned
scope: projects/Auto_Storage
project: Auto Storage
tool: Minecraft/NeoForge
status: active
confidence: high
evidence: >-
  Current source, project design notes, and 184 focused static regression tests
  cover recipe-snapshot indexing, transient cache release, server-owned
  presentation, and deterministic station layout.
created: 2026-08-20
updated: 2026-08-20
tags:
  - auto-storage
  - craftable
  - cache
  - gui
  - minecraft
source_refs:
  - Auto_Storage:src/main/java/com/swear/autostorage/CraftableRecipeCatalog.java
  - Auto_Storage:scripts/test_static_regressions.py
  - Auto_Storage:docs/notes.md
  - Auto_Storage:scripts/run_prism_gui_session.py
redaction: passed
generated_by: openai-codex/gpt-5.6-luna
---

# Catalog boundary

The Craftable catalog is server-owned. Build and cache its index from the live
recipe snapshot, key cache reuse to that snapshot identity, invalidate on
reload, and release transient recipe matches after presentation work. Do not
make the client scan `RecipeManager`, infer recipes from display stacks, or
retain full variant graphs after the menu has materialized its presentation.

Candidates should be narrowed by indexed ingredient requirements before the
full craftability simulation. Exact recipe identity, adapter obligations,
variant resolution, and simulate-then-commit validation remain server-side.

# Station presentation

Station geometry is derived from the current descriptor set and available
panel bounds. Keep processing work as a separate aggregate value from the
installed item count, use bounded deterministic paging, and preserve stable
ordering for incomplete final rows. A large aggregate must remain a `long`
value outside the item icon count; it must not overflow or alter layout
selection.

These are automated contracts, not a fullscreen visual verdict. A rendered GUI
still requires its separate visual review.
