# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

The documentation site for [IPF](https://github.com/oehf/ipf) (Open eHealth Integration Platform), published via
GitHub Pages at https://oehf.github.io/ipf-docs. It is a Jekyll site based on the Minimal Mistakes theme.
There is no application code here — content lives in Markdown under `_pages/`, structured data in `_data/`.

## Build & deploy

* **Deploy**: automatic. GitHub Pages rebuilds `master` on push and serves it at
  https://oehf.github.io/ipf-docs — that deployed site is the product. The classic Pages builder ignores this
  repo's `Gemfile` and uses its own pinned gem set (`github-pages`, currently Jekyll 3.10), so Gemfile changes
  affect only local builds and the Algolia workflow, never the deployed site.
* **Local preview**: `bundle exec jekyll serve` → http://localhost:4000/ipf-docs. `remote_theme` works locally
  too, so no `_config.yml` swap is needed.
  Note `incremental: true` in `_config.yml`; delete `_site/` and `.jekyll-metadata` if a change doesn't show up.
  Ruby/bundler are *not* installed in this environment. To verify a build here, use Docker:
  `docker run --rm -v "$PWD":/w -w /w ruby:3.3 bash -c 'bundle install && bundle exec jekyll build'`.
* **Search index**: Algolia, pushed by `.github/workflows/algolia.yml` on every `master` push (and on
  `workflow_dispatch`) via `bundle exec jekyll algolia`. Needs the `ALGOLIA_API_KEY` repository secret — an Algolia
  *admin* key; the search-only key in `_config.yml` cannot write. This is the only CI in the repo; the site build
  itself is GitHub Pages, not Actions.

## Content architecture

### IHE transaction pages are data-driven

`_data/ihe.yml` is the single source of truth for every supported eHealth transaction. Each top-level key
(e.g. `iti119`) carries `transaction`, `link`, `profile`, `description`, `component`, `transport`, `format`,
`client-actor`, `server-actor`, `module`, `section`, `section-link`.

~70 pages under `_pages/ihe/**` start with `{% assign tx = site.data.ihe['<key>'] %}` and then interpolate
`{{ tx.component }}`, `{{ tx.module }}`, `{{ tx.transport }}` etc. into prose, the Maven dependency snippet and the
endpoint URI format. **Fix component names, module names, actors or spec links in `_data/ihe.yml`, not in the page.**

`_pages/ihe/ihe.md` renders the master transaction table by looping over `site.data.ihe` in *file order*, linking to
`{{ tx.link }}/` relative to `/docs/ihe/`. Keep the YAML ordered by transaction, and keep `link` equal to the last
segment of the page's `permalink`.

Adding a transaction therefore means four coordinated edits:
1. entry in `_data/ihe.yml` (in the right position);
2. page at `_pages/ihe/<group>/<key>.md` with `permalink: /docs/ihe/<key>/`;
3. sequence diagram `_src/<key>.plantuml`;
4. link in the relevant group page (`mllp/`, `hl7v3/`, `xds/`, `fhir/`, `hpd/`, `xacml20/`, `svs/`, `ws/`, `hl7v2ws/`)
   and, if it is a top-level topic, in `_data/navigation.yml`.

Pages are grouped by transport/format, mirroring the IPF Maven modules: `mllp` (HL7v2/MLLP), `hl7v3` (SOAP),
`xds` (ebXML/SOAP), `fhir` (REST), `hpd` (DSMLv2), `xacml20`, `svs`, `hl7v2ws`, plus cross-cutting
`atna.md`, `messageValidation.md`, `payloadLogging.md`.

### Diagrams

Actor/sequence diagrams are PlantUML sources in `_src/*.plantuml`, rendered to committed SVGs in
`assets/images/<same-name>.svg` and embedded with
`{% include figure image_path="/assets/images/<name>.svg" alt="..." caption="..." %}`.
Both source and SVG must be committed. `plantuml` is available on this machine:

```bash
plantuml -tsvg -o "$PWD/assets/images" _src/iti119.plantuml
```

### Page conventions

Every page in `_pages/` sets an explicit `permalink` — the config-level `permalink: /:categories/:title/` is not
relied upon. Frontmatter defaults (`layout: single`, `sidebar.nav: sidebar`) come from the `defaults` block in
`_config.yml`; pages add `toc: true` + `toc_icon: align-left` + `toc_sticky: true` for long reference pages, or
`classes: wide` for wide tables and overview pages.

Internal links use reference-style Markdown with Jekyll's `link` tag, which fails the build on a bad path — this is
the intended safety net, so prefer it over hand-written URLs:

```markdown
[ATNA auditing]: {{ site.baseurl }}{% link _pages/ihe/atna.md %}
```

`_data/navigation.yml` drives both the masthead (`main`) and the left sidebar (`sidebar`, nested one level).
Its URLs are permalinks *without* `site.baseurl` — the theme prepends it.

### Versioned migration guides

`_pages/migration/migration-<version>.md` (one per IPF release, `permalink: /docs/migration-<version>/`), indexed from
`migration.md`. New releases get a new file plus an index entry; existing guides are historical and not rewritten.

## Theme

The Minimal Mistakes theme is consumed unmodified through `remote_theme` in `_config.yml`, pinned to an exact tag.
The repo used to carry a full vendored copy of the theme's `_layouts`/`_includes`/`_sass`/`assets`, which shadowed
`remote_theme` and silently pinned the live site to 4.13 no matter what the config said; that copy is gone. Do not
re-add whole theme directories — override single files only, and only with a reason.

The one deliberate override is `_includes/figure`, which adds a `width` parameter that 8 diagram pages pass as
`width="75%"`. It carries a comment saying so; re-apply upstream changes to it when bumping the theme.

Site-level customizations that used to be theme edits now live in `_config.yml`: `atom_feed.hide` (no posts, so no
footer feed link), `copyright`/`copyright_url`, and `author.name`. `minimal_mistakes_skin` is unset, so the default
skin applies and `_sass/.../skins/*` is irrelevant. See README.md for the theme-update procedure.

`_data/navigation.yml` and `_data/ihe.yml` are site content; `_data/ui-text.yml` is *not* — it comes from the theme.

## Do not edit

* `_pages/apidocs/**` — ~10k generated Javadoc HTML files, checked in and served as static content.
* `README-minimal-mistakes.md` — upstream theme documentation, kept for reference and linked from README.md.
