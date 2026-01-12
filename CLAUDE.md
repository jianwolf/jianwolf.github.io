# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a personal portfolio website (jianwang.me) hosted on GitHub Pages. It's a static HTML site with no build system or dependencies.

## Development

**No build commands required.** Edit files directly and push to master - GitHub Pages deploys automatically.

**Local preview:** Open `index.html` directly in a browser.

## Architecture

```
├── index.html                    # Single-page site (all content here)
├── style.css                     # Custom styles with minimal grid
├── resume_jianwang.pdf           # Linked resume
├── CNAME                         # Custom domain (jianwang.me)
└── jian_mount_si_washington.jpg  # Profile photo
```

## Typography

| Element | Font | Weight | Why |
|---------|------|--------|-----|
| Headings (H1) | Libre Baskerville | Bold (700) | Signals "I write, I think, I have taste." |
| Headings (H2) | Libre Baskerville | Regular (400) | Same refined feel, lighter hierarchy |
| Body | Inter | Regular (400) | Signals "I am modern, clean, and practical." |

Fonts loaded from Google Fonts with `display=swap` for performance.

## Key Constraints

- Pure HTML/CSS only (no JavaScript, no frameworks)
- Custom minimal grid (~20 lines replaces Bootstrap)
- Mobile-first responsive design (breakpoints at 576px for grid, 600px for profile photo)
- Keep the minimal, clean aesthetic
