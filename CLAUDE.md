# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a personal portfolio website (jianwang.me) hosted on GitHub Pages. It's a static HTML site with no build system or dependencies.

## Development

**No build commands required.** Edit files directly and push to master - GitHub Pages deploys automatically.

**Local preview:** Open `index.html` directly in a browser.

## Architecture

```
├── index.html              # Single-page site (all content here)
├── css/
│   ├── main.css           # Custom styles
│   └── bootstrap.min.css  # Bootstrap 5 framework
├── resume_jianwang.pdf    # Linked resume
├── CNAME                   # Custom domain (jianwang.me)
└── jian_*.jpg             # Profile photos
```

## Typography

| Element | Font | Weight | Why |
|---------|------|--------|-----|
| Headings (H1) | Libre Baskerville | Bold (700) | Signals "I write, I think, I have taste." |
| Headings (H2) | Libre Baskerville | Regular (400) | Same refined feel, lighter hierarchy |
| Body | Inter | Regular (400) | Signals "I am modern, clean, and practical." |
| Code | JetBrains Mono | Regular (400) | Signals "I am a serious engineer who cares about tooling." |

Fonts loaded from Google Fonts with `display=swap` for performance.

## Key Constraints

- Pure HTML/CSS only (no JavaScript)
- Bootstrap 5 for responsive layout
- Maintain mobile-first responsive design (breakpoints at 800px and 600px in main.css)
- Keep the minimal, clean aesthetic
