---
title: Experiment evidence needs oracle, fresh artifacts, and fail-closed gates
scope: global
status: active
confidence: high
tags:
  - evidence
  - experiments
  - benchmarks
  - verification
  - fail-closed
source_refs:
  - agent-memory:issue-10
  - agent-memory:issue-12
  - agent-memory:issue-13
  - agent-memory:issue-16
created: 2026-08-20
updated: 2026-08-20
generated_by: openai-codex/gpt-5.6-luna
---

# Claim boundary

An owner-attested execution establishes that a run was performed and that it
emitted the reported result. It does not by itself establish the root cause,
soundness, or generalization of that result. Treat exact measurements as G4
until their method, scope, and configuration support a broader claim.

# Candidate and verifier

Generated suggestions are untrusted candidates, not verdicts. Keep the
original verifier model intact and gate each candidate with the applicable
syntax/type, satisfiability, initiation/consecution, inductiveness, or
vector-equivalence checks before it can affect a result. A guessed invariant
or constraint must never be injected as an unchecked model restriction.

# Oracle and proxy

A proxy such as structural coverage, a heuristic score, or an incomplete
fault profile is not an exact outcome. Compare it with the complete oracle
when one exists, account for skipped or unreachable classes, and label the
result as a proxy when the oracle or denominator is incomplete.

# Fresh, bounded run artifacts

Keep task inventory, execution inventory, and outcome records distinct. Each
run needs stable task or fault keys, an immutable input/configuration manifest,
run-scoped output, and an explicit policy for solved, failed, unknown, and
timeout states. Bound per-target work and clear stale artifacts before a run;
a timeout remains a timeout until the current process emits valid evidence.

Use fixed denominators and stable keys when comparing progressive or partial
runs. Do not combine numbers from different builds, datasets, solver
portfolios, seeds, time limits, parallelism, or timeout policies into one
experiment.

# Fail closed

Provider, transport, configuration, parser, resource, and telemetry failures
must be observable and separate from solver or model outcomes. A missing
request, abnormal process exit, or zero-count smoke result is an environment
failure until the run manifest proves otherwise. Fallback behavior must be
explicit in the result; never silently replace an experimental path with a
different implementation and call it equivalent.

# Conflicts and duplicate views

Collapse exact, normalized, semantic, and shared-filesystem views before
counting evidence. Preserve conflicting configurations and measurements as
separate runs with their provenance; deduplication must not select a winning
number. Cross-project reuse applies to the acceptance rule or mechanism, not
to a project-specific score.

# Scope

The rules above are G1 experimental-method boundaries. Tool or fleet
implementation details are G2, project integrations are G3, and individual
benchmark values or incidents remain G4 unless a later review establishes a
broader scope.
