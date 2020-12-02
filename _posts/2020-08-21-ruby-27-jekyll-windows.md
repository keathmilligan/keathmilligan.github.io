---
layout: post
title: Running Jekyll on Windows with Ruby 2.7
date: 2020-08-21 08:12:12 -0500
categories: [dev]
tags: [ruby, jekyll, github-pages, windows]
image: /assets/images/icon-ruby-notext-color.svg
---

On Windows, Jekyll will not run out of the box with Ruby 2.7.x. A few additional steps are required. Unfortunately many of the guides out there are out of date and/or refer to tools that are no longer supported so here are the steps that worked for me as of the time of this post.
<!--more-->

### Ruby 2.7

This guide assumes you are working with the latest Ruby 2.7 (currently 2.7.1.1) installed from [Chocolatey](https://chocolatey.org/) or some other source.

> Until recently the "nokogiri" package did support Ruby 2.7, but this has since been resolved.

Using Chocolatey, install ruby with:

```
choco install ruby -y
```

In an administrator shell.

### MSYS2

Next install MSYS2 (also in the admin shell):

```
choco install msys2 -y
```

Now you can close the admin shell and switch to a regular shell.

### Replace the `eventmachine` package

```
gem uninstall eventmachine
gem install eventmachine --platform ruby
```

> Note: if you update your installation later (`bundle update`), you will need to repeat this step.
{: .warning}

### Gemfile

Your gemfile should look something like this:

```
source 'https://rubygems.org'
gem 'tzinfo-data'
gem 'wdm', '>= 0.1.0' if Gem.win_platform?
gem 'github-pages', group: :jekyll_plugins do
    gem "jekyll-feed"
    gem "jekyll-seo-tag"
end
```

### Install the bundle

In your Github Pages project root, install the bundle with:

```
bundle install
```

Now you should be able to serve your site locally with:

```
bundle exec jekyll serve
```
