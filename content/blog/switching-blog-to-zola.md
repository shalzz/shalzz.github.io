+++
title = "Blogging: From Jekyll to Zola"
date=2019-02-01
draft=true
[extra]
tags="blog, ssg, static-site, jekyll, zola, rust, SSG, static, site, blogging, hackers"
+++

When I first created this blog I used [Jekyll] for my blogging needs mostly
because that was the first
time I had come across the concept of a Static Site Generator (SSG) from markdown posts
and Jekyll was and still is the most popular SSG to date. Also because of GitHubs
free hosting for Jekyll sites.

Jekyll gained its popularity by being a simple yet powerful framework.
It combined the Liquid templating language with a markdown processor to create 
an engine that spits out a completely static website with no database or any dependencies.

<!-- more -->

Once you have or create a theme you can focus on what matters the most,
writing the content, without worring about the formating or the layout or the
specifies of a markup language like HTML while you are in the thought process
of writing a blog.

With Markdown you can still control how the text looks and add addition images
and links which makes it a lot better to compose and write than HTML.
A very pleaseant offline writing experience and ease of development 
is what makes Jekyll and SSGs as a whole great.

## Performance

Jekyll has a lot of great themes made by available by generatous people which
make it very easy to pick one, customise it and start blogging. It does have
a short learning curve though and once you start writing a lot more posts you have to
consider the performance of how long it's taking to actually generate the site
from your posts.

Building 15 pages without LSI and a atom feed generation and jekyll-assets
plugin for image processing takes `56.202 seconds`. That's a long time!

```
$ bundle exec jekyll build
Configuration file: /home/shalzz/dev/shalzz.github.io/_config.yml
            Source: /home/shalzz/dev/shalzz.github.io
       Destination: /home/shalzz/dev/shalzz.github.io/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
       Jekyll Feed: Generating feed for posts
                    done in 56.202 seconds.
 Auto-regeneration: disabled. Use --watch to enable.
```

On subsequent builds with incremental build enabled, it takes `2.89 seconds`.
Much better but still very slow.

```
$ bundle exec jekyll build --incremental
Configuration file: /home/shalzz/dev/shalzz.github.io/_config.yml
            Source: /home/shalzz/dev/shalzz.github.io
       Destination: /home/shalzz/dev/shalzz.github.io/_site
 Incremental build: enabled
      Generating...
       Jekyll Feed: Generating feed for posts
                    done in 2.896 seconds.
 Auto-regeneration: disabled. Use --watch to enable.
```

## Zola

I recently started learning [Rust] and when I started looking around for a faster
alternative than Jekyll for my blog, I found [Zola] a static site generator written
in Rust that compiles to a binary with performance magnitutes of order better
than Jekyll ( slighty slower than Hugo as of now ) with no plugins and everything
in built, it was perfect. 

Compared to Jekyll's Liquid template engine Zola used Tera
as its template engine which is also written in rust.
It has the same syntax plus the concept of `block`s that
can be inheireted and overriden when you
extend a template in another template. If you are also looking at Hugo, Tera is a huge
step up from it's Go based template language syntax.

After making a some modifications to my sites layouts/templates due to the subtle
difference between the two frameworks and their structuring I had now effectively
extracted my sites templates and css to its own [theme][1] that can be reused by anyone else.

### Performance

Building 16 pages (including this one) with 8 images and no cache with Zola
takes only `0.13 seconds`.
Zola turns out to be an order of magnitude faster than Jekyll. That's a huge improvement.

```
$ zola build
Building site...
-> Creating 16 pages (0 orphan), 2 sections, and processing 8 images
Done in 130ms.
```

Combining that fast engine with a [GitHub Action][2] that reuses a cached docker image
in the background, from a git push to being published on GitHub Pages takes on
average only ~20 seconds compared to Jekyll + Travis total time of ~2 minutes.

In the end I'm very happy with Zola and it's architure and there's more improvements
to come with Zola 0.6 and above.

### Further Reading

 * [Zola Docs](https://www.getzola.org/documentation/getting-started/installation/)
 * [Tera Docs](https://tera.netlify.com/docs/templates/#templates)

### More Links

 * [Zola theme - Butler][1]
 * [Zola Deploy GitHub Action](http://github.com/shalzz/zola-deploy-action)

[Jekyll]: https://jekyllrb.com
[Rust]: https://rust-lang.org
[Zola]: https://getzola.org
[1]: https://github.com/shalzz/butler
[2]: https://github.com/features/actions
