# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Working file

All work happens in `HTMLs/index.html`. No build step — everything (CSS, JS, SVG) is inline in that single file.

## Preview server

The preview MCP server is named `"xray"`, runs on port 8123, serving `/tmp/xray_index.html`. After any edit to `index.html`, sync before verifying:

```bash
cp "HTMLs/index.html" /tmp/xray_index.html
```

Navigate to `http://localhost:8123/xray_index.html` in the browser pane. The cover page must be dismissed before the main UI is interactive — use `document.getElementById('cover-enter').click()` via `javascript_tool`.

Static assets are referenced relative to `HTMLs/index.html` (e.g. `../Content/bagels_on_sunday.pdf`), but the preview server resolves from `/tmp/` root. Copy assets to `/tmp/Content/` if needed for preview.

## Architecture

### Peace sign navigation

`#peace-stage` (`position: fixed; inset: 0; z-index: 10`) centers `#peace-ring`, a square div containing the entire interactive UI. The ring holds:

- **`#peace-svg`** — SVG with 3 sector `<path>` elements (`#sector-brain`, `#sector-heart`, `#sector-spirit`) that are the hover/click targets. The circle is split into 3 equal 120° sectors. Lines go from center to the top and diagonally to lower-left/lower-right.
- **`.peace-icon`** SVGs — brain, heart, meditation figure — absolutely positioned within the ring at their sector centroids.
- **`#man-figure`** — the character SVG, positioned at `top:50%; left:50%` (the intersection of all peace sign lines). The `figureFloat` keyframe animation must include `translate(-50%, -50%)` in both keyframes or the centering breaks.
- **`.zone-side-panel`** (`#zone-panel-left`, `#zone-panel-right`) — absolutely positioned text panels that appear left/right of the ring on hover. Positioned with `calc(50% + min(28vw, 36vh) + 28px)`.

### Pointer events architecture

`#peace-stage` and `#peace-ring` have `pointer-events: none`. `#peace-svg` has `pointer-events: all`. `#peace-circle` and `.peace-line` have `pointer-events: none` (critical — without this they intercept mouse events intended for the sector paths beneath them). `.peace-icon` and `.peace-label` elements have `pointer-events: none`.

### Hover system

`setHover(zone)` sets `hover-<zone>` class on **both** `#peace-ring` (for CSS rules targeting child elements like icons) and `document.body` (for CSS rules targeting the side text panels). All hover classes must be removed from both when clearing. The `HOVER_CLASSES` constant holds the full list.

### Cover page

`#cover` (`z-index: 500`) is full-screen. `coverVisible` (boolean in JS closure) gates all interaction — sectors' `mouseenter` and `click` handlers return early when `coverVisible` is true. `hideCover()` sets `coverVisible = false` and uses `classList.remove('cover-visible')` in the setTimeout — **do not** use `document.body.className = ''` as it wipes hover classes.

### Content panel

`#content-panel` slides up from `translateY(100%)` to `translateY(0)` via class `open`. Clicking a sector calls `openPanel(zone)` directly (no intermediate definition panel). Cards are built by `buildCard(item)` from `CONTENT[zone]`.

Card fields: `{ type, title, date?, note (HTML ok), pdf?, link?, img? }`. PDF cards use pdf.js (loaded from cdnjs). Lightbox (`#img-lightbox`, `z-index: 9999`) must appear in DOM **before** the `<script>` tag.

### About panel

`#about-panel` (`z-index: 620`) slides in from `translateY(100%)`. Triggered by the corner "ABOUT" link, not from the peace sign sectors.

### z-index stack

| Element | z-index |
|---|---|
| `#peace-stage` | 10 |
| `#about-panel` | 620 |
| `#cover` | 500 |
| `#content-panel` | 400 |
| `#img-lightbox` | 9999 |

### Content data

```js
const CONTENT = {
  brain:  [ /* items */ ],
  heart:  [ /* items */ ],
  spirit: [ /* items */ ],
};
```

### Zone colors

- Brain: pink `rgba(234, 89, 110, …)`
- Heart: blue `rgba(100, 160, 230, …)`
- Spirit: green `rgba(24, 179, 94, …)`

## Reference files (do not delete)

- `HTMLs/xray.html` — prior iteration; useful for skeleton proportions and animation values.
- `Content/bagels_on_sunday.pdf` — linked from a Heart card.
- `Content/rooftop_boston.jpg` — linked from a Spirit card (converted from HEIC via `sips`).
- `SVGs/Profile SVG.svg` — source for the about/profile icon.
