---
title: "Embracing Static Site Generation: WordPress to Hugo Migration"
date: 2023-01-23T00:33:31-05:00
author: Adrien Poupa
url: /embracing-static-site-generation-wordpress-to-hugo-migration/
tags:
    - hugo
    - wordpress
---

This is it! I finally migrated my blog to [Hugo](https://gohugo.io/)! 

## Static Site Generation?

After close to a decade of running WordPress and months of procrastination, 
I finally took the decision to join the cool kids  and drink the static website kool aid.
There are numerous benefits with this approach:
- I no longer need to rent a dedicated server. So long [Kimsufi](https://www.kimsufi.com/), it's been great!
- Attack surface is much smaller, if not nonexistent
- I can use Markdown to write posts and stop [fighting Gutenberg](https://www.reddit.com/r/Wordpress/comments/ox31tv/why_does_gutenberg_suck_so_fucking_hard/)
- No need to worry about backups, WordPress and PHP upgrades
- Hosting is free
- GTmetrix performance is 100%

## Selecting a Static Site Generator

There is no shortage of static site generators these days.

After reviewing the [different options here](https://jamstack.org/generators/), I reduced my selection to Jekyll, Hugo and Zola.

They are all great, really. Jekyll was the precursor of this whole static website trend, it still works great,
and I am using it for my [resume](https://resume.adrien.poupa.net/). However, it feels a bit dated and can be slow to 
compile bigger sites, not that it matters for mine.

Zola is a no-frill generator that consists of a single binary, built in Rust. How cool is that? However, development seems
a bit slow, and more importantly the [theme selection](https://www.getzola.org/themes/) sounds a bit lackluster for me.
I did consider [Anpu](https://github.com/zbrox/anpu-zola-theme) for a bit, but the lack of dark mode was a deal-breaker.

Hugo, written in Go, is extremely popular and well maintained. There is a [good amount](https://themes.gohugo.io/) of themes available.
It is also very fast to compile, compared to other SSGs. I settled on the [PaperMod theme](https://github.com/adityatelange/hugo-PaperMod) 
as it felt very clean, is [packed with features](https://github.com/adityatelange/hugo-PaperMod/wiki/Features), 
is popular and actively maintained.

## Migration from WordPress

### Migrating Posts

The migration itself was almost painless, at least much more than I had envisioned.

While there is a [WordPress to Hugo migration plugin](https://github.com/SchumacherFM/wordpress-to-hugo-exporter), I used 
the [Jekyll flavor](https://github.com/benbalter/wordpress-to-jekyll-exporter) of it as it seemed it was better at filtering
out HTML from the posts. 

I had to adjust some header attributes to convert them from Jekyll to Hugo, but at the end of the day, Markdown is Markdown.
After stripping out some HTML tags the plugins did not remove, 
I had all my posts conveniently converted to Markdown in less than an hour.

### Migrating Comments

Now, this is a bit trickier. See, static sites are, well... static, typically removing any user interaction such as comments
or contact forms. I had considered removing all comments, after all I was not impressed with all the spam that went through Askimet.

However, Hugo is [really good](https://gohugo.io/content-management/comments/) at handling comments, shipping with Disqus.
Because this is a dev blog, I thought using GitHub as a spam filter would be pretty convenient and stumbled upon [utterances](https://utteranc.es/).
After all, using issues on the repository I use for this website's code seemed like the perfect hack.
I later found [giscus](https://giscus.app/), which is using discussions rather than issues, and I was sold.

Now, I had to migrate the comments to Giscus. While there is no official way to do so, I was in luck given Rémi Peyronnet
did [all the work](https://www.lprp.fr/2022/11/Migrate-wordpress-comments-to-giscus/) for me here. Thank you, Rémi!
Even though PaperMod does not ship with Giscus support, 
adding it was [simple enough](https://cdwilson.dev/articles/using-giscus-for-comments-in-hugo/) with Hugo's templating system.

## Selecting a Hosting Provider

There are many free static hosting providers, such as Netlify, GitHub Pages or CloudFlare Pages.

I initially started to use GitHub Pages, pretty much by default as I am hosting the code on GitHub, and it was working
fine, but I needed to deal with GitHub actions, creating a second `gh-pages` branch for the static code 
(or enable some beta option to get rid of it), then dealing with the workflow's permission,
and then figure out whether I needed to setup a `CNAME` file after updating my DNS record. It felt a bit messy overall.

Enter CloudFlare Pages. I did not even consider it before seeing somebody mention it on HackerNews. Given I already use
too many CloudFlare products, I figured why not try another one. The setup process was very smooth and code-free, all
I had to do was authorize the CloudFlare integration into my repository, specify the generator,
and it was already building it. Then I added the DNS record in 2 clicks, with the seamless CloudFlare DNS integration.
I was impressed by the ease of the process.

The free tier is also extremely [generous](https://developers.cloudflare.com/pages/platform/limits): 500 builds per month, 
preview deployments and no traffic limit.

I am really happy with this new setup, and wonder why I have not done this years ago. One less excuse for not blogging, I guess.
Let me know if you are interested in a similar migration, or if you have been through one :)