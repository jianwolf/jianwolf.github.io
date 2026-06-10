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
├── style.css                     # Custom styles with minimal grid + @font-face
├── fonts/                        # Self-hosted WOFF2 (Inter 400 static, Libre Baskerville variable 400-700) + OFL licenses
├── images/
│   ├── jian_mount_si_washington.jpg  # Profile photo MASTER (2048px, not served)
│   ├── jian_mount_si_washington_{400,800,1600}.{avif,webp}  # Responsive variants (<picture>)
│   ├── jian_mount_si_washington_800.jpg  # Fallback src + og:image
│   ├── favicon.ico               # "J" monogram
│   └── apple-touch-icon.png
├── resume_jianwang.pdf           # Linked resume (keep at root — URL is shared externally)
├── CNAME                         # Custom domain (jianwang.me)
└── web-image-optimization.md     # Notes on image formats & responsive serving
```

## Images

The profile photo is served via `<picture>` (AVIF → WebP → JPEG) with
400/800/1600 rungs for a 320px box (sized with ~25% headroom for zoom/DPR).
To regenerate variants from the 2048px master in `images/`, use Pillow (AVIF quality=60,
WebP quality=80, JPEG quality=82). See `web-image-optimization.md` for the
full rationale.

## Typography

| Element | Font | Weight | Why |
|---------|------|--------|-----|
| Headings (H1) | Libre Baskerville | Bold (700) | Signals "I write, I think, I have taste." |
| Headings (H2) | Libre Baskerville | Regular (400) | Same refined feel, lighter hierarchy |
| Body | Inter | Regular (400) | Signals "I am modern, clean, and practical." |

Fonts are self-hosted in `fonts/` as latin-subset WOFF2 with `font-display: swap`
(no third-party requests). `@font-face` rules live at the top of `style.css`.

## Key Constraints

- **Maintainability is the top priority.** Never trade it for micro-optimizations —
  e.g., never inline `style.css` into `index.html`; keep CSS in its own file.
- Pure HTML/CSS only (no JavaScript, no frameworks)
- Custom minimal grid (~20 lines replaces Bootstrap)
- Mobile-first responsive design (breakpoints at 576px for grid, 600px for profile photo)
- Keep the minimal, clean aesthetic
