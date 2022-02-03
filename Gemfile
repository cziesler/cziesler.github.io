source "https://rubygems.org"

# Hello! This is where you manage which Jekyll version is used to run.
# When you want to use a different version, change it below, save the
# file and run `bundle install`. Run Jekyll with `bundle exec`, like so:
#
#     bundle exec jekyll serve
#
# This will help ensure the proper Jekyll version is running.
# Happy Jekylling!
#gem "jekyll", "~> 3.9.0"

# This is the default theme for new Jekyll sites. You may change this to anything you like.
#gem "minima", "~> 2.5.1"
gem "minima", :github => 'jekyll/minima'

# If you want to use GitHub Pages, remove the "gem "jekyll"" above and
# uncomment the line below. To upgrade, run `bundle update github-pages`.
gem "github-pages", "~> 223", group: :jekyll_plugins

# If you have any plugins, put them here!
group :jekyll_plugins do
  #gem "jekyll-feed", "~> 0.15.1"
end

# Windows does not include zoneinfo files, so bundle the tzinfo-data gem
gem "tzinfo-data", platforms: [:mingw, :mswin, :x64_mingw, :jruby]

# For Cygwin
gem "bigdecimal"
gem 'wdm', '>= 0.1.0' if Gem.win_platform?

# https://stackoverflow.com/questions/65989040/bundle-exec-jekyll-serve-cannot-load-such-file
gem "webrick"

# https://stackoverflow.com/questions/63335953/jekyll-error-building-page-related-to-kramdown-parser
#gem "kramdown-parser-gfm"
