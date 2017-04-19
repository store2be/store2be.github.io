---
date: 2017-04-18
title: Finally switching to vim
categories:
- Vim
author_staff_member: 01_peter
medium_link:
practical_dev_link:
---

This article is just a selection of things I found helpful when switching to vim. So when someone asks for help because he or she wants to switch to vim, I just pass him or her the link to this article. Some of the links in this article also describe why you should switch to vim. So I won't get into this here.

***

Before I started using vim, I thought switching would be just a timeframe of 2-4 weeks of being less productive and learning vim and then I would have it all set up and I am used to the vim style of doing things. But then I realized switching to vim is more like a long and ongoing journey.

**The most important thing about vim is that you can't just start with a complete-guide-to-vim and then you're good to go. It is a steady process of learning new things (and be overwhelmed by them). The secret in the beginning is to give yourself some time to get up to speed and understand the brilliance behind the concepts of vim.**

***

I recommend to start by [watching this introduction talk to vim by Mike Coutermash](https://www.youtube.com/watch?v=_NUO4JEtkDw). He describes really good what I was experience as well when starting with vim. Here are also his [blog article](https://mikecoutermarsh.com/learning-vim-in-a-week/) and [his slides for the talk](https://mikecoutermarsh.com/boston-vim-learning-vim-in-a-week/).

In order to get a first impression [I started with an interactive online tutorial](http://www.openvim.com/). This tutorial provides a very good interactive introduction to the principles of vim. [And this is a very good read about the why of vim.](http://www.viemu.com/a-why-vi-vim.html)

After that I realized that I need a list of all these movement commands. [Here is a cheat sheet to start with](https://bitbucket.org/tednaleid/vim-shortcut-wallpaper/raw/tip/vim-shortcuts2560x1600.png). Or even better you print out a more complete vim cheat sheet: [This](https://rumorscity.com/wp-content/uploads/2014/08/10-Best-VIM-Cheat-Sheet-02.jpg) or [this](http://michael.peopleofhonoronly.com/vim/vim_cheat_sheet_for_programmers_print.png). The vim wiki has [a getting started page](http://vim.wikia.com/wiki/Category:Getting_started) that I found quite useful.

In order to train the basic movements you can start playing [VIM Adventures](https://vim-adventures.com/).

[This Github repository](https://github.com/mhinz/vim-galore) provides a long guide to vim.

[Here are 68 free screencasts about vim](http://vimcasts.org/episodes/archive/) and [here are two more free ones and other paid ones](https://thoughtbot.com/upcase/onramp-to-vim).

You also might wanna have a look at [neovim](https://neovim.io/).

Vim has a very rich ecosystem of plugins. There are multiple ways of installing plugins. I started with [pathogen.vim](https://github.com/tpope/vim-pathogen) and it was ok. But my coworkers showed me the advantages of [vim-plug](https://github.com/junegunn/vim-plug) and as they are using vim-plug I switched, too.

Then you have to decide which plugins to use (and not to use 😉) and how to customize your settings and keymaps. As a starting point I will list some plugins someone might have a look at. The most important thing here is: read through the documentation of each plugin and decide whether you need it or not!

Here are some plugins that I found useful for the start:

* [vim-sensible](https://github.com/tpope/vim-sensible)
* [fzf](https://github.com/junegunn/fzf)
* [vim-surround](https://github.com/tpope/vim-surround)
* [vim-gitgutter](https://github.com/airblade/vim-gitgutter)
* [nerdtree](https://github.com/scrooloose/nerdtree)

And every programming language normally has a vim plugin on its own.

Finally, [start playing vimgolf](http://www.vimgolf.com/) from the beginning. It's never too early to start (I had that thought). Just look at the other solutions and try to understand what they did.

***

After one week of vim I'm not as productive as with RubyMine. If I really need to get something done fast I still open RubyMine. But most of the time I use vim now and I enjoy working with it. Two more weeks and RubyMine stays closed. What I miss the most is the awesome multi cursor implementation of RubyMine. But I already have been told that in vim you do things differently 😉
