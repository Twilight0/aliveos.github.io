# AliveOS

A minimalist, Arch-based, developer-optimized Linux distribution.

## Website

This repository hosts the static landing page for the AliveOS project, deployed
via [GitHub Pages](https://pages.github.com/) at **https://aliveos.org**.

## Structure

```
.
├── index.html          # Landing page
├── aliveos-logo.png    # Logo referenced by index.html
├── CNAME               # Custom domain (aliveos.org)
├── .nojekyll           # Disables Jekyll processing on GitHub Pages
└── README.md
```

## Local preview

No build step required. Serve the directory with any static file server, e.g.:

```sh
python3 -m http.server 8000
```

Then open http://localhost:8000.

## Deployment

Push to the `main` branch. GitHub Pages must be enabled in
**Settings > Pages** with the source set to this branch. The `CNAME` file
configures the custom domain `aliveos.org`.
