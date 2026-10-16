# Anoushka Aggarwal — Portfolio

Single-page portfolio site. **Everything lives in one file: `index.html`** (inline `<style>` + inline `<script>`, vanilla JS, no build step, no dependencies). Assets are under `assets/`.

This file is read automatically at the start of every Claude Code session — it's the source of truth for how this project works and the design rules to follow. Keep it updated when conventions or the project state change.

---

## How to run / verify
- No build. It's a static file. Serve locally and open in a browser:
  ```bash
  python3 -m http.server 8000
  ```
  then visit `http://localhost:8000/index.html`.
- Always verify changes at **desktop (~1280px) and mobile (375px)** — check for horizontal overflow and console errors.
- macOS tooling notes: **no `pdftoppm`/poppler and no `ffmpeg`**. Render PDFs to PNG via `osascript -l JavaScript` + PDFKit; use `sips` for image resize/convert/crop and dimensions. `qlmanage -t` for thumbnails.
- **Pasted images are not files** — they can't be read. The user must save them to disk (usually `~/Downloads`) first; then detect the newest file and copy/optimize it into `assets/`.

## Git / deploy
- Repo: `github.com/nnusshh/portfolio`, branch `main`. **Direct-to-main workflow** (every commit is a portfolio update); pushing publishes the site.
- End commit messages with: `Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>`
- Commit/push only when the user asks (they usually do, explicitly).
- The code is the durable record. If a chat is lost, `git clone`/`git pull` restores everything; this file restores the context.

---

## Architecture (all inside `index.html`)
- **SPA routing:** `go(id, opts)` toggles `.on` on `[data-view="<id>"]` sections and `[data-nav]` sidebar links, resets scroll, updates the breadcrumb from `PAGES`.
- **Views (`data-view`):** `home`, `about`, and the case studies.
- **`CASE_VIEWS = ['tiaa','articulating','nav','uxaudit','carma','auskey']`** — these get scroll-reveal (`initRevealFor`), flow anim, etc.
- **`PAGES`** — `{id: {label, icon}}` for breadcrumb labels.
- Scroll-reveal: `initRevealFor(id)` (IntersectionObserver; respects `prefers-reduced-motion`; never leaves content stuck hidden).

## Projects (display order: Auskey → Carma → Navigator → Unilever)
| view id | Name | Notes |
|---|---|---|
| `auskey` | **Auskey** — Bluetooth digital stethoscope companion app | palette: teal `#0E716D`, sage `#A6BFB1`, mint `#D3E4D7`; live App Store link; credits **Studio Carbon** |
| `carma` | **Carma** (Agtuall) — climate/agri-insurance web app | accent orange `#ff5d00` |
| `nav` | **Navigator** — FIS Vantage 360 (FP&A dashboard) | palette teal `#00566A`/`#00576B`, mint `#5CCAB1` |
| `uxaudit` | **Unilever** — UX Audit | |
| "The Work Between": `tiaa`, `articulating` | shorter article-style pieces | |

Project order is set in two places: the **sidebar** (`.n-list` under "Projects") and the **home cards** (`.hp-proj`, numbered 01–04). Keep both in sync.

## Case-study layout pattern
`.p-hero` (banner img) → `.p-wrap` ( `.p-title` name, `.p-tag` headline, `.p-meta` facts grid, `.p-git` ) → `.vg-shell` ( sticky scroll-spy nav `.vg-toc` + `.vg-content` of `.cs-row` sections, each with `.cs-aside .cs-aside__ttl` kicker + `.cs-body` ) → `.p-cta`.
- Vantage uses `#vgToc` / `initVgToc`; Auskey uses `#akToc` / `initAkToc` (mirror of it). Section kickers double as per-section headers synced to the nav.
- Most case-study CSS is **scoped per view**, e.g. `[data-view="auskey"] .cs-body p { ... }`.

## Section nav — desktop vs mobile
- **Desktop:** `.vg-toc` is a vertical sticky scroll-spy list (left column). `.vg-toc__trigger` hidden; `.vg-toc__list` is `display:contents` so items flow in the column.
- **Mobile (≤860px):** `initTocDropdowns()` turns each `.vg-toc` into a compact **"jump to section" dropdown** — a sticky bar (`.vg-toc__trigger`, accent dot + current section + chevron) that expands an overlay `.vg-toc__list`. Label stays synced to the active section via a MutationObserver on `.on`.
- Scrolling to a section uses **`smoothScrollTo()`** (requestAnimationFrame — native `behavior:'smooth'` silently no-ops in some environments; this also respects reduced-motion) and **`navOffset(toc)`** (offsets by the pinned sticky-bar height on mobile so the heading clears the bar).

---

## Design rules (important — the user cares about these)
- **Body & section text caps at 16px.** Section headings, section titles, body copy — all ≤16px. Build hierarchy with weight, letter-spacing, color, and spacing, not size. (Holds even when a source PDF uses big display type.)
- **Hero is the exception:** on every project page the **project name (`.p-title`) is 24px** and the **header line / tagline (`.p-tag`) is 32px**; mobile ≤768px steps down to **20px / 24px**. Consistent across all four projects.
- **No grey body text.** Avoid muted greys (`--muted` #9a9a97 / `--dim`). Use white `#fff` for emphasis, near-white `#ededed` / light `#d3dbd8` for secondary. When rebuilding from a PDF, sample and match the PDF's real colors.
- **Each case study gets its own solid color / palette.**
- **Case studies need subtle scroll-reveal** so content isn't static.
- **Adequate, even gaps between sections** (e.g. `.cs-row` vertical padding).
- Em dashes were removed from portfolio copy (contextual mix of alternatives). Keep new copy em-dash-free to match.

## Design tokens (`:root`)
- Radii: `--r-xs:6px --r-sm:8px --r-md:12px --r-lg:16px`
- Durations: `--dur-fast:0.15s --dur-mid:0.2s --dur-slow:0.6s`
- Easing: `--ease-out:cubic-bezier(0.22,0.61,0.36,1)`, `--ease-spring:cubic-bezier(0.34,1.56,0.64,1)`
- Spacing: `--sp-1 … --sp-8`
- Color: `--bg:#1A1A19 --sb:#282826 --text:#ededed --muted:#9a9a97 --dim:#787874 --border:#33322e`
- Layout: `--sw:250px` (sidebar), `--bc:40px` (breadcrumb bar height)

## Gotchas learned the hard way
- **Lazy images must have intrinsic `width`/`height`.** Missing sizes caused CLS that broke anchor/section scrolling (page was short at click time, then grew). All `loading="lazy"` images carry width/height.
- **Container queries:** an element's own `cqw` units reference an *ancestor* container, not itself — put `cqw` only on descendants of the `container-type` element.
- **`getComputedStyle` returns mid-transition interpolated values** — to check a target value, read the inline/target or wait for the transition to settle.
- When verifying in the preview pane, JS `scrollTo` screenshots can lag/blank — trust JS measurements, or use a real wheel scroll before screenshotting.

## Current state / housekeeping
- The home "valley / travel-cards on a map" section was **removed**. `assets/home/valley.jpg` is now **orphaned** (untracked, unused) — safe to delete.
- Untracked files **not part of the portfolio** (do not commit with portfolio changes): `wedding-invite.html`, `wedding-invite/`, `assets/wedding/`, `flex-academy.html`, `font-options.html`.
- This repo is cloned in more than one place (e.g. `~/Documents/CLAUDE` and a OneDrive copy). Always confirm which clone you're editing, and `git pull` before starting.
