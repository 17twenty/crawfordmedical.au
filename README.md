# crawfordmedical.au

Single static holding page for Crawford Medical. No build step, no dependencies —
the files in this repo are what gets served.

## Files

| File | Purpose |
| --- | --- |
| `index.html` | The whole page: markup, styles and the one animation, inlined. |
| `404.html` | Copy of `index.html`, so any path shows the page instead of GitHub's 404. |
| `CNAME` | Custom domain. Managed by the Pages settings UI — if you change the custom domain on GitHub it rewrites this file, so pull before you push. |
| `favicon.svg` | The river mark. |
| `.nojekyll` | Stops GitHub running the files through Jekyll. |

## Deploying

1. Push this directory to a GitHub repo.
2. Settings → Pages → Source: **Deploy from a branch**, branch `main`, folder `/ (root)`.
3. Settings → Pages → Custom domain: `www.crawfordmedical.au`. Setting this rewrites
   the `CNAME` file in the repo, so `git pull` afterwards.
4. Tick **Enforce HTTPS** once the certificate is issued (can take up to an hour).

### If `https://crawfordmedical.au` shows a certificate error

GitHub issues one Let's Encrypt cert covering both the apex and `www`, but only for
names that resolved **when the cert was requested**. If the apex DNS records were added
after the domain was first configured, the existing cert covers `www` alone and will
not pick the apex up on its own.

Check what the cert actually covers:

```sh
echo | openssl s_client -connect 185.199.108.153:443 -servername www.crawfordmedical.au 2>/dev/null \
  | openssl x509 -noout -ext subjectAltName
```

You want both `crawfordmedical.au` and `www.crawfordmedical.au` in the SAN list. If only
`www` is there, force a re-issue: Settings → Pages → clear the custom domain → Save →
re-enter `www.crawfordmedical.au` → Save. The site drops out for a minute or two and
**Enforce HTTPS unticks itself**, so re-tick it once the new cert appears.

This matters more than it looks: browsers try HTTPS before HTTP, so a bare-domain visit
hits the cert error and shows a security warning rather than following the redirect.

## DNS

Registrar is Porkbun. **`www.crawfordmedical.au` is the canonical address** — it is
what the Pages custom domain is set to, and it is what `CNAME` in this repo contains.
The apex redirects to it.

```
A      @    185.199.108.153
A      @    185.199.109.153
A      @    185.199.110.153
A      @    185.199.111.153
AAAA   @    2606:50c0:8000::153
AAAA   @    2606:50c0:8001::153
AAAA   @    2606:50c0:8002::153
AAAA   @    2606:50c0:8003::153

A      www  185.199.108.153
A      www  185.199.109.153
A      www  185.199.110.153
A      www  185.199.111.153
AAAA   www  2606:50c0:8000::153
AAAA   www  2606:50c0:8001::153
AAAA   www  2606:50c0:8002::153
AAAA   www  2606:50c0:8003::153
```

The apex records exist only so the bare `crawfordmedical.au` resolves; GitHub answers
on those IPs and 301s across to `www`. Without them the bare domain returns NODATA and
fails to load entirely, which is easy to miss because the `www` URL keeps working.

`www` uses the same literal IPs rather than `CNAME www → 17twenty.github.io`. Both
work — GitHub Pages routes on the **Host header**, so the CNAME target carries no repo
identity and pointing several domains at one `github.io` name does not make them
collide. Literal IPs are used here only to keep this domain independent of the other
sites on the account. The tradeoff is that if GitHub ever changes its Pages IPs these
need updating by hand; check
<https://docs.github.com/pages/configuring-a-custom-domain-for-your-github-pages-site>.

**Never put a plain `CNAME` record on the apex.** A CNAME at the zone apex cannot
coexist with other records and will take the MX records down with it, breaking Kate's
email forwarding. Use `A`/`AAAA` as above, or Porkbun's apex-safe `ALIAS` type.
(A CNAME on `www` is harmless — that restriction applies only to the apex.)

### Email

Forwarding is handled by the registrar and is independent of Pages:

```
MX  @  10 fwd1.porkbun.com
MX  @  20 fwd2.porkbun.com
```

Nothing in this repo touches email. Just don't let a DNS edit clobber the MX records.

## Editing

Everything lives in `index.html`. The things most likely to change:

- **Contact address** — the `mailto:` and the link text, near the bottom of the markup.
- **The sentence about Kate** — the `.lede` paragraph. It deliberately avoids a start
  date so it doesn't go stale.
- **Colours** — the custom properties in `:root`.

If you change `index.html`, re-copy it over `404.html`:

```sh
cp index.html 404.html
```

## Previewing locally

```sh
python3 -m http.server 8000
```

Then open http://localhost:8000.
