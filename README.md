# SearchPilot demo site

A small, static, four-page marketing site used as the **audit target** for SearchPilot
demonstrations. "Northline Analytics" is a fictional B2B brand; the site exists so a
demo operator can point SearchPilot at a real public URL and show deterministic findings
landing on the right pages.

Plain HTML5 + one stylesheet. No build step, no JavaScript, no frameworks, no external
fonts, CDNs, analytics, or tracking. Light theme by default with a `prefers-color-scheme`
dark variant.

## Files

| File | Purpose |
| --- | --- |
| `index.html` | Healthy home page: unique title, meta description, one H1, links to every page. |
| `services.html` | Deliberately missing its meta description, and deliberately thin. |
| `about.html` | Deliberate image without `alt`, deliberate heading-level skip, title/description shared with `contact.html`. |
| `contact.html` | Reuses the About page `<title>` and meta description verbatim. |
| `styles.css` | Shared stylesheet for all four pages. |
| `team.svg` | Local illustration used by the `<img>` that is missing `alt` on `about.html`. |

## Run it locally

From the repository root:

```bash
python3 -m http.server 8080 -d demo-site
```

Then open <http://localhost:8080/>. The site is plain static files, so any static server
works; `python3 -m http.server` is the lowest-friction option.

**Do not submit `http://localhost:8080` to SearchPilot.** See the SSRF caveat below.

## The deliberate issues

Expected findings, with the rule IDs and severities produced by SearchPilot's
deterministic SEO engine. Severity is `high`/`medium`/`low` as stored on the finding.

| Page | Rule ID | Severity | What is wrong on the page |
| --- | --- | --- | --- |
| `services.html` | `missing_meta_description.v1` | medium | No `<meta name="description">` element at all. |
| `services.html` | `thin_content.v1` | high | Body copy is deliberately thin: ~45 words in the hero section, ~79 words in the whole `<main>` (threshold is 300). |
| `services.html` | `weak_topical_coverage.v1` | low | Expected knock-on effect of the thin page in a four-page site. |
| `about.html` | `image_missing_alt.v1` | medium | `team.svg` is referenced by an `<img>` with no `alt` attribute (1 of 1 images missing alt). |
| `about.html` | `heading_structure.v1` | low | Heading outline skips a level: `h2` → `h4` ("How we started" → "The first eighteen months"). |
| `about.html` | `duplicate_title.v1` | medium | Shares the `<title>` "About Northline Analytics" with `contact.html`. |
| `about.html` | `duplicate_meta_description.v1` | low | Shares the meta description verbatim with `contact.html`. |
| `contact.html` | `duplicate_title.v1` | medium | Same `<title>` as `about.html` (flagged as the duplicate of the pair). |
| `contact.html` | `duplicate_meta_description.v1` | low | Same meta description as `about.html` (flagged as the duplicate of the pair). |

`index.html` is intentionally clean: it produces **no** findings. It carries a unique
title, a unique meta description, exactly one H1, a self-referencing canonical, a
non-skipping heading outline, and enough prose (~353 words) to stay clear of the
thin-content threshold.

The issues are marked with HTML comments in the source, for example:

```html
<!-- Deliberate gap: this page ships without a meta description so the demo
     triggers the missing_meta_description.v1 rule. -->
```

Everything else about the site is production-shaped on purpose: semantic landmarks
(`header`/`nav`/`main`/`footer`), a skip link, visible focus rings, labelled form
fields, `aria-current` on the active nav item, `lang` and viewport metadata, and
internal links that all resolve to the four pages above.

## Hosting it publicly

SearchPilot crawls a **public** URL, so the demo site has to be reachable from the
internet for the demo to work end to end.

**GitHub Pages.** Push `demo-site/` to the repository (or a dedicated branch), then in
the repository go to *Settings → Pages*, set *Source* to *Deploy from a branch*, pick the
branch and the `/demo-site` folder (choose *root* if the folder is the repository root),
and save. The site is then at `https://<user>.github.io/<repo>/demo-site/`. Note that
Pages is served over HTTPS with no build step, which is exactly what this site needs.

**Netlify.** Drag the `demo-site/` folder into the Netlify drop zone, or connect the
repository and set *Publish directory* to `demo-site` with *Build command* left empty.
The site is then at `https://<site-name>.netlify.app/`. Netlify serves `index.html` for
the directory root automatically.

Either way, submit the **directory URL** (the one ending in `/`) as the audit target.
All links inside the site are relative, so the site works unchanged under any host or
sub-path.

## SSRF caveat (important)

SearchPilot refuses to crawl targets that resolve to a non-public address. Its SSRF
guard blocks loopback, private, link-local, and reserved ranges, cloud metadata hosts
such as `169.254.169.254` and `metadata.google.internal`, and the hostnames
`localhost`, `*.localhost`, `*.local`, and `*.internal`.

Consequences for the demo:

- `http://localhost:8080` and `http://127.0.0.1:8080` are **blocked**. The API answers
  with `blocked_target` (HTTP 400). This is the guard working correctly, not a bug — see
  the `Security note` in the repository README and `backend/tests/security/`.
- Serve the site locally for **reading only**, and use the public hosted URL for the
  audit itself.
- If no public URL has been provisioned, submit **`https://example.com`** instead. It
  is small and stable, but it is not tuned to trigger every finding in the table above.
- For a containerised demo, the target must be a public URL that resolves outside the
  container's private network; `https://example.com` is the safe fallback.

## Notes

- The site is a demo fixture, not a real product page: no analytics, no cookies, no
  network requests of any kind beyond loading `styles.css` and `team.svg`.
- `contact.html` contains a labelled form with no backend. Submitting it is a no-op in
  the static build; the page says so under the message field.
- Keep the deliberate issues intact. If they are "fixed", the demo loses the findings
  it exists to produce.
