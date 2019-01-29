+++
title = "Blogging: From Jekyll to Zola"
date=2019-01-29
draft=true
[extra]
tags="blog, ssg, static-site, jekyll, zola, rust, SSG, static, site, blogging, hackers"
+++

When I first created this blog I used Jekyll for my blogging needs because that was the first
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

## Zola

I recently started learning [Rust] and when I started looking around for a faster
alternative than Jekyll for my blog, I found [Zola] a static site generator written
in Rust that compiles to a binary with performance magnitutes of order better
than Jekyll ( as of now, slighty slower than Hugo) with no plugins and everything
in built, it was perfect. 


