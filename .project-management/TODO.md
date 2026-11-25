# TODO — `guillaumemaiano.github.io` (`T480` Rebuild)

## Context & Constraints
- **Stack:** Hugo (`v0.146.0+`), `theme = 'T480'`, `defaultContentLanguage = 'en'`, languages `en` and `fr`, deployed to GitHub Pages.
- **2026 Hugo Conventions:** Root `layouts/` templates (no `_default/`), `layouts/_partials/`, and `partials.Include` syntax.
- **Zero-Bloat & Zero-JS:** No `<script` tags, no external UI/icon CDNs, CSS-only theme toggle (`:root` + `@media (prefers-color-scheme: dark)` + `:checked` inversion), and native `.Translations` with fallback to `site.Home.AllTranslations` for `EN`/`FR` switching.
- **Reference Harness:** Mirror Playwright + TypeScript test structure and CI conventions from the art site (local sibling directory).

## Tasks
- [ ] **Inspect art site reference:** Read `playwright.config.ts`, `package.json`, `tests/helpers/assertions.ts`, `tests/layout/`, and `.github/workflows/` from the art site to align test and CI patterns.
- [ ] **TDD Red — Scaffold Playwright suite:** Add `package.json`, `playwright.config.ts`, `tests/helpers/assertions.ts`, and specs failing on unbuilt site requirements:
  - `hugo --gc --minify --panicOnWarning` exits `0`.
  - Core routes (`/`, `/404.html`, `/journal/`, `/about/`, `/fr/`, `/fr/about/`) render non-empty without broken internal `href`/`src` or leading-space URLs (`/%20...`).
  - Zero `<script` tags in generated HTML.
  - `#theme-toggle` checkbox toggle + `:checked` CSS state and `EN`/`FR` switcher links are present.
  - Valid RSS at `/index.xml` (`xmllint --noout` + `<item>` entries + full post content when using custom `rss.xml`).
- [ ] **Write `themes/T480/theme.toml`:** Define theme metadata (`name = "T480"`).
- [ ] **Add custom ThinkPad favicon & head links:** Create `themes/T480/static/favicon.svg` (carbon-to-pure-black gradient, TrackPoint-red center circle, bottom-right `GEM` monogram) plus minimal SVG/PNG assets, and wire clean root-relative `<link>` tags in `themes/T480/layouts/_partials/head.html`.
- [ ] **Populate `themes/T480/layouts/_partials/` (`head.html`, `header.html`, `footer.html`):** Include `IBM Plex` font setup, inline `16x16`/`18x18` stroke SVGs (no external icon CDN), `<input type="checkbox" id="theme-toggle">` + `<label>` toggle, and `.Translations` / `site.Home.AllTranslations` `EN`/`FR` switcher link.
- [ ] **Populate `themes/T480/layouts/` (`baseof.html`, `home.html`, `list.html`, `single.html`, `404.html`, and optional full-text `rss.xml`):** Use 2026 `partials.Include` calls; add full-text `rss.xml` for full `.Content` in feeds or remove it to use Hugo's built-in summary RSS.
- [ ] **Write zero-JS `themes/T480/assets/css/main.css`:** Define `:root` + `@media (prefers-color-scheme: dark)` matte carbon/paper tokens, `:has(#theme-toggle:checked)` / sibling combinator inversion, `IBM Plex` typography, and TrackPoint-red accents.
- [ ] **Audit `content/`:** Set `draft = true` on `content/journal/test.md` (or remove it), add `content/journal/_index.fr.md` if needed, and fill in placeholder Atreides GRUB and Caladan OMZSH theme URLs in `content/journal/post_20261124.md`.
- [ ] **TDD Green & CI:** Run `hugo --gc --minify --panicOnWarning`, `xmllint --noout public/index.xml`, and Playwright tests locally until green; wire checks into `.github/workflows/`, push `main`, and verify GitHub Actions build + Pages deploy succeed.
- [ ] **Publish Post #2:** Write and publish the follow-up post documenting the `T480` theme, zero-JS switches, and CI rebuild.