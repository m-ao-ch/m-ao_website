# Design: intent and reasons

This document explains why the site is the way it is. It is drawn from
the working sessions in which it was designed (October 2026), so that
later changes can keep its identity instead of drifting from it. The
mechanics are in [`ARCHITECTURE.md`](ARCHITECTURE.md).

## The idea

M+AO is an architecture office. The website has a single image: the
wordmark. The wordmark is not decoration in front of the content; it is
the instrument used to reach the content. The gap between M and A is
where the office "operates": add, multiply, subtract, divide. Each
operation opens a different part of the practice. The original about
text says it directly: *the wordmark is an operator rather than a
label; the site performs the same logic it describes.*

The site should therefore feel like **one object being handled**, not a
set of pages. There is no menu, no routing and no page change: one mark,
one camera, and things unfolding from the letters.

## How it should feel

The brief for the motion, in the founder's words: movement that
"releases serotonin", that you feel in your head as if you were moving
the objects yourself, with a satisfying click. In practice:

- **Physical, not timed.** Everything is a spring with mass and
  momentum, never a fixed-duration curve. Interrupt any movement and it
  changes direction from where it is. The first version used CSS
  transitions and `setTimeout` chains; it felt mechanical, jumped, and
  jammed under fast clicks. That was the starting complaint.
- **The click.** Pressing makes the wordmark sink slightly. Releasing
  springs it back with a small overshoot. The operator opens with a touch
  of bounce. This is where the satisfaction comes from.
- **Direct manipulation.** On touch, the wordmark follows the finger
  1:1 before snapping. The wheel and the timeline are thrown and caught,
  not stepped through.
- **Crisp at all times.** Zooming must look like leaning in towards a
  drawing, not like enlarging a picture. At rest the page is redrawn at
  the new size, and lines stay hairline-thin while moving.

## References

**[rauno.me](https://rauno.me)** (Rauno Freiberg) was chosen as the
reference over the other candidates (Family, Emil Kowalski, Linear,
Apple's Dynamic Island). The appeal for architects is in the
experiments and prototypes, which read like diagrams:

- thin lines alternating with thick ones, and **red only for what
  matters**;
- the *wheel interface* → the Projects drum;
- the *running diary*, a bar and red dot scrubbing through time → the
  Publications timeline;
- text that appears like a *terminal / split-flap letter board* → the
  Contact board and the changing dimension labels.

From there came the decision to give **each section its own behaviour**,
matched to its operator. The aim was to build in that spirit, not to
copy the prototypes.

The second reference is the **architectural drawing**: dimension lines,
45° tick marks, hairlines, chains of dimensions that add up. The About
section literally draws the logo the way an architect would draw a
building.

## Visual language

- **Colour.** White paper, ink `#111`, one red `#e10600` for the
  operator, the needles, the selected numbers and the dimension labels.
  Grey (`#c9c9c9`, `#a6a6a6`) for secondary ticks and text. Red is never
  decorative: if it is red, it is the current or important thing.
- **Lines.** The thinnest line the screen can draw: 1 device pixel, so
  0.5 px on retina screens. Grey for minor marks, ink for major ones,
  slightly heavier red for indicators. In the drawings, the slash marks
  are heavier than the dimension line, as in drafting convention.
- **Type.** The wordmark is a Swiss grotesque. Inter Tight was chosen as
  a free, very close relative of Helvetica that can be served from the
  site itself (Helvetica would need a paid web licence). Black (900) is
  the reference weight. Bold and Extra Bold can be selected in the CMS.
  For icons and graphics outside the site, TeX Gyre Heros Bold (a free
  Helvetica clone) was used to match the original M+AO artwork.
- **The operator** is drawn, never typed. It has one stroke thickness
  for every bar, proportioned like the Swiss cross (arms 1/6 longer than
  wide), and is derived from the weight of the letters so that it
  belongs to them.
- **Alignment.** Everything aligns to the letters' **ink**, not to their
  boxes: dimensions start and stop exactly where a letter starts and
  stops, and the Contact columns hang from the M and the O. The
  Projects needle sits on the operator's axis. The Publications axis is
  the wordmark's baseline. The Projects and Publications blocks occupy
  the cap height of the letters beside them.

## Section by section

**+ Contact. The proportions are sacred.** This was the only part of the
original site the founder asked to keep exactly: its layout, alignments
and proportions. Email centred. Swiss office under the M, left-aligned.
Italian office under the O, **right-aligned**, so the two columns mirror
each other and form one box. The split-flap entrance had to read as
**flipping**, not falling: letters turn on a hinge, with the seam
visible.

**× Projects. A drum, perpetual and three-dimensional.** Rows recede,
shrink and flatten as they turn away; ticks on a ruler; a fixed red
needle. Project numbers are the office's own job numbers (103, 104, 108…),
so they are set by hand in the CMS rather than generated.

**− About. The logo as a drawing.** The dimensions give the **real
physical size of the logo on the screen you are looking at**: centimetres
at rest, millimetres when you lean in. The founder measured the screen
with a ruler and found the first version more than 10 mm off. Browsers
do not know a screen's physical size, so the site estimates it per
device and offers a calibration against a bank card, "the one ruler
everyone carries". The annotations were extended at the founder's
request, as on a real drawing:

- a chain of dimensions below, letter after letter, forming one
  continuous line;
- on the right, the O measured from its top to the axis of the − and
  from there to its bottom;
- every dimension ending exactly at the letter's edge;
- short, contained end marks: the perpendicular tick as long as the
  oblique one, not long extension lines, which made the whole drawing
  look off;
- chain parts that add up to the total.

**÷ Publications. No beginning, no end.** The first timeline had hard end
ticks and a start and an end. It felt finite, unlike the wheel's
perpetual quality, which the founder loved. It became a timeline that
continues forever and fades at both sides, with a fixed red needle and
time moving underneath. The list above rolls with it and fades, so the
design holds for any number of publications. Other details:

- the red number sits at the end of the line, **towards the M**,
  counter-intuitively but deliberately;
- there is breathing room between the date and the text;
- the grey lines are hairlines.

## Navigation

- **Left quarter back, rest forward.** A half-and-half split was tried
  first. Going forward is the main action, so it gets three quarters of
  the screen.
- **Click the content to lean in, click again to lean back.** The
  wordmark never disappears when zoomed: it stays at the edges, cropped,
  "as if you were simply looking closer". Zoomed views can carry more
  detail than the overview.
- **No instructions on screen.** An early "← click →" hint under the
  wordmark was removed at the founder's request. The site is meant to be
  discovered by touching it.

## Decision log

| Decision | Reason | Rejected alternative |
|---|---|---|
| Spring physics for all motion | Interruptible, momentum-preserving; the core of the "feel" | CSS transitions + timers (mechanical, jumpy, jammed on fast clicks) |
| Compute final geometry before animating | Panels never aim at a moving target | Measuring during the animation (caused the "jumps") |
| Layout decided by real content and screen, no breakpoints | Works on any device; tablets were broken with fixed breakpoints | One breakpoint at 640 px |
| Inter Tight, self-hosted | Close to Helvetica, free, no requests to Google (privacy, GDPR) | Helvetica (licence); Google Fonts |
| Uniform operator stroke | Founder: the + looked off with a thick vertical and thin horizontal | Stroke varying with angle, like the font's own contrast |
| Real-size dimensions with estimate + card calibration | Founder measured the screen: the 96 dpi default was >10 mm off | Fixed CSS-pixel conversion |
| Re-layout at rest for zoom | Text and lines must stay crisp in every browser, including Safari | Transform-only zoom (blurry) |
| Wheel as a computed 2D projection | Holds its depth in Safari; no skewed text | Nested `preserve-3d`; `perspective()` with a side vanishing point |
| Perpetual timeline + rolling list | Matches the wheel's perpetuity; scales to any number of publications | Finite timeline with end caps; static list |
| Content in `content.json`, edited with **Pages CMS** | Free, open source (MIT), edits JSON straight in the GitHub repo; no code for the office | Decap CMS (relies on Netlify Identity, deprecated in 2025); editing HTML |
| Hosting on **GitHub Pages**, publishing via GitHub Actions | Free, one account for code, CMS and hosting; the workflow writes link-preview tags into the HTML | "Deploy from a branch" (cannot write the preview tags, and conflicts with the workflow if both are on) |
| Site settings (title, preview, icon, weight, size) in the CMS | The office can change its own identity details without code | Hard-coded in `index.html` |
| Left quarter = back | Forward is the main action | Left half / right half |
| No on-screen hint | Cleaner; the mark is meant to be discovered | "← click →" under the wordmark |

## Keeping the identity

When changing the site, check against these:

1. Does it still feel like handling one object? No pages, no menus, no
   pop-ups.
2. Is the motion a spring that can be interrupted, and does it run on
   `transform` and `opacity` only?
3. Is red used only for what is current or important?
4. Do lines stay hairlines, and does everything align to the letters'
   ink?
5. Are the Contact proportions untouched?
6. Is it crisp when zoomed, in Safari as well?
7. Can the office still change it from Pages CMS without code?
