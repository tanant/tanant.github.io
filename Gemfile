source "https://rubygems.org"
# Hello! This is where you manage which Jekyll version is used to run.
# When you want to use a different version, change it below, save the
# file and run `bundle install`. Run Jekyll with `bundle exec`, like so:
#
#     bundle exec jekyll serve
#
# This will help ensure the proper Jekyll version is running.
# Happy Jekylling!
# gem "jekyll", "~> 4.0"
# This is the default theme for new Jekyll sites. You may change this to anything you like.
gem "minima", "~> 2.5"
#gem "minima", path: "/mnt/h/repos/tanant/minima"

# gem "jekyll-theme-hacker", "~> 0.1"
#
# gem "jekyll-theme-hacker", git: 'https://github.com/tanant/hacker'

# If you want to use GitHub Pages, remove the "gem "jekyll"" above and
# uncomment the line below. To upgrade, run `bundle update github-pages`.

# okay. Decision to go this way, because 
gem "github-pages", group: :jekyll_plugins
# If you have any plugins, put them here!
group :jekyll_plugins do
  gem "jekyll-feed", "~> 0.12"
  gem "kramdown", ">= 2.3.1"
  gem "rouge"

end

# Windows and JRuby does not include zoneinfo files, so bundle the tzinfo-data gem
# and associated library.
platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", "~> 1.2"
  gem "tzinfo-data"
end

# Performance-booster for watching directories on Windows
gem "wdm", "~> 0.1.1", :platforms => [:mingw, :x64_mingw, :mswin]



# todo - verify the entire remote theme thing. let's base it off minima 251, for now
# okay. So i can continue to muck around with this, we can hack around with minima directly
# and then if we need we can directly just you know, do a diff and port over. This works for me

# step 1 - let's get the dev loop going

