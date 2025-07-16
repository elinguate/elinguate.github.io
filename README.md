# elinguate.github.io

# Quick Start

Docs for testing locally:
https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll

> bundle install
> bundle exec jekyll serve --livereload --force-polling

Make sure to use --livereload (and --force-polling) above, as it will ensure that live changes get updated automatically *and* the theme won't break when you do so!

It can help to tick "Disable Cache" in the browser Network tab, as sometimes the caching can cause issues.

If the theme breaks just terminate, and recall the command.

## General Install Guide

Install Ruby: https://www.ruby-lang.org/en/documentation/installation/
Install RubyGems: https://rubygems.org/pages/download
Install GCC: Annoying + Add environment variable
Install Make: https://gnuwin32.sourceforge.net/packages/make.htm - Setup program + Add environment variable

Install Jekyll: 
> gem install jekyll bundler
