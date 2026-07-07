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
