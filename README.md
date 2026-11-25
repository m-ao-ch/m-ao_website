# M+AO

The website for **M+AO**, an architecture office based in Locarno (CH) and
Cavenzano (IT). Live at [m-ao.ch](https://www.m-ao.ch).

The whole site is the firm's wordmark. There is no menu: **M A O** is both
the logo and the navigation. Clicking it steps through an operator —
**+ × − ÷** — that appears between the letters, and each operator is a
section of the site, chosen to fit what it shows:

| State | Operator | Section | Why that symbol |
|---|---|---|---|
| 0 | *(none)* | — | Resting wordmark, "MAO\*" (the asterisk notes "Architecture office") |
| 1 | **+** | Contact | Addition: two offices joined, one on each side of the mark |
| 2 | **×** | Projects | Multiplication: the practice's output, browsed as a wheel |
| 3 | **−** | About | Subtraction: the mark reduced to a dimensioned drawing of itself |
| 4 | **÷** | Publications | Division: writing about the work, laid out as a timeline |

Clicking again advances to the next state (and wraps back to the plain
wordmark after Publications). The transition between operators is drawn,
not swapped — the + and × share the same two bars, rotating into place;
the − is those two bars folding flat onto each other; the ÷ is the same
flat bar with two dots splitting off it. Nothing about the symbols is a
font glyph; they're shapes the page draws and animates itself, at the
same weight as the wordmark's own strokes.

Clicking the mark a second time while already on a section **zooms in**
on that section's content — the whole page is one camera that pushes in
and frames the section, rather than a popup or a page change. Clicking
once more zooms back out and returns to the resting wordmark.

## The sections

**Contact** lays the two offices (Swiss and Italian) under the M and the
O, each column starting exactly at the letter above it.

**Projects** is a wheel of numbered entries, scrollable/draggable, listing
the practice's work: number, title, typology, place, and — once zoomed
in — the year.

**About** is the most literal expression of the concept: it draws the
wordmark itself as an architectural drawing, with real dimension lines
giving the letterforms' width and height. Those numbers aren't arbitrary
— the page estimates the physical pixel density of the visitor's own
screen (from device/DPI heuristics, refined by an optional one-time
calibration against a real object such as a credit card, saved locally)
so the dimensions shown are the **actual physical size, in millimetres,
of the logo on that visitor's screen**. Zooming in switches the units
from cm to mm. The firm's "about" text sits beside the drawing.

**Publications** is a rolling, loop-free timeline of press and
publications, newest first, positioned along a continuous line by month
and year.

## Content

Everything editable — projects, publications, the about text, contact
details, the wordmark's weight and size, the tab title and link-preview
image — lives in [`content.json`](content.json), not in the HTML. It's
edited through [Pages CMS](https://pagescms.org) (configuration in
[`.pages.yml`](.pages.yml)), which gives non-technical editors a form
instead of raw JSON. Saving there commits straight to this repo.

`index.html` fetches `content.json` at load and renders from it — add a
project or a publication in the CMS, and it appears on the wheel or the
timeline in that order, no code change needed.

## Deployment

[`.github/workflows/publish.yml`](.github/workflows/publish.yml) runs on
every push to `main` (including CMS saves) and publishes to GitHub Pages
at the custom domain in [`CNAME`](CNAME):

1. Copies `index.html`, `content.json`, `fonts/`, and `media/` into the
   build output.
2. Reads `content.json` and writes the tab title, meta description, and
   Open Graph / Twitter preview tags straight into `index.html`'s
   `<head>`, between the `SITE-HEAD:START`/`SITE-HEAD:END` markers — this
   has to happen at build time because link-preview scrapers (WhatsApp,
   iMessage, Slack, …) read plain HTML and never run the page's own
   JavaScript.
3. Deploys the result with `actions/deploy-pages`.

## Structure

```
index.html        — the entire site: markup, styles, and behaviour
content.json       — all editable text and settings (edited via Pages CMS)
.pages.yml         — Pages CMS form definition for content.json
fonts/             — self-hosted Inter Tight, weights 400–900
media/             — logo, icon, and link-preview images
CNAME              — custom domain for GitHub Pages
.github/workflows/ — build + deploy to GitHub Pages
```

See [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md) for how the page
itself is built — the camera/zoom system, the drawn operators, and the
real-world dimensioning behind the About section.
