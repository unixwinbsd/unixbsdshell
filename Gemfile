source "https://rubygems.org"
# Hello! This is where you manage which Jekyll version is used to run.
# When you want to use a different version, change it below, save the
# file and run `bundle install`. Run Jekyll with `bundle exec`, like so:
#
#     bundle exec jekyll serve
#
# This will help ensure the proper Jekyll version is running.
# Happy Jekylling!


gem "rake"
gem "jekyll"
gem "github-pages", group: :jekyll_plugins

# If you have any plugins, put them here!
group :jekyll_plugins do
gem "jekyll-paginate"
gem "jekyll-feed"
gem "jekyll-redirect-from"
gem 'jekyll-seo-tag', git: 'https://github.com/adamsdesk/jekyll-seo-tag.git', branch: 'fix-json-ld-alt'
gem "jekyll-sitemap"
gem "jekyll-github-metadata"
gem "jekyll-gist"
gem "jekyll-polyglot"
gem "jemoji"
gem "csv"
gem "wdm", "~> 0.1.1" if Gem.win_platform?
gem "octokit"
gem "jekyll-archives"
end

# Windows and JRuby does not include zoneinfo files, so bundle the tzinfo-data gem
# and associated library.
platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo"
  gem "tzinfo-data"
end

# Performance-booster for watching directories on Windows
gem "wdm"

# Lock `http_parser.rb` gem to `v0.6.x` on JRuby builds since newer versions of the gem
# do not have a Java counterpart.
gem "http_parser.rb"
gem "webrick"
