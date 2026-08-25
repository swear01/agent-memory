---
title: ICCAD2026B GitHub Actions runner on Mazu
scope: machines/mazu
project: ICCAD2026B
status: active
confidence: high
evidence: PR 167 job 97825964168 passed the Rust checks on the replacement runner image.
created: 2026-08-25
updated: 2026-08-25
tags:
  - github-actions
  - docker
  - rust
  - mazu
source_refs:
  - github:swear01/ICCAD2026B#167
redaction: passed
generated_by: openai-codex/gpt-5.6-sol
---

# ICCAD2026B GitHub Actions runner

- Runner `mazu-iccad2026b` is a reusable Docker runner for `swear01/ICCAD2026B` with labels `ubuntu-22.04` and `mazu-ci`.
- Its image source is `$HOME/.local/share/github-actions-runner/iccad2026b/Dockerfile`.
- The image must provide PyYAML plus rustup, stable Rust, rustfmt, and clippy. Rust lives under `/opt/rustup` and `/opt/cargo`, with `/opt/cargo/bin` on `PATH`.
- Current image tag (2026-08-25): `iccad2026b-actions-runner:2.336.0-jammy-rustup`.
- Persistent volumes: `mazu-iccad2026b-config-v2:/runner-config` and `mazu-iccad2026b-work:/_work`.
- Replacing a reusable runner can briefly yield `A session for this runner already exists`; wait for the old GitHub session to expire and verify the new listener logs `Listening for Jobs` before rerunning queued checks.
- Verified on PR 167: Rust toolchain install, `cargo fmt`, `cargo clippy`, and `cargo test` all passed on job `97825964168`.
