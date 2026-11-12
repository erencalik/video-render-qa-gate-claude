---
name: video-render-qa-gate
description: Operations card for rendering short-form videos on a shared, memory-constrained machine, unsticking a hung render, and running the pre-delivery quality gate (transcript match, -14 LUFS, frozen frames, safe areas, logo). Use when rendering, when a render hangs, and before delivering any video.
---

# Render & QA Gate

Temporary or dated states (who holds the render slot, today's queue) never go on this card; check the coordinator's latest instruction before starting.

## Setup (fill in once)

| Setting | Example |
|---|---|
| `WORKDIR` (all project files) | `D:\Video` or `~/video` |
| `TMPDIR` (inside WORKDIR, safe to wipe) | `D:\Video\_tmp` |
| `PROTECTED` folders never touched | personal folders, other teams' folders |
| `RAM` | 8 GB |
| `BUSY_WINDOWS` when other scheduled jobs run | e.g. emulator or scraping jobs |
| `OUTPUT` / `DELIVERY` | local output folder / shared drive folder per series |

## Boundaries
- All files under `WORKDIR`; don't write to the system drive. Point TEMP at `TMPDIR`.
- Don't touch `PROTECTED`. Ask before deleting anything.
- ONE render at a time on the machine. Get "go" from whoever coordinates the queue before starting; report when done.
- With limited RAM: no emulators or extra heavy apps open during a render. A render that won't finish before a `BUSY_WINDOWS` slot doesn't start.

## Speed
- Use the hardware encoder if available; reuse cached unchanged scenes.
- When a new rule arrives, NO full re-render: only the faulty scene/section. If the owner pointed at a spot, fix only that spot.

## Hang prevention (required)
- In ffmpeg, give `color` and `-loop` sources a `-t` equal to the video length (or `d=D`), give overlays `eof_action=endall`, give the output `-frames:v`.
- In Node/Chromium layers, wrap `browser.close()` in a 10 s timeout followed by `process.exit(0)`.
- Log a frame counter every 10 s. Check in the first minute that it is moving; if node CPU time is flat, it is hung: kill it and limit resources.

## Quality gate (before delivery, one report line)
- Transcript check with Whisper against the script/quote: no missing or half words, no cut-off sentences.
- −14 LUFS, no frozen frames, correct length.
- Text inside the safe area (top 250, bottom 420, right 140 px clear for 9:16), subtitles horizontally centred, no text over faces, faces not cropped.
- Logo colours match the brand kit.
- Plus any series-specific checks (e.g. from `quote-reels-editing-guide`: no sound effects, written closing line, no voice on the closing).
- Report format: `file | length | LUFS | gates: pass/fail (why)`.

## Delivery
- Output to `OUTPUT/<Series>/` and the series folder in `DELIVERY`. Nothing goes to the scheduling tool before the owner approves.
- Send the owner a short "ready" message with the link; don't attach the file to the chat.
