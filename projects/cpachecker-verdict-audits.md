---
title: CPAchecker verdict audits
scope: project
status: active
updated: 2026-09-06
---

# CPAchecker verdict audits

- For SV-COMP verdict triage, the frozen dataset/official SV-COMP label remains the correctness ground truth. Agreement from k-induction or independent tools is diagnostic evidence and must not relabel a disagreement.
- Keep arithmetic/bitvector, array-permutation, and heap/list-structural explanations separate unless source inspection identifies a shared cause. A machine-model mismatch is only a hypothesis until independently reproduced with exact provenance.
- In issue #54/#56 (2026-09-06), the 12 historical disputes were source/property/hash checked against frozen records and the official SV-COMP 2026 table; the six crash/unqualified tasks and #178 LP64 observation remain separate lanes.
