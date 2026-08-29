---
title: ICCAD2026B GitHub Actions runner on Mazu
scope: machines/mazu
project: ICCAD2026B
status: active
confidence: high
evidence: PR 167 job 97825964168 passed the Rust checks on the replacement runner image.
created: 2026-08-25
updated: 2026-08-30
tags:
  - github-actions
  - docker
  - rust
  - mazu
source_refs:
  - github:swear01/ICCAD2026B#167
  - github:swear01/ICCAD2026B#178
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
- Normal integration CI and formal release are intentionally separate. Normal CI uses
  `mazu-ci`; formal release jobs use the dedicated `mazu-ci-release` label.
- The formal release runner exposes rootless Podman and must not expose
  `/var/run/docker.sock`. The workflow verifies both properties before building.
- Dedicated release deployment details: outer Docker container
  `mazu-iccad2026b-release`, image
  `iccad2026b-actions-runner:2.336.0-jammy-rustup-release`, and image source
  `$HOME/.local/share/github-actions-runner/iccad2026b-release/Dockerfile`. Persistent
  volumes are `mazu-iccad2026b-release-config:/runner-config`,
  `mazu-iccad2026b-release-work:/_work`, and
  `mazu-iccad2026b-release-podman:<remote-home>/.local/share/containers`.
- Nested rootless Podman requires `slirp4netns` plus
  `runner:100000:65536` in both `/etc/subuid` and `/etc/subgid`. On this Docker host the
  outer runner container needs `--privileged` for Podman namespaces, but it still runs as
  UID 1001 (`runner`) and deliberately has no Docker socket bind. Starting that container
  with the image default `RUN_AS_ROOT=true` while overriding `--user runner` causes a
  restart loop with `ERROR: RUN_AS_ROOT env var is set to true ... UID '1001'`; set
  `RUN_AS_ROOT=false` when reusing the saved runner config.
- The persisted Podman store currently uses `vfs`; `podman info` emits
  `User-selected graph driver "overlay" overwritten by graph driver "vfs" from database`.
  This is noisy but the cached, cold-cache, publish, and Drive jobs all completed through
  that store. Do not delete libpod state during an active job merely to silence it.
- Release ABI comes from `manylinux_2_28_x86_64`, not from the Ubuntu host image.
  `stable-2.3` verified the dedicated runner path through cached build, cold-cache build,
  GitHub publish, and Google Drive upload.
