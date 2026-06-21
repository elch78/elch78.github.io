# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

This is a Jekyll-based static site (version 4.4.1) configured for GitHub Pages hosting at elch78.github.io. The site uses the Minima theme and jekyll-feed plugin.

## Development Environment

The project uses a DevContainer setup with:
- Jekyll 2 on Debian Bullseye
- Node.js LTS
- Claude Code pre-installed
- RubyMine backend for JetBrains IDEs
- Port 4000 forwarded for local preview

## Common Commands

The project is developed inside a VM without a local Ruby toolchain. Jekyll runs
in a Docker container defined by `docker-compose.yml` (Ruby 3.3 image). Gems are
installed into a named volume (`bundle`) on first start and cached afterwards.

### Build and Serve
```bash
docker compose up        # foreground, logs visible
docker compose up -d     # detached
```
Runs `bundle install` then `bundle exec jekyll serve` inside the container.
Serves at http://localhost:4000 with auto-regeneration (`--force_polling` for
reliable file watching on mounted volumes) and LiveReload on port 35729.

```bash
docker compose down      # stop and remove the container
docker compose logs -f   # follow logs
```

### Build Only
```bash
docker compose run --rm jekyll bundle exec jekyll build
```
Generates static site to `_site/` directory without starting a server.

### Update Dependencies
```bash
docker compose run --rm jekyll bundle install
```
Run after Gemfile changes (it also runs automatically on `docker compose up`).

## Architecture

### Key Configuration
- `_config.yml`: Site-wide settings (title, description, theme, plugins)
  - **Important**: Changes require server restart (not hot-reloaded)
- `Gemfile`: Ruby dependencies and Jekyll plugins

### Content Structure
- `_posts/`: Blog posts following `YEAR-MONTH-DAY-title.MARKUP` naming convention
  - Must include YAML front matter with layout, title, date, categories
- `_site/`: Generated output (git-ignored, never edit directly)
- `about.markdown`, `index.markdown`: Static pages
- `404.html`: Custom error page

### Layouts and Theming
Site uses Minima theme. Theme files are in gem, not repository. To customize:
1. Copy files from gem to project (find location with `bundle info minima`)
2. Override specific files in `_layouts/`, `_includes/`, or `_sass/` directories

### Front Matter
All content files use YAML front matter:
```yaml
---
layout: post
title: "Title Here"
date: YYYY-MM-DD HH:MM:SS +0000
categories: category1 category2
---
```
