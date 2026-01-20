# The Agentic Web - Project Guide

## Overview

Marketing website for "The Agentic Web" - a platform promoting AI-native endpoints for enterprise. The site explains how companies can create endpoints that AI agents can interact with, providing verified responses and enabling actions.

**Live site:** https://agentic-web.ai/

## Tech Stack

- **Frontend:** Static HTML/CSS (no build system)
- **Fonts:** Outfit, Plus Jakarta Sans, JetBrains Mono (Google Fonts)
- **Deployment:** AWS Amplify
- **No dependencies** - vanilla HTML/CSS/JS

## Project Structure

```
├── index.html         # Main landing page
├── nlweb.html         # NLWeb standards & how Agentic Web extends it
├── sequence.html      # Interaction sequence diagram
├── specification.html # Technical MCP extension specification
├── sitemap.xml        # SEO sitemap
├── robots.txt         # Search engine crawling rules
├── favicon.svg        # Site favicon
├── amplify.yml        # AWS Amplify deployment config
└── CLAUDE.md          # This file
```

## Running Locally

Since this is a static site with no build step, use any HTTP server:

### Python (built-in)
```bash
python -m http.server 8000
# Then open http://localhost:8000
```

### Node.js
```bash
npx serve
# Or install globally: npm install -g serve && serve
```

### VS Code
Install "Live Server" extension, then right-click `index.html` → "Open with Live Server"

## Pages & SEO

| Page | URL Path | Purpose |
|------|----------|---------|
| Home | `/` | Main landing, value proposition |
| NLWeb | `/nlweb.html` | NLWeb standards explanation |
| Sequence | `/sequence.html` | Interaction flow diagram |
| Specification | `/specification.html` | Technical MCP spec |

## Design System

### Colors (CSS Variables)
- `--accent-cyan: #1E90FF` - Primary brand color
- `--accent-amber: #FF6B35` - Secondary accent
- `--accent-emerald: #10b981` - Success states
- `--bg-primary: #030712` - Dark background

### Fonts
- **Display:** Outfit (headings)
- **Body:** Plus Jakarta Sans
- **Code:** JetBrains Mono

## Deployment

Push to `main` branch triggers automatic AWS Amplify deployment.

## Common Tasks

### Adding a new page
1. Create `newpage.html` in root
2. Add navbar (copy from existing page)
3. Update `sitemap.xml` with new URL
4. Add link to navbar on all pages if needed

### Updating SEO
- Each page has `<title>` and `<meta name="description">` tags
- Open Graph and Twitter Card meta tags on index.html
- Update `sitemap.xml` when adding/removing pages
