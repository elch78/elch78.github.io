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

### Build and Serve
```bash
bundle exec jekyll serve
```
Starts development server at http://localhost:4000 with auto-regeneration enabled.

### Build Only
```bash
bundle exec jekyll build
```
Generates static site to `_site/` directory without starting a server.

### Install Dependencies
```bash
bundle install
```
Run after Gemfile changes or initial setup.

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
