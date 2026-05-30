# WSIC Anonymiser

A browser-based utility for anonymising NHS primary-care exports (WSIC,
Tableau crosstabs, generic CSV/TSV) before sharing for audit or
AI-assisted analysis. Runs entirely client-side. No data leaves the
browser.

Built for use at a single GP practice in NW London. Open source so
local IT and Caldicott Guardians can audit it line-by-line before
approving its use.

The running build version is shown in the page footer and recorded in
the session activity log, so you can always confirm which version
processed a given file.

## Demo

Deploy this repo to [Netlify](https://www.netlify.com) and the entire
app is a single static HTML page. There is no backend.

## What it does

- Replaces NHS numbers with stable Patient IDs (`Patient 0001`, `Patient 0002`, …).
- The same NHS number always becomes the same Patient ID, so multiple
  extracts (Frailty Radar, A&E Activity, SMI register, etc.) can be
  joined on the Patient ID.
- **Detects NHS numbers two ways:** by column header, and by validating
  the cell contents against the NHS Modulus-11 check digit. A column of
  valid NHS numbers is pseudonymised *even if its header is not
  recognised* (e.g. `NHS #`). This closes the failure mode where an
  unexpected header would let NHS numbers pass through silently.
- **Refuses to export a file that still contains a live NHS number.**
  After anonymisation, every output cell is re-checked against
  Modulus-11. If any cell still validates as an NHS number, the download
  is blocked and the offending column and row are flagged on screen
  (with the number masked). It is therefore not possible to save an
  output file that still contains a valid NHS number.
- Flags "suspicious" columns: a column made up mostly of 10-digit
  numbers that do *not* pass Modulus-11 is left untouched but highlighted
  for manual review, in case it is a malformed or non-standard
  identifier.
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

The "Save key file" button is enabled only once the key holds at least
one patient — i.e. once at least one NHS number has been pseudonymised.
If a run detects no NHS numbers, the key stays empty and there is
nothing to save.

## What this tool will not do

- It will not *pseudonymise* free-text fields (e.g. a "Notes" column).
  It will, however, **block export** if a valid NHS number is found in
  any cell, including free text — so an NHS number typed into a note
  stops the file rather than leaking. Other identifiers in free text
  (names, addresses, dates of birth written into prose) are not
  detected; eyeball free-text columns before sharing.
- It will not anonymise PDFs, screenshots or images.
- It will not protect against re-identification from very small
  cohorts with rare attribute combinations. For cohorts under
  ~10 patients, treat the output as still identifiable.

Note on the export guard: roughly 1 in 11 random 10-digit numbers will
pass the Modulus-11 check by chance, so a genuinely non-patient 10-digit
column could occasionally trigger a block. This is the safe direction to
fail — it stops the file and asks a human to look rather than waving it
through. If you hit a legitimate false block, make that column's
contents obviously non-NHS rather than weakening the guard.

If in doubt, run the output past your Caldicott Guardian or DPO.

## Local development

```
git clone <this-repo>
cd WSIC_Anon
# Serve index.html with any static server, e.g.:
python3 -m http.server 8000
# Then open http://localhost:8000
```

There are no build steps and no dependencies.

After changing behaviour, bump `APP_VERSION` / `BUILD_DATE` at the top
of the script in `index.html` so the footer build stamp reflects the
deployed version.

## Deploying to Netlify

1. Push this repo to GitHub (public or private).
2. Sign in to Netlify, "Add new site → Import an existing project".
3. Pick this repo. Leave build command empty. Publish directory: `.`.
4. Done.

`netlify.toml` sets strict security headers including a
`Content-Security-Policy` that blocks all outbound network calls from
the page. Confirm these headers are present on the deployed site
before relying on the privacy guarantee.

Because the app is a single cached HTML file, after deploying a change
do a hard refresh (Cmd+Shift+R / Ctrl+F5) and confirm the footer build
stamp matches the version you just pushed before relying on the new
behaviour.

## Licence

MIT — see `LICENSE`.
