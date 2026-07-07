# Site Refresh Plan

Goal: rebuild reischadance.com on a modern stack that renders correctly on
desktop and mobile. The current site is table-based HTML from ~2013-2018
(fixed 898px layout, `<font>` tags, an image-map nav bar, Highslide/Lightbox+
Prototype+Scriptaculous JS from the mid-2000s) with no responsive behavior at
all.

## Decisions locked in
- **Framework:** Astro. Renders to static HTML/CSS by default, ships JS only
  for interactive bits (gallery lightbox, mobile nav toggle), static export
  is the default output mode.
- **Hosting:** stays on GitHub Pages (existing `CNAME` -> reischadance.com).
  Build via GitHub Actions (`astro build` -> `dist/` -> Pages).
- **Contact form:** current `Contact/sendemail.php` doesn't actually run on
  GitHub Pages (static host, no PHP) - replace with a static form posting to
  Formspree (requires Reischa/Jay to create a Formspree account/endpoint).

## Phase 0 - Scaffold & tooling (done)
- Astro project lives in `astro-site/` alongside the existing static HTML
  (old site stays live at repo root until cutover).
- `astro-site/src/layouts/BaseLayout.astro` - shared header/nav/footer with
  a working hamburger toggle below a 48rem breakpoint, replacing the
  `<map>`/`<area>` image-map nav. Styling is a plain functional baseline for
  now; full brand look (palette, type, grain texture) is Phase 1.
- Stub page created for every planned route (`/`, `/about/`, `/classes/`,
  `/events/`, `/gallery/`, `/videos/`, `/acting/`, `/links/`, `/contact/`) so
  the nav has no dead links while Phase 2 fills in real content.
- `astro.config.mjs` sets `site: 'https://reischadance.com'`;
  `public/CNAME` carries the custom domain into `dist/` on build.
- `.github/workflows/deploy-astro.yml` builds `astro-site` and deploys to
  GitHub Pages via `actions/deploy-pages`. **Trigger is `workflow_dispatch`
  only for now** so it can't clobber the live site mid-rebuild. At Phase 5
  cutover: (1) switch the trigger to `push: branches: [main]`, and (2) in
  the GitHub repo Settings > Pages, change the source from "Deploy from a
  branch" to "GitHub Actions".
- Verified: `npm run build` in `astro-site/` produces all 9 pages plus
  `dist/CNAME`; dev server serves the homepage with nav markup intact.

## Phase 1 - Design system (done)
- Brand identity sourced from the old site's actual graphics
  (`Main_graphic_6-4-2014.jpg` wordmark, `bottom.jpg` nav bar, `Grain.jpg`
  texture) rather than guessed: deep-maroon script wordmark, pink/mauve
  gradient header, tiled grain texture on a `#ffc5e8` page background.
- Fonts (self-hosted via `@fontsource`, no third-party font requests):
  Alex Brush (script, wordmark only), Playfair Display (headings), Mulish
  (body/nav - replaces the old Arial default with something more refined
  but still highly legible on mobile).
- Design tokens as CSS custom properties in `astro-site/src/styles/global.css`.
- Content sits on a white/blush card (`--color-surface`) with a dark plum
  ink color instead of the old white-text-on-pink combo, fixing the
  original's low-contrast body text.
- Verified with Playwright screenshots (mobile 390px, desktop 1280px, and
  the mobile nav opened) - no console errors. Caught and fixed a real
  accessibility bug this way: the header gradient's lighter stop gave white
  nav text ~2.4:1 contrast (WCAG AA needs 4.5:1) - narrowed the gradient to
  two stops that both stay >= 4.5:1, and switched the nav hover/focus style
  from a lighter text color (also failed contrast) to an underline.

## Phase 2 - Content migration (page by page) (done)
| Old | New |
|---|---|
| `index.html` | `/` (real bio copy + "More on the artist" list, done in Phase 0/2) |
| `Belly_Dance_.html` | `/what-is-belly-dance/` |
| `Bios/Reischa_.html`, `Resume_.html` | `/about/`, `/about/resume/` |
| `Classes/Classes_.html` | `/classes/` |
| `Classes/Attire_.html` | `/classes/attire/` |
| `Classes/Millennium.html` | `/classes/millennium/` |
| `Classes/Vandy_.html`, `Vandy2_.html` | `/classes/vanderbilt-bellydance/`, `/classes/vanderbilt-flamenco/` (migrated as-is per decision below) |
| `Events/Events_.html`, `Perf_Past_.html` | `/events/`, `/events/past/` |
| `Links/Links_.html` | `/links/` |
| `Contact/Contact_.html` | `/contact/` (text + form UI now; Formspree wiring is Phase 4) |
| `Acting/acting.html` | `/acting/` (headshots copied to `public/images/acting/`, restyled to match the design system) |
| `Pictures/*`, `Video/*` | still Phase 3 - these are media-heavy pages rebuilt alongside the gallery/video lightbox work, so left as stubs for now |

Stripped table/font markup, pulled actual text content, dropped dead/
commented-out entries (several old pages had HTML comments hiding stale
venue names, and a dead empty `RSK_.html` link on the old Classes page).

**Decision:** `Vandy_.html`/`Vandy2_.html` (Vanderbilt Dance Program classes,
Fall 2014 semester) are not linked from the current live Classes page and
describe a program Reischa no longer teaches for (her 12-year Vanderbilt
role has ended) - user chose to migrate them as-is anyway, so they're live
at `/classes/vanderbilt-bellydance/` and `/classes/vanderbilt-flamenco/`,
linked from `/classes/` under an "archived" heading.

**Resume page:** the original's two/three-column layout tables were
verified to line up row-for-row (count-checked each section) before being
restructured as `{title, role, venue}` triples in a responsive
`.credit-list` component - safer than guessing pairings, and much more
readable on mobile than the original tables.

**Content needing the user's own update (not a migration bug):** the
Millennium class listing and the Events page both carry specific dates
that are already in the past relative to today - these are ported
verbatim from the old site's live content and will need Reischa/Jay to
update them directly in `astro-site/src/pages/classes/millennium.astro`
and `astro-site/src/pages/events/index.astro`.

Verified with Playwright screenshots (desktop + mobile) across all new
pages - no console errors.

## Phase 3 - Media (done)
- Photo galleries (32 "Live" + 16 "Studio"): old hand-made `*thumb.jpg`
  files were dropped - all originals moved to `src/assets/gallery/{live,studio}/`
  and Astro's `astro:assets` pipeline now generates every size (build
  output showed 452kB/570kB/etc. JPEGs down to 14-70kB webp thumbnails, 96
  images total). `GalleryGrid.astro` renders a CSS-columns masonry grid
  (natural aspect ratios, no cropping) plus a single reusable lightbox
  built on the native `<dialog>` element - vanilla JS only (no dependency)
  for open/close/prev/next, keyboard arrow + Escape support.
- Video pages: dropped Highslide entirely. `VideoEmbed.astro` originally
  rendered a plain responsive `iframe` (youtube-nocookie.com,
  `loading="lazy"`); Phase 5 Lighthouse testing found this loaded all 4
  iframes at once (7s LCP), so it's now a click-to-load facade instead -
  see Phase 5. Split into `/videos/` (solo dances, was `Video_.html`) and
  `/videos/group-choreography/` (was `Video_2.html`), cross-linked like
  the originals.
- Verified via Playwright: build output confirms all 96 images optimized
  with no errors/warnings; exercised the lightbox open/next/close flow and
  confirmed all 4 video iframes on both pages actually load
  (`page.frames()` showed all 4 youtube-nocookie.com embeds resolve once
  scrolled into view - confirmed a "blank" 4th video in one screenshot was
  just a lazy-load timing artifact of the screenshot itself, not a bug).

## Phase 4 - Contact form (done)
- Static form posts to the real Formspree endpoint
  (`https://formspree.io/f/mlgyvpzr`) in place of the non-functional
  `sendemail.php`. Includes a `_gotcha` honeypot field, a `_subject`
  line, and a `_next` redirect to `/contact/thanks/`.
- New `/contact/thanks/` page recreates the original's ornate "Thanks"
  graphic as real text in the Alex Brush script font (design-system
  consistent) instead of embedding the old low-res JPEG.
- Verified via Playwright without spending a real submission: confirmed
  the form posts to the correct endpoint with the right hidden fields,
  and that HTML5 `required` validation blocks an empty submission
  (page doesn't navigate). The user submitted the real first test
  message themselves, since Formspree's free tier ties activation to
  the account owner confirming via email.

## Phase 5 - QA & launch
### QA (done)
- Automated crawl (Playwright) of all 20 routes: zero broken internal
  links, zero 404s, zero console errors.
- Automated accessibility scan (axe-core) of all 20 routes found 2 real
  issues, both fixed:
  - `/contact/thanks/` was missing a real `<h1>` (used a styled `<p>`
    instead) - fixed.
  - Two remaining flagged items (`aria-prohibited-attr` on `/videos/*`)
    were traced to a specific DOM node (`#movie_player`,
    `data-version=".../player_embed.vflset/..."`) - confirmed via node
    inspection this is YouTube's own embedded-player markup, not ours;
    not fixable from our side and inherited by any site embedding
    YouTube.
- Lighthouse (performance/accessibility/best-practices/SEO) on a
  representative page per profile:
  - `/` (text-heavy): 99 / 100 / 100 / 100
  - `/gallery/live/` (image-heavy, 32 photos): 92 / 100 / 100 / 100
  - `/videos/` (embed-heavy): was 60/100/96/100 with a 7.0s LCP and
    1.9MB page weight from loading all 4 YouTube iframes at once -
    fixed by converting `VideoEmbed.astro` to a click-to-load facade
    (static `i.ytimg.com` poster thumbnail + play button; real iframe
    only created on click, with a `<noscript>` fallback for no-JS).
    Re-measured: 98 / 100 / 100 / 100, LCP 2.1s, page weight 196KB.
    Verified by clicking through in Playwright that the real embed still
    loads and autoplays correctly.
- Full responsive screenshot pass (mobile 390px + desktop 1280px) across
  every page not already spot-checked in earlier phases.

### Launch (remaining)
- Cut over Actions deploy to the new Astro output: switch
  `.github/workflows/deploy-astro.yml` trigger from `workflow_dispatch`
  to `push: branches: [main]`, switch the repo's Settings > Pages source
  from "Deploy from a branch" to "GitHub Actions", merge `astro-rebuild`
  into `main`, confirm CNAME/DNS still resolves post-cutover.
- This is a live production change to the public site - do not perform
  without explicit user go-ahead at the time of cutover.

## Design rework - match the old site's look (done)
After Phase 5 QA passed, Reischa felt the new site deviated too much from
the old site's look. Root-caused to three things and fixed all three:

1. **Homepage hero photo was missing.** Restored the actual old hero photo
   (`Main_graphic_6-4-2014.jpg`, baked-in logo + tagline) as a full-width
   banner.
2. **Per-page background art was missing.** Every old page had a bespoke
   background image (photo + ornate script title, or pure scrollwork
   texture in the same purple family) that got dropped during rebuild.
   Restored via a new `PageHero.astro` component that reuses the actual
   old images, rendered as a full-width responsive banner (natural aspect
   ratio, no cropping, `max-height: 70vh` safety cap for any future
   portrait image). Two modes: `showTitle=false` (image already has a
   baked-in title, e.g. ClassesBG/EventsBG/ContactBG/LinksBG/LiveBG/
   StudioBG/VideoBG/BioBG - a visually-hidden real `<h1>` carries the
   text for accessibility/SEO) or `showTitle=true` (a real `<h1>` in Alex
   Brush script renders over the image, for textless generic images -
   `ClassesBG2.jpg` reused across Attire/Millennium/both Vanderbilt pages/
   Resume, `Belly_DanceBG.jpg` for What is Belly Dance, `BioBG2.jpg` -
   Reischa's own newer, untracked bio photo - for About).
   `Acting` and `Resume` had no original bespoke asset, so both reuse the
   generic `ClassesBG2.jpg` texture (Acting's own headshots are portrait
   orientation, unsuitable as a full-width banner - see the `max-height`
   note above). `Gallery` index and `Contact/Thanks` intentionally have no
   PageHero (each already has its own strong visual treatment).
3. **Nav felt flat.** (Addressed via the header gradient background,
   already part of the design system - no separate change needed here
   since the ornate-photo banners now carry most of the "old look" job.)

Also flipped the content section below each hero from a light card
(dark-ink-on-white, Phase 1's original choice) to a deep-plum gradient
with white/lavender text (`--color-content-bg`, `--color-content-heading`,
`--color-content-muted`), matching the old site's actual purple-and-white
mood far more closely while keeping strong contrast (~14:1 body text,
still passes WCAG AA/AAA).

Verified: full Playwright + axe-core crawl of all 20 pages after the
rework - zero broken links, zero console errors, zero accessibility
violations (one real one was caught and fixed: the hero banner sat
outside any landmark region; moved it inside `<main>`). Lighthouse
re-checked on the two heaviest pages (Home, About) post-rework: still
99/100/100/100 with sub-2s LCP despite much heavier hero images -
Astro's image pipeline handled it (e.g. the new About photo was a 12MB
source JPEG, optimized down to 177KB).

## Working style note
Work lands in small, committable increments (one phase or a few pages at a
time) rather than one long uninterrupted pass, so progress is checkpointed
and sessions can pause/resume cleanly.
