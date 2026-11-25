# How the site works

`index.html` is the entire site — markup, CSS, and JavaScript in one
file, with content pulled in at runtime from `content.json`. There's no
build step for the page itself (the GitHub Actions workflow only injects
`<head>` tags for link previews; see the main README). This doc covers
the mechanisms inside that file, for anyone changing its behaviour
rather than its content.

## 1. The wordmark is a drawn shape, not type

The letters **M A O** are real text, but the operator that appears
between M and A — **+ × − ÷** — is never a font glyph. It's two bars
(`barA`, `barB`) and two dots (`dotT`, `dotB`), positioned with
transforms and driven from JS so one shape can smoothly become the next:

- **+** and **×** are the same two bars at 90° to each other, just
  rotated 45°: going between them is a literal rotation.
- **−** is those two bars rotated until they lie flat on top of each
  other — a *closing* motion, not a swap.
- **÷** is that same flat bar, with the two dots splitting away from its
  centre, above and below.

Because a straight bar looks identical every 180°, each transition picks
whichever equivalent target angle is numerically closest to the bar's
*current* angle, so the rotation always turns the short way rather than
spinning arbitrarily. The very first appearance of **+** (from the
resting wordmark) is a special case: the bars start collapsed to zero
length on a flat line and grow outward as the M–A gap opens, then fold
into the cross — mirroring, in reverse, the ×→− closing motion.

The operator's thickness, length, and spacing are not fixed — they're
computed from the wordmark's own stem width (`stemOf`, measured with a
canvas context against the current font weight) and kept in Swiss-cross
proportion (arms 7:6) to the letters. Changing the wordmark's weight or
size in the CMS rescales the operator with it automatically (`opEm()`).

## 2. One camera, not four pages

There's a single `world` element containing everything; `state` (0–4)
tracks which section is current, and `zoomed` tracks whether the camera
is pushed in on it. There's no routing and no hidden/shown pages in the
usual sense — the whole page is panned and scaled:

- `aimCamera()` computes a target scale/position so the active section's
  panel fills most of the viewport, then animates three springs
  (`C.s`, `C.x`, `C.y` — scale, x, y) toward it.
- While the camera is moving, content is shown *scaled* via CSS
  transform (cheap, but blurry at high zoom).
- Once the camera settles, `commitZoom()` sets `Z` (the "real" zoom
  factor) and re-runs `layout()`, which rebuilds the page's actual CSS
  sizes at that zoom level — so text and lines end up crisp, native-
  resolution, not a magnified bitmap. `uncommitZoom()` reverses this
  seamlessly just before the camera moves again.

`go(to)` changes `state` (wrapping 0–4), swaps the operator shape via
`setSymbol()`, and shows/hides the relevant panel. Clicking the wordmark
while already zoomed out zooms in; clicking while zoomed in backs out to
the resting wordmark state (see the `wm.addEventListener('click', …)`
and `zoomIn`/`zoomOut` wiring near the bottom of the script).

## 3. About: the wordmark as a real-world drawing

The About section (state 3, **−**) draws dimension lines around the
letterforms and labels them with the letters' actual on-screen size.
Getting an honest number requires knowing the device's physical pixel
density, which browsers don't expose directly, so `estimateMmPerPx()`:

1. Checks `localStorage` for a saved calibration value first.
2. Falls back to a heuristic by device: parses `navigator.userAgent`,
   screen dimensions, and `devicePixelRatio` against known values for
   common iPhone/iPad/Android/Mac screens.
3. Offers a manual **calibration** overlay (`openCalib()` /
   `renderCal()`) where the visitor drags a bar to match a real object's
   width (a credit card, by default) against the screen; the result is
   stored (`CAL_KEY`) and reused on later visits.

`layoutAbout()` and `placeDim()`/`labelDim()` then lay out dimension
lines and labels against the real glyph metrics (`glyph()`, cached per
weight/character since canvas text metrics are comparatively slow), and
`relabelAbout()` switches the unit between cm and mm depending on
whether the camera is zoomed in.

## 4. Projects: a wheel

`makeWheel()` builds a vertically scrollable, draggable list (pointer
drag, mouse wheel, and arrow-key support all feed the same spring-driven
position, `s`). `project(f)` maps a fractional index to a screen
position with perspective-like scaling and fade, so entries recede
convincingly as they scroll away from the centre rather than just
sliding linearly.

## 5. Publications: a timeline with no ends

`makeTimeline()` is the same drag/wheel/keyboard-driven pattern as the
wheel, but laid out along a horizontal line positioned by date (year +
month) rather than by list order — entries further back in time sit
further along the line, so the spacing itself carries information.

## 6. Small things worth knowing

- **Reduced motion**: a `prefers-reduced-motion` media query
  (`mqReduce`) is checked live; when it matches, flap/spring animations
  are skipped rather than just shortened.
- **Haptics**: `buzz(ms)` fires a short `navigator.vibrate` pulse on
  state changes and zoom, on devices that support it — a no-op
  elsewhere.
- **Flap animation**: `flap()` drives a brief split-flap-style digit
  flicker (used where numbers appear/change), independent of the main
  spring system; `stopAllFlaps()` cancels all of them at once before a
  zoom commit, so nothing is mid-flip when the layout rebuilds.
- **Fine-pointer hover**: on devices with a precise pointer
  (`mqFine`, i.e. not touch-only), hovering the resting wordmark gives a
  small preview — the gap eases open slightly and the asterisk dims —
  without triggering a real state change.
- **Accessibility**: a visually-hidden live region (`live`) announces
  the current section name on every state or zoom change, and the
  wordmark's `aria-label` updates to describe what clicking it will do
  next.

## Content flow, end to end

```
Pages CMS (pagescms.org)
   → edits content.json, commits to `main`
        → GitHub Actions (.github/workflows/publish.yml)
             → writes <head> tags (title / OG / icon) from content.json
             → deploys index.html + content.json + fonts/ + media/ to GitHub Pages
                  → browser loads index.html, which fetch()-es content.json itself
                       → renders projects, publications, about text, contact details
```

The build step only ever touches the document `<head>`; everything a
visitor actually sees and interacts with is rendered client-side from
`content.json` at load time.
