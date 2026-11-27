---
title: Isolated Cursor ACP tests on macOS
scope: tools/cursor
status: active
updated: 2026-11-28
---

For disposable HAPI tests that start the real Cursor ACP client with a temporary HOME, set `AGENT_CLI_CREDENTIAL_STORE=memory` **after** sanitizing the child environment. Keep the ordinary protected token handoff scoped to the Cursor session and apply the same environment to replacement processes. Leave the real account and global Cursor settings intact.

On Cursor CLI `2026.11.22-e9042b2`, default storage on macOS can probe the keychain with a write, even when a token is already provided. HAPI upstream main and the native history runner pass the parent selection through. A rewind starts a second ACP client. To verify the boundary, use a fake ACP executable for the unset baseline, then run the real isolated session with memory storage. Check the store selection and token presence for both session children, without recording values. A runner capability probe can be a separate process without a session token.

A time-bounded unified-log query for authd, subsystem `com.apple.Authorization`, and `system.keychain.create.loginkc` distinguishes new authorization requests from older ones. Check a known historical interval as a positive control. These events do not establish whether a dialog was displayed.
