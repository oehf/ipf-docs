# IPF Documentation

This is a GitHub Pages project documenting the [eHealth Integration Platform](https://github.com/oehf/ipf) (IPF).
The documentation is deployed on github at [this](https://oehf.github.io/ipf-docs) location.
The previous (obsolete) documentation can still be found [here](https://oehf.github.io/ipf).

The layout is based on the Minimal Mistakes Theme (see description in README-minimal-mistakes.md).
The theme is pulled in unmodified via the `remote_theme` property in `_config.yml`; the only local
override is `_includes/figure`, which adds a `width` parameter to the upstream include.

## Build remotely

There is nothing to be done. GitHub Pages are automatically rebuilt and redeployed after a commit.

## Build locally

In order to build and work locally,

* Follow [these](https://help.github.com/articles/setting-up-your-github-pages-site-locally-with-jekyll/) instructions.
  Hint: install/download the required ruby gems and jekyll plugin without using a proxy!
* Run `bundle exec jekyll serve` — `remote_theme` works locally too, so no change to `_config.yml` is needed.
* The site is served on `http://localhost:4000/ipf-docs`

## Update the theme

* Check the [Minimal Mistakes releases](https://github.com/mmistakes/minimal-mistakes/releases) and its
  [CHANGELOG](https://github.com/mmistakes/minimal-mistakes/blob/master/CHANGELOG.md) for breaking changes.
* Bump the version in the `remote_theme` property of `_config.yml` (and the `minimal-mistakes-jekyll`
  version in the `Gemfile`, which is only a fallback for `theme`-based builds).
* Re-apply the upstream changes to `_includes/figure` if that include changed upstream.
* Build locally and check the home page, a transaction page (e.g. `/docs/ihe/iti9/`) and the search page.

## TO-DOs

* Javadocs not included yet
* Add description here on how/where to add or edit pages
* Links missing, some polishing