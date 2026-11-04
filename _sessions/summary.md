# The Future from Memory to video: session summary

6 October 2026, working with Claude Code, with 4K versions rendered on 3 November 2026 (see [4K versions](#4k-versions-november-2026)). The full conversations are in [transcript.md](transcript.md).

## Aim

The Future from Memory (Emma Kay, produced by e-2, Chisenhale Gallery, 2003) was five Flash movies on a CD, played full-screen from projector programs. Flash is gone, so Pete asked whether the piece could be turned into video. It was always a projection rather than a website, so the video is now the piece; there is no site to port.

## What the files are

- **The CD version** (`ffm-CD/`, Flash 6, February 2003) is the final one. The root and `backup*` folders hold earlier Flash 4/5 builds with slightly different frame counts (one labelled "no tints").
- **Five segments**, 800×600 at 25 frames a second, each a fixed timeline of zooming text paragraphs with no sound and nothing interactive. The only scripts go full-screen, hide the mouse, stop each paragraph clip at its end, and `loadMovie` the next segment; segment 5 loads segment 1, so the piece loops.
- **Each segment hands over early**, before its last frame. Segment 4's last ~1,900 frames (about 76 seconds) never played. The video stops each segment where the CD version did:

  | Segment | Frames played | Length |
  |---|---|---|
  | 1 | 13,789 | 9m 11.6s |
  | 2 | 11,112 | 7m 24.5s |
  | 3 | 11,904 | 7m 56.2s |
  | 4 | 13,387 | 8m 55.5s |
  | 5 | 14,561 | 9m 42.4s |
  | **Whole piece** | **64,753** | **43m 10s** |

- **The text** is vector outlines (rendered from 3D with Swift 3D), so no fonts are needed. 269 paragraphs under 30 headings; paragraphs are animated over 14–16.6 seconds, headings over 4.4–9.7.

## How it was rendered

- **Ruffle**, the open-source Flash emulator, built from source (Rust installed for it). Its `exporter` plays a SWF and saves every frame as a PNG at any scale, with no screen recording, so no frames are dropped.
- **A fix to the exporter.** It kept every frame in memory until the end, about 85 GB for one segment, which brought the machine to a standstill. Claude changed `exporter/src/lib.rs` (in `/mnt/data/home/src/ruffle`) to write each frame as soon as it is rendered; memory stays around 117 MB.
- **Settings:** 1.8× scale (1440×1080), at the lowest CPU and disk priority, writing to `/mnt/data` rather than `/tmp` (which is in RAM). About 100 frames a second: the whole piece in about 10½ minutes.
- **Encoding:** H.264 (x264, preset slow, CRF 16, tuned for animation), 25 frames a second. The segments are joined without re-encoding; each starts and ends on a blank white frame, so the joins don't show.
- **Widescreen:** the 16:9 version (1440×810) removes the top 270 pixels. The darkest pixel in that band was measured in all 64,753 frames: it is pure white in every one, so no text is lost. The highest text comes about 55 pixels below the cut.
- `ffm-render/render_segments.sh` is the script that rendered segments 2–5 and checked the band.

## Output

- `futurefrommemory_full_1440x1080.mp4` and `futurefrommemory_full_1440x810_16x9.mp4`: 43m 10s, 1.08 GB each, plus each segment in both shapes.
- The videos and posters were uploaded to `https://media.bjorsq.net/ffm/`.

## The preview page (`redux/`)

`redux/index.html` shows the full piece and the five segments, each in both shapes side by side, with a poster frame and an information panel, styled like The Story of Art's page. It was made so that Pete's colleague and Emma could look through the videos, and it's published by a GitHub Actions workflow. The video now also has its own page on the e-2 site (with a "how it was made" page), so much of `redux/` may no longer be needed; Pete is keeping it for now.

- **Posters:** for each video, the frame with the most text where a whole paragraph is inside the 16:9 area and above the player's controls. The full piece and segment 1 use the opening paragraph ("During the twenty-first century the human lifespan…").
- At first a switch swapped between 4:3 and 16:9; Pete asked for every video on the page instead, so the page now has no JavaScript.

## 4K versions (November 2026)

Rendered for Emma, in case she wants them for a large-scale projection; only the full piece was needed. This was done in a Claude Code session that also rendered The Story of Art at 4K (see the `storyofart` repository) and worked on other e-2 projects.

- **Rendering:** Ruffle at 4.8× scale (3840×2880), the same top crop for UHD (3840×2160, `crop=3840:2160:0:720`).
- **DCI (4096×2160):** cropping the 16:9 picture to DCI height would cut text: it reaches into the bottom band in about half the frames, and into the top band in 66. So the UHD picture is padded with 128 pixels of white on each side instead; the background is white, so the padding doesn't show.
- **Files**, in `/mnt/scratch/4k/`, not in the repository:

  | File | Picture | Size |
  |---|---|---|
  | `futurefrommemory_3840x2160_uhd.mp4` | 3840×2160 (UHD) | 2.7 GB |
  | `futurefrommemory_4096x2160_dci.mp4` | 4096×2160 (DCI 4K) | 2.7 GB |

  Both are 64,753 frames, 43m 10s, H.264 (CRF 16, tuned for animation).
- **Memory:** a first attempt, running alongside The Story of Art's 4K render with no limit, ran the machine out of memory. The renders were re-run one piece at a time under a 20 GB memory cap (`systemd-run --user -p MemoryMax=20G -p MemorySwapMax=0`), from `/mnt/scratch/4k/run-4k.sh`.

## Status

- `ffm-render/` (11 GB: the rendered segments, frames, poster scripts and logs) is in the repository folder but untracked.
- The Rust toolchain (`~/.cargo`, `~/.rustup`) and the patched Ruffle checkout (`/mnt/data/home/src/ruffle`) are still installed.
- The preview page is published from `redux/` by an Actions workflow (`.github/workflows/static.yml`), not from `docs/` like the other e-2-org repositories. As the piece is a video, and `redux/` only holds previews, this was left as it is.
