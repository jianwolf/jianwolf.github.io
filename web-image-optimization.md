# Web Images: A Field Guide

Notes from optimizing this site's profile photo (June 2026). The original was a
2048×2048, 616KB JPEG displayed in a 320px box — these notes record why that
mattered and everything learned fixing it.

## 1. What counts as "large" on the web

616KB is a rounding error in ML, but web budgets are set by **transfer time on
the critical render path**, not storage:

- **Core Web Vitals**: the Largest Contentful Paint (LCP) element — here, the
  photo — should render in **< 2.5s at the 75th percentile** of real-user
  conditions. Google ranks on this.
- Measured on this site's actual image: 0.4s unthrottled, 0.76s at ~6 Mbps
  (decent 4G), **3.7s at ~1.6 Mbps** (weak signal) — the original blew the
  entire LCP budget by itself on a bad connection.
- The real metric is the **waste ratio**: 2048² = 4.2M pixels shipped; the
  320px CSS box on a 2× display renders 640² = 410K pixels. 10× more pixels
  than any screen would sample — like serving float64 when int8 is bitwise
  identical for the use case.
- "The median page is 2MB" is a descriptive statistic of a bloated ecosystem,
  not a target. Grading against it is grading on the curve of the failing class.

### Why "it loads instantly on my phone" proves nothing

- **Warm cache**: GitHub Pages serves `cache-control: max-age=600` + `ETag`.
  After expiry the browser revalidates and gets `304 Not Modified` — the image
  may never be re-downloaded after the first visit, ever.
- **Friendly sample**: testing from a metro area next to a CDN edge
  (`x-served-by` header shows which) is the p10 of visitor conditions, not p75.
- Web perf is a **tail-latency discipline**: the visitor who matters is on
  conference Wi-Fi or a train. Same as benchmarking inference with the model
  already in GPU memory — n=1 from the training distribution.
- Proper test: Chrome DevTools → Network → "Disable cache" + "Slow 4G"
  throttle → hard reload. Or https://pagespeed.web.dev.

## 2. Pixels: CSS px vs device px

- Browsers lay out in **CSS pixels**; hardware renders each CSS px with
  DPR² device pixels (DPR = device pixel ratio).
- **Physical size is irrelevant.** A 100-inch 8K TV doesn't request more
  pixels — it reports a ~1920×1080 CSS viewport at DPR 4. The worst case in
  existence for a 320 CSS px box is 320 × 4 = **1280 device px**.
- A 6K Pro Display XDR is DPR 2 → just 640 device px. "6K" means a big
  desktop, not denser images.

## 3. `srcset`: one copy per DPR bucket

For a fixed-width box, export one file per DPR class (1×, 2×, 3–4×). The
browser downloads exactly **one**, chosen by `sizes` × its own DPR. For fluid
(full-width) images, use rungs ~1.5–2× apart instead.

**Buffer the rungs.** This site serves 400/800/1600 for a 320px box (~25%
headroom) rather than exact-fit 320/640/1280, because:

- **Browser zoom raises effective DPR** — a 2× screen at 150% zoom needs 3×
  pixels. Pinch-zoom on mobile samples beyond the layout size too.
- Future CSS tweaks (box widened to 360px) need no re-export.
- Slight downscale at render time is benign; upscale is visible.

**Critical gotcha: bare `srcset` cannot negotiate formats.** The browser picks
by size descriptor *before* knowing the file format, so a browser that
understands `srcset` but can't decode the format renders a **broken image** —
the `src` fallback does not rescue it. Single-format `srcset` is only safe
with a universally supported format (WebP). For anything newer, use
`<picture>`, which exists precisely to do `type` negotiation and falls through
gracefully.

## 4. The format wars (what a photographer should know)

The web's rule: **delivery formats are chosen by patent lawyers, not image
quality engineers.** Universal decode support requires royalty-free status.

| Format | Tech | Fate on the web |
|---|---|---|
| **GIF** (1987) | 256-color palette + LZW | Photos get posterized/dithered — that's color *quantization* (24-bit → 8-bit palette), a different disease from codec compression. Still decodes everywhere 39 years later: web formats are immortal. |
| **JPEG** (1992) | 8×8 DCT, Huffman | The compatibility floor. Still the right choice for `og:image`, email, uploads, print labs — the last mile you don't control. |
| **WebP** (2010) | VP8 intra-frame | Royalty-free, ~30% under JPEG, universal since 2020. Limits: 8-bit only, lossy locked to 4:2:0 chroma. In maintenance mode (Google moved on to AVIF) — but "legacy" ≠ dying; decoders never get removed. |
| **HEIC** | HEVC intra + HEIF | Technically excellent, patent minefield (multiple royalty pools) → Chrome/Firefox will never ship it. A capture/storage format (what iPhones shoot), never a delivery format. |
| **AVIF** (2019) | AV1 intra + HEIF | The royalty-free answer to HEIC, built by AOMedia (Google, Apple, Mozilla, Netflix…). ~50% under JPEG, 10/12-bit, HDR, 4:4:4, film grain synthesis. Universal in current browsers since ~2023. |
| **JPEG XL** | — | The photographer's favorite (lossless JPEG recompression −20%, progressive, float HDR). Safari ships it; Chrome removed it in 2023 → not deliverable. Codec politics, again. |

### Who can't decode AVIF (2026)

Current versions of everything support it. The gaps are old hardware:

- **Safari < 16.4** (Mar 2023), and Safari's decoder is the **OS's** —
  iPhones capped at iOS 15 (iPhone 7 and earlier) and Macs stuck before
  Ventura never get it, even though Chrome/Firefox on the same Mac decode it
  fine (they bundle decoders).
- **Edge < 121** (Jan 2024 — Microsoft kept it disabled for years despite
  Chromium).
- Old smart-TV/console browsers, webviews on old iPhones, IE11 kiosks.
- **Link unfurlers and email clients** (Outlook desktop notably) — keep the
  social-card image JPEG.

≈95% global support, but the missing 5% fails *hard* on bare `srcset` —
hence `<picture>`.

### Why serve scrapers at all? (the AI-agent-age question)

"Scrapers" here means every non-browser client that fetches the site — and
for a personal site they may outnumber human visits:

1. **Link unfurlers are the first impression.** Paste jianwang.me into
   Slack, iMessage, LinkedIn, WhatsApp, or Discord and the *platform's bot*
   fetches the page, reads `og:image`, downloads that one URL, and renders a
   preview card — before anyone clicks, and for most people *instead of*
   clicking. A broken card (because the bot can't decode AVIF) is a broken
   first impression with no error message to anyone.
2. **These bots are primitive decoders by design.** They're headless
   fetchers, not browsers: no `<picture>` negotiation, no `srcset` — they
   read exactly one URL from a meta tag and decode it with whatever library
   they ship. JPEG/PNG always work; WebP is spotty; AVIF often fails
   (Outlook desktop renders email with *Word's* engine). You can't negotiate
   with them, so you hand them the lowest common denominator.
3. **Search engines** read the JSON-LD `image` and index for Google Images /
   rich results. Google handles modern formats; the long tail of engines and
   aggregators doesn't.
4. **The AI-age part — a real shift, but it points the same way.** An
   increasing share of traffic is now AI crawlers (GPTBot, ClaudeBot,
   PerplexityBot) and agents fetching the page on a user's behalf ("tell me
   about Jian Wang"). Being legible to *them* is the new SEO. They mostly
   consume text, alt attributes, and JSON-LD rather than pixels — but
   multimodal agents increasingly do fetch images, and they inherit the same
   property as 2010s preview bots: an unknown decoder you can't negotiate
   with. The defense is identical: one universal-format image plus
   machine-readable metadata.

So the rule wasn't *caused* by the AI age — it was caused by social preview
bots circa 2012 — but the AI age re-derives it: the population of
non-browser readers keeps growing and diversifying, and the insurance
premium is one JPEG that doubles as the legacy-browser fallback. Cost:
zero extra files. Payoff: every context that *describes* the site — chat
apps, search results, AI answers — renders it correctly.

(Size matters for scrapers too: platforms re-fetch `og:image` per share,
another reason it's an 800px/113KB JPEG and not the 616KB original.)

### Does WebP/AVIF degrade a photo? (measured, not vibes)

Pixel-diff of this site's 640px WebP (q80) against a lossless reference:

- Systemic color shift: **0.275/255 worst channel (~0.1%)** — GIF-style color
  distortion simply isn't a failure mode of full-color codecs.
- Mean error ~2.5/255, PSNR 36.7 dB — visually transparent band for web.
- Worst errors sit on high-contrast edges, where the eye's contrast masking
  hides them. Codecs are built to put error where vision can't see it.
- Modern lossy failure mode at low quality is *softening* (texture goes
  plastic), never palette banding.

## 5. Lightroom / photography workflow

Format follows **destination**:

- **Web on modern devices → AVIF**: 10 bits/component (kills gradient
  banding), quality ~50–65 (AVIF 60 ≈ JPEG 85 — don't reuse JPEG quality
  habits), sRGB for predictability or Display P3 for Apple-heavy audiences,
  resize to display size.
- **Uploads / sharing / print labs / email → JPEG q85+**: platforms re-encode
  anyway, labs and old clients reject modern formats.
- **Masters → 16-bit TIFF/PSD** — though RAW + the Lightroom catalog *is* the
  archive. Exports are disposable renders: cattle, not pets. Future-proofing
  lives in the master, not the delivery format; re-exporting is ten minutes.

Why Lightroom has no WebP export: not licensing — product timing. Adobe added
modern formats in the HDR era (~2023) and picked the two that can carry 10-bit
HDR output (AVIF, JXL); WebP's ceiling (8-bit, 4:2:0) is below the renderer's
output, and web variant generation is considered the CDN/build step's job.
(Photoshop *does* have WebP — its audience hand-authors web assets.)

## 6. What this site ships

```html
<picture>
  <source type="image/avif" srcset="…_400.avif 400w, …_800.avif 800w, …_1600.avif 1600w" sizes="320px">
  <source type="image/webp" srcset="…_400.webp 400w, …_800.webp 800w, …_1600.webp 1600w" sizes="320px">
  <img src="…_800.jpg" alt="…" width="320" height="320" fetchpriority="high" decoding="async">
</picture>
```

| File | Size | Served to |
|---|---|---|
| `_400.avif` / `_400.webp` | 25 / 27 KB | DPR-1 desktops |
| `_800.avif` / `_800.webp` | 70 / 79 KB | Most visitors (DPR 2–2.5) |
| `_1600.avif` / `_1600.webp` | 164 / 200 KB | DPR 3+ phones, 8K TVs, deep zoom |
| `_800.jpg` | 113 KB | Pre-2020 browsers, `og:image`, scrapers |

The 2048×2048 `images/jian_mount_si_washington.jpg` stays in the repo as the
unreferenced master for future re-exports. Typical visitor cost went from
**616KB to 70KB (−89%)**, with graceful fallback for every client back to the
1990s.
