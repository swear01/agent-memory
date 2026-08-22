---
title: Template-based PowerPoint editing and visual QA workflow
scope: domains/other
tool: PowerPoint, Synopsys presentation template, artifact-tool
status: active
confidence: high
created: 2026-08-22
updated: 2026-08-22
tags:
  - powerpoint
  - pptx
  - presentation
  - template
  - visual-qa
  - speaker-notes
---

# Template-based PowerPoint editing

## Content and layout

- Start from the supplied template or a template-derived starter deck. Keep the template-owned legal/confidential page, footer, purple visual system, and master framing intact.
- Keep visible slide text short. Put the complete personal review, context, caveats, and source notes in speaker notes.
- Use one language for visible descriptions unless a proper name or template-owned text requires otherwise. Do not leave mixed-language placeholder bullets in the finished deck.
- Treat speech-to-text as a draft: verify store names, Chinese characters, addresses, landmarks, and dates against authoritative or public sources before placing them on slides.
- Separate location labels from product descriptors. A neighborhood or landmark and a food characteristic should be independent bullets when both are useful.

## Image selection

- Classify every image before use: food, storefront, interior, courtyard, or other environment. Match the image class to the slide's purpose.
- A food-review slide should use a same-visit food photo when available; an environment thumbnail should show the storefront, interior, courtyard, or seating area. Do not use an empty bowl or a post-meal photo as an environment image.
- When the user requests a specific existing photo, inspect multiple candidates visually and preserve the requested main image if the user says not to change it.
- If a personal photo is unavailable, a visually inspected public business photo can fill the gap, but record its source and disclose visible watermarks or attribution concerns.
- For small context thumbnails, enlarge them enough to communicate the scene; two environment thumbnails around 260 × 160 on a 16:9 slide fit below a short three-bullet body while staying above the footer.

## Verification

- Re-export the PPTX after every material edit, render all slides to images, run the slide overflow test, run the template-fidelity check, and visually inspect the slides with changed images or text.
- The bundled presentation helpers require the runtime environment variables for Node, Node modules, Python, and override binaries. A missing `RUNTIME_NODE_MODULES` or `RUNTIME_NODE` causes helper scripts to fail before the deck is checked.
- A passing automated test is not enough for image correctness: visual inspection caught the difference between an environment photo and a food/post-meal photo in this workflow.

