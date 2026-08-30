---
title: GitHub Actions runners on Mazu
scope: machines/mazu
projects:
  - shared-skills
  - transfer_MAC
status: active
confidence: high
created: 2026-08-30
updated: 2026-08-30
tags:
  - github-actions
  - docker
  - mazu
---

# GitHub Actions runners

- `mazu-shared-skills` is the intended reusable runner for `swear01/shared-skills`, with the `mazu-ci` label.
- An offline registration does not prove that the runner was rebuilt persistently. On 2026-08-30, GitHub still listed `mazu-shared-skills` offline, while Mazu had no matching listener process, service, persistent runner directory, or container.
- The recovery containers used `myoung34/github-runner:2.336.0-ubuntu-noble`, but logged `Runner reusage is disabled` and `Ephemeral option is enabled`. Each processed one job, removed its runner configuration, and exited with Docker restart policy `no`.
- The durable setup uses repo runners `mazu-shared-skills` and `mazu-transfer_MAC`, container image `myoung34/github-runner:2.336.0-ubuntu-noble`, label `mazu-ci`, Docker restart policy `unless-stopped`, separate persistent `/runner-config` and `/_work` volumes, and the host Docker socket for nested gitleaks.
- Set `CONFIGURED_ACTIONS_RUNNER_FILES_DIR=/runner-config` and `DISABLE_AUTOMATIC_DEREGISTRATION=true`; do not set `EPHEMERAL` at all because this image treats any non-empty value, including `false`, as enabled.
- After initial registration, wait until the runner is idle, recreate the container from the saved config without `RUNNER_TOKEN`, and allow the old GitHub session to expire if the listener reports `A session for this runner already exists`. On 2026-08-30, both token-free containers reconnected, completed CI jobs, remained online, and retained their reusable configuration.
