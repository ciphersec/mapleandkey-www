# mapleandkey.com — Maple & Key marketing site

Public-facing landing site for the **Maple & Key** property-management SaaS for small Canadian landlords.

The actual portal app lives at `app.mapleandkey.com` (see `~/Projects/tenants-portal/SPEC.md`).

## Brand

Visual aesthetic matches `manohomes.ca` — warm, organic, paper-cream:

| | |
|---|---|
| Background | Paper cream `#faf7f0` / cream `#f6f1e7` |
| Accent | Maple brick-red `#a23a2a` (warm, NOT bright red) |
| Highlight | Gold `#b08a3e` (used on dark sections + cta cards) |
| Text | Warm ink `#1d1916`, soft `#4a423b`, muted `#8a7e72` |
| Dark band | `#1b1815` (pricing section) · `#14110f` (footer) |
| Borders | `#e3dcce` |
| Wordmark | "Maple & Key" in Playfair Display 600, ink-color, italic maple ampersand |
| Headings | Playfair Display |
| Body | Inter |

**Brand-history note:** earlier draft used royal blue + bright red (RE/MAX-coded). Repalette'd 2026-05-30 to match `manohomes.ca` aesthetic — warm cream backdrop, brick-maple accents, no RE/MAX color overlap.

## Stack

Static HTML/CSS — no build step, no JS framework, no external CDNs.

- `index.html` — single landing page
- `style.css` — vanilla CSS with custom properties
- `assets/` — images (currently empty; logo PNG/SVG to be added later)

## Local preview

```bash
cd ~/Projects/mapleandkey-www
python3 -m http.server 8080
# open http://localhost:8080
```

## Deploy to realty-server (manual checklist — script TBD)

```bash
# 1. backup the current /var/www/mapleandkey
ssh realty 'sudo tar czf ~/backups/mapleandkey-pre-wipe-$(date +%F).tar.gz -C /var/www mapleandkey'

# 2. rsync new files
rsync -av --delete ~/Projects/mapleandkey-www/ realty:/tmp/mapleandkey-new/
ssh realty 'sudo rsync -av --delete /tmp/mapleandkey-new/ /var/www/mapleandkey/ && sudo chown -R www-data:www-data /var/www/mapleandkey && rm -rf /tmp/mapleandkey-new'

# 3. rewrite nginx config (drop the 301, serve static)
ssh realty 'sudo cp /etc/nginx/sites-available/mapleandkey.com /etc/nginx/sites-available/mapleandkey.com.bak-pre-wipe-$(date +%F)'
# edit /etc/nginx/sites-available/mapleandkey.com: replace the `return 301` location with:
#   root /var/www/mapleandkey;
#   index index.html;
#   location / { try_files $uri $uri/ =404; }
# keep the ACME challenge location and TLS cert paths as-is

# 4. apply
ssh realty 'sudo nginx -t && sudo systemctl reload nginx'
```

DNS and Zoho MX are untouched.
