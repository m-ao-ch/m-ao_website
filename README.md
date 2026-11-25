# M+AO

The website of **M+AO**, an architecture office in Locarno (CH) and
Cavenzano (IT). Live at [www.m-ao.ch](https://www.m-ao.ch).

The whole site is the wordmark. There is no menu: **M A O** is both the
logo and the navigation. Moving through the site opens a gap between the
M and the A, and an operator, **+ × − ÷**, appears in it. Each operator
is a section, and each section has its own way of showing its content:

| State | Operator | Section | How the content behaves |
|---|---|---|---|
| 0 | *(none)* | — | The resting wordmark, **MAO\***. The asterisk refers to the footnote "\*Architecture office". |
| 1 | **+** | Contact | The text arrives like a split-flap departure board, letter by letter. |
| 2 | **×** | Projects | A drum ("wheel") of numbered projects, turned by scrolling or dragging. |
| 3 | **−** | About | The wordmark is drawn as a dimensioned architectural drawing of itself, at its real size on your screen. |
| 4 | **÷** | Publications | A timeline with no beginning or end, scrubbed under a fixed red needle. |

The site performs the logic it describes: the mark is an operator, not a
label. The operators are not font glyphs. They are two bars and two dots
that the page draws and animates, so + turns into ×, × folds shut into −,
and − splits into ÷.

Why the site looks and moves the way it does is in
[`docs/DESIGN.md`](docs/DESIGN.md). How it is built is in
[`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md).

## Using the site

| Action | Result |
|---|---|
| Click or tap the **right three quarters** of the screen | Next section |
| Click or tap the **left quarter** | Previous section |
| Swipe left / right | Next / previous; the wordmark follows the finger |
| Click a section's own content | Zoom in on it: the page leans in, and the rest of the wordmark stays at the edges, cropped |
| Click that content again, or anywhere outside it | Zoom back out |
| Keyboard | ← → sections · `Z` zoom · `Esc` zoom out, then back to the wordmark |

When you are zoomed in, some content shows more detail: map links under
the offices, the year of each project, measures in mm instead of cm, and
a link to calibrate the measures.

## Editing the content

Everything that changes lives in [`content.json`](content.json) and is
edited through [Pages CMS](https://pagescms.org) (app.pagescms.org, with
the GitHub login). The form is defined in [`.pages.yml`](.pages.yml).
Saving commits to this repository and the site updates in a minute or
two.

| Form section | What it controls |
|---|---|
| **Site** | Tab title; link preview title, text and image (1200 × 630 px); tab icon (square PNG, at least 180 px, solid background); wordmark weight (Bold / Extra Bold / Black) and size (80–110 %). |
| **Projects (×)** | Number (free text, e.g. `103`; empty = numbered by position), title, type, place, year (shown when zoomed in). Order = order on the wheel. |
| **Publications (÷)** | Publisher, title, year, month. Sorted by date automatically, newest first; numbered automatically. |
| **About (−)** | One entry per paragraph. |
| **Contact (+)** | Email and up to two offices. The first sits under the M, the second under the O. "Map search" is what the map link looks up. |

Two things to know:

- **Link previews and tab icons are cached** by the apps and browsers
  that show them, sometimes for days. A changed preview appears at once
  for a link nobody has shared yet; to test it, add something like
  `?v=2` to the address. To check a new icon, open the site in a private
  window.
- `content.json` on your own computer is not the live one. Edit content
  only through Pages CMS, never by uploading `content.json`, or you will
  overwrite what was saved online.

## Publishing

[`.github/workflows/publish.yml`](.github/workflows/publish.yml) runs on
every push to `main`, including every save from Pages CMS:

1. It copies `index.html`, `content.json`, `fonts/` and `media/` into the
   build.
2. It reads the **Site** settings from `content.json` and writes the
   title, description, Open Graph / Twitter preview tags and the icon into
   the `<head>` of `index.html`, between the `SITE-HEAD:START` and
   `SITE-HEAD:END` markers. This has to happen at build time: WhatsApp,
   iMessage, Slack and the rest read plain HTML and never run the page's
   JavaScript.
3. It deploys to GitHub Pages.

**Repository setting this depends on:** *Settings → Pages → Build and
deployment → Source* must be **GitHub Actions**. If it is set to "Deploy
from a branch", GitHub runs its own publish as well. That publish puts
`index.html` online without the preview tags, and because it finishes
later it overwrites the correct version. This happened once, on
22 October 2026: the link preview kept showing the defaults.

The domain is `www.m-ao.ch`, set in *Settings → Pages*. The bare
`m-ao.ch` redirects to it through four A records pointing at GitHub
Pages, and `www` is a CNAME to `m-ao-ch.github.io`. The registrar's MX
and TXT records belong to the email and must not be touched.

## Discoverability (SEO)

The goal: someone who types **"M+AO"** into Google should land on
`www.m-ao.ch` itself, not on a social profile or a directory listing,
and a link to the site shared anywhere should preview as the office,
not as a bare URL.

**Already covered**, entirely from `content.json` → **Site**, written
into `<head>` at publish time (see *Publishing* above):

- `<title>` and `<meta name="description">`
- Open Graph / Twitter preview tags (title, description, image)
- the tab icon / apple-touch-icon

**Not yet in the repo**, worth adding later without touching the
page's behaviour (static `<head>` tags and two root-level files only
— none of this affects the wordmark, motion, or any live content):

- `robots.txt` and `sitemap.xml` at the repo root, so crawlers are
  pointed at the single page explicitly instead of relying on default
  crawl behaviour
- a `<link rel="canonical" href="https://www.m-ao.ch/">` tag, so
  `m-ao.ch` and `www.m-ao.ch` are never read as two different pages
- a JSON-LD block (`schema.org` `ArchitectureFirm` or `Organization`)
  naming the office, its email and its two office locations, so
  search engines can resolve "M+AO" to this specific entity rather
  than inferring it from prose
- registering the domain with **Google Search Console** and
  submitting the sitemap, which is also how indexing of a brand-new
  domain can be requested rather than waited for

None of this requires a build step beyond what `publish.yml` already
does; the JSON-LD values would come from the same `content.json`
fields already used for the Open Graph tags (title, description) plus
`contact.email` and `contact.offices`.

## Structure

```
index.html            the whole site: markup, styles, behaviour, and a fallback copy of the content
content.json          all editable content and site settings (edited via Pages CMS)
.pages.yml            Pages CMS form definition for content.json
fonts/                Inter Tight 400–900, self-hosted (no requests to Google)
media/                icon, link-preview images, and images uploaded through Pages CMS
CNAME                 custom domain
.github/workflows/    build + deploy to GitHub Pages
docs/DESIGN.md        identity, intent, references, and the reasons behind the choices
docs/ARCHITECTURE.md  how index.html works
```

Opening `index.html` straight from disk works for a quick look, but the
browser blocks reading `content.json` from a local file. In that case the
page uses the fallback copy of the content written into the script.
