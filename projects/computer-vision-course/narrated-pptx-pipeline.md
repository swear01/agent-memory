---
title: Computer-vision narrated PPTX uses Beamer PDF plus work_azure audio
scope: projects/computer-vision-course
project: computer-vision-course
tool: LaTeX Beamer/Python/PowerPoint
status: needs-review
confidence: high
evidence: >-
  Direct source inspection found Beamer main.tex and the work_azure audio
  directory; the build produced 16 slides, Open XML validation passed, and
  PowerPoint opened 16 slides. Full slideshow playback was not verified.
created: 2026-08-18
updated: 2026-08-18
tags:
  - latex
  - beamer
  - pptx
  - audio
  - python
  - pep668
  - needs-review
source_refs:
  - codex-b001-computer-vision:019e96b3-5dd1-7700-8ee7-19d9632794ae
redaction: passed
generated_by: openai-codex/gpt-5.6-luna
---

# Paths

For the computer-vision presentation project:

- Beamer source: `slide/2026-06-03-frame-gen-slide/main.tex`
- PDF: `slide/2026-06-03-frame-gen.pdf`
- Narration input: `narration_pipeline/work_azure/audio/001.mp3` through `016.mp3`

# Pipeline

The verified pipeline is PDF -> PNG pages -> full-slide images in PPTX -> per-slide audio -> Open XML audio/timing patch. The build reported matching 16 pages, audio files, and slides; Open XML validation and a PowerPoint open test passed.

Homebrew Python rejected direct user installation with PEP 668 `externally-managed-environment`. Use the project-local `build/pptx_venv` for `python-pptx`, `mutagen`, and `lxml`.

Full slideshow playback and MP4 export remain unverified.
