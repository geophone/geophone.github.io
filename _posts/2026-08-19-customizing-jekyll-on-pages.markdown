---
layout: post
title: "Jekyll customization"
date: 2026-08-19
---
I found that it's rather involved to customize jekyll on github to run a proper css /scss customization.

First you need minima 3.0 to override hooks which you need to specify in your Gemfile so that it pulls in the right version

Then you need to override the default jekyll builds on github which use an old minima verison which doesn't have the scss override hook.

The repository this is based on [this github](https://github.com/geophone/geophone.github.io) has examples of how to fix all this stuff up one of the key things I found is that the scss file custom-styles.scss in _sass/minima/... is very specific as a hook
from the repository root
```
_sass/minima/custom-styles.scss
has content like
@include url('cssfile.css'); 
a bunch of css overrides
```

This works although the include wasn't working for some reason to allow custom stylization

```
with:
          ruby-version: '3.3' # Not needed with a .ruby-version file
          bundler-cache: true # runs 'bundle install' and caches installed gems automatically
          cache-version: 2 # Increment this number if you need to re-download cached gems
        env:
          BUNDLE_GEMFILE: ${{ github.workspace }}/Gemfile
```
key changes I made to the default jekyll version 
Also in my Gemfile

```
gem "jekyll", "~> 4.4.1"
# This is the default theme for new Jekyll sites. You may change this to anything you like.
gem "minima", github: "jekyll/minima"
```
pinning both jekyll to the latest and pulling minima from the github to get version 3.0.0 (with the scss hooks)

All this means that the pages site looks slightly more fashionable
	
