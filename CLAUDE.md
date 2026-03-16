# The Agentic Web - Project Guide

## Overview

Marketing website for "The Agentic Web" - a platform for turning any business into an AI-native endpoint using MCP. The site explains how companies can create MCP servers that AI agents connect to directly, providing verified responses and enabling actions. Not enterprise-only — this is for any business (Wix sites, B2C, B2B, bakeries, SaaS).

**Live site:** https://agentic-web.ai/

## Tech Stack

- **Frontend:** Static HTML/CSS (no build system)
- **Fonts:** Outfit, Plus Jakarta Sans, JetBrains Mono (Google Fonts)
- **Deployment:** AWS Amplify
- **No dependencies** - vanilla HTML/CSS/JS

## Project Structure

```
├── index.html         # Main landing page
├── specification.html # MCP Server Profile specification
├── blog/index.html    # "Under the Hood" technical blog post
├── sequence.html      # Interaction sequence diagram
├── nlweb.html         # NLWeb standards & how Agentic Web extends it
├── why.html           # Why it's good for everyone (buyers/vendors/LLMs)
├── a2a-chat.html      # A2A chat replay demo
├── a2a-sequence.html  # A2A interaction sequence (cinematic)
├── blog-agentic-web.md # Blog post source content (markdown)
├── TODOS.md           # Tracked follow-up work
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
| Specification | `/specification.html` | MCP Server Profile spec (JSON-RPC 2.0) |
| Blog | `/blog/` | "Under the Hood" technical blog post |
| Sequence | `/sequence.html` | Interaction flow diagram |
| NLWeb | `/nlweb.html` | NLWeb standards explanation |
| Why | `/why.html` | Value prop for buyers, vendors, LLMs |
| A2A Chat | `/a2a-chat.html` | Agent-to-agent chat replay |
| A2A Sequence | `/a2a-sequence.html` | Agent-to-agent cinematic sequence |

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
