# ilune.dev

The coming-soon website for Ilune, hosted on GitHub Pages at <https://ilune.dev>.

The site is static HTML and CSS with no build step, no JavaScript, and no third-party requests. All fonts, icons, and stylesheets are served from the site itself.

## Repository contents

| Path | Description |
|---|---|
| `site/` | Published site root. The GitHub Actions workflow deploys only this directory. |
| `site/index.html`, `site/404.html`, `site/styles.css` | Landing page, 404 error page, and shared stylesheet. |
| `site/fonts/` | IBM Plex Sans (weights 400, 600, and 700, Latin subset) from Fontsource 5.3.0, with its license (`OFL.txt`). |
| `site/.well-known/security.txt` | Vulnerability disclosure contact ([RFC 9116](https://www.rfc-editor.org/rfc/rfc9116)). |
| `site/llms.txt` | Concise Markdown summary of the site for AI tools and crawlers. |
| `site/og.png` | Open Graph preview image (`1200 × 630`), generated from `tools/og.html`. |
| `site/favicon.*`, `site/favicon-32.png`, `site/apple-touch-icon.png` | Site icons. `tools/apple-touch-icon.svg` is the source for `apple-touch-icon.png`. |
| `site/CNAME`, `site/.nojekyll` | Legacy files for branch-based publishing. GitHub Actions deployments ignore both files. |
| `.github/workflows/pages.yml` | Deploys `site/` to GitHub Pages on every push to `main`. |

## Previewing the site locally

With Node.js 24, run:

```sh
npx --yes serve@14.2.8 site
```

Open <http://localhost:3000>. Any unknown path, such as <http://localhost:3000/nope>, renders `site/404.html`, matching GitHub Pages behavior.

## Validating changes

Run the HTML validator, workflow linter, and security scanner before pushing:

```sh
npx --yes html-validate@11.23.0 site/index.html site/404.html
actionlint .github/workflows/pages.yml
zizmor --persona=pedantic .github/workflows/
```

## Regenerating icons and social images

Generate the PNG and ICO icons from `site/favicon.svg` and `tools/apple-touch-icon.svg` using Inkscape and ImageMagick:

```sh
inkscape site/favicon.svg --export-type=png --export-filename=site/favicon-32.png -w 32 -h 32
inkscape tools/apple-touch-icon.svg --export-type=png \
  --export-filename=site/apple-touch-icon.png -w 180 -h 180
inkscape site/favicon.svg --export-type=png --export-filename=/tmp/fav-16.png -w 16 -h 16
inkscape site/favicon.svg --export-type=png --export-filename=/tmp/fav-48.png -w 48 -h 48
magick /tmp/fav-16.png site/favicon-32.png /tmp/fav-48.png site/favicon.ico
```

To regenerate `site/og.png`:

1. Serve the repository root: `npx --yes serve@14.2.8 .`
2. Open <http://localhost:3000/tools/og> in a browser window sized to `1200 × 630`.
3. Save a screenshot of the page to `site/og.png`.

## Deployment and domain configuration

Pushes to `main` in **`ilunesec/ilunesec.github.io`** trigger `.github/workflows/pages.yml`, which uploads `site/` (including hidden files such as `.well-known/security.txt`) and deploys it to GitHub Pages.

Using the `<org>.github.io` repository name hosts the root site at `https://ilunesec.github.io/` and allows future project sites (for example, documentation from `ilunesec/ilune`) to be served under `https://ilune.dev/<repository>/`.

### GitHub Pages settings

- **Source:** In the repository, **Settings** > **Pages** > **Build and deployment** > **Source** is set to **GitHub Actions**.
- **Custom domain:** Configured as `ilune.dev` with **Enforce HTTPS** enabled. (Because `.dev` is on browsers' HSTS preload list, browsers always connect over HTTPS.)
- **Organization domain verification:** `ilune.dev` is verified under **Organization Settings** > **Pages** using the `_github-pages-challenge-ilunesec.ilune.dev` `TXT` record. Keep this `TXT` record active to prevent domain takeovers.

### DNS records for `ilune.dev`

| Type | Name | Value |
|---|---|---|
| `A` | `@` | `185.199.108.153` |
| `A` | `@` | `185.199.109.153` |
| `A` | `@` | `185.199.110.153` |
| `A` | `@` | `185.199.111.153` |
| `AAAA` | `@` | `2606:50c0:8000::153` |
| `AAAA` | `@` | `2606:50c0:8001::153` |
| `AAAA` | `@` | `2606:50c0:8002::153` |
| `AAAA` | `@` | `2606:50c0:8003::153` |
| `CNAME` | `www` | `ilunesec.github.io.` |
| `CAA` | `@` | `0 issue "letsencrypt.org"` |
| `TXT` | `_github-pages-challenge-ilunesec` | Organization verification token |

These addresses match GitHub's [custom domain documentation](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site). Do not add a wildcard record (`*.ilune.dev`), because wildcard records allow subdomain takeovers even on verified domains.

To verify DNS resolution:

```sh
dig ilune.dev A +short
dig ilune.dev AAAA +short
dig www.ilune.dev CNAME +short
dig ilune.dev CAA +short
```

### Redirecting `ilune.io` to `ilune.dev`

GitHub Pages supports one custom domain per site. To forward `ilune.io` and `www.ilune.io` to `https://ilune.dev`:

1. Configure an HTTPS-capable `301` redirect (for example, a Cloudflare Redirect Rule) for `ilune.io` and `www.ilune.io` to `https://ilune.dev`, preserving the request path. Do not use HTTP-only registrar forwarding or a `CNAME`, because `https://ilune.io` requests fail TLS validation without a matching certificate.
2. Add a `CAA` record on `ilune.io` for the certificate authority used by your redirect provider.
3. Keep `ilune.io` verified under **Organization Settings** > **Pages** on GitHub so no other account can claim it.

## Ongoing maintenance

- **Security contact:** Confirm that `security@ilune.dev` (referenced in `site/index.html` and `site/.well-known/security.txt`) receives mail.
- **Renew `security.txt`:** `site/.well-known/security.txt` expires on **November 1, 2027**. Update the `Expires` field before that date; RFC 9116 recommends keeping the expiration within one year.

## License

This site is licensed under the [MIT License](LICENSE). IBM Plex Sans is licensed under the SIL Open Font License 1.1 (`site/fonts/OFL.txt`). The MIT License does not grant rights to the Ilune name or mark.
