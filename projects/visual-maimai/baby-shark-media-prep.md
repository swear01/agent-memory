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
segment was stable at `115 BPM`, with the first beat phase near `0.10` seconds after
the trimmed audio start. The source video was preserved unchanged; the prepared
folder contains a separate MP3, video-only PV, background PNG, and an editable empty
Master template. No note chart was invented when the destination folder contained no
note data.
