# 6,684 Feet: A Gift, Revisited

**URL:** https://mt-mitchell.up.railway.app

## Overview

A single-page photo/video/essay website documenting ten years of experiences on Mt. Mitchell, NC. Built as a single HTML file with no frameworks or build tools.

---

## Technology Stack

| Component | Detail |
|---|---|
| Hosting | Railway (static site via `npx serve`) |
| Repository | https://github.com/streamrD/mt-mitchell.git |
| Image hosting | Backblaze B2 — bucket: `mtmitchell` |
| Image base URL | `https://s3.us-east-005.backblazeb2.com/mtmitchell/` |
| Videos | YouTube (unlisted), embedded via iframe on click |
| Fonts | Google Fonts — Cormorant Garamond, Spectral, Overpass |
| Analytics | Umami — https://todd-umami.up.railway.app |

---

## Repository Structure

```
mt-mitchell/
├── index.html        ← entire site (HTML/CSS/JS, ~850 lines)
├── sorter.html       ← gallery ordering tool (internal use)
├── serve.json        ← Railway static serve config
├── package.json      ← defines start command for Railway
├── README.md         ← this file
```

---

## index.html Structure (top to bottom)

1. `<head>` — OG/Twitter meta tags, Google Fonts, CSS, Umami script tag
2. `#landing` — full-viewport hero with panorama image (`panorama-2.jpg`)
3. `#prologue` — short introductory text section
4. `#essay-2026` — April 2026 essay with pull quotes
5. `.elevation-divider` — decorative divider
6. `#gallery` — photo/video grid
7. `.interlude` — "Nine years earlier" separator
8. `#essay-2016` — July 2017 essay (warm color scheme)
9. `<footer>` — credit line, author name
10. `#lightbox` — hidden overlay for full-size images and video playback
11. `<script>` — scroll reveal, lightbox logic, gallery click handlers with Umami tracking

---

## CSS Design System

All colors defined as CSS custom properties in `:root`:

- `--haze-dark: #2a3540` — dark blue-gray (landing, footer, video cells)
- `--haze-mid: #7a9aaa` — mid blue-gray (pull quote accents)
- `--page-bg: #f2efeb` — warm off-white (essay backgrounds)
- `--page-bg-warm: #f5f0e8` — slightly warmer (2017 essay)
- `--ink: #1e2a30` — body text
- `--gap: 2px` — gallery grid gap

Fonts:
- `--font-display: 'Cormorant Garamond'` — headings, pull quotes, titles
- `--font-body: 'Spectral'` — essay body text
- `--font-sans: 'Overpass'` — labels, metadata, UI elements

---

## Gallery Grid

CSS Grid, 3 columns, 2px gap. Cell override classes:

- `.wide` — spans 2 columns, 2:1 aspect ratio
- `.full` — spans 3 columns, 3:1 aspect ratio
- `.tall` — spans 2 rows, 1:2 aspect ratio
- `.featured` — spans 2×2, square

**Video cells** (`.video-cell`):
- Always span full width (`grid-column: 1 / -1`)
- 16:9 aspect ratio
- Show YouTube thumbnail from Backblaze (e.g. `clouds-thumb.jpg`)
- On click: open YouTube embed in lightbox
- Thumbnails: `clouds-thumb.jpg`, `pye-thumb.jpg`, `morning-thumb.jpg`, `perception-thumb.jpg`, `tao-thumb.jpg`

**Photo cells:**
- On click: open full-size image in lightbox
- Navigate with arrow keys or ← → buttons
- Click image or background to close

---

## Videos (YouTube, unlisted)

| Title | Date | YouTube ID |
|---|---|---|
| The Clouds | April, 2026 | `nk68MfdAjwA` |
| Joe Pye Weed | August, 2018 | `ufYKXkjgal8` |
| Morning at Maple Camp Bald | June, 2017 | `xfbcg2UW0Go` |
| The Doors of Perception on Middle Creek | October, 2016 | `V1gexqIlWgU` |
| Homage to the Chinese Taoist Poets | August, 2016 | `tevDGK7_cBs` |

---

## Analytics (Umami)

- Script loaded in `<head>` with `defer`
- Website ID: `30432572-8b0f-4704-98fe-e5b2e76e4c19`
- Click tracking uses `window.umami.track(label)` inside the gallery cell click handler
- Labels: `photo: FILENAME.jpg` or `video: Video Title`

---

## Key Workflow Rules

1. **Never copy files from Downloads** — the browser caches downloaded files and the stale version will wipe Backblaze URLs from index.html
2. **Always edit index.html directly** in Terminal using Python's `str.replace()` for targeted changes, or VS Code for larger edits
3. **Always verify after edits:** `grep -c "backblazeb2" index.html` should return **60**
4. **Standard deploy:**
```bash
git add index.html
git commit -m "description"
git push
```
5. **To revert:** `git checkout HEAD -- index.html`
6. **To restore from a specific commit:** `git checkout COMMIT_HASH -- index.html`

---

## Railway Configuration

`serve.json`:
```json
{
  "headers": [{"source": "**", "headers": [
    {"key": "Access-Control-Allow-Origin", "value": "*"},
    {"key": "X-Robots-Tag", "value": "all"}
  ]}],
  "cleanUrls": false,
  "trailingSlash": false
}
```

> **Note:** Railway blocks Facebook's crawler (`facebookexternalhit`) at the network level — OG previews work on all other platforms (iMessage, Slack, LinkedIn, Twitter) but not Facebook.

---

## OG / Social Meta

- **Title:** "A Gift, Revisited"
- **Image:** `https://mtmitchell.s3.us-east-005.backblazeb2.com/og.jpg`
- **URL:** `https://mt-mitchell.up.railway.app`

---

## Sorter Tool

Available at `https://mt-mitchell.up.railway.app/sorter.html`

- Drag-and-drop gallery ordering tool
- All 57 items (52 photos + 5 videos) pre-loaded in current curated order
- Generates gallery HTML code for pasting into index.html
- Has built-in preview mode

---

## Known Issues / Gotchas

- The chat interface mangles CSS selectors like `p.video-title-label` into markdown links — always use Python string replacement or VS Code for edits involving that string
- Backblaze CORS is set to "Share with all HTTPS origins"
- All five video thumbnails confirmed working: `clouds-thumb.jpg`, `pye-thumb.jpg`, `morning-thumb.jpg`, `perception-thumb.jpg`, `tao-thumb.jpg`
