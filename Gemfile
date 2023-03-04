# frozen_string_literal: true
source "https://rubygems.org"

gem "github-pages", group: :jekyll_plugins

gem "tzinfo-data"
gem "wdm", "~> 0.1.0" if Gem.win_platform?

# If you have any plugins, put them here!
group :jekyll_plugins do
  gem "jekyll-paginate"
  gem "jekyll-sitemap"
  gem "jekyll-gist"
  gem "jekyll-feed"
  gem "jemoji"
  gem "jekyll-include-cache"
  gem "jekyll-algolia"
end
gem "webrick"

# commenting below to remove dependency with "github-pages" 
# gem "github-pages", group: :jekyll_plugins

gem "jekyll-seo-tag"
gem "jekyll-sitemap"

# https://github.com/jekyll/jekyll/issues/8523#issuecomment-751409319
# When running locally, we run into the following error —
# `require': cannot load such file -- webrick (LoadError)
# adding this avoids it
gem "webrick"

# adding the following gems to support removal of "github-pages" dependency
gem "jemoji"
gem "kramdown-parser-gfm"
