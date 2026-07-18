# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Personal blog/portfolio site for Vaibhav Kumbhar, built with **Jekyll** (via the `github-pages` gem) using the remote theme `mmistakes/minimal-mistakes` (v4.24.0, `minimal_mistakes_skin: default`). No Node/JS build step — pure Ruby/Jekyll, no `package.json`. Deployed automatically via GitHub Actions to GitHub Pages at **https://vkumbhar94.github.io/personal-website/** (a project page, not a custom domain — no CNAME file). No Ruby version is pinned (no `.ruby-version`, no `ruby` directive in the `Gemfile`).

## Setup & common commands

```bash
bundle install                 # install gems (github-pages, jekyll-* plugins)
bundle exec jekyll serve       # local dev server -> http://localhost:4000/personal-website/
bundle exec jekyll build       # production build -> _site/
```

> **Heads-up:** these commands currently **fail on the maintainer's machine** — the `Gemfile` pins `liquid ~> 4.0.4` (needs Ruby `< 4.0`) against an installed Ruby 4.0.2, so `bundle install` can't resolve. See Known issues; verification usually happens via the deployed site instead.

- There is no separate "preview a production build" script; serve `_site/` with any static file server, or just use `jekyll serve` (it rebuilds and serves with the same permalink/baseurl behavior as `build`).
- Analytics (GTM) only renders in production builds, so it is intentionally absent from `jekyll serve` output.
- No lint, format, or test tooling exists in this repo (no `.rubocop.yml`, no `.eslintrc`, no test directory). The only "check" that runs is the build itself, via CI.
- `Gemfile.lock` and `_site/` are gitignored (`.gitignore`), so `bundle install` will generate a fresh lockfile locally each time.

## Repository structure

```
_config.yml           # Site-wide Jekyll config: title, author, nav defaults, plugins, front-matter defaults
_data/navigation.yml  # Top nav bar links (Posts, Categories, Tags, About, Resume)
_layouts/default.html # Local override of the theme's default layout (see Styling notes below)
_includes/            # Local overrides of theme includes (all deliberate — see Styling notes)
  head/custom.html                 # Injected into <head>: light/dark theme toggle (CSS + JS)
  author-profile.html              # Vendored theme include; adds target="_blank" to author.links
  analytics-providers/custom.html  # Google Tag Manager container snippet
_pages/                # Standalone pages, rendered via `include: [_pages]` in _config.yml
  about.md              # /about/ — bio page
  404.md                # /404.html — not-found page
  category-archive.md   # /categories/ — auto-generated category index
  tag-archive.md         # /tags/ — auto-generated tag index
  year-archive.md        # /posts/ — chronological post index (this is what "/posts/" on the live site is)
_posts/blogs/           # Blog posts (only subfolder currently in use)
assets/images/           # All images, referenced from post/page markdown via /assets/images/...
.github/workflows/jekyll-docker.yml  # CI/CD: builds and deploys to GitHub Pages
Gemfile                  # Ruby gem dependencies (github-pages, jekyll plugins)
README.md                # **Rendered as the site home page (/)** — see Known issues; also duplicates the About bio copy
```

There is no `_sass/` directory — all theme styling comes from the remote theme. `_includes/` and `_layouts/` contain only the deliberate local overrides listed above; every other include/layout resolves to the remote theme's own files.

## Content authoring conventions

### Adding a new blog post

- File location: `_posts/blogs/`
- Filename pattern: `YYYY-MM-DD-Title-With-Dashes.md` (must match the post's `date`)
- Front matter (from the 3 existing posts):
  ```yaml
  ---
  title: "Post Title"
  date: 2025-08-10
  categories:
    - blog
  tags:
    - some-tag
    - another-tag
  ---
  ```
  - `title` (required, string)
  - `date` (required, `YYYY-MM-DD`)
  - `categories` (required — list; every existing post uses `blog` as its only category)
  - `tags` (optional — list, free-form kebab-case)
  - Do **not** set `layout`, `author_profile`, `read_time`, `comments`, `share`, `related`, or `seo` per-post — these are all supplied by the `defaults` block in `_config.yml` for everything under `_posts`.
- Use `<!--more-->` in the body if you want an explicit excerpt break (configured as `excerpt_separator` in `_config.yml`); otherwise Jekyll auto-truncates.
- Post body is plain Markdown (kramdown), fenced code blocks work as shown in existing posts (e.g. ` ```go `).

### Adding a "portfolio" item

There is **no dedicated portfolio content type, directory, or layout** in this repo, and no category other than `blog` is currently in use. Treat "portfolio item" as: a normal post in `_posts/blogs/` (or a new `_posts/<subfolder>/` — subfolder doesn't affect routing) with a `categories:` value such as `[projects]`. Confirm the category name with the user before inventing one, since none exist yet and category pages are generated from whatever posts declare.

## Deployment

- Workflow: `.github/workflows/jekyll-docker.yml`, name "Build Jekyll site"
- Trigger: automatic on every push to `main` (no manual dispatch configured)
- Steps: checkout → `actions/configure-pages@v5` → `actions/jekyll-build-pages@v1` (builds the site) → `actions/upload-pages-artifact@v3` → `actions/deploy-pages@v4`
- Requires the repo's **Settings → Pages → Source** to be set to "GitHub Actions" (not "Deploy from a branch"). There is no `gh-pages` branch and no `/docs` folder involved.
- No `baseurl` is set in `_config.yml`; `actions/jekyll-build-pages` derives and injects the correct base path (`/personal-website`) automatically from the Pages configuration at build time. Do not hardcode `/personal-website` into `_config.yml`'s `baseurl` — it isn't needed and could conflict with the Action's injection.
- No manual steps beyond merging/pushing to `main`; there's no staging environment.

## Styling / theming notes

- Visual theme comes entirely from the **remote theme** `mmistakes/minimal-mistakes` (declared in `_config.yml` as `remote_theme: mmistakes/minimal-mistakes`, skin set via `minimal_mistakes_skin: default`). To change the color skin, change `minimal_mistakes_skin` in `_config.yml` (theme ships several named skins) — do not try to hand-edit theme CSS, since none of it is vendored into this repo.
- `_layouts/default.html` is a **local override** of the theme's own `default.html` (copied near-verbatim, per the "Minimal Mistakes Jekyll Theme 4.24.0" header comment inside it), plus one local addition: the Google Tag Manager `<noscript>` iframe right after `<body>`. It references `{% include head/custom.html %}` and `{% include footer/custom.html %}` hooks. `_includes/head/custom.html` **now exists locally** (it holds the dark-mode toggle); `_includes/footer/custom.html` does not and still resolves to the theme's empty stub. To inject custom head/footer HTML, edit/create those hook files rather than editing `_layouts/default.html` further.
- **Vendored theme files go stale.** `_layouts/default.html` and `_includes/author-profile.html` are pinned copies, so they will **not** pick up upstream theme fixes. If layout or the author sidebar looks broken after a theme update, diff them against the current upstream `mmistakes/minimal-mistakes` versions. (`author-profile.html` carries a header comment saying exactly this.)
- **Dark mode** is a local feature, not a theme skin. Minimal Mistakes compiles a single skin at build time, so the runtime toggle lives entirely in `_includes/head/custom.html`: a floating button, `data-theme="dark"` set on `<html>`, `localStorage` persistence, `prefers-color-scheme` as the default, a pre-paint script to avoid a flash, and a dark CSS layer that overrides the light skin's surfaces (uses `!important` deliberately, since the theme bakes colors into element selectors). To restyle dark mode, edit that file — do not change `minimal_mistakes_skin`.
- **Analytics is Google Tag Manager**, wired via the theme's `custom` analytics provider: `_config.yml` sets `provider: "custom"` with the GTM container id in `analytics.google.tracking_id` (`GTM-NLK6G9NP`), and `_includes/analytics-providers/custom.html` emits the container snippet. The theme only injects analytics when `jekyll.environment == 'production'`, so GTM never loads on local `jekyll serve`. **The GA4 Measurement ID (`G-…`) intentionally does not appear anywhere in this repo** — GA4 is delivered by a tag configured *inside* the GTM container. Seeing a `G-` id in the page source would mean a duplicate, double-counting install.
- **External links open in a new tab**, done three different ways depending on what renders the link: nav links use a `target:` field per entry in `_data/navigation.yml` (theme-native); the author sidebar icons need the vendored `_includes/author-profile.html` (the theme hardcodes those anchors with no `target`); in-body Markdown links use a kramdown attribute list, e.g. `[text](url){:target="_blank" rel="noopener"}`. Footer links (`footer.links` in `_config.yml`) are still same-tab — making them new-tab would require vendoring the theme's `footer.html` too.

## Conventions & style guide

- **Commit style**: Conventional-ish, lowercase, `type(scope): short description`, e.g. `fix(resume): remove resume page and link to resume site`, `feat(resume): pandoc resume`. Types seen: `fix`, `feat`, `ci`. There is no enforced/linted convention (no commitlint), just an observed pattern — follow it.
- No branch-naming convention is evident; recent history is linear commits directly reflected on `main`.
- No linting/formatting config exists for Markdown, YAML, or Ruby — match the existing front-matter key order and indentation style shown in existing posts/pages when adding new files.

## Known issues / gotchas

- **`README.md` IS the site home page.** There is no `index.html`/`index.md` and no `_pages` file with `permalink: /`. GitHub Pages' `jekyll-readme-index` (bundled in the `github-pages` gem) therefore renders `README.md` at the site root, using the `home` layout — so the live home page is README's "About Me" prose followed by the paginated Recent Posts feed. **Editing `README.md` changes the live home page.** It also duplicates the bio copy in `_pages/about.md`, and the two are **not** auto-synced — update both when changing bio content.
- **Local builds currently fail on this machine.** `bundle install` can't resolve because the `Gemfile` pins `liquid ~> 4.0.4` (which requires Ruby `< 4.0`) while the installed Ruby is 4.0.2. No Ruby version is pinned in the repo (`.ruby-version` absent). CI is unaffected — `actions/jekyll-build-pages` runs its own compatible Ruby in Docker — so changes can only be verified by pushing and checking the deployed site, or by installing an older Ruby locally (e.g. via rbenv). Factor this in before promising a local preview.
- **~189 orphaned images**: `assets/images/` (ds_portfolio/, gpt-screenshots/, llm-comparison/, linkedin-post-analysis/, viz_journey/, viz_retriever/, plus loose MakeoverMonday files) are left over from previously-deleted blog posts (data-viz posts removed in commit `13cfddf`). **No current post references any file under `assets/images/`.** The owner has explicitly decided to **keep** these for reuse in future posts — do not propose deleting them again.
- **Only the `blog` category exists.** Every post uses `categories: [blog]`. Earlier site copy advertised `data-viz`, `reading-notes`, and `projects` categories that were never populated; that copy has since been corrected. Don't assume any other category exists.
- **`jekyll-algolia` was removed** from the `Gemfile` — the site's `search: true` uses minimal-mistakes' built-in Lunr search. Don't re-add Algolia without config to match.

## Quick task recipes

**Publish a new blog post**
1. Create `_posts/blogs/YYYY-MM-DD-Title-With-Dashes.md` (date = today or intended publish date, must match filename).
2. Add front matter: `title`, `date`, `categories: [blog]` (or another category — see caveat above), optional `tags`.
3. Write the post body in Markdown below the `---` closing front matter.
4. Commit to `main` (directly or via PR) — the `jekyll-docker.yml` workflow builds and deploys automatically on push to `main`. No other steps needed.

**Add a new portfolio/project post**
1. Same mechanics as a blog post, but ask the user which `categories` value to use — no category other than `blog` exists yet, so there is no prior example to copy from.
2. Place images under a new subfolder in `assets/images/` (e.g. `assets/images/<project-name>/`) and reference them from the post body as `/assets/images/<project-name>/file.png` (relative to site root; no baseurl prefix needed).

**Update site metadata / bio**
1. Site-wide title/description/social links: edit `_config.yml` (`title`, `description`, `author`, `footer`).
2. Bio copy: it lives in **two** places that are not auto-synced — `_pages/about.md` (the `/about/` page) and `README.md` (**the home page**). Update both.
3. Nav bar links/order: edit `_data/navigation.yml` (add `target: "_blank"` on an entry to open it in a new tab).
4. Restart `bundle exec jekyll serve` after editing `_config.yml` — it is **not** hot-reloaded (noted in the file's own header comment).

**Change or verify analytics**
1. The GTM container id lives in `_config.yml` → `analytics.google.tracking_id`; the snippet is `_includes/analytics-providers/custom.html` and the `<noscript>` is in `_layouts/default.html`.
2. GA4 tags/measurement IDs are configured **in the GTM UI**, not this repo — changes there need no deploy.
3. To verify a deploy actually shipped the container:
   `curl -fsSL https://vkumbhar94.github.io/personal-website/ | grep -o "GTM-[A-Z0-9]*"` (expect 2 hits: script + noscript).

**Fix a broken image link**
1. Confirm the image exists under `assets/images/` (`find assets/images -iname "*<name>*"`).
2. Reference it in Markdown as `![alt](/assets/images/path/to/file.png)` — root-relative, no `baseurl` prefix (the theme/Jekyll handles the `/personal-website` prefix automatically via `relative_url`/`site.url` in the theme's own templates; for raw Markdown image syntax in post bodies, this repo's existing posts don't currently embed any images, so there's no in-repo precedent — verify rendering with `bundle exec jekyll serve` locally before pushing).
