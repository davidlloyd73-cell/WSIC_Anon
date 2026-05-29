# Deployment guide — GitHub + Netlify

Step-by-step. Reading time ~3 minutes, doing time ~10 minutes.

## What you need first

1. A **GitHub account** (free) — github.com
2. A **Netlify account** (free) — netlify.com — sign in with your GitHub
   account so they're already linked
3. The **wsic-anonymiser-web** folder this guide came in

Total cost: zero. Netlify's free tier is enough for this project
forever.

---

## Step 1 — Create a GitHub repository (3 min)

1. Sign in at github.com.
2. Top-right "+" → "New repository".
3. Repository name: **wsic-anonymiser** (or whatever you prefer — it'll
   show up in the URL).
4. Description: "Browser-based anonymiser for WSIC primary care exports."
5. **Public** is the right choice for the privacy story (your Caldicott
   Guardian and IT lead can read the code). If you'd rather keep it
   private at first you can — Netlify works with both.
6. Tick "Add a README file" — we'll overwrite it in the next step.
7. "Create repository".

You'll land on the empty repository page.

## Step 2 — Upload the files (2 min)

The easiest path — no Git knowledge needed:

1. On the empty repo page click "Add file" → "Upload files".
2. From your Mac, open the `wsic-anonymiser-web` folder and select all
   the files inside it:
   - `index.html`
   - `README.md`
   - `LICENSE`
   - `netlify.toml`
   - `.gitignore`
   - `DEPLOY.md` (this file)
3. Drag them all into the GitHub upload area.
4. Scroll down. Commit message: "Initial commit".
5. "Commit changes".

GitHub now shows your repository with all the files. The README is
displayed underneath.

## Step 3 — Connect Netlify (3 min)

1. Sign in at app.netlify.com (use "Sign up with GitHub" if you haven't
   yet).
2. "Add new site" → "Import an existing project".
3. "Deploy with GitHub". Authorise Netlify if asked.
4. Pick your **wsic-anonymiser** repository from the list.
5. The site settings should already be correct:
   - **Build command**: (leave blank)
   - **Publish directory**: `.`
   - Netlify reads these from `netlify.toml`.
6. "Deploy site".
7. Netlify provisions a URL like
   `https://random-words-12345.netlify.app`. Click it.

You should see the WSIC Anonymiser page.

## Step 4 — Give it a sensible URL (2 min)

The random URL works, but you'll want a nicer one for the partner email
and Caldicott review.

1. In Netlify, on your site dashboard → "Site configuration" → "Domain
   management" → "Options" next to the current `*.netlify.app` URL.
2. "Edit site name". Pick something like `ridgeway-anonymiser` or
   `wsic-anonymiser-harrow`.
3. Your URL becomes `https://ridgeway-anonymiser.netlify.app`.

If later you want a fully custom domain (e.g. via the practice or
your own personal domain), Netlify supports that too — but the free
`.netlify.app` URL is perfectly respectable for this use case.

## Step 5 — Test, then verify the privacy guarantee (3 min)

1. Open the deployed URL in any browser.
2. Run a WSIC export through it — the anonymised file should download
   automatically.
3. Verify the privacy promise:
   - Right-click the page → "View Page Source" → confirm it's a single
     HTML file with embedded JS.
   - Open DevTools (F12) → "Network" tab → hard-refresh (Cmd-Shift-R /
     Ctrl-Shift-F5). You should see the page load and three or four
     resources (HTML, favicon, maybe a font). After that, process a
     file: no further network requests appear.
   - DevTools → "Application" → "Local Storage" — should be empty unless
     you ticked "Remember in this browser".
   - DevTools → "Network" → look at the response headers for the page.
     You should see `content-security-policy: ... connect-src 'none' ...`.
     That header is the browser-enforced ban on outbound calls.

## Step 6 — Update the partner-meeting email

Add this line to the "Locally built, locally owned" bullet:

> Hosted as a static page at https://YOUR-URL.netlify.app and the
> source is open at https://github.com/YOUR-USERNAME/wsic-anonymiser
> for IT and Caldicott review.

---

## Updating the site later

If you (or I, via you) make changes to `index.html`, the workflow is:

1. Edit the file in GitHub directly (pencil icon at the top right of
   the file view) OR upload a new version via "Add file" → "Upload
   files" → drop the new `index.html` → commit.
2. Netlify detects the commit automatically and redeploys within ~30
   seconds.

No manual steps needed on the Netlify side once the connection is set
up.

## Pulling it down

If you ever want to take the site offline: Netlify dashboard → site →
"Site configuration" → "Stop builds" then "Delete this site". The
GitHub repo can stay or be deleted independently.
