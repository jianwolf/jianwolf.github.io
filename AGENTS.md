# Repository Guidelines

## Project Structure & Module Organization
- Root-level static site: `index.html` holds all content, `style.css` holds all styles.
- Assets live at the repo root: `jian_mount_si_washington.jpg` (profile photo) and `resume_jianwang.pdf`.
- `CNAME` defines the custom GitHub Pages domain.
- `CLAUDE.md` documents constraints and design intent for contributors and agents.

## Build, Test, and Development Commands
This site has no build system or dependencies; edit files directly.
- Local preview: open `index.html` in a browser.
- Optional static server for link/path parity: `python3 -m http.server` (then visit `http://localhost:8000`).
- Deployment: push changes to the default branch and GitHub Pages serves the update.

## Coding Style & Naming Conventions
- Indentation: 4 spaces in HTML and CSS.
- HTML: prefer semantic elements (`header`, `section`, `nav`); keep markup minimal and readable.
- CSS: keep custom properties in `:root` and reuse them; class names use kebab-case (e.g., `profile-photo`).
- Avoid JavaScript and frameworks; keep the lightweight, responsive layout intact.

## Testing Guidelines
No automated tests currently.
- Manual checks: load the page, verify layout at mobile and desktop widths (breakpoints at ~576px/600px),
  and confirm external links plus `resume_jianwang.pdf` download.

## Commit & Pull Request Guidelines
- Commit messages are short, sentence-case, and action-oriented (e.g., "Update resume_jianwang.pdf",
  "Optimize site: ...").
- PRs should include a brief summary, note any asset changes, and add before/after screenshots for layout or
  typography updates. Link related issues if they exist.

## Agent-Specific Instructions
- Follow the constraints in `CLAUDE.md` (pure HTML/CSS, minimal grid, clean aesthetic, Google Fonts usage).
