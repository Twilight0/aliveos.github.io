# AliveOS

A minimalist, Arch-based, developer-optimized Linux distribution.

## Website

This repository hosts the project landing page **plus a Jekyll-powered news
blog**, deployed via [GitHub Pages](https://pages.github.com/) at
**https://aliveos.org**.

GitHub Pages builds the site with Jekyll automatically on every push &mdash;
there is no build step to run locally. Markdown news posts become HTML pages
under `/news/<slug>/`.

## Structure

```
.
├── _config.yml          # Jekyll config (url, permalink, defaults)
├── _layouts/
│   ├── default.html     # Shared shell: <head>, CSS, header + nav, footer
│   └── post.html        # Single news post wrapper (uses default)
├── _posts/              # News posts as YYYY-MM-DD-title.md
├── news/index.html      # News listing (loops over site.posts)
├── index.html           # Landing page (uses default layout)
├── assets/              # Logo, favicons, background wallpaper
├── CNAME                # Custom domain (aliveos.org)
└── README.md
```

## Writing a news post

Create a file in `_posts/` named `YYYY-MM-DD-my-post-slug.md` with front matter:

```markdown
---
layout: post
title: "My Post Title"
date: 2026-07-03 15:30:00 +0000
author: Your Name
---

Your markdown content here...
```

The URL becomes `https://aliveos.org/news/my-post-slug/`. Commit and push &mdash;
GitHub Pages rebuilds and publishes automatically. You can also edit posts
directly in GitHub's web editor.

## Local preview (optional)

```sh
# Requires Ruby + Bundler
gem install bundler
bundle init && bundle add github-pages
bundle exec jekyll serve
```

Then open http://localhost:4000.

## Custom domain

`aliveos.org` is configured via the `CNAME` file and Cloudflare DNS (A/AAAA
records pointing at GitHub Pages, grey-clouded / DNS-only). HTTPS is enforced
on the GitHub Pages side.
