---
date: 2017-04-18
title: Finally switching to vim
categories:
  - Vim
author_staff_member: 01_peter
medium_link:
practical_dev_link:
---

First, a little history of my editor usage. 10 years ago I started with notepad and fastly switched to Notepad++. Then there came a time of Eclipse but I never loved working with it. A few years ago I got to know PhpStorm and RubyMine and really liked working with these awesome IDEs of JetBrains.

***

Two of my colleagues are already using vim and are super happy with it. So I decided I should give it a try, too. I knew it would be hard and that I had to plan with not being productive for at least the first two weeks.

Before using vim I thought you can do every awesome thing with vim out of the box. Now I know better 🙃 Don't get me wrong here, vim is awesome without using any plugins already. But if you are used to a fully featured IDE you expect a lot of the features that made you productive. One example is fuzzy file opening. Vim already is quite good at this out of the box, but with something like [fzf](https://github.com/junegunn/fzf) I like it a little bit more.

Before I started using vim, I thought about writing a blog article about switching to vim. I imagined to give some helpful resources for others to do the same. So when someone else in my company or any other friends wants to switch, I just give this person the link to this article and the person can start immediately.

But then I realized the switch to vim is more like a journey. And a journey needs to be done by everyone who wants to switch.

**The most important thing about vim is that you can't just start with a complete-guide-to-vim and then you're good to go. In the beginning it is a steady process of learning new things (and be overwhelmed by them). The secret is to give yourself some time to get up to speed and understand the brillance behind the concepts.**

## Here is how my journey started

To get the first impression [I started with an interactive online tutorial](http://www.openvim.com/). This tutorial provides a very good introduction to the principles of vim.

After that tutorial I realized that I need a list of all these movement commands. [Here is one to start with](http://vim.wikia.com/wiki/All_the_right_moves). This wiki also has [a getting started page](http://vim.wikia.com/wiki/Category:Getting_started).

In order to train the basic movements you can start playing [VIM Adventures](https://vim-adventures.com/).

The next thing was to install the first plugins. There are multiple ways of achieving this task. I started with [pathogen.vim](https://github.com/tpope/vim-pathogen) and it was ok. But my coworkers showed me the advantages of [vim-plug](https://github.com/junegunn/vim-plug) and as they are using vim-plug I switched, too.

You also might wanna have a look at [neovim](https://neovim.io/).

The last (and maybe hardest) part of starting with vim is deciding which plugins to use (and not to use 😉) and how to customize your settings and keymaps. As a starting point I will list some plugins someone might have a look at. The most important thing here is: read through the documenation of each plugin and decide whether you need it or not!

* [vim-sensible](https://github.com/tpope/vim-sensible)
* [fzf](https://github.com/junegunn/fzf)
* [vim-surround](https://github.com/tpope/vim-surround)
* [vim-gitgutter](https://github.com/airblade/vim-gitgutter)
* [YouCompleteMe](https://github.com/Valloric/YouCompleteMe)

And every programming language normally has a vim plugin on its own.

***

After one week I'm not as productive as with RubyMine. If I really need to get something done fast I still open RubyMine. But most of the time I use vim now and I enjoy working with it. Two more weeks and RubyMine stays closed. What I miss the most is the awesome multi cursor implementation of RubyMine. But I already have been told that in vim you do things differently 😉
