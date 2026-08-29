---
title: shared-skills GitHub Actions runner on Mazu
scope: machines/mazu
project: shared-skills
status: active
confidence: high
created: 2026-08-30
updated: 2026-08-30
tags:
  - github-actions
  - docker
  - mazu
---

# shared-skills GitHub Actions runner

- `mazu-shared-skills` is the intended reusable runner for `swear01/shared-skills`, with the `mazu-ci` label.
- An offline registration does not prove that the runner was rebuilt persistently. On 2026-08-30, GitHub still listed `mazu-shared-skills` offline, while Mazu had no matching listener process, service, persistent runner directory, or container.
- The recovery containers used `myoung34/github-runner:2.336.0-ubuntu-noble`, but logged `Runner reusage is disabled` and `Ephemeral option is enabled`. Each processed one job, removed its runner configuration, and exited with Docker restart policy `no`.
- A durable repair must create a reusable, supervised listener rather than another ephemeral recovery container. Verify that it remains online after completing a job and after its supervisor restarts it; remove stale GitHub registrations only after the replacement listener is healthy.
