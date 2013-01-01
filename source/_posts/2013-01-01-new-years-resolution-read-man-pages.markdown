---
layout: post
title: "New Year's Resolution: Read man Pages"
date: 2013-01-01 16:00
comments: true
categories: 
---

I've become addicted to [tmux](http://tmux.sourceforge.net/), the terminal multiplexer.  It lets me turn one terminal window into a multi-tabbed, multi-paned hub for all my terminal-based exploits.  

I've been mulling for a while if there is a way to emulate [Cluster SSH](http://sourceforge.net/projects/clusterssh/) using tmux, being able to type a command once and have it be executed in multiple sessions on different servers.  I spent a few hours today Googling for ideas on how to do such a thing, stumbling across [a gist that seemed to do something close to what I want](https://gist.github.com/2773454), then sinking some more effort into adapting it to my needs.  The result worked, but was a bit messy.

While trying to clean up my solution, I did some searching in the tmux man page.  In order to understand what I was reading, I had to page back to review the section header and stumbled upon the `synchronize-panes` window option.  It was exactly the functionality I was looking for, already baked into tmux.

So my New Year's resolution is to spend less time Googling and more time reading the official documentation.  Instead of waiting until I need some specific feature, I'm going to set aside a little time every day to get a broader view on the tools I rely on to be productive.
