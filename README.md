# 6,684 Feet: A Gift, Revisited

**URL:** https://mt-mitchell.up.railway.app

## Overview

A single-page photo/video/essay website documenting ten years of experiences on Mt. Mitchell, NC. Built as a single HTML file with no frameworks or build tools.

There are **two** things in this repo, and forgetting the second one causes
real damage:

1. **`index.html`** — the site itself. One file, everything in it.
2. **`sorter.html`** — a drag-and-drop tool for reordering the gallery, which
   *generates the gallery HTML you paste back into `index.html`*. See
   **[Sorter Tool](#sorter-tool)** — read it before touching the gallery.

Photos are not in this repo. They live in a Backblaze B2 bucket, and the page
reads downscaled **[Image Tiers](#image-tiers)**, never the full-resolution
originals.

---

## Technology Stack

| Component | Detail |
|---|---|
| Hosting | Railway (static site via `npx serve`) |
| Repository | https://github.com/streamrD/mt-mitchell.git |
| Image hosting | Backblaze B2 — bucket: `mtmitchell` |
| Image base URL | `https://s3.us-east-005.backblazeb2.com/mtmitchell/` |
| Image tiers | `display/1600/` (grid), `display/2800/` (enlargements) — see **Image Tiers** below |
| Videos | YouTube (unlisted), embedded via iframe on click |
| Fonts | Google Fonts — Cormorant Garamond, Spectral, Overpass |
| Analytics | Umami — https://todd-umami.up.railway.app |

---

## Repository Structure

```
mt-mitchell/
├── index.html        ← entire site (HTML/CSS/JS, ~740 lines)
├── sorter.html       ← gallery ordering tool — GENERATES index.html's gallery
│                       markup. Must stay in sync with index.html. See below.
├── serve.json        ← Railway static serve config
├── package.json      ← defines start command for Railway
├── README.md         ← this file
```

Local `*.jpg` / `*.JPG` / `*.jpeg` / `*.HEIC` files are the photo originals.
They are **gitignored** — the repo has never contained them. They exist on the
Mac and in the B2 bucket, and nowhere else, so B2 is the only offsite copy.

---

## Sorter Tool

> **This is easy to forget exists, and that is exactly how the gallery gets
> damaged.** It is not a viewer — it *writes* the markup `index.html` uses.

**Local:** `http://localhost:PORT/sorter.html` · **Live:** https://mt-mitchell.up.railway.app/sorter.html

A drag-and-drop tool for reordering the gallery. All 60 items (55 photos +
5 videos) are pre-loaded in the current curated order, with a built-in preview
mode. You rearrange, click generate, and paste the result into `index.html`.

It reads `display/1600/` for its own thumbnails and `display/2800/` in preview —
never the bucket-root originals.

### The rule

**`generateCode()` must emit markup identical to what `index.html` ships.** It
currently produces:

- `src` on the tier implied by the override class — `wide` / `full` /
  `featured` → 2800, everything else → 1600
- `data-full` on 2800 for every photo
- Backblaze video posters, *not* YouTube thumbnails

If you change how images are referenced in `index.html`, change
`generateCode()` in the same commit. Otherwise the next paste silently reverts
it, and nothing will error — the page just quietly gets worse.

### Why the warning is this strong

As of Aug 2026 the sorter had drifted from the gallery in three ways, none of
which would have thrown an error:

| Drift | What the next paste would have done |
|---|---|
| List missing `waterfall.jpg`, `middle-creek-rapid.jpg`, `mtmitchell-from-mtfarm.jpg` | **Dropped three live photos** off the site |
| `generateCode()` emitted bucket-root originals | Reverted the image tiers — back to 203.5 MB |
| Video posters used YouTube's 320×180 `mqdefault.jpg` | Undone `c2c0970`, downgrading all five posters |

All three are fixed. The item list is now rebuilt from `index.html`'s live
order, and its output is verified to match the live gallery cell-for-cell. If
you ever suspect drift, the check is to generate code and diff it against the
`<div class="gallery-grid">` blocks in `index.html` — they should be identical.

---

## Performance

The page ships downscaled derivatives instead of full-resolution originals.
Current cost of a visit:

| | Before (Aug 2026) | Now | |
|---|---|---|---|
| Scroll the entire gallery | 203.5 MB | **31.6 MB** | −84.5% |
| Landing hero (LCP element) | 3.68 MB | **295 KB** | −92% |
| Open one photo | free — already downloaded | **1.21 MB** avg | prefetched on hover |
| Scroll everything + open 20 photos | 203.5 MB | **55.8 MB** | −73% |
| Bucket objects / size | 77 / 274.6 MB | 188 / 341.0 MB | derivatives added |

The originals were being loaded straight into grid cells a few hundred pixels
wide — `IMG_6968.JPG` is 11.2 MB at 5712×4284, rendered into roughly 480 px.
The hero was 8981 px wide for a `100vh` `object-fit: cover` backdrop.

The 31.6 MB also arrives *first*, with enlargements fetched only for photos
someone actually shows interest in — so time-to-usable-page improved far more
than the totals alone suggest.

Trade-off worth knowing: previously the grid image *was* the enlargement, so
clicking was free. Now a click needs a different file. Three mechanisms hide
that — see **Latency** under [Image Tiers](#image-tiers).

---

## index.html Structure (top to bottom)

1. `<head>` — OG/Twitter meta tags, Google Fonts, CSS, Umami script tag
2. `#landing` — full-viewport hero with panorama image (`panorama-2.jpg`)
3. `#prologue` — short introductory text section
4. `#essay-2026` — April 2026 essay with pull quotes
5. `.elevation-divider` — decorative divider
6. `#gallery` — photo/video grid
7. `.interlude` — decorative dots separator
8. `#essay-2016` — July 2017 essay (warm color scheme), with "A Gift" title before the original essay text
9. `<footer>` — credit line, author name
10. `#lightbox` — hidden overlay for enlargements and video playback
11. `<script>` — scroll reveal, lightbox logic (paints the cached grid copy instantly, then swaps in the enlargement), enlargement prefetching, gallery click handlers with Umami tracking

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
- On click: open the enlargement (`display/2800/`) in the lightbox
- Navigate with arrow keys or ← → buttons
- Click image or background to close

---

## Image Tiers

The bucket root holds the **untouched original** of every photo. Nothing on the
page links to those any more — the page reads from two generated tiers instead:

| Tier | Used by | Files | Size |
|---|---|---|---|
| `display/1600/` | the 52 one-third-width grid cells, plus all 55 photos for the sorter's thumbnails | 55 | 27.9 MB |
| `display/2800/` | all 55 enlargements, the 8 full-bleed grid cells (5 video posters + 2 `.full` + 1 `.wide`), and the landing panorama | 61 | 69.0 MB |

Both tiers are JPEG quality 82, progressive, generated with Pillow/Lanczos.
Filenames and capitalization match the original exactly, so a tier URL is just
the base URL plus a path prefix.

The 1600 tier carries three photos the *page* does not use at that size —
`35527968665_2af45f4333_o.JPG`, `IMG_6360.jpeg`, `IMG_7681.JPG` are full-bleed
cells that render from 2800. They exist at 1600 so the sorter has a small
thumbnail for every photo.

**Why:** scrolling the whole gallery used to pull 203.5 MB of full-resolution
originals into cells a few hundred pixels wide. It now pulls 31.6 MB — 84.5%
less. Ten older Flickr-sourced images are narrower than 2800px and were not
upscaled; they sit at their native width in the 2800 tier.

**Markup.** Each photo cell carries both tiers:

```html
<img src=".../display/1600/IMG_7681.JPG"
     data-full=".../display/2800/IMG_7681.JPG"
     alt="Mt. Mitchell" loading="lazy" onload="this.classList.add('loaded')">
```

Video posters have **no** `data-full` — they open YouTube, not an image.

**Latency.** Splitting the tiers means a click could have waited on a download,
so three things prevent that:

1. The lightbox paints the already-cached `src` (grid copy) immediately, then
   swaps in `data-full` once it loads. It never waits on the network to show
   something.
2. `pointerenter` / `touchstart` on a cell prefetches its enlargement — hover
   typically lands 200–500 ms before the click.
3. While the lightbox is open, neighbours `i±1` are prefetched so arrow-key
   navigation is instant.

Prefetches are deduped through a module-level `Set`. Nothing is prefetched
speculatively on load — only in response to actual intent.

### Regenerating tiers after adding a photo

A new photo needs both tiers before it can be referenced, or it 404s:

```bash
# 1600 (grid) and 2800 (enlargement) — omit the 1600 for a full-bleed cell
sips -Z 1600 -s format jpeg -s formatOptions 82 NEW.jpg --out /tmp/1600/NEW.jpg
sips -Z 2800 -s format jpeg -s formatOptions 82 NEW.jpg --out /tmp/2800/NEW.jpg
b2 file upload mtmitchell /tmp/1600/NEW.jpg display/1600/NEW.jpg
b2 file upload mtmitchell /tmp/2800/NEW.jpg display/2800/NEW.jpg
```

Upload the original to the bucket root too, so the archive stays complete.

> `sips -Z` fits the **longest** edge, so for a portrait photo it yields less
> than 1600/2800 px of width. The originals were generated with Pillow fitting
> the **width**; use Pillow if that difference matters for a given image.

**EXIF orientation — read before regenerating with Pillow.** Seven of the
iPhone originals store portrait photos as landscape pixels plus an EXIF
Orientation tag, and browsers apply that tag when rendering. Pillow does *not*:
`Image.open()` gives you the raw, unrotated pixels, and saving drops the tag —
so a naive resize yields a derivative that is silently rotated 90° or 180°.

Always transpose first, which bakes the rotation into the pixels and removes the
tag, so nothing downstream has to interpret it:

```python
from PIL import Image, ImageOps
im = ImageOps.exif_transpose(Image.open(src)).convert("RGB")   # <- required
```

Note the dimensions swap for the six `Orientation=6` images: a 4032×3024 source
is really a 3024×4032 portrait, so fitting to width 1600 gives 1600×2133, not
1600×1200. Affected files:

```
122DE275-...2022-10-23_10-48-28_459.jpeg   (180°)
IMG_6259.jpeg  IMG_7002.JPG  IMG_7087.JPG
IMG_7329.JPG   IMG_7548.JPG  IMG_7734.JPG   (90° CW)
```

`sips` preserves the tag rather than dropping it, so the `sips` recipe above is
not affected — this is a Pillow-specific trap.

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
3. **Always verify after edits:** `grep -c "backblazeb2" index.html` should return **63**
   > Still 63 after the image-tier change. `grep -c` counts matching *lines*, not matches, and each photo cell's `src` and `data-full` live on the same line. The occurrence count is now 118 (`grep -o backblazeb2 index.html | wc -l`) — use the line count, as documented, since that is the number that stayed stable.
4. **Adding a photo requires generating both tiers first** — see *Regenerating tiers after adding a photo*. Referencing a bucket-root original directly works but ships a multi-megabyte file into a small grid cell, which is exactly what the tiers exist to prevent.
5. **Standard deploy:**
```bash
git add index.html
git commit -m "description"
git push
```
6. **To revert:** `git checkout HEAD -- index.html`
7. **To restore from a specific commit:** `git checkout COMMIT_HASH -- index.html`

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

## Known Issues / Gotchas

- The chat interface mangles CSS selectors like `p.video-title-label` into markdown links — always use Python string replacement or VS Code for edits involving that string
- Backblaze CORS is set to "Share with all HTTPS origins"
- All five video thumbnails confirmed working: `clouds-thumb.jpg`, `pye-thumb.jpg`, `morning-thumb.jpg`, `perception-thumb.jpg`, `tao-thumb.jpg`
- **EXIF orientation** breaks Pillow-generated derivatives — see the warning under *Regenerating tiers*. Seven originals are affected. Symptom: correct on disk and in Preview, rotated 90° or 180° on the page
- **The sorter can silently damage the gallery** if it drifts from `index.html` — see [Sorter Tool](#sorter-tool). It fails quietly, never loudly
- `railway.json` and `package.json` specify different start commands. Production demonstrably serves `serve.json`'s headers (`x-robots-tag: all`, `access-control-allow-origin: *`), so `package.json` is winning — but the two disagree, and a Railway config change could flip which one applies
- Browser caching hides derivative fixes: regenerated images keep the same URL, so a hard refresh (Cmd+Shift+R) is required to see corrections
- Serving the repo locally, `localhost:PORT/` shows a directory listing rather than the site — use `localhost:PORT/index.html`. Production root is unaffected

---

## Changelog

### Aug 7, 2026 — image tiers, prefetching, sorter resync

**`cb7de57` — serve resized display tiers, prefetch enlargements**

Moved every image on the page onto the two generated tiers and added the
latency machinery. Gallery scroll 203.5 MB → 31.6 MB, hero 3.68 MB → 295 KB.

Also fixed a bug this change would otherwise have introduced: the lightbox
found its index by comparing `item.src` to `img.src`, which are different tiers
now — every click would have silently opened the *first* photo.

**`dc794fb` — resync sorter with index.html, put it on the display tiers**

The sorter had drifted three ways, each of which would have damaged the site on
the next paste. See [Sorter Tool](#sorter-tool). Its item list is now rebuilt
from `index.html`'s live order, and its generated markup is verified to match
the live gallery cell-for-cell.

**`cf6f67a` — document the EXIF orientation trap**

Seven photos rendered rotated after the first tier generation, because Pillow
ignores the EXIF Orientation tag and dropped it on save. Six were also the wrong
aspect ratio — portrait sources treated as landscape. All 14 affected
derivatives were regenerated with `ImageOps.exif_transpose` and re-verified;
the trap is documented under *Regenerating tiers*.

**Housekeeping (no commit — bucket only)**

- Deleted 5 stale `*-thumb.png` video posters, superseded by `.jpg` in
  `58fc44e`. Bucket 193 → 188 objects, 29.0 MB freed.
- Pulled `IMG_7661.jpg` and `IMG_7746.JPG` down to the Mac. They had existed
  **only** in B2 — no local copy anywhere. Both verified byte-identical.

**Still true and worth watching:** the bucket-root originals remain the only
offsite copy of every photo. `.gitignore` excludes them, so git will never
protect them.
