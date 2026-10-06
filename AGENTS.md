# AGENTS.md

Personal blog built with Jekyll 3.9 (`github-pages` gem) and the Jasper2 theme. Ruby 2.7.4 (`.ruby-version`).

- Serve locally: `bundle exec jekyll serve` → http://localhost:4000/blog/ (`baseurl: /blog/`)
- Posts: `_posts/YYYY-MM-DD-slug.markdown`; images: `assets/images/` (keep them small — resize to ≤1920px wide)
- Custom plugins in `_plugins/` (so the site is built in CI, not by GitHub's safe-mode builder)
- Deploy: every push to `master` runs `.github/workflows/deploy.yml`, which builds `_site` and deploys it with GitHub Pages (source: GitHub Actions). There is no `gh-pages` branch.

## Commits

English [Conventional Commits](https://www.conventionalcommits.org/): `type: summary`, e.g. `feat: add post \`rubyconftw-2023\``, `fix: ...`, `chore: ...`, `ci: ...`, `docs: ...`.
