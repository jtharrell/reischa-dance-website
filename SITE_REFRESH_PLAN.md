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

## Phase 1 - Design system
- One shared `BaseLayout.astro` (header/nav/footer) instead of per-page
  copy-pasted table markup.
- Global stylesheet: CSS Grid/Flexbox, relative units, mobile-first
  breakpoints. Keep the site's actual brand identity (pink/magenta palette,
  grain background texture, script-style heading font) rather than
  genericizing it.

## Phase 2 - Content migration (page by page)
| Old | New |
|---|---|
| `index.html` | `/` |
| `Belly_Dance_.html` | `/what-is-belly-dance` |
| `Bios/Reischa_.html`, `Resume_.html` | `/about`, `/about/resume` |
| `Classes/*` (Classes, Attire, Millennium, Vandy, Vandy2) | `/classes` + subpages |
| `Events/Events_.html`, `Perf_Past_.html` | `/events`, `/events/past` |
| `Pictures/Pictures_.html`, `Live_.html`, `Studio_.html` | `/gallery`, `/gallery/live`, `/gallery/studio` |
| `Video/Video_.html`, `Video_2.html` | `/videos` |
| `Links/Links_.html` | `/links` |
| `Contact/Contact_.html` | `/contact` |
| `Acting/acting.html` | `/acting` |

Strip table/font markup, pull actual text content, drop dead/commented-out
entries (several old pages have HTML comments hiding stale venue names).

## Phase 3 - Media
- Photo galleries (32 "Live" + 16 "Studio" images, each with a hand-made
  `*thumb.jpg`): replace Lightbox/Prototype/Scriptaculous with a CSS Grid of
  images + a lightbox built on the native `<dialog>` element (no dependency).
- Video page: drop Highslide, embed YouTube via responsive `iframe` in an
  `aspect-ratio` container - no JS needed.
- Compress/resize the 135 source images; use Astro's built-in image
  optimization for responsive `srcset` + lazy loading.

## Phase 4 - Contact form
- Static form posting to Formspree in place of `sendemail.php`.

## Phase 5 - QA & launch
- Test mobile/tablet/desktop breakpoints, verify all internal links.
- Lighthouse pass for performance/accessibility (current site has missing
  alt text and low-contrast white-on-pink text in places).
- Cut over Actions deploy to the new Astro output, confirm CNAME/DNS still
  resolves.

## Working style note
Work lands in small, committable increments (one phase or a few pages at a
time) rather than one long uninterrupted pass, so progress is checkpointed
and sessions can pause/resume cleanly.
