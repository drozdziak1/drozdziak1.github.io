---
layout: post
title: "DSP-17 #0: Better RSS + Muut comments = an easy recipe for improving
your Jekyll blog"
author: Stan Drozd
date:   2017-02-24 8:00:00 +0100
categories: dsp17 meta
tags: dsp17 tutorial jekyll rss muut comments feed disqus alternative
published: true
---

When I first heard about Jekyll,  it  was  an  epiphany.   The  main  highlights
include:
* Free hosting on [GitHub][ghpages]
* Battle-tested, breeze-to-use Markdown syntax
* An approachable template system
* Efortless security through static documents

# The catch
Although the defaults work decently, the stock Minima theme leaves something  to
be desired, like categorized RSS feeds (or category support in general) and, in
my case, an embeddable comment section that would just **work**. In this first
tutorial post we'll take a closer look at the two topics.

# Prerequisites
* A Jekyll project (preferably without drastic customizations)
* A [Muut][muut] account with a new community to hook up with your project
* Rudimentary grasp on [Liquid][liquid] templates

## Category killer RSS
As I mentioned in my [first post]({%post_url 2017-02-15-bin-init%}), the idea
for starting this blog sparked from the [Daj się poznać/Get Noticed 2017][dsp]
competition (which BTW starts on March 1). One of its notable requirements is to
provide an RSS feed containing blog posts meant specifically for the challenge.

If you're like me, you'd probably like a barely minimal and easy to use solution
that would meet the requirement just right. With the help of [this
repo][jekyll-rss-feeds], I have come up with a no-brainer feed template:
{% raw %}
```xml
---
category_name: dsp17
---
<?xml version="1.0" encoding="UTF-8"?>
<rss version="2.0" xmlns:atom="http://www.w3.org/2005/Atom">
	<channel>
		<title>{{ site.title | xml_escape }}</title>
		<description>Posts categorized as {{ page.category_name }}</description>
		<link>{{ site.url }}</link>
		<atom:link href="{{ site.url }}/{{ page.path }}" rel="self" type="application/rss+xml" />
		{% for post in site.categories[page.category_name] %}
		<item>
			<title>{{ post.title | xml_escape }}</title>
			{% if post.excerpt %}
			<description>{{ post.excerpt | xml_escape }}</description>
			{% else %}
			<description>{{ post.content | xml_escape }}</description>
			{% endif %}
			<pubDate>{{ post.date | date_to_rfc822 }}</pubDate>
			<link>{{ site.url }}{{ post.url }}</link>
			<guid isPermaLink="true">{{ site.url }}{{ post.url }}</guid>
		</item>
		{% endfor %}
	</channel>
</rss>
```
{% endraw %}

All you have to do is just copy the snippet above into an xml file in your
project's directory, e.g. `feed.<category>.xml`. Then, change the
`category_name` variable to whatever category you want and you're **done**!
:boom: :confetti_ball:

If your post titles aren't browseable in production (e.g. when using Firefox to
access the xml file on the target GitHub Pages website), please **make sure that
you specified the `url` variable in your `_config.yml` file**.

This template could easily be transformed into a category listing layout - e.g.
by reusing the `_layouts/home.html` file from Minima. Here's a sample
`category.html` layout I have created:
{% raw %}
```html
---
layout: default
---

<div class="category">

  <h1 class="page-heading">Posts from the {{page.category_name}} category</h1>

  {{ content }}

  <ul class="post-list">
    {% for post in site.categories[page.category_name] %}
      <li>
        <span class="post-meta">{{ post.date | date: "%b %-d, %Y" }}</span>

        <h2>
          <a class="post-link" href="{{ post.url | relative_url }}">{{ post.title | escape }}</a>
        </h2>
      </li>
    {% endfor %}
  </ul>

  {% capture feed_file %}/feed.{{ page.category_name }}.xml{%endcapture%}
  <p class="rss-subscribe">subscribe <a href="{{ feed_file | relative_url }}">via RSS</a></p>

</div>
```
{%endraw%}

To use it, you just need to create a new markdown file in the root of your
blog's directory, specify the layout as `category` and set the `category_name`
variable. I went with `dsp17.md` for the competition:
{% raw %}
```
---
layout: category
category_name: dsp17
title: DSP'17
---
```
{% endraw %}

Keep in mind that `category.html` shows a link to a `feed.<category_name>.xml`
file, which will result in a `404` error unless you create the file manually (e.g.
using my XML template from above).

## The comment madness
The default Jekyll theme supports Disqus comments by specifying your blog's
shortname in `_config.yml`. Unfortunately for me, the setup wasn't going that
smoothly as the comment section wouldn't load due to a [Content Security
Policy][csp] violation, which (after considerable time spent debugging) forced
me to jump ship and try something else.

# Muut to the rescue!
This is where Muut - a drop-in replacement for Disqus - comes in. While for me
it was necessary to switch, others might also find its clean style prettier than
what vanilla Disqus has to offer.

Let's get cracking. Here's what the original Muut embed code looks like at the
time of writing this post:

{% raw %}
```html
   <a class="muut" href="https://muut.com/i/community-name">Your Community Title</a>
   <script src="//cdn.muut.com/1/moot.min.js"></script>
```
{% endraw %}

In order to get Muut up and running you only need to change a couple lines in
just two files. First, let's have a look at the `_layouts/post.html` layout
freshly copied from Minima into our project:

{% raw %}
```html
---
layout: default
---
<article class="post" itemscope itemtype="http://schema.org/BlogPosting">

  <header class="post-header">
    <h1 class="post-title" itemprop="name headline">{{ page.title | escape }}</h1>
    <p class="post-meta"><time datetime="{{ page.date | date_to_xmlschema }}" itemprop="datePublished">{{ page.date | date: "%b %-d, %Y" }}</time>{% if page.author %} • <span itemprop="author" itemscope itemtype="http://schema.org/Person"><span itemprop="name">{{ page.author }}</span></span>{% endif %}</p>
  </header>

  <div class="post-content" itemprop="articleBody">
    {{ content }}
  </div>

  {% if site.disqus.shortname %}
    {% include disqus_comments.html %}
  {% endif %}
</article>
```
{% endraw %}

As you can see, Jekyll creates the stock comment section by checking whether
your page's Disqus shortname is set and then including a partial.

Let's change the `site.disqus.shortname` variable to `site.muut.name` or
whatever sounds more relevant to you. In my case, I'm also opting out of putting
the Muut's embed code in a separate include file. I don't mind the comment
section showing outside production and only add the check for whether comments
are on.

After pasting in the embed code, we fill in the `community-name` placeholder and
append the post file name for Muut to automatically start new comment threads.
My final result for the post layout is something along these lines:

{% raw %}
```html
---
layout: default
---
<article class="post" itemscope itemtype="http://schema.org/BlogPosting">

  <header class="post-header">
    <h1 class="post-title" itemprop="name headline">{{ page.title | escape }}</h1>
    <p class="post-meta"><time datetime="{{ page.date | date_to_xmlschema }}" itemprop="datePublished">{{ page.date | date: "%b %-d, %Y" }}</time>{% if page.author %} • <span itemprop="author" itemscope itemtype="http://schema.org/Person"><span itemprop="name">{{ page.author }}</span></span>{% endif %}</p>
  </header>

  <div class="post-content" itemprop="articleBody">
    {{ content }}
  </div>

  {% if site.muut.name and page.comments != false %}
    <a class="muut " href="https://muut.com/i/{{ site.muut.name }}/{{ page.path }}">{{ site.muut.name }} forum</a>
    <script src="//cdn.muut.com/1/moot.min.js"></script>
  {% endif %}
</article>
```
{% endraw %}

The last thing we need to do is to just fill in `_config.yml` with the setting
we chose earlier:
```yaml
muut:
  name: drozdziak1-github-io
```

# Conclusion
Congrats! :champagne: You just learned how to start a blog with enough
functionality to take part in ["Get Noticed!"][dsp].

Even though the topic isn't too sophisticated, I've really had fun taking a stab
at writing my first tutorial - seems like a nice warmup before the competition.
I hope someone finds it helpful, but should you have any questions, feel free to
drop a line below.

[csp]: https://content-security-policy.com/
[dsp]:http://dajsiepoznac.pl/
[ghpages]: https://pages.github.com/
[jekyll-rss-feeds]: https://github.com/snaptortoise/jekyll-rss-feeds
[liquid]: http://shopify.github.io/liquid/
[muut]: https://muut.com/
