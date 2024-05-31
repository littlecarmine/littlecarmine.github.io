---
layout: post
title:  "Hosting a blog on github Pages with Jekyll"
date:   2022-12-25 15:25:00 +0900
categories: blogging
---
## Returning to forthoney.github.io
After having left my github page unattended for a year or so, I've finally come back. But returning to the project, I found myself with two big questions about the project's future. How would I finish building it, and what would I build in it?

I probably should have asked myself the latter question first. But at the time (i.e. two days ago), the sheer will to partake in the act of building overshadowed deciding what to build. Naturally, I headed over to [github's own guide page](https://pages.github.com/) and looked for how to build stuff here, and there I found something called *Jekyll*. It had a pretty cool name so I just decided I'll try building something with Jekyll.

## Not so cool setup
Despite being officially hosted by github, actually setting Jekyll was pretty rocky. Github does provide documentation on how to set it up, but it's very specific to hosting documentation of a project. For example, [this page](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/creating-a-github-pages-site-with-jekyll) doesn't quite explain why they create an orphan branch rather than just a regular branch. Having built the project, I can surmise that they create an orphan branch because they are assuming the page will hold documentation on a project. The project would be stored on the main branch while the documentation for the project is held on an orphan branch. Then, they would host the pages from the orphan branch.

The same applies to configuring the publishing source. They simply state that you can specify the branch and directory from which you'll publish the page from, but don't mention why you would want to change the source. This again likely comes from them anticipating this feature be used for documentation hosting. If you want to keep the documentation on the same repo as the project, you can create a docs directory, move the documentation to docs/ and set the publishing source to docs/. This way, you can find documentation about the project without changing repos, but also host it in a nice fancy way.

I ran into an even bigger problem when setting up the `_config.yml`, one that prevented my site from being properly hosted at all. The description they gave for setting the `baseurl` and `url` didn't mention what to do if your publishing source was not the main branch. After wrestling with the issue for an hour, I gave up on using the orphaned branch as my publishing source.

Later I found that the instructions on the [local testing guide](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll) didn't work when using a remote theme. While the issue was quickly resolved with adding the line `gem "remote-jekyll-theme"` like so,

{% highlight ruby %}
group :jekyll_plugins do
  gem "jekyll-feed", "~> 0.12"
  gem "jekyll-remote-theme"
end
{% endhighlight %}

it would have been preferable to find out from github's guide rather than stack overflow.

## Very cool development
Thankfully, everything after the setup was fabulous. On the backend, Jekyll's file system one of the most intuitive approaches I've seen. Posts are in the `_posts` directory and pages are in the root directory. Actually writing stuff is very easy since you can write in markdown and yaml. On the frontend, there's a vast array of free themes to choose from available online and without downloading (if using `jekyll-remote-theme`). One minor complaint I had was how divergent the themes were, meaning depending on what theme I used, I may need to completely overhaul `_config.yml`. But besides this, it was *so* pleasant. It's difficult to express how easy it really is, but I hope how much shorter this section is compared to the setup section of the post goes on to show how little I have to criticize about Jekyll itself.

## The final question
Having built something with Jekyll, I now face the dreaded question of what I want this Jekyll site to be. The primary purpose of Jekyll is for documentation and blogging. I don't really have anything that needs this caliber of a documentation. So, I'm leaning towards just making this my personal blog where I write about whatever I want: some technical, most not (probably).

Some things on my mind are (in no particular order)
- [ ] Thoughts on framworks
- [ ] Thoughts on FOSS
- [ ] Monopolistic rule of game publishers on e-sports
- [ ] *Amadeus* by Peter Schaffer
- [ ] Balancing in e-sports titles

I hope that as I write more, the blog will naturally develop a direction/identity and until then, I'll just freely write stuff I am thinking about.
