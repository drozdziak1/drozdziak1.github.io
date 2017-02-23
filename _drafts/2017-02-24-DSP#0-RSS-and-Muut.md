---
layout: post
title: "DSP-17 #0: Better RSS + Muut comments = an easy recipe for improving
your Jekyll blog"
date:   2017-02-24 8:00:00 +0100
categories: dsp17 meta
tags: dsp17 tutorial jekyll rss muut comments feed disqus alternative
---

When I first heard about Jekyll,  it  was  an  epiphany.   The  main  highlights
include:
* Free hosting on [GitHub][ghpages]
* Battle-tested, breeze-to-use Markdown syntax
* An approachable template system
* Efortless security through static documents

# The catch
Although the defaults work decently, the stock Minima theme leaves something  to
be desired, like categorized RSS feeds and - in my case - an embeddable comment
section that would just **work**. In this first post we'll take a closer look at
the two topics.

# Prerequisites
* A Jekyll project (preferably without drastic customizations)
* A Muut account with a new community to hook up with Jekyll

# RSS

# The comment madness
The default Jekyll theme supports Disqus comments by specifying your blog's
shortname in `_config.yml`. Unfortunately for me, the setup wasn't going that
easy as the comment section wouldn't load due to a [Content Security
Policy][csp] violation, which (after considerable time spent debugging) forced me
to switch to something else.

# Muut to the rescue!
This is where Muut - a drop-in replacement for Disqus - comes in. While for me
it was necessary to switch, others might find it's simplistic style prettier
than what Disqus has to offer.

Assuming that you've already got your Muut community registered, we'll start by
looking at the `_layouts/post.html` layout.

[ghpages]: https://pages.github.com/
[csp]: https://content-security-policy.com/
[muut]: https://muut.com/
