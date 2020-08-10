sudo apt install ruby
sudo gem install bundler

add this to Gemfile:
source "https://rubygems.org"
gem 'github-pages', group: :jekyll_plugins

bundle install
failed with:
Installing commonmarker 0.17.13 with native extensions
Gem::Ext::BuildError: ERROR: Failed to build gem native extension.

    current directory: /tmp/bundler20190818-12608-1seg4iqcommonmarker-0.17.13/gems/commonmarker-0.17.13/ext/commonmarker
/usr/bin/ruby2.5 -r ./siteconf20190818-12608-mth4s8.rb extconf.rb
creating Makefile

current directory: /tmp/bundler20190818-12608-1seg4iqcommonmarker-0.17.13/gems/commonmarker-0.17.13/ext/commonmarker
make "DESTDIR=" clean
sh: 1: make: not found

current directory: /tmp/bundler20190818-12608-1seg4iqcommonmarker-0.17.13/gems/commonmarker-0.17.13/ext/commonmarker
make "DESTDIR="
sh: 1: make: not found

make failed, exit code 127

Gem files will remain installed in /tmp/bundler20190818-12608-1seg4iqcommonmarker-0.17.13/gems/commonmarker-0.17.13 for inspection.
Results logged to /tmp/bundler20190818-12608-1seg4iqcommonmarker-0.17.13/extensions/x86_64-linux/2.5.0/commonmarker-0.17.13/gem_make.out

An error occurred while installing commonmarker (0.17.13), and Bundler cannot continue.
Make sure that `gem install commonmarker -v '0.17.13' --source 'https://rubygems.org/'` succeeds before bundling.

In Gemfile:
  github-pages was resolved to 198, which depends on
    jekyll-commonmark-ghpages was resolved to 0.1.5, which depends on
      jekyll-commonmark was resolved to 1.3.1, which depends on
        commonmarker



sudo gem install bundler

sudo apt-get install ruby-dev
sudo apt-get install ruby-all-dev zlib1g-dev libxslt1-dev libxml2-dev
sudo gem install commonmarker