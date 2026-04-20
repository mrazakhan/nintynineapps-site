# nintynineapps-site

Static site for **NintyNineApps** — a landing page + privacy notice for the 67s game.

Pure HTML/CSS, no build step, no dependencies.

## Files

- `index.html` — landing page (company flyer + 67s product card)
- `privacy.html` — privacy notice covering NintyNineApps + 67s
- (add images/assets here later if needed)

## Local preview

Open `index.html` directly in a browser, or serve the folder:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000/
```

## Deploying to the production host

Target host: `66.179.255.201` (Ubuntu 22.04). Apache handles `:80`.

The site should live alongside the other apps without disturbing AI Tutor
(`/aitutor/*`) or DeepTutor++ (`/*`). The clean way is a dedicated path like
`/99apps/` served as plain static files from Apache.

### 1. Copy the site onto the server

```bash
# from the repo root on your laptop
rsync -av --delete ./ root@66.179.255.201:/var/www/nintynineapps/
```

### 2. Add an Apache location block

Edit `/etc/apache2/sites-enabled/deeptutor.conf` and add (above the
catch-all `ProxyPass / http://127.0.0.1:3782/` line, so it matches first):

```apache
Alias /99apps /var/www/nintynineapps
<Directory /var/www/nintynineapps>
    Require all granted
    Options -Indexes
    DirectoryIndex index.html
</Directory>

# Make sure the static path is NOT proxied to DeepTutor++
ProxyPass /99apps !
```

Then reload:

```bash
apachectl configtest && systemctl reload apache2
```

### 3. Verify

```
http://66.179.255.201/99apps/           → landing page
http://66.179.255.201/99apps/privacy.html
```

## Notes

- This repo is **separate** from AI-Tutor; deploy/operate it independently.
- HTTP-only host (no TLS). Don't put anything here that depends on
  `navigator.clipboard`, Service Workers, or other secure-context APIs.
- If `/99apps/` ever needs internal relative links, keep them relative
  (`privacy.html`, not `/privacy.html`) so the base path is portable.
