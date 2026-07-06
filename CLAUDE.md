# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Working file

All work happens in `HTMLs/index.html`. No build step — everything (CSS, JS, SVG) is inline in that single file.

## Preview server

The preview MCP server is named `"xray"`, runs on port 8123, and serves from the project root directory. After any edit to `index.html`, sync it before verifying:

```bash
cp "HTMLs/index.html" /tmp/xray_index.html
```

Static assets (PDFs, images) are referenced relative to `HTMLs/index.html` in the actual file (e.g. `../Content/bagels_on_sunday.pdf`), but the preview server resolves them from the project root (e.g. `http://localhost:8123/Content/rooftop_boston.jpg`). Copy assets to `/tmp/Content/` if you need them to load in the preview.

## Architecture

### Visual layer

An SVG skeleton ("x-ray man") floats in the left ~18% of the screen. It is wrapped in `#organ-stage` and shifts rightward (`left: 18%`) when a zone is active. Three clickable organ SVGs — `#organ-brain`, `#organ-heart`, `#organ-meditation` (spirit) — are hidden by default and shown via `body.zone-<name>` CSS classes.

**Critical CSS specificity rule:** `#organ-stage svg { display: none; }` hides all SVGs inside the stage (specificity 1,0,1). Zone-specific show rules use `body.zone-brain #organ-brain` (1,1,1). The rotating text ring (`#organ-ring`) lives *outside* `#organ-stage` — moving it inside would break `position: fixed` because transformed parents create a new stacking context.

### Zone system

Three zones cycle in scroll order (bottom-to-top of the body): `spirit → heart → brain`.

```js
const ZONES = ['spirit', 'heart', 'brain'];
let zoneIndex = 2; // starts on brain
```

Zone changes are driven by a `wheel` event listener on `window` — `e.preventDefault()` is only called when actively navigating (cover dismissed, panel closed). The `#content-panel` has `pointer-events: none` when closed to prevent it from intercepting wheel events despite being `position: fixed; inset: 0`.

### Cover page

`#cover` is a full-screen overlay (`position: fixed; inset: 0`). `coverVisible` (boolean in JS closure) tracks state. Click to dismiss; click the name tag (`#name-tag`) to return.

### Content panel

`#content-panel` slides up from `translateY(100%)` to `translateY(0)` (class `open`). It contains a rolodex-style card system:

- Cards are built by `buildCard(item)` using the `CONTENT` object keyed by zone name.
- `ZONE_ICONS` maps zone name → inline SVG for the card icon bullet.
- Cards support: `type`, `title`, `date`, `note` (HTML allowed), `pdf` (click → open file), `link` (click text area → open URL), `img` (displays full-width at bottom of card; click → fullscreen lightbox).
- When a card has both `link` and `img`, clicking the text opens the link; clicking the image opens the lightbox (`#img-lightbox`).
- Cards get `max-height: 80vh; overflow-y: auto` for long content.

### Lightbox

`#img-lightbox` is `position: fixed; inset: 0; z-index: 9999`. It must be in the DOM *before* the `<script>` tag — placing it after causes a null-reference crash that silently kills all event listeners.

### Content data

```js
const CONTENT = {
  brain: [ /* items */ ],
  heart: [ /* items */ ],
  spirit: [ /* items */ ],
};
```

Each item: `{ type, title, date?, note, pdf?, link?, img? }`.

## Reference files (do not delete)

- `HTMLs/xray.html` — prior iteration; useful reference for skeleton proportions and animation values.
- `Content/bagels_on_sunday.pdf` — linked from the Heart entry.
- `Content/rooftop_boston.jpg` — linked from the Spirit entry (converted from HEIC via `sips`).
- `SVGs/` — source SVG assets; organ icons were extracted from these and inlined.
