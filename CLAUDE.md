# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Personal website (thilohohlt.com) built with the [Zola](https://www.getzola.org) static site generator. No JS build step, no package manager, no tests.

## Commands

- `zola serve`: local dev server with live reload
- `zola build`: build into `public/` (gitignored)
- `zola check`: build and validate, including external links
- `sh optimize-images.sh`: generate WebP versions of new photos (needs ImageMagick `magick`)

Deployment: every push to `main` builds and publishes to GitHub Pages through `.github/workflows/deploy.yml` (`shalzz/zola-deploy-action`). Keep the action's version in step with the local Zola version (currently 0.23.x).

## Zola 0.23 / Tera v2 constraints

Templates use Tera v2, which is stricter than v1:
- Accessing a field of an undefined variable is an error. `base.html` is rendered for pages, sections and the 404 page, so it uses optional chaining (`page?.title or section?.title or "..."`) instead of `default()`.
- A child template can only override blocks the parent defines, and `{% endblock name %}` must match its opening name.
- `%+` is not a supported date format. Use `%Y-%m-%dT%H:%M:%S%:z` for ISO 8601.

## Architecture

- `templates/base.html` holds the whole page shell (nav, header with title/date, footer, inline SVG icons). Child templates fill `content` and optionally `body_scripts`, `title`, `description` and `header`.
- CSS is not linked. `sass/style.scss` is compiled by Zola (`compile_sass = true`), and `base.html` inlines the result with `load_data(path="style.css")` inside a `<style>` tag.
- Each content page chooses its template in front matter (`template = "..."`). `content/blog/_index.md` sets `blog.html` for the paginated list (`paginate_by = 3`) and `page_template = "blog-post.html"` for posts.
- Blog posts live in `content/blog/YYYY-MM-DD_slug/index.md` as page bundles, with co-located images. Front matter has `title`, `description`, `authors` and `date`.
- Search: `build_search_index = true` with `index_format = "fuse_json"` writes `search_index.en.json`. `templates/search.html` fetches it on the client and searches it with Fuse.js loaded from jsDelivr.
- RSS goes to `rss.xml`, using `title`/`description` from `zola.toml`.

### Photography gallery

- Originals: `static/photography/DSC*.JPG`. WebP versions: `static/photography/optimized/DSC*.WEBP`. Both are stored in **Git LFS** (see `.gitattributes`), and CI checks out with `lfs: true`.
- The photo list is data in `content/photography/index.md` front matter: `[[extra.photos]]` entries with `file`, `caption` and `year`. `photography.html` renders them in reverse order (newest at the bottom of the file shows first) and maps `X.JPG` to `optimized/X.WEBP` by string replacement. Filenames therefore have to keep the uppercase `.JPG` extension.
- To add a photo: copy the JPG into `static/photography/`, run `optimize-images.sh` (it skips files that already exist), then append an `[[extra.photos]]` entry.
- The lightbox works through focus (`tabindex` plus CSS `:focus`), with arrow keys, prev/next buttons and Escape handled by a small inline module script.
