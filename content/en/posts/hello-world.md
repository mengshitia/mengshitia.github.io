+++
title = 'Hello, World!'
date = '2026-02-12T15:28:44+08:00'
description = 'A brand new start.'
draft = false
isCjkLanguage = false
categories = ['articles']
tags = ['blog', 'hello world']
[build]
  list = 'always'  # Use 'never' to exclude this page from all page collections.
[params]
  comments = true  # Set 'true' to enable.
+++

## A wish

I like browsing all kinds of personal websites and blogs.
No matter what these websites are.
They can be used to show off skills, share one's daily life or simply just be a online notebook.
For me, they are just the bridges that connect me -- the reader, and the creators.
So I could walk into the eden of the creators and enjoy.

After seeing these wonderful websites and blogs, I wish I could have one too.


## The opportunity

Just like the other experience when I switched to a Linux distribution,
I found Hugo in the corner of the Internet.

Well, if I hadn't had *that wish* first, I might have completely dismissed it with just a quick glance.

## Try out

Following the instructions in the documentation, a brand new website was up and running.
The black-box technique was amazing.

But then I found a problem immediately, which can be very crucial to me:
The Ananke theme doesn't have a dark theme.

As a heavy browser user, how come it came without a dark theme?
"Fine, forget about it", I said.
Then I went for some other Hugo themes that may be suitable for me.

After searching and trying out, I chose these themes:
[Paper](https://github.com/nanxiaobei/hugo-paper), [Paper-Mod](https://github.com/adityatelange/hugo-PaperMod) and [Stack](https://github.com/CaiJimmy/hugo-theme-stack).
All these themes have a switch for toggling dark/light theme, soft color scheme and great design of the UI plus a fluent UX.

However, when I was using and tweaking these themes,
I realized they have so many features that not all of them are useful to me.

Moreover, I was not satisfied with the dark/light theme toggle feature given by them.
Because Paper and Paper-Mod are lacking the `'auto-mode'` feature which follows the system settings or just provide the feature, but users can't decide when to use it, like the Stack theme.


## Tweak? Write a new one!

"What if I write a theme which gives me just enough features including the dark/light theme switching feature myself?"

Well, this thought came out not so long after me trying out these themes,
and it seems so crazy at that time, in hindsight.

With that thought in my mind, I dived into the Hugo Documentation and started digging in.
Unfortunately, it barely has descriptions about the Hugo theme,
the only thing I found was: "A theme is a Hugo module, just like a Hugo site, it's a module too."
Therefore, it is no exaggeration to say that I spent a month.
The process in between was one of trying to understand, using it, failing and starting over...
This part would be boring for readers, so I'll skip it.

Side note: [Archie](https://github.com/athul/archie), a simple theme, even using the old version of the template system, helped me grasp how Hugo template system works.


## Finally

I finished writing my Hugo theme [Mage](https://github.com/mengshitia/hugo-theme-mage) at last.
It has all the features I need: displaying my avatar, listing articles on the home page, having the 'Archive' section, the 'About' section, the taxonomies, the tags and the most important, dark/light theme switching feature.
It also comes with soft color scheme and the card-based layout design those themes have.

With the right theme, I settled down and finished this article.

So, first came a wish, then I forgot the middle part, but in the end I still managed to pull it off!


## Afterword

Though a little bit odd, I want to thank all my friends for your help and support,
even the support that is not relevant to this.
