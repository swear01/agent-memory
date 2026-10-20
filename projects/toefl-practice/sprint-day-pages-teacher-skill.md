---
title: TOEFL daily pages and precomputed teacher skill
scope: projects/toefl-practice
status: verified-with-limits
updated: 2026-10-21
---

# TOEFL daily pages and teacher preparation

## Current course baseline

- The private course now uses the fixed `2026-10-17-v5` speaking and writing bank. Its manifest covers 14 full days plus a warm-up: 158 speaking items, 19 unique writing tasks, and 7 sentence-building sets of 10 items each. Each sentence set has a six-minute practice timer and includes extra word blocks. The 33 scheduled writing assignments reuse some of the 19 unique tasks. This is course material, not ETS-calibrated content.
- Verify `.agents/skills/toefl-sprint-teacher/references/manifest.json` and its source hashes before grading. The bank, lessons, and schedule hashes were verified against that manifest on 2026-10-21. Use the day's `dayNN.json` and only its mapped `writing-*-v5.json` references; the warm-up has its own file.
- The project keeps original pre-v5 question snapshots in `參考資料/題庫重製前_20261017/`. Older Day 1/2 attempts belong to those original prompts; they are not v5 attempts. Keep first attempts, retakes, and corrections distinct. A prefilled correction field may simply copy the first attempt.
- The study target remains stable 4.0 with 45–60 minutes a day, focused on speaking and writing. Follow actual submitted work rather than calendar dates. Do not add a full four-section mock, light-day variant, or extra reading assignment.

## Local pages and files

All paths below are relative to `<project-root>`.

- `00_使用導覽.md`, `02_目前程度與進度.md`, and `03_練習紀錄.md` establish current state; then read `01_托福衝刺時程.md` and `練習材料/說寫課表.json`.
- Start or reuse both local services with `練習材料/每日口說閱讀/開始每日練習.command`. The daily site uses `127.0.0.1:8768/`, with day pages such as `/day/day02`; writing routes use `/writing?day=day02&task=day2-discussion-v5`. The speaking service uses port 8769.
- The student's v5 writing JSON files live in `作答與評估/寫作課程/`; speaking attempts live under `作答與評估/每日口說閱讀/2026-10-17-v5/`. Preserve old attempt files and original Day 1/2 recordings.
- The teacher source is `.skillshare/skills/toefl-sprint-teacher/`; `.agents/skills/toefl-sprint-teacher/` is the project copy. Read that local copy, the rubric, feedback guide, and only the relevant day/task references. The skill files are outside the answering website.

## Teaching and scoring boundaries

- Do not invent replacement question sets or reveal answers before the first attempt. V5 examples are teacher-created and not ETS-scored exemplars. Essay tasks allow different relevant arguments and wording.
- A suggested word count is not a hard score cutoff. Score email/discussion as an overall integer 0–5 teaching estimate when evidence permits; do not linearly map it to an official 1–6 section score. For Build a Sentence, report matches to the key and discuss reasonable variants separately.
- Focus feedback on task completion, changed meaning, clarity, and consequential grammar. Give at most two priorities, show the learner's own wording and a small correction, then ask for their revision. Polished teacher models do not count as student work or proof of improvement.
- Transcript and signal tools can support content, duration, word-count, speaking-rate, or pause-candidate checks. They do not replace reliable listening. Do not infer pronunciation from ASR, mechanical pause thresholds, or tool metrics; keep a full speaking task score blank if the voice evidence is not reliable.
- Missing work is not zero. Read the answer files before making progress claims, and update the progress record only from actual submissions.

## Verification boundary

The project records local checks for the teacher-bank hashes, page mappings, storage, exports, and responsive layout. Browser checks use isolated test data; they do not count as student practice. The sources were not score-equated to ETS difficulty. Verify cloud synchronization separately when that matters.
