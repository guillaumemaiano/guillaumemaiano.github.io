# TODO

- [ ] Write `themes/T480/theme.toml` manifest.
- [ ] Populate `themes/T480/layouts/_partials/` (`head.html`, `header.html`, `footer.html`), including the `<input type="checkbox">` + `<label>` theme toggle and `.Translations` / `site.Home.AllTranslations` `EN`/`FR` switcher link in `header.html`.
- [ ] Populate `themes/T480/layouts/` templates (`baseof.html`, `home.html`, `list.html`, `single.html`, `404.html`).
- [ ] Add full-text `themes/T480/layouts/rss.xml` (or delete it to use Hugo's built-in summary RSS).
- [ ] Write zero-JS `themes/T480/assets/css/main.css` (`IBM Plex`, `:root` + `@media (prefers-color-scheme: dark)` light/dark tokens, `:has(#theme-toggle:checked)` / sibling combinator inversion, TrackPoint-red accents).
- [ ] Add minimal SVG/PNG favicons to `themes/T480/static/` and wire clean root-relative `<link>` tags in `head.html`.
- [ ] Ensure `content/journal/test.md` has `draft = true` (or remove it) and add `content/journal/_index.fr.md` if needed.
- [ ] Run `hugo --gc --minify --panicOnWarning` and `xmllint --noout public/index.xml` locally until green.
- [ ] Push `themes/T480/` and verify GitHub Actions build + Pages deploy turn green.
- [ ] Write and publish the follow-up post documenting the rebuild.