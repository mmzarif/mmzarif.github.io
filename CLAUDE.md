# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal portfolio website for Mustahsin Zarif, hosted on GitHub Pages at mmzarif.github.io. Built on the "Free to Engineer" Jekyll template.

## Development Commands

```bash
bundle install          # Install dependencies
bundle exec jekyll serve   # Local dev server at http://localhost:4000
bundle exec jekyll build   # Build to _site/
```

## Architecture

- **Static site generator**: Jekyll with the `github-pages` gem (no standalone Jekyll gem)
- **Templating**: Liquid templates with Kramdown markdown
- **Styling**: Single CSS file at `css/styles.css` with theme colors defined in `_config.yml`

### Key Files

- `_config.yml` — All site configuration: personal info, social links, skills list, color theme, and Jekyll build settings. The `colors:` block is the active theme; other theme blocks are examples only.
- `_layouts/wrapper.html` — Base HTML shell (head, navbar, footer)
- `_layouts/post.html` — Project detail page layout
- `index.html` — Home page, composes includes: hero, projects grid, skills, contact form

### Projects (Collection)

Projects live in `_projects/{project-name}/index.md` with associated images in the same folder. Each project requires YAML frontmatter:

```yaml
---
layout: post
title: Project Title
description: Short description
skills:
  - Skill 1
  - Skill 2
main-image: /image-filename.png
---
```

The `main-image` path is relative to the project folder (prefixed automatically) unless it contains "http". Projects are sorted by date (reverse) on the home page.

### Includes

- `hero.html` — Profile section with image, name, headline, social links
- `projects.html` — Grid of project cards (used on both index and /projects/)
- `skills.html` — Skills grouped by category from `_config.yml`
- `contact.html` — Formspree-powered contact form
- `image-gallery.html` — Responsive image embed (usage: `{% include image-gallery.html images="img1.png, img2.png" height="400" %}`)
- `youtube-video.html` — YouTube embed (usage: `{% include youtube-video.html id="11-digit-id" autoplay="false" %}`)

### Reference

`Reference/template.md` contains a blank project frontmatter template for creating new projects.
