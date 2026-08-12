source "https://rubygems.org"
gem "minimal-mistakes-jekyll", "~> 4.28" # keep in sync with `remote_theme` in _config.yml
# Pinned to the gem set GitHub Pages itself deploys (see https://pages.github.com/versions/).
# Unpinned this resolves to 222 (liquid 4.0.3), which breaks `jekyll algolia` on Ruby >= 3.2.
gem "github-pages", "~> 232", group: :jekyll_plugins
gem "tzinfo-data", platforms: [:mingw, :mswin, :x64_mingw, :jruby]
gem "wdm", "~> 0.1.0" if Gem.win_platform?

# If you have any plugins, put them here!
group :jekyll_plugins do
  gem "jekyll-mermaid"
  gem "jekyll-paginate"
  gem "jekyll-sitemap"
  gem "jekyll-gist"
  gem "jekyll-feed"
  gem "jemoji"
  gem "jekyll-include-cache"
  gem "jekyll-algolia"
end
gem "webrick", "~> 1.8"
