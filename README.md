# 340B Daily Intelligence

A self-contained static site that tracks developments in the 340B Drug Pricing Program — manufacturer contract-pharmacy restrictions and rebate models, federal litigation, HRSA/OPA guidance, federal and state legislation, oversight reports, and stakeholder positioning. Designed to compound daily into a sourced, searchable corpus.

## Files

- `index.html` — the deployable site. Single HTML file, inline CSS and JS, embedded corpus. No external assets.
- `corpus.json` — source of truth for the corpus. Structured per the schema in `340b_prompt.txt`.
- `build_site.py` — regenerates `index.html` from `corpus.json`. Run it after every corpus update.
- `340b_prompt.txt` — the analyst prompt used to drive each day's run.
- `HOSTING.md` — step-by-step instructions for putting this on GitHub Pages.

## Run the daily intelligence pass

1. Open a fresh chat with Claude.
2. Paste the contents of `340b_prompt.txt`, then in your next message paste the current corpus inside `<corpus>...</corpus>` tags. On the first day, that's `<corpus>[]</corpus>`; on day N it's yesterday's `corpus.json`.
3. Save the model's `<corpus_out>` content to `corpus.json`.
4. Rebuild and push:

```bash
python3 build_site.py
git add corpus.json index.html && git commit -m "Daily update: $(date +%F)" && git push
```

GitHub Pages will redeploy within a minute or two.

## What's in the first-pass corpus

This initial corpus has 27 entries covering roughly Dec 2025 – May 2026, weighted heavily toward the rebate-model litigation arc (AHA v. Kennedy in D. Maine, 1st Circuit, vacatur, RFI) and the state contract-pharmacy circuit split (5th Cir Louisiana, MN Court of Appeals, 4th Cir West Virginia, W.D. Wash). Manufacturer claims-data mandates (Lilly, Novo Nordisk), TPA shifts (340B ESP → Kalderos Truzo), federal bills (SUSTAIN, PROTECT 340B, House FQHC bill), and stakeholder positions (AHA, RWC-340B, ASCO) are all represented.

Treat this as a seed corpus — the daily runs are where the value compounds.
