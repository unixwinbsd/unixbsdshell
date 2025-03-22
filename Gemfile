source "https://rubygems.org"

 require 'rbconfig'
    if RbConfig::CONFIG['target_os'] =~ /(?i-mx:bsd|dragonfly)/
      gem 'rb-kqueue', '>= 0.2'
    end

# This will help ensure the proper Jekyll version is running.
# Happy Jekylling!
gem "rake", "~> 12"
gem "jekyll", "~> 3.10.0"
gem "github-pages", "~> 232", group: :jekyll_plugins

# If you want to use Jekyll native, uncomment the line below.
# To upgrade, run `bundle update`.

install_if -> { RUBY_PLATFORM =~ %r!mingw|mswin|java! } do
gem "tzinfo"
  gem "tzinfo-data"
end

# If you have any plugins, put them here!
group :jekyll_plugins do
gem "jekyll-paginate"
gem "jekyll-feed"
gem "jekyll-redirect-from", "~> 0.16.0"
gem 'jekyll-seo-tag', git: 'https://github.com/adamsdesk/jekyll-seo-tag.git', branch: 'fix-json-ld-alt'
gem "jekyll-sitemap", "~> 1.4.0"
gem "jekyll-github-metadata"
gem "jekyll-gist"
#gem "jekyll-polyglot"
gem "jemoji"
gem "csv"
gem "wdm", "~> 0.1.1" if Gem.win_platform?
gem "octokit", ">= 4.25.0"
  gem "jekyll-archives"
end
gem "webrick", "~> 1.8"