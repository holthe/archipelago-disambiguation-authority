<p align="center">
  <img src="assets/logo.svg" alt="Archipelago Disambiguation Authority" width="620">
</p>

<p align="center">
  <strong>Office of the Registrar &middot; Not for navigation.</strong><br>
  Fifteen definitions &middot; Nine territories &middot; Eleven distinct patterns.
</p>

---

This repository contains the public site for the Archipelago Disambiguation
Authority (archipelago.besteffortindustries.com), which determines which
team you play for, which passport you hold, and which of those is currently
true.

## The determination

British Isles, Great Britain, the United Kingdom and Team GB are four
different places. So are the Six Nations, Eurovision, the Commonwealth
Games and the Crown Dependencies. Each is a real boundary drawn for a real
purpose, each includes and excludes a different set of the same nine
territories, and no two of them agree.

The Authority does not resolve this. It projects each definition onto the
same map and shows you which land rises.

## What the site actually does

Everything runs client-side:

* **The chart** takes any of the fifteen definitions and lights the
  territories inside it, leaving the rest as the same land, excluded by the
  same map.
* **The matrix** puts every definition against every territory at once, so
  the eleven distinct patterns are visible as patterns rather than as a
  list of exceptions.
* **The register** records each definition with the instrument that created
  it and the reason it draws the line where it does.

---

## Development notes

The parody ends here. The rest of this file is accurate.

### Layout

A static, zero-build, zero-dependency site. Two HTML files and a handful of
generated images. There is no framework, no bundler and no `package.json`.
Cloudflare Pages serves the repository root exactly as it appears here.

```
index.html            the site, chart, matrix and register included
404.html              catch-all, served automatically by Cloudflare Pages
favicon.svg           icon, written by hand (no text, so nothing to outline)
favicon.ico           16/32/48, generated
apple-touch-icon.png  180x180, generated
og.png                1200x630 share image, generated
assets/logo.svg       wordmark, text outlined, used at the top of this README
tools/og.html         source for og.png
tools/logo-src.svg    source for assets/logo.svg, text still live
tools/favicon-16.svg  pixel-grid 16px icon, used for the smallest .ico entry
Makefile              asset regeneration only, never runs at deploy time
_headers              Cloudflare Pages header rules
robots.txt            permissive
wrangler.toml         Cloudflare Pages configuration
mise.toml             pins the Wrangler version used to deploy
```

### One deviation from the fleet

Every other division on the register makes **zero** requests to any external
domain. This one loads three webfonts from Google Fonts: Instrument Serif,
Inter and IBM Plex Mono. The typography is the design, and substituting
system faces would change the site rather than tidy it.

The consequence is stated rather than hidden: a visitor's browser contacts
`fonts.googleapis.com` and `fonts.gstatic.com` on load, and the page falls
back to Georgia, a system sans and a system monospace if those are blocked.
If the fleet's no-external-requests rule is ever made absolute, the fix is
to inline the three faces as base64 `@font-face` sources and delete the two
`preconnect` links and the stylesheet link from `index.html` and `404.html`.

### Brought in from outside

`index.html` began as a standalone page and was adapted to fleet standards
rather than rewritten:

* The masthead carried **Best Effort Industries · Division 17**. Both halves
  broke a rule: the parent is named in the footer only, and no site records
  its own division number, because the register on besteffortindustries.com
  is the sole authority on numbering and it renumbers when the queue closes
  up. The masthead now identifies the Authority itself.
* The footer's existing mention of the parent was kept, and now links to it.
* A `DOC: BEI-ADA` line was added, derived from the Authority's own name so
  it cannot go stale.
* The head gained `rel=canonical`, the Open Graph and Twitter blocks, the
  favicon wiring and `theme-color`. It previously had a title and a
  description and nothing else, which meant no link preview at all.

### The production domain

The Authority has no domain of its own, so its canonical host is a subdomain
of the parent: `archipelago.besteffortindustries.com`. That is the host every
absolute URL on the page points at, so link previews resolve. If the site is
ever promoted to a domain of its own, the canonical host changes in the
places below and nothing else derives it:

| File | What to change |
| --- | --- |
| `index.html` | `rel=canonical`, `og:url`, `og:image`, `twitter:image` |

The `og:image` and `twitter:image` URLs carry a `?v=` suffix. It is a cache
key: `og.png` is served with a seven-day `max-age` and its path never changes,
so a regenerated card stays invisible behind the edge cache, and behind the
copy each social platform keeps, until the URL itself changes. **Bump `?v=`
in `index.html` every time `make og` is run**, or the new card will not be
the one that circulates.
| `404.html` | nothing, the 404 uses only root-relative paths |
| `tools/og.html` | the domain printed in the footer of the share image |
| `README.md` | this table, and the mentions above it |

After changing `tools/og.html`, re-run `make og`.

### Local preview

```sh
make serve          # python3 -m http.server 8000
```

Then open `http://localhost:8000`. A local server is preferable to opening
the file directly because the icon paths are root-absolute.

### Regenerating images

Only needed when the wordmark, the icon or the share image changes.
Requires `google-chrome`, ImageMagick 7 (`magick`) and Inkscape on the
machine doing the regenerating; none of them is needed to deploy, because
the outputs are committed.

```sh
make assets         # everything below
make og             # og.png     <- tools/og.html, via headless Chrome
make favicon        # favicon.ico + apple-touch-icon.png <- the SVG sources
make logo           # assets/logo.svg <- tools/logo-src.svg, text outlined
```

`make logo` outlines the wordmark's text so the README renders the same
whether or not the viewer has the fonts. Inkscape rewrites the whole file,
so the `GENERATED` comment at the top has to be pasted back afterwards.
`favicon.svg` is not generated; it has no text in it.

The map on the share card is not a redrawing. Its nine paths were lifted
verbatim from the live page: `pathFor()` in `index.html` builds them from the
`GEO` coordinates, and `tools/og.html` embeds the strings it produced, in the
same 584x835 viewBox. The card therefore shows the same territories in the
same projection and palette as the site, with Team GB selected. **If the
geometry in `index.html` changes, re-extract those paths** rather than editing
them by hand; the header comment in `tools/og.html` says the same thing.

### Deploying

Wrangler is configured via `wrangler.toml`, so a deploy is one command from
an authenticated shell:

```sh
make deploy         # wrangler pages deploy .
```

The Wrangler version is pinned by `mise.toml` (this machine manages its
Wrangler through [mise](https://mise.jdx.dev/); the global config tracks
`latest`, the repo pins an exact version). To move the pin, edit
`mise.toml`, run `mise install`, and deploy once to confirm nothing moved
underneath.

### Which Cloudflare account this deploys to

This machine has two Cloudflare identities, and picking the wrong one
deploys this site into an unrelated organisation.

**Pages configuration cannot pin the account.** `account_id` is a
Workers-only key; putting it in a Pages `wrangler.toml` makes Wrangler
refuse to run. So the account is selected by **an auth profile bound to this
directory**, recorded in
`~/.config/.wrangler/profiles/directory-bindings.json`:

```sh
wrangler auth activate personal    # already done; re-run after moving the repo
wrangler whoami                    # must print: Active profile: personal
```

Without a binding, Wrangler falls back to the `default` profile, which here
is the other organisation, and it will deploy there without asking. **Check
`whoami` before deploying.** The binding lives outside the repo, so a fresh
clone, a moved directory, or another machine all need `wrangler auth
activate` again.

One extra trap: Wrangler caches the resolved account in the untracked
`.wrangler/cache/wrangler-account.json` inside this directory. If a deploy
ever went to the wrong account from here, activating the right profile is
**not** enough; delete `.wrangler/` as well, or the cached account ID wins
and the API call fails with `Authentication error [code: 10000]`.

For CI, where profiles do not exist, set `CLOUDFLARE_ACCOUNT_ID` (the account
to deploy into) and `CLOUDFLARE_API_TOKEN` (credentials scoped to it) as
environment variables.

The Pages project is `archipelago`, production branch `main`, with no build
command and the build output directory set to `/`. If you ever recreate it
from the dashboard, use exactly those values; there is nothing to build, and
any build command entered there will only make the deployment worse.

To wire the Git integration instead, connect the
`holthe/archipelago-disambiguation-authority` repository under **Workers &
Pages -> Create -> Pages -> Connect to Git** with the same settings.

### Custom domain

Deploy at least once first, so the project exists. Then, in the dashboard
under **Workers & Pages** -> `archipelago` -> **Custom domains** -> **Set up
a custom domain**, add `archipelago.besteffortindustries.com`. The zone is
already on Cloudflare, so the CNAME is created for you; do not create the
record by hand first, because a pre-existing record blocks the flow.
Universal SSL already covers one level of subdomain on that zone, so the
certificate needs no extra step.

One trap worth recording: `archipelago.pages.dev` belongs to an unrelated
project, so Cloudflare assigned this one a suffix and the deployment answers
on **`archipelago-ffl.pages.dev`**. Any DNS record for it must target that
hostname, not the project name. The Pages project itself is still called
`archipelago`.

Until the domain is attached the site is reachable at
`archipelago-ffl.pages.dev`.

### Related

The Authority is a division of
[Best Effort Industries](https://besteffortindustries.com). The register
there is the only authority on division numbering, and this repository
deliberately records none: the site files itself as `BEI-ADA`, which is
derived from the Authority's own name and cannot go stale when the register
renumbers.

## License

Parody. The definitions are real, the territories are real, the
disagreements between them are real, and the Authority is the only party
involved that never existed.
