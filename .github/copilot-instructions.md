# Copilot Instructions — techmyth.github.io

## Overview

Hugo-based bilingual blog (Spanish + English) deployed to GitHub Pages via the PaperMod theme (Git submodule). Default language is Spanish; English is secondary.

## Build & Preview

```bash
# Local development server with live reload
hugo server

# Build for production (output → ./public/)
hugo --minify

# Build with explicit base URL (mirrors CI)
hugo --minify --baseURL "https://techmyth.github.io/"
```

Hugo extended edition is required (`extended: true` in the CI workflow). The PaperMod theme lives in `themes/PaperMod/` as a Git submodule — always run with `--recurse-submodules` on fresh clones.

## Repository Structure

```
config.yml          # Site-wide config: languages, menus, params, services
archetypes/         # default.md — template for new posts (draft: true)
content/
  posts/            # All blog posts — bilingual pairs (*.es.md / *.en.md)
  about.{es,en}.md
  projects.{es,en}.md
  search.{es,en}.md
static/
  img/              # Images organised by year/post-slug/
    2025/abr-veeam-vbr-0_8_22/
    2026/abr-veeam-vbr-1_0_1/
layouts/
  _default/_markup/render-link.html   # Opens external links in new tab
  partials/comments.html              # Giscus comments integration
  shortcodes/embed-pdf.html           # PDF.js inline PDF viewer
themes/PaperMod/    # Git submodule — do not edit directly
.github/workflows/
  hugo.yml          # CI: build + deploy to GitHub Pages on push to main
  dependabot.yml
```

## Content Conventions

### Bilingual posts

Every post must have **both** language files. Filename pattern:

```
content/posts/<slug>.es.md   # Spanish (primary)
content/posts/<slug>.en.md   # English
```

### Front matter

```yaml
---
title: 'Post title'
date: '2026-04-22T10:00:00-04:00'   # ISO 8601 with timezone offset
draft: false
tags:
    - TAG1
    - TAG2
---
```

- `draft: true` while writing; set to `false` to publish.
- Tags use title-case; common tags: `VEEAM`, `AsBuiltReport`, `Powershell`, `VMware`, `NetApp`, `Powershell`.

### Images

Images are stored under `static/img/<year>/<post-slug>/` and referenced with an absolute path:

```markdown
![alt text](/img/2026/abr-veeam-vbr-1_0_1/image-name.webp)
```

Preferred format is **WebP**. The `<year>` directory corresponds to the post's publication year.

### Post anatomy (blog style)

Posts follow a consistent narrative structure:
1. Opening greeting: `Hola desde el Caribe 🥥🌴🌺🌅🌊,` (ES) / `Hello from the Caribbean 🥥🌴🌺🌅🌊,` (EN)
2. Body sections with `####` headings
3. Changelog block in a fenced ` ```markdown ``` ` code block
4. Sample report link
5. Closing: `¡Hasta la Próxima!`
6. Ko-fi donation badge:
   ```markdown
   [![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F8DEV80)
   ```

### Shortcodes

Embed a PDF inline:

```markdown
{{< embed-pdf url="/path/to/file.pdf" >}}
```

Optional params: `hidePaginator="true"`, `hideLoader="true"`, `renderPageNum="2"`.

## Deployment

Pushes to `main` trigger the `hugo.yml` workflow which builds and deploys to GitHub Pages automatically. The `public/` directory is the build artifact — it is not committed.
