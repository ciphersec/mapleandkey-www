# mapleandkey.com — Maple & Key marketing site

Public-facing landing site for the **Maple & Key** property-management SaaS for small Canadian landlords.

The actual portal app lives at `app.mapleandkey.com` (see `~/Projects/tenants-portal/SPEC.md`).

## Brand

| | |
|---|---|
| Primary | Royal Blue `#003DA5` |
| Accent | Maple Red `#DC1C2E` |
| Text | `#1b1815` near-black, `#7a7a7a` muted |
| Wordmark | "Maple & Key" set in Georgia, blue/amp/red |

**Brand-history note:** the palette is inherited from the original Maple & Key real-estate-sales-rep brand (which used RE/MAX co-branding on the business cards). For this SaaS site the colors stay but **all RE/MAX co-branding has been dropped** — Maple & Key here is a pure property-management software brand, no RE/MAX affiliation.

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
