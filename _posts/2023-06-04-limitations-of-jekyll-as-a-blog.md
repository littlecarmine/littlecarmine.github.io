---
layout: post
title:  "Limitations of Jekyll as a Blog"
date:   2023-06-04 22:29:00 +0900
categories: blogging 
---

This is a tough one to write. When I first started this little side project, I worshipped Jekyll. It's simple to set up, hosted for free by GitHub, and written in Ruby (which always holds a dear spot in my heart). I still stand by these advantages, especially the free hosting on GitHub after Heroku eliminated free hosting. But, having used it intermittently for half a year, I am realizing that, for blogging, Jekyll maybe isn't so great.

And all this doubt stems from one critical issue: it's pretty damn unappealing to actually _write_ on Jekyll. Due to Jekyll's static nature, all new posts have to be written in Markdown and added to the site as commits. 
Markdown is great until you want to write 800-word, text heavy posts every week. Committing your posts sounds like a simple, no-frills way of posting, but it becomes a huge hassle after a while. Fixing something as simple as a typo becomes a matter of making an entirely new commit and waiting for the changes to go thruogh GitHub Actions.

I really don't blame this shortfall on Jekyll. Jekyll is a **static** site generator. It works wonders for hosting documentation, but it doesn't try too hard to be a tool for making personal blogs; it will simply allow you to do it without necessarily encouraging it. Ultimately, the problem boils down to Jekyll not being a tool optimized for personal blogging which it never claims to be.

All this realization comes after my little honeymoon phase with Ruby and all things Ruby. If I'm being honest, I have been a bit of a blind fanboy for the last year for Ruby. It was a fun language, and it worked pretty well for any small toy projects I had. But in the past few months, I've been realizing that it's not the best option for every scenario. In fact, it's the best option in only a few select scenarios, as is the case with all programming languages.

Do I regret choosing Jekyll as my platform? Not really. I think Jekyll has done its job handsomely as my personal website. But, I do think I might be able to find something better for the blog side of the website. But until then, I'll relish this quirky site builder for what it is, flaws and all.
