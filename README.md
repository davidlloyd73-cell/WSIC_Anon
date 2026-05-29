# WSIC Anonymiser

A browser-based utility for anonymising NHS primary-care exports (WSIC,
Tableau crosstabs, generic CSV/TSV) before sharing for audit or
AI-assisted analysis. Runs entirely client-side. No data leaves the
browser.

Built for use at a single GP practice in NW London. Open source so
local IT and Caldicott Guardians can audit it line-by-line before
approving its use.

## Demo

Deploy this repo to [Netlify](https://www.netlify.com) and the entire
app is a single static HTML page. There is no backend.

## What it does

- Replaces NHS numbers with stable Patient IDs (`Patient 0001`, `Patient 0002`, …).
- The same NHS number always becomes the same Patient ID, so multiple
  extracts (Frailty Radar, A&E Activity, SMI register, etc.) can be
  joined on the Patient ID.
- Redacts columns whose headers match common patient-identifier
  patterns (name, DOB, address, postcode, phone, email) by overwriting
  the cell with `[REDACTED]`.
- Output is UTF-8 tab-separated.

## Privacy model

This is the central design choice. The page makes no network calls of
any kind after the initial HTML/CSS/JS download from Netlify. To verify:

1. View source (Ctrl+U / Cmd+Option+U). The page is a single HTML file.
2. Search for `fetch`, `XMLHttpRequest`, `WebSocket`, `navigator.sendBeacon`,
   `<img`, `<iframe`, or `<link rel=preload>` pointing to external URLs.
   You will find none.
3. Open browser DevTools → Network tab, hard-refresh the page, then
   process a file. After the initial page load there are no further
   requests.
4. Disconnect your computer from the network. The page still works.

The Content-Security-Policy header set by `netlify.toml`
(`connect-src 'none'`) instructs the browser to block any future
network calls even if a regression introduced one.

## Master key

The Patient ID ↔ NHS Number mapping is the bit that turns anonymised
data back into identifiable data. Two storage options:

- **File-based (default, recommended for shared NHS PCs).** Use the
  "Save key file" / "Load key file" buttons. The key file lives on
  your local disk under your control. Do not store it on iCloud,
  OneDrive, Dropbox, Google Drive or any cloud-synced folder.
- **Browser persistence (personal device only).** Tick "Remember in
  this browser" to persist the key in browser localStorage. Useful on
  a personal laptop where you control the device. Inappropriate on a
  shared NHS PC because localStorage is not encrypted and persists
  across user sessions for the same browser profile.

## What this tool will not do

- It will not anonymise free-text fields (e.g. a "Notes" column with
  identifiers written in by clinicians). Eyeball free-text columns
  before sharing.
- It will not anonymise PDFs, screenshots or images.
- It will not protect against re-identification from very small
  cohorts with rare attribute combinations. For cohorts under
  ~10 patients, treat the output as still identifiable.

If in doubt, run the output past your Caldicott Guardian or DPO.

## Local development

```
git clone <this-repo>
cd wsic-anonymiser-web
# Serve index.html with any static server, e.g.:
python3 -m http.server 8000
# Then open http://localhost:8000
```

There are no build steps and no dependencies.

## Deploying to Netlify

1. Push this repo to GitHub (public or private).
2. Sign in to Netlify, "Add new site → Import an existing project".
3. Pick this repo. Leave build command empty. Publish directory: `.`.
4. Done.

`netlify.toml` sets strict security headers including a
`Content-Security-Policy` that blocks all outbound network calls from
the page. Confirm these headers are present on the deployed site
before relying on the privacy guarantee.

## Licence

MIT — see `LICENSE`.
