# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Repo Is

Static front-end assets for the Transpara AI hive on nucbuntu. No build step, no framework, no server-side rendering — just standalone HTML files served via GitHub Pages, a local file server, or opened directly in a browser.

This is one of three repos in the telemetry system:
- **lovyou-ai-hive** — telemetry writer (Go, runs in the hive process)
- **lovyou-ai-work** — telemetry API (Go, runs in the work-server)
- **summary** (this repo) — dashboard and static pages (HTML only)

## Files

- `index.html` — Static architecture poster showing the complete dependency hierarchy. No external dependencies.
- `dashboard.html` — Live mission control dashboard that polls the work-server telemetry API (`GET {api}/telemetry/status` every 10s). Configured via URL query params `?api=...&key=...`.
- `docs/designs/` — Design documents for the telemetry/mission-control system.

## Development

No build, no install, no tests. Open HTML files directly in a browser or serve with:

```
python3 -m http.server 9000
```

Dashboard requires a running work-server instance for live data. Without `api` and `key` query params, it shows a configuration screen.

## Architecture Notes

- Both HTML files are fully self-contained — all CSS and JS are inline. No external dependencies, no CDN links.
- `dashboard.html` uses vanilla JS fetch to poll the telemetry API. Connection state is tracked with a pulsing indicator (green=live, gray=stale, red=failed).
- `index.html` supports light/dark mode via `prefers-color-scheme`.
- The work-server has `Access-Control-Allow-Origin: *` so cross-origin fetch works from any serving origin.
