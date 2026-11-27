# ilune.dev

The coming-soon page for Ilune, served by GitHub Pages at <https://ilune.dev>.

It is one static page with no build step, no JavaScript and no third-party requests: the fonts,
icons and styles are all served from the site itself.

## What is here

| Path | What it is |
|---|---|
| `site/` | Everything that is published. The workflow uploads this folder and nothing else. |
| `site/index.html`, `site/404.html`, `site/styles.css` | The page, the not-found page and their shared styles. |
| `site/fonts/` | IBM Plex Sans 400, 600 and 700, Latin subset, from Fontsource 5.3.0, with its licence (`OFL.txt`). |
| `site/.well-known/security.txt` | Where to report a security issue ([RFC 9116](https://www.rfc-editor.org/rfc/rfc9116)). |
| `site/og.png` | The image shown when the link is shared. `tools/og.html` is its source. |
| `site/favicon.*`, `site/favicon-32.png`, `site/apple-touch-icon.png` | Icons. `tools/apple-touch-icon.svg` is the square source of the last one. |
| `site/CNAME`, `site/.nojekyll` | Kept for branch publishing. A GitHub Actions deployment ignores both: the domain comes from the Pages settings, and no Jekyll build runs. |
| `.github/workflows/pages.yml` | Deploys `site/` to GitHub Pages on every push to `main`. |

## Preview locally

With Node.js 24:

```sh
npx --yes serve@14.2.8 site
```

Then open <http://localhost:3000>. Any unknown address, such as <http://localhost:3000/nope>,
shows the not-found page, as it will on GitHub Pages.

## Check a change

```sh
npx --yes html-validate@11.23.0 site/index.html site/404.html
actionlint .github/workflows/pages.yml
zizmor --persona=pedantic .github/workflows/
```

All three report nothing on the current files.

## Regenerate the images

The icons come from `site/favicon.svg` with Inkscape and ImageMagick:

```sh
inkscape site/favicon.svg --export-type=png --export-filename=site/favicon-32.png -w 32 -h 32
inkscape tools/apple-touch-icon.svg --export-type=png \
  --export-filename=site/apple-touch-icon.png -w 180 -h 180
inkscape site/favicon.svg --export-type=png --export-filename=/tmp/fav-16.png -w 16 -h 16
inkscape site/favicon.svg --export-type=png --export-filename=/tmp/fav-48.png -w 48 -h 48
magick /tmp/fav-16.png site/favicon-32.png /tmp/fav-48.png site/favicon.ico
```

For `site/og.png`, serve the repository root (`npx --yes serve@14.2.8 .`), open
<http://localhost:3000/tools/og> in a browser window sized to 1200 × 630, and save a screenshot of
the page.

## Publish the site

Do these in order. Steps 2 and 3 come before any DNS change for the site, so nobody else can
claim `ilune.dev` on GitHub Pages in between.

### 1. Create the repository and push

Create the repository as **`ilunesec/ilunesec.github.io`**, public, with no README or licence
(this repository has both). Then:

```sh
git remote add origin git@github.com:ilunesec/ilunesec.github.io.git
git push -u origin main
```

Why this name rather than, say, `ilunesec/site`: a repository named `<org>.github.io` is the
organization's own Pages site. Its default address is `https://ilunesec.github.io/`, which is
exactly what the `www` record below points at, and the organization's other Pages sites (for
example, documentation published later from `ilunesec/ilune`) are then served under the same
domain at `https://ilune.dev/<repository>/`. A project repository such as `ilunesec/site` works
as well, but its default address has the repository name in its path, and the organization can
only ever have one site at its root domain anyway.

The first push runs the workflow, which fails until Pages is turned on in step 3. Re-run it
afterwards from the **Actions** tab, or push again.

### 2. Verify `ilune.dev` for the organization

This stops anyone else from publishing a GitHub Pages site on `ilune.dev` or its subdomains.

1. Open the organization's **Settings** > **Pages** (under "Code, planning, and automation"),
   not the repository's settings.
2. Select **Add a domain**, enter `ilune.dev` and follow the instructions to add a DNS `TXT`
   record. Its name is `_github-pages-challenge-ilunesec.ilune.dev`.
3. Wait until `dig _github-pages-challenge-ilunesec.ilune.dev TXT +short` shows the value, then
   select **Verify**. Keep the `TXT` record for as long as the site exists.

Separately, **Settings** > **Verified and approved domains** verifies the domain for the
organization profile and email notifications. It is optional, and it does not protect Pages.

### 3. Turn on Pages with GitHub Actions

In the repository: **Settings** > **Pages** > **Build and deployment** > **Source**: select
**GitHub Actions**.

### 4. Set the custom domain

Still in **Settings** > **Pages**, under **Custom domain**, enter `ilune.dev` and select
**Save**. With an Actions deployment this value is what counts; GitHub does not read `CNAME`.

### 5. Add the DNS records for ilune.dev

Remove any default `A`, `AAAA`, `ALIAS` or parking records your DNS provider created for `@` and
`www`, then add:

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
| `TXT` | `_github-pages-challenge-ilunesec` | the value from step 2 |

These are the addresses in GitHub's documentation, [Managing a custom domain for your GitHub
Pages site](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site),
as of 27 November 2026. Check them there before you add them.

- GitHub Pages gets its certificate from Let's Encrypt, so the `CAA` record allows only that
  authority. If another service issues certificates for `ilune.dev` or a subdomain (an email
  provider, for example), add an `issue` record for its authority as well.
- Do not add a wildcard record such as `*.ilune.dev`. GitHub warns that it allows takeovers even
  of a verified domain.
- With the `www` record in place, GitHub redirects `www.ilune.dev` to `ilune.dev`.

Check the records:

```sh
dig ilune.dev A +short
dig ilune.dev AAAA +short
dig www.ilune.dev CNAME +short
dig ilune.dev CAA +short
```

### 6. Enforce HTTPS

Once **Settings** > **Pages** shows the DNS check as successful and the certificate as issued
(this can take up to 24 hours), select **Enforce HTTPS**. If the certificate does not appear
after an hour, remove the custom domain, save it again and wait.

`.dev` is on browsers' HSTS preload list as a whole top-level domain, so browsers only ever
connect to `ilune.dev` over HTTPS, even before this setting is on.

### 7. Make security@ilune.dev receive mail

The page and `/.well-known/security.txt` both name `security@ilune.dev`. Set up the mailbox or a
forward, and send it a test message, before the site goes live.

`security.txt` expires on 1 November 2027. Move its `Expires` date forward before then; RFC 9116
recommends a date less than a year ahead.

### Later: send ilune.io to ilune.dev

A GitHub Pages site has one custom domain, so `ilune.io` cannot be added as a second one. Use a
redirect at the DNS provider instead:

1. At the DNS provider for `ilune.io`, create a permanent (301) redirect for `ilune.io` and
   `www.ilune.io` to `https://ilune.dev`, keeping the path. Cloudflare calls this a Redirect
   Rule; many registrars call it URL forwarding. Choose one that serves HTTPS on `ilune.io`
   itself, since links to `https://ilune.io` fail otherwise.
2. Add a `CAA` record on `ilune.io` for the authority that provider uses.
3. Verify `ilune.io` for the organization too (step 2 above), even though Pages does not serve
   it. That keeps anyone else from claiming it on GitHub Pages if a record ever points there.

## Licence

The site is under the [MIT licence](LICENSE). MIT covers the markup and styles as well as the
words, with one short notice to keep, which suits a page that is mostly code. CC BY 4.0 is
written for text and images, and Creative Commons advises against it for software. IBM Plex Sans
is under the SIL Open Font License 1.1 (`site/fonts/OFL.txt`). The licence does not grant rights
to the Ilune name or mark.
