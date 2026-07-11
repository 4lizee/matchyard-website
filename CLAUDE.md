# CLAUDE.md

Guidance for AI assistants working in this repository.

## What this is

The public marketing site for **MatchYard** (matchyard.us) — a mobile app that
connects people with pickup sports games and players nearby. The app itself
lives elsewhere; this repo is only the landing page plus the Privacy/Terms
pages, deployed as a static site on **Vercel**.

There is **no build system, no package.json, no framework, and no tests**.
Every file in the repo root is served as-is. Verify changes by opening
`index.html` in a browser (or `python3 -m http.server`) — there is nothing to
compile.

## File map

| File | Purpose |
| --- | --- |
| `index.html` | The entire landing page — markup, all of its CSS (inline `<style>` blocks), and all of its JS (inline `<script>` blocks). ~1,500 lines, self-contained. |
| `style.css` | Shared stylesheet for `privacy.html` and `terms.html` **only**. The landing page does NOT link it — don't add landing-page styles here. |
| `privacy.html`, `terms.html` | Static legal pages. Dependency-free by design: no CDNs, no JS, dark-first with a `prefers-color-scheme: light` variant. |
| `vercel.json` | Vercel config: `cleanUrls: true` (pages are linked as `/privacy` and `/terms`, without `.html`), no trailing slashes. |
| `shots/` | Drop-in slots for real app screenshots (see below). Contains only a README until PNGs are added. |
| `logo.svg`, `logo-mark.svg` | Wordmark and square "MY" monogram (also the favicon). |
| `og-image.png` (+ `.svg` source) | 1200×630 link-preview image referenced by absolute `https://www.matchyard.us/` URLs in the OG/Twitter meta tags. |
| `hero-tennis.mp4`, `hero-poster.jpg` | Hero background video and its poster frame. |
| `photo-*.jpg` | Sports photos used by the hero swipe deck. |
| `badge-app-store.svg`, `badge-google-play.svg` | Store badges (links are "coming soon" placeholders). |

## index.html anatomy

The page is deliberately a single file. Order matters:

1. **Head**: meta/SEO (OG + Twitter cards pointing at `https://www.matchyard.us/`),
   Google Fonts (Space Grotesk), then one large `<style>` block with all
   landing-page CSS.
2. **Body sections**, each a `<section>` with an id used by the nav:
   hero (swipe deck + phone mockup) → trust ticker → stats → `#features`
   (bento grid) → `#how` (pinned steps) → `#nearby` → `#app-preview` →
   `#stories` → `#roadmap` → `#faq` → `#support` → `#download` (waitlist
   form) → footer.
3. **JSON-LD** structured-data `<script type="application/ld+json">`.
4. **Script block 1** (vanilla, no dependencies): screenshot-slot reveal,
   condensing sticky nav, waitlist form, stats counters, showcase/roadmap
   reveals, FAQ accordion, mobile nav, IntersectionObserver scroll-reveals.
5. **Script block 2** (CDN-dependent): GSAP 3.12.5 + ScrollTrigger +
   Lenis 1.1.13 loaded from cdnjs/jsdelivr, driving the rotating headline
   word, draggable hero swipe deck, scroll-zoom, the pinned how-it-works
   sequence, and desktop-only cursor tilt. Everything here degrades
   gracefully if the CDNs fail — block 1 must keep the page fully usable
   on its own.

### Conventions inside index.html

- **Brand tokens** are CSS custom properties on `:root`: lime `#A3E635`
  (primary accent), teal `#00E5C3` (secondary), navy `#0A1120` background,
  `--gradient` lime→teal, Space Grotesk for everything. Always use the
  variables, not raw hex. Note the legacy aliases (`--indigo`, `--violet`,
  `--emerald`) are mapped onto lime/teal — don't introduce actual
  indigo/violet colors.
- **Dark-first**: the landing page is dark-only; navy text (`--on-accent`)
  goes on lime fills.
- **Motion is opt-out aware**: both script blocks check
  `prefers-reduced-motion` and skip/neutralize animations when set. Any new
  animation must do the same, and fine-pointer-only effects must gate on
  `(hover:hover) and (pointer:fine)`.
- **Accessibility**: skip link, `:focus-visible` styles, `aria-expanded` on
  the hamburger, `role="status"` on form feedback. Keep this bar.
- The **waitlist form is a front-end stub** — it fakes success client-side
  and submits nowhere. If wiring a real backend, replace the handler in
  script block 1 (`/* 1d. WAITLIST FORM */`).

## Screenshot swap system (`shots/`)

The app-screen visuals are CSS mockups. Each has a hidden
`<img class="shot">` sibling with a `<!-- swap: ... -->` comment above it.
Dropping a correctly named PNG into `shots/` and setting its `src` (per
`shots/README.md`) makes the real screenshot cover the mockup; script block 1
auto-reveals `.shot` images that load and re-hides them on error, so broken
paths fall back safely. Slot ids: `shot-discover` (hero), `shot-game`,
`shot-chat`, `shot-match`, `shot-profile` (features grid). Spec: portrait
iPhone, dark mode, ~9:19.5 (e.g. 1179×2556).

## Development workflow

- **Branches**: work happens on feature branches (`feat/...` or
  `claude/...`) merged into `main` via PR. `main` is what Vercel deploys.
- **Commits**: Conventional-commit style with a scope, e.g.
  `feat(landing): ...`, `fix(seo): ...`, `content(landing): ...`. Use
  `content` for copy-only changes.
- **Verification**: serve the directory locally and check in a real
  browser. Test at mobile widths (hamburger nav kicks in), with
  reduced-motion enabled, and with the CDN scripts blocked (page must still
  work). There is no linter or CI.
- **URLs**: internal links to legal pages use clean URLs (`/privacy`,
  `/terms`) because of `vercel.json`; absolute URLs (OG tags, JSON-LD)
  always use `https://www.matchyard.us/` — never a `*.vercel.app` preview
  domain (that was a real bug, fixed in `e872451`).

## Gotchas

- `style.css` looks like the site stylesheet but only styles the legal
  pages. Landing-page CSS edits go in `index.html`'s `<style>` blocks.
- Large binaries (`hero-tennis.mp4` ~3.9 MB, `hero-poster.jpg` ~1 MB) are
  committed directly — compress new media before adding it.
- The hero `<h1>` uses `white-space: nowrap` and a JS-rotated word
  (`#rotator`); headline copy changes must stay one line at desktop widths.
- The FAQ accordion closes other items when one opens; section reveals use
  one-shot IntersectionObservers — re-triggering requires a reload.
