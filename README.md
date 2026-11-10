# davidpereira.cc

My personal website: a links page at **https://davidpereira.cc** and a page for my
children's book at **https://davidpereira.cc/lagosta**.

Plain HTML and CSS, with no framework, no build step and no dependencies. It's hosted
for free on Cloudflare Workers (static files only) and deployed automatically on every
push to `main`.

## ⚠️ The one rule: never break `/lagosta`

`https://davidpereira.cc/lagosta` is printed as a QR code in physical copies of
*Cadê a lagosta?*. It must keep working forever.

- Never rename, move or delete `public/lagosta/index.html`.
- Keep that page self-contained, with its own styles. A homepage redesign must not
  touch it.
- Before any hosting or DNS change, test `/lagosta` and `/lagosta/` on the new setup.
- Keep the domain on auto-renew. If the domain lapses, every printed QR code dies.

## Structure

```
public/                  everything served by the site
  index.html             homepage (links page)
  lagosta/index.html     book page (Portuguese, unlisted: noindex)
  lagosta/*.png          the book page's own favicons
  404.html               not-found page
  assets/                photo, book cover, share images (WebP + JPEG)
  favicon.ico, favicon-32.png, apple-touch-icon.png
  _headers               security headers, noindex for /lagosta, caching
  robots.txt, sitemap.xml
wrangler.jsonc           Cloudflare config (static assets only, no Worker script)
```

## Domain setup (Cloudflare)

- `davidpereira.cc` is attached to the Worker as a custom domain, declared in `wrangler.jsonc`.
- `www.davidpereira.cc` is **not** attached to the Worker. A placeholder DNS record
  (`A www 192.0.2.1`, proxied) and a zone *Redirect Rule* send it to
  `https://davidpereira.cc` with a 301, keeping the path and query.
- *Always Use HTTPS* is on. DNSSEC is on.
- The domain sends no email: `MX 0 .`, SPF `v=spf1 -all`, DMARC `p=reject`.
- There's no public `*.workers.dev` copy (`workers_dev: false`). Preview URLs for
  branches stay on (`preview_urls: true`).

## Run locally

Requires Node.js. Nothing gets installed into the project.

```bash
npx wrangler dev
```

Then open http://localhost:8787. This honours `_headers` and the trailing-slash
setting, which a plain static file server doesn't.

## Deploy

Push to `main`. Cloudflare Workers Builds deploys it in about a minute (GitHub shows a
"Workers Builds" check on the commit).

Any other branch gets a preview instead, at:

```
https://<branch-name>-davidpereira-cc.p65kt8j49g.workers.dev
```

The address isn't shown in the GitHub check, so build it from the branch name. Previews
need the empty `"previews": {}` block in `wrangler.jsonc`: don't remove it. For bigger
changes, work on a branch, check the preview on a phone, then merge to `main` and
delete the branch.

## Common changes

| Change | What to edit |
|---|---|
| Bio, links | `public/index.html` (one `<li>` per link) |
| Profile photo | Replace `public/assets/david-256.*` and `david-512.*`, keeping the names |
| Book cover or description | `public/lagosta/index.html` and `public/assets/lagosta-cover-*` |
| **Revised PDF, same link** | In Google Drive: right-click the PDF → *File information* → *Manage versions* → *Upload new version*. Check sharing is still *Anyone with the link – Viewer*. No site change needed |
| PDF moves elsewhere | Change the download button's `href` in `public/lagosta/index.html` |
| New edition or language | Add another button on `/lagosta`. Don't change what `/lagosta` itself is |
| Share images | `public/assets/og-home.jpg`, `og-lagosta.jpg` (1200 × 630) |

## If a deploy goes wrong

Cloudflare dashboard → *Workers & Pages* → `davidpereira-cc` → *Deployments* → roll back
to the previous version. Then fix it in Git (`git revert`) and push.

## Moving to another host

1. Deploy `public/` to the new host.
2. On the new host's temporary URL, test `/`, `/lagosta`, `/lagosta/` and the PDF
   button. Recreate the `_headers` rules in that host's format.
3. Only then switch DNS. Afterwards, scan a printed QR code with a phone.

## Licence

No open-source licence has been chosen. The photo, book cover and illustrations are
© David Pereira, all rights reserved.
