# Dennisdoomen.github.io

The future home of <http://www.continuousimprover.com/>.

## How to Build this Site

### Prerequisites

* Ruby 4.x (or Ruby 3.x). The project is compatible with modern Ruby versions.
* An easy way to install Ruby is to download the latest version from the [RubyInstaller site](https://rubyinstaller.org/downloads/) for Windows, or use a version manager like `rbenv` or `rvm` on Linux/macOS.
* The `bundler` gem (`gem install bundler`). If you receive SSL-related errors while running `gem install`, try running `refreshenv` first. 

### Building

* Clone this repository
* `cd` into the root of the repository
* Run `bundle install`. 
* Run `bundle exec jekyll serve`. To have it monitor your working directory for changes, add the `--incremental` option. 

## Troubleshooting

* Do you receive an error around `jekyll-remote-theme` and `libcurl`? See [this issue on the pages-gem repo](https://github.com/github/pages-gem/issues/526).
* Do you receive an error `Liquid Exception: SSL_connect returned=1 errno=0 state=error: certificate verify failed`? Check out [this solution in the Jekyll repo.](https://github.com/jekyll/jekyll/issues/3985#issuecomment-294266874)
* 