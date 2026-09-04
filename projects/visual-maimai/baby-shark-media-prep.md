---
title: Majdata song-folder media and timing preparation
scope: projects/visual-maimai
project: Visual Maimai
machine: Windows
status: active
confidence: high
evidence: Verified against existing MaiCharts folders, MajdataEdit source, and a Baby Shark media preparation run on 2026-09-04
created: 2026-09-04
updated: 2026-09-04
tags:
  - visual-maimai
  - majdata
  - maidata
  - audio-video
  - bpm
---

# Verified folder contract

Song folders use `maidata.txt` for chart metadata, `track.mp3` for gameplay audio,
`bg.png` (or another supported image extension) for the jacket/background, and
optional `pv.mp4` for the background video. The existing library uses a video-only
`pv.mp4` when `track.mp3` is supplied separately.

# Timing contract

`MajdataEdit` initializes chart time from `&first` in seconds, so the first timing
row and any first note are measured from that value. Keep an explicit inline BPM at
the beginning of each `&inote_*` section, for example `(115)`, even when
`&wholebpm=115` is also present.

# Baby Shark result

For the supplied Pinkfong video, visual inspection and audio segmentation identified
the main song as source time `10.02–105.48` seconds. Beat-period analysis of that
segment was stable at `115 BPM`, with the first stable beat phase near `0.22` seconds after
the trimmed audio start. An earlier coarse analysis reported `0.10`; that value was
wrong because it locked onto an intro transient rather than the sustained beat grid.
The source video was preserved unchanged; the prepared
folder contains a separate MP3, video-only PV, background PNG, and an editable empty
Master template. No note chart was invented when the destination folder contained no
note data.

# Visual Maimai import contract

Visual Maimai's Open flow must select `track.mp3` or `track.ogg`; it auto-imports
`maidata.txt` and `pv.mp4` only when those files are beside the selected audio. The
separate File > Import maidata command is the reliable route for an existing chart.
Opening the media as a new chart can create a blank `chart.json` with default offset
and BPM instead of importing the maidata. After import, verify the UI values before
saving: Baby Shark should show offset `0.22` and BPM `115`; the prepared template is
intentionally empty of note tokens, so an empty note list alone is not proof that
import failed. However, the standalone parser rejects a completely empty chart
with `max() iterable argument is empty`; a minimal importable starter must contain
at least one legal Note. The repaired Baby Shark template therefore uses one
placeholder `1` at the first beat followed by `,,,E`; the placeholder can be
deleted when real charting begins.
