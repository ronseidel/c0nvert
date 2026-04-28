# c0nvert — file converter

v17 · 20260424

Drop files, pick format, done. Images, HEIC, SVG, PDF, audio, video — all
client-side, no uploads.

## What's in here

- `index.html` — entry point. Identical to `c0nvert-20260424-17.html`.
- `c0nvert-20260424-17.html` — versioned archive copy.
- `og.png` — 1200×630 social preview, Space Mono + Inter, brand-true zero in --accent.
- `favicon.png` — 32×32 fallback favicon (the inline SVG favicon in <head> handles modern browsers).
- `apple-touch-icon.png` — 180×180 iOS home-screen icon.
- `_headers` — Cloudflare Pages caching, security, **and COOP/COEP for ffmpeg.wasm**.

## Deploy to Cloudflare Pages

```
cd c0nvert-bundle
wrangler pages deploy . --project-name c0nvert
```

Or via the dashboard: drag this whole folder into a new Pages project. The
`_headers` file is picked up automatically.

## DNS

Point `c0nvert.app` at the Pages deployment. Cloudflare issues the cert.

## ⚠ Cross-origin isolation matters here

Unlike a typical static site, c0nvert needs `Cross-Origin-Opener-Policy:
same-origin` and `Cross-Origin-Embedder-Policy: require-corp` so the
browser will hand out `SharedArrayBuffer`, which `ffmpeg.wasm` needs to
re-encode video.

These headers are already in `_headers`. After deploy, verify in the
browser console:

```js
typeof SharedArrayBuffer  // should be 'function', not 'undefined'
crossOriginIsolated       // should be true
```

What works without these headers:
- Image, HEIC, SVG, PDF, audio (incl. AIFF) conversion — all pure JS
- MP4 ↔ MOV via brand-swap (no ffmpeg)

What requires these headers:
- WebM ↔ MP4 / MOV / anything else needing re-encoding via ffmpeg.wasm

## Verifying after deploy

- `https://c0nvert.app/` → app loads
- Drop `test-tone-440hz.aif` → MP3 → confirms AIFF parser
- Drop a small MP4 → MOV → confirms brand-swap (instant)
- Drop a WebM or older format → confirms ffmpeg.wasm path (proves COOP/COEP works)
- Drop an iPhone HEIC → confirms libheif-js
- Paste URL into Slack/iMessage to verify social preview
- Test in light + dark; toggle should flip cleanly

## Pre-deploy production-readiness checklist

Static client-side app, so most of the 20-point list doesn't apply.
What does:

- [x] No hardcoded API keys (none used)
- [x] No localStorage of sensitive data (only `c0nvert-theme` preference)
- [x] Input sanitization: every user-entered string is HTML-escaped before
      rendering. File contents go in via `textContent` or as binary blobs.
- [x] Error boundaries: each conversion wrapped in try/catch, surfaces error
      to the user with file name + format context.
- [x] Permissions-Policy header denies geo/mic/camera (in `_headers`)
- [x] X-Frame-Options SAMEORIGIN prevents clickjacking
- [x] Referrer-Policy strict-origin-when-cross-origin
- [x] **COOP/COEP** for SharedArrayBuffer (required by ffmpeg.wasm video path)

Not applicable here (no server, no auth, no DB):
rate limiting, httpOnly cookies, Stripe webhook verification, DB indexing,
admin role checks, async emails, health checks.

## Lazy-loaded libraries (loaded on demand from CDN)

- **pdf.js** (cdnjs) — loaded eagerly, ~600KB. Drives PDF page rasterization.
- **JSZip** (cdnjs) — loaded eagerly, ~95KB. Bundles multi-output downloads.
- **lamejs** (jsdelivr) — lazy, ~50KB. Loaded on first MP3 conversion.
- **libheif-js** (jsdelivr) — lazy, ~1.5MB. Loaded on first HEIC file.
- **ffmpeg.wasm 0.11.6** (unpkg) — lazy, ~25MB. Loaded only on video
  re-encode (not for MP4↔MOV brand-swap).

All run entirely in the browser. No file ever leaves the device.

## Roadmap

- v18: Multi-threaded ffmpeg via ffmpeg-core 0.12.x for faster video
- v18: FLAC / OGG / M4A audio output options
- v19: Batch presets ("favicon set", "social sizes", "podcast clip")
