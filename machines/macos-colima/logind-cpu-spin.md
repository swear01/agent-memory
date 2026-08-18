---
title: Colima VM high CPU can originate in guest systemd-logind
scope: machines/macos-colima
tool: Colima
status: needs-review
confidence: high
evidence: >-
  Host Virtualization VM stayed near 100% CPU while Docker used 0%; guest
  systemd-logind used 99.7%, D-Bus queues were full, and SIGKILL plus respawn
  reduced host VM CPU to about 1.5%; root trigger remains unknown.
created: 2026-08-18
updated: 2026-08-18
tags:
  - colima
  - lima
  - virtualization
  - systemd-logind
  - dbus
  - cpu
source_refs:
  - codex-b001-linked-versedx:019ef36a-e21b-7060-884c-74ac3729fbf7
redaction: passed
generated_by: openai-codex/gpt-5.6-luna
---

# Finding

When a Colima VM causes near-100% host CPU, inspect the guest before blaming Docker. In this incident the Docker container used 0.00% CPU while guest `systemd-logind` used 99.7%; `loginctl` and D-Bus reported full-queue/time-out errors.

Restarting the service and sending SIGTERM did not recover it. Sending SIGKILL to the stuck guest process allowed systemd to respawn it, after which host VM CPU fell to about 1.5% while Docker remained running.

This identifies an immediate recovery workaround, not the underlying trigger. Keep the root cause unresolved until a recurrence is investigated.
