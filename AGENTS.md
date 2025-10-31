# Agents Guidelines

- This project intentionally follows the Sembr “one sentence per line” approach when editing long-form prose in HTML. See https://sembr.org/ for the rationale: Sembr keeps each sentence on its own line to simplify diff reviews, surface small changes, and avoid noisy merges. Please preserve that formatting when contributing new copy.
- Avoid em dashes; prefer commas, colons, or ASCII hyphen-minus substitutions so the site stays typographically consistent.
- Pages are rendered via Jekyll layouts; include the appropriate front matter and reuse the shared `_layouts/default.html` structure when adding or editing pages.
- Manage dependencies with Bundler (`bundle install`) and preview with `bundle exec jekyll serve` to keep local builds aligned with GitHub Pages.
- Deployments go out via GitHub Pages, so commits merged to the `gh-pages` branch publish the site.
