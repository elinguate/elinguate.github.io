# elinguate.github.io

# Quick Start

Docs for testing locally:
https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll

> bundle install
> bundle exec jekyll serve --livereload --open-url

Make sure to use --livereload, as it will ensure that live changes get updated automatically *and* the theme won't break when you do so!

The --open-url will just automatically open the locally hosted address when it's ready!

It can help to tick "Disable Cache" in the browser Network tab, as sometimes the caching can cause issues (stylesheet doesn't update, etc.).

Sometimes the theme will turn off when editing files that aren't the .css - if so just tab back and re-save the .css file to reapply it.

If the theme breaks fully just terminate, and recall the command.

## General Install Guide

- Install Ruby: https://www.ruby-lang.org/en/documentation/installation/
- Install RubyGems: https://rubygems.org/pages/download
- Install GCC: Annoying + Add environment variable
- Install Make: https://gnuwin32.sourceforge.net/packages/make.htm + Setup program + Add environment variable

Install Jekyll: 
> gem install jekyll bundler
