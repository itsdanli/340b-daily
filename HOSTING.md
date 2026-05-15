# Hosting on GitHub Pages — step by step

This folder contains a fully self-contained site (`index.html`) plus the source data (`corpus.json`) and a tiny builder (`build_site.py`). To put it on the public web you only need to publish `index.html`. Everything else is helpful machinery for the daily-update workflow.

## What you'll end up with

- A public URL like `https://<your-github-username>.github.io/340b-daily/` that opens the latest corpus.
- A repo you push to once per day after the model produces a fresh `corpus.json` and rebuilds `index.html`.
- No build server, no analytics, no external dependencies.

## One-time setup

### 1. Create the GitHub repo

On github.com, click **New repository**. Suggested values:

- Repository name: `340b-daily` (anything is fine — this affects the URL path)
- Visibility: **Public** (required for free GitHub Pages on a non-Pro account)
- Initialize: leave everything unchecked. You'll push existing files.

### 2. Make this folder a git repo and push

From a terminal, in the folder that contains `index.html`:

```bash
cd "340B Research"
git init -b main
git add index.html corpus.json build_site.py HOSTING.md README.md 340b_prompt.txt
git commit -m "Initial 340B intelligence corpus and site"
git remote add origin https://github.com/<your-username>/340b-daily.git
git push -u origin main
```

If you'd rather keep the prompt or raw research private, just don't `git add` it. The site only needs `index.html`.

### 3. Turn on Pages

In the GitHub repo UI:

1. **Settings → Pages**
2. Under **Build and deployment**, set **Source** to **Deploy from a branch**.
3. Branch: `main`, Folder: `/ (root)`. Save.
4. Wait ~30 seconds. GitHub will print the public URL at the top of the Pages settings page.

Open the URL. You should see the same site you see locally.

### 4. (Optional) Custom domain

If you own a domain you'd like to use:

1. Add a `CNAME` file to the repo containing your domain (e.g., `340b.example.com`).
2. Add a DNS CNAME record at your registrar pointing that subdomain to `<your-username>.github.io`.
3. In **Settings → Pages**, enter the custom domain and check **Enforce HTTPS** once the cert provisions.

## The daily update loop

Each morning you run the model with yesterday's `corpus.json` pasted into `<corpus>`. The model returns a fresh `corpus.json` inside `<corpus_out>` tags and the HTML site. Then:

```bash
# 1. Save the new corpus
#    (paste the contents of <corpus_out>...</corpus_out> into corpus.json,
#     overwriting the file)

# 2. Rebuild the embedded site
python3 build_site.py

# 3. Commit and push
git add corpus.json index.html
git commit -m "Daily update: $(date +%Y-%m-%d)"
git push
```

Pages will redeploy automatically within a minute or two. If you'd rather skip the build step, the model's HTML output can be saved directly as `index.html` — the embedded data and the site are equivalent.

## Optional: automate the rebuild on push

If you'd prefer to push only `corpus.json` and let GitHub rebuild `index.html`, add this workflow at `.github/workflows/build.yml`:

```yaml
name: Build site
on:
  push:
    paths: ["corpus.json", "build_site.py"]
jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: write
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: "3.x"
      - run: python3 build_site.py
      - run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add index.html
          git diff --cached --quiet || git commit -m "Rebuild site"
          git push
```

This isn't required — running `python3 build_site.py` locally is just as good and avoids the small write-permissions surface area of a self-pushing workflow.

## Things to know

- **No PHI lives in this repo.** Everything in the corpus is public-domain regulatory, litigation, and policy news. Keep it that way — never paste internal data into `corpus.json`.
- **Cache invalidation:** GitHub Pages serves with sensible default cache headers, but if you change something and don't see it, hard-refresh (Cmd-Shift-R / Ctrl-F5).
- **Mobile QR-code workflow:** point your phone at the public URL, save to home screen — the site is designed to read cleanly on a phone.
- **Searchability:** the site is fully client-side and indexable by search engines. If you don't want it indexed, add a `robots.txt` containing `User-agent: *\nDisallow: /` at the repo root.

## Restoring from scratch

If you ever lose `index.html`, you can regenerate it as long as `corpus.json` and `build_site.py` exist:

```bash
python3 build_site.py
```
