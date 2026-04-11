# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Dev Servers

Defined in `.claude/launch.json`. Use `preview_start` with the server name to launch:

- **Maggie Quote Generator** — `C:/Python314/python -m http.server 8082 --directory F:/Maggiesapp` → http://localhost:8082/maggie-quote-generator.html
- **Ashleigh Olney Site** — `python -m http.server 8081 --directory F:/Maggiesapp/ashleigholney` → http://localhost:8081

No build step. Both are static file servers.

## Project: Maggie Quote Generator (`F:/Maggiesapp/maggie-quote-generator.html`)

Single-file HTML/CSS/JS app — no framework, no bundler, no dependencies except two CDN scripts (jsPDF 2.5.1, jsPDF-AutoTable 3.8.2).

### Architecture

**Email parsing** (`buildFieldMap` + `fillFromFields`): Handles FormSubmit booking emails pasted or dragged in. Uses three strategies: tab-separated, colon-separated, and alternating-line (the real format). Strategy 3 scans for the first KNOWN_KEY anchor to skip Gmail header noise, then pairs alternating lines as key/value. The standalone `"Value"` line from Gmail table headers is filtered out.

**Travel fee** (`onLocationChange` → `setMiles`): Checks studio keywords → fuzzy venue match → manual entry. `VENUES` array stores **one-way** miles; the app doubles them internally (`currentMiles` = RT). The miles input field always displays one-way; `onMilesChange()` doubles whatever the user types.

**Venue photos**: JPGs live in `F:/Maggiesapp/venue-photos/`. Filename convention: `venue_name.jpg` (lowercase, spaces → underscores). `venueNameToFilename()` converts matched venue names. `generic.jpg` is used as fallback for unmatched venues. Photos display in the travel panel UI and are embedded in the generated PDF.

**Quote calculation** (`calcQuote`): Reads `extraItems[]` (added via quick-add buttons or custom entry) + early bird checkbox + additional artists number input. Service type dropdown and hair/makeup counts are informational only — pricing comes entirely from `extraItems`. Travel fee = $75 base + $1/mile RT if not studio.

**PDF generation** (`generatePDF`): Uses jsPDF. Captures `logoBase64` (loaded on page load from `logo.png`) and `venuePhotoB64` (grabbed from the DOM img element at generation time). Logo aspect ratio stored as `logoAspect`; venue photo aspect stored as `venuePhotoAspect` — always compute both width and height from one dimension to avoid squishing.

### Key Constants (in the script block)
- `STUDIO_ADDRESS` — "1033 E 2100 S Unit 207, Salt Lake City, UT 84106"
- `PRICE` object — all service prices; `travelBase: 75`, `travelPerMile: 1`
- `VENUES` array — ~65 entries, miles are ONE-WAY

### Static Assets (all in `F:/Maggiesapp/`)
- `logo.png` — business logo, loaded as base64 for PDF embedding
- `venue-photos/*.jpg` — ~48 venue photos; `generic.jpg` is the fallback
