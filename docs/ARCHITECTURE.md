# How the site works

`index.html` is the whole site: markup, CSS and JavaScript in one file,
with no framework and no build step for the page itself. Content comes
from `content.json` at load time. The publish workflow only rewrites the
`<head>`; see the README. This document is for anyone changing the
page's behaviour rather than its content. The reasons behind the choices
are in [`DESIGN.md`](DESIGN.md).

## 0. Boot

`boot()` loads `content.json` and the fonts in parallel. If the fetch
fails, for example when the file is opened from disk, it falls back to
the `FALLBACK` object in the script. `normalize()` then cleans the data:
it trims strings, keeps at most two offices, pads one-digit project
numbers (`7` → `07`), sorts publications newest first and numbers them.

Next it applies the **Site** settings. The wordmark weight goes into
`--wmw` (that weight is loaded before anything is measured), the size
into `WM.size`, and the tab title and icon are set at runtime too, so
they change even before the next publish. Then it builds the four panels
and runs `layout()`. Every measurement waits for the real font: measuring
the fallback font would put every alignment in the wrong place.

## 1. Motion: one spring engine

Everything that moves is a `Spring`, a damped mass-spring integrated at
240 Hz inside a single `requestAnimationFrame` loop. A spring that
changes target mid-flight keeps its velocity, so every animation can be
interrupted, reversed or retargeted without a jump. The frame step is
capped at 1/30 s, so a tab switch or a hiccup never produces a leap.
Delayed steps use `timerGroup()` and are cancelled as a group, so fast
clicking never stacks stale callbacks.

The presets in `P` are the feel of the site:

| Preset | k / c | Used for |
|---|---|---|
| `press` | 900 / 48 | the wordmark sinking under the pointer |
| `release` | 520 / 17 | springing back, with a small overshoot (the "click") |
| `bouncy` | 330 / 19 | operator rotations, selections |
| `soft` | 190 / 25 | panels appearing, rig placement |
| `follow` | 1600 / 80 | tracking a finger or a drag 1:1 |
| `exit` | 800 / 56 | things leaving quickly |
| `cam` | 135 / 23.2 | the camera, critically damped so it never wobbles |

Only `transform` and `opacity` are animated. Layout is never animated: an
earlier version animated `width` and stuttered.

## 2. The wordmark and the operator

**Geometry is computed, not read back.** `layout()` places the letters
from the font's own advance widths, measured by `glyph()` at 1000 px and
then scaled, with letter-spacing −0.035 em. The baseline comes from the
font metrics. Two reasons: `offsetLeft` rounds to whole pixels, and
canvas metrics at small sizes are rounded too. Computed positions are
identical at every zoom level, so nothing drifts. All final positions
are computed for the **open** wordmark before anything animates, so a
panel never aims at a moving target.

**Opening the gap.** The spring `S.open` (0 → 1) shifts M left and A + O
right by half the slot each, using transforms only. The wordmark stays
centred and the operator's centre never moves.

**The operator** is `barA`, `barB`, `dotT` and `dotB`:

- **+** is bars at 0° and 90°. **×** is 45° and 135°: a literal rotation.
- **−** is both bars at 0°: × *closes* onto itself.
- **÷** is the flat bar with the two dots springing out above and below.
- A bar looks the same every 180°, so `nearest()` always picks the target
  angle closest to the current one, and rotations take the short way.
- From the resting wordmark, the line first grows from zero length while
  the gap opens, then folds into the sign 130 ms later.

**Proportions** (`opEm()`): Black (900) is the reference. The bar is
0.17 em thick, which is 0.863 × the measured stem, and 0.567 em long,
which follows the Swiss cross (arms 1/6 longer than wide). The slot is
0.70 em, the dots 0.19 em, the dot offset 0.255 em. For Bold or Extra
Bold, everything scales by the ratio of that weight's stem to Black's
(`stemOf()`), so a lighter MAO gets a proportionally lighter operator.
All bars have one thickness. Weight-dependent thickness, matching the
font's thick verticals and thin horizontals, was tried and dropped (see
DESIGN.md).

**The asterisk** sits with its ink top exactly on the cap line, computed
from the glyph metrics. **Hover** with a mouse in the resting state opens
the gap by 12 % and shrinks the asterisk slightly.

## 3. Layout and screens

`fitDesktop()` looks for the largest wordmark size (at most 176 px, 15 %
of the viewport width, 26 % of its height) that leaves enough room on
the side for Projects and Publications beside the letters. The room is
measured from the actual longest entry in `content.json`. If no size
fits, the page switches to **compact** mode, used for phones and
portrait tablets. There, panels stack below the wordmark, the wordmark
is about 80 % of the screen width, panel text has a 12 px minimum, and
the whole group (`rig`) re-centres vertically with a spring on each
change. There are no fixed breakpoints: the decision comes from the real
content and the real screen. The **MAO size** setting scales the
wordmark only; panel text keeps its size.

`placeRig()` keeps the wordmark and the active panel inside the
viewport.

## 4. Navigation and input

`go(to)` changes `state` (0–4, wrapping), updates the operator, hides
the old panel and shows the new one. In compact mode the new panel waits
90 ms, so two stacked panels never overlap. One pointer handler on the
stage decides what a gesture means:

- a horizontal drag over 10 px locks into a **swipe**. The wordmark
  follows with rubber-banding and the operator tilts. Releasing past 60 px
  or faster than 0.45 px/ms changes section;
- a **tap** on a link does what a link does;
- a tap on the active panel toggles the **zoom**;
- a tap anywhere else zooms out if zoomed in, otherwise goes **back**
  in the left quarter of the screen (`BACK_ZONE = 0.25`) and **forward**
  in the rest. On press, the wordmark sinks and leans 5 px toward the
  direction it is about to go.

The wheel, the publication list and the timeline handle their own
pointers. A tap on an unselected row selects it, and a tap on the
selected row toggles the zoom. After a mouse or touch interaction they
**blur themselves**, so the arrow keys go back to changing sections
instead of staying captured by the widget. They keep their own arrow
keys only when reached with the keyboard (`:focus-visible`).

## 5. The camera: zoom that stays sharp

Everything lives inside `world`. `aimCamera()` frames the active panel's
`box()` at up to 2.5× on desktop and 1.9× in compact mode, and drives
three springs: `C.s`, `C.x` and `C.y`.

- **While moving**, the zoom is a CSS transform. Line weights are divided
  by the current scale (`--zs`, `--px`, `--hair`), so hairlines stay
  hairlines instead of thickening as the page grows.
- **At rest**, `commitZoom()` sets `Z` to the camera scale and runs
  `layout()` again at that size, with the transform back at 1. Text and
  lines are then drawn natively and stay crisp. `uncommitZoom()` reverses
  this in a single frame before the camera moves again.
- Every length in `layout()` is multiplied by `Z`. `placeRig()` and
  `aimCamera()` work in 1× units and divide boxes by `Z`. The rig
  transform is applied immediately after a re-layout, so a commit never
  shows a one-frame jump.

Zoom state shows extra elements marked `.zx`: map links, project years,
the calibration note and the drawing scale.

## 6. The sections

### + Contact
The email is centred. The **Swiss** column starts at the M's left ink
edge and is left-aligned; the **Italian** column ends at the O's right
ink edge and is right-aligned. Together the two columns make one box.
On narrow screens the box widens symmetrically if the columns would
touch. On entry, `flap()` runs over the panel: each character turns
over on its hinge (`perspective` + `rotateX`), with the board's seam
across the middle while it turns, cycling through a few board letters
before landing. Every cell is locked to its final width, so the layout
never moves. When the animation ends, the original text nodes are put
back, which restores kerning.

### × Projects
`makeWheel()` puts every row on a drum of radius R, seen from 3R away.
`project()` computes the projection itself (height on the drum, depth
scale, flattening by cos θ) and applies it as a **plain 2D transform**,
anchored on the ruler. Two earlier versions were replaced: a nested
`preserve-3d` (Safari flattens it) and a `perspective()` vanishing at the
left edge (it skewed the text). The ruler has a major tick per project,
three minor ones between, and a fixed red needle on the operator's axis.
Input: drag with momentum, mouse wheel (one notch = one row), trackpad
with snapping after 110 ms, ↑ ↓ Home End, and a tap on a row.

### − About
The panel text sits above the wordmark, between its ink edges, and
appears word by word from a blur. Around the letters, `layoutAbout()`
places four dimensions (`placeDim()`):

1. **overall width**, from M's left ink edge to O's right ink edge, above
   the letters;
2. **cap height** of the M, which has a flat top and foot, left of the
   wordmark;
3. **the O split at the operator's axis**, right of the wordmark: O top →
   centre of the − → O bottom;
4. **a chain below**, letter by letter: M | slot | A | O. A and O share
   one mark, since their ink touches. Any span too short to read is
   merged, so a negative or zero measure cannot appear. `chainMM()`
   rounds the parts so they **add up to the overall width**, as on a
   drawing.

Each end has a perpendicular tick and a 45° slash of the same length
(0.8 × the text size). Labels break the line; if one does not fit, it
moves outside the line. On narrow screens the side dimensions move in so
their labels stay on screen. Units are cm at rest and mm when zoomed,
and they switch with a flap.

**Real size.** `estimateMmPerPx()` uses a saved calibration if one
exists. Otherwise it estimates from the device: iPhone, iPad and Android
classes, and MacBook vs. 27-inch Macs from screen size and
`devicePixelRatio`, corrected for browser zoom. The default is the CSS
reference pixel (96/in), which on a MacBook is about 25 % off. The
estimate is good to roughly ±5 %. The **calibration** overlay
(`openCalib()`) draws an ISO/IEC 7810 ID-1 card outline (85.60 × 53.98 mm,
3.18 mm corners) with its own dimensions. The visitor holds a bank card
against the screen and drags anywhere to match it. The further the
pointer is from the card vertically, the finer the control ("adaptive
precision"). On phones the card stands upright, since 85.6 mm is wider
than the screen. The result is saved per screen size in `localStorage`.

### ÷ Publications
`makeTimeline()` has a rolling list above and a horizontal drum timeline
below, sitting on the wordmark's baseline. Both are driven by one spring,
`t`, measured in months. The red needle is fixed at the centre and time
moves under it: newest on the right, towards the M. The axis has month
ticks (grey) and year ticks (ink), with years labelled below. It
continues in both directions and fades at the edges: there is no first
or last tick. Publications are vertical marks, the selected one red and
full height.

The list rolls with time. `fOf()` and `tOf()` map months to a fractional
list position, piecewise between publications, so the publication under
the needle is always the middle row, with one neighbour fading above and
below. This works the same for 3 publications or 30. Dragging the
timeline horizontally or the list vertically, or using the wheel, moves
the same spring, and on release it snaps to the nearest publication.
Between the list and the date readout there is a fixed breathing space
of 1.1 × the text size.

## 7. Details

- **Fonts**: Inter Tight 400/500/700/800/900, self-hosted woff2. The
  page makes no third-party requests.
- **Reduced motion**: with `prefers-reduced-motion`, springs jump to
  their targets, flaps are skipped, and CSS transitions drop to 1 ms.
- **Haptics**: `buzz()` sends short vibrations on section change, zoom
  and wheel ticks. It works on Android only; iOS Safari ignores it.
- **Accessibility**: hidden panels are `inert` and `aria-hidden`; a live
  region announces section and zoom changes; the wordmark's `aria-label`
  says where it leads; the wheel is a `listbox` and the timeline a
  `slider`.
- **Resize / rotation**: `onResize()` resets `Z`, lays out again and
  re-aims the camera if zoomed.
- `window.__mao` exposes `go`, `zoomIn`, `zoomOut`, `openCalib` and some
  state for testing.

## Content flow

```
Pages CMS ─ commits content.json (and uploaded media) to main
   └─ GitHub Actions: publish.yml
        ├─ writes <head>: title, description, Open Graph, icon (from content.json → site)
        └─ deploys index.html + content.json + fonts/ + media/ to GitHub Pages
             └─ browser loads index.html → fetches content.json → normalize() → builds panels → layout()
```
