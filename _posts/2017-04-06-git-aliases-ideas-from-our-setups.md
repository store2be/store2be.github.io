---
date: 2017-04-07
title: "Git rich or die tryin' 01: Ideas from our Git setups"
categories:
  - CLI
  - Git
  - Alias
author_staff_member: 02_fausto
---

This post is part of a series about our Git setups. In this series we will focus on the Git aliases we use the most, particularly the ones we find interesting (i.e., not `c = commit`) and that have increased our productivity using Git.

A complete compilation can be found [here]({{ site.url }}/never-forgit).

# Rebasing

For the following aliases we're going to define shell functions inside of an alias and call them directly afterwards.

## rbb

`rbb = "!f() { git rebase -i HEAD~$1; }; f"`

Interactive rebase by the amount of commits passed through the first argument. Using `git rbb 3` in a project with at least three commits will leave you in your editor of choice with something like this:

![interactive rebase alias]({{ site.url }}/images/git_aliases/rbb.jpg)

## cfix

We can also use backticks surrounding a command to evaluate with the shell and pass the results as a part of our git alias. We use this with `rev-parse` in the following alias:

``cfix = "!f() { git commit --fixup `git rev-parse --short HEAD` && git rebase -i HEAD~2 -i --autosquash; }; f"``

We use `cfix` when the following situation arises. Say you've committed something with the message `Fix the terrible bug` only to find out that the fix did not actually work the way you expected. So you make your changes and go:

`git add .`

Now you commit the new, real fix:

`git commit -m "Fix the terrible bug... again"`

Wouldn't it be nice to just have the new changes added to the last commit? Well, you can do that with a `rebase`:

`git rebase -i HEAD~2`

This will leave you with your editor looking like the first image in this section. Then you can change the `pick {COMMIT HASH} Fix the terrible bug... again` line to `fixup {COMMIT HASH} Fix the terrible bug... again` and save the file, with the desired results.

You can do exactly the same but avoiding the numerous commands by using `cfix`. Here's an example screen after adding changes and running `git cfix`:

![use latest add as a fixup for last commit]({{ site.url }}/images/git_aliases/cfix.jpg)

A similar result can be achieved using `git commit --amend`, but with `cfix` you can also `squash`, or even use the commit, and/or `reword` the previous one. Some people, however, don't recommend rewriting Git history. You might want to read up on the topic starting with posts like the ancient [Thou Shalt Not Lie: git rebase, amend, squash, and other lies](http://paul.stadig.name/2010/12/thou-shalt-not-lie-git-rebase-ammend.html).

# Logging

There are many different options when it comes to customizing `git log`. Some of the most common options are passed using flags like `--graph` or `--decorate`, but the main differentiator is `--pretty`. Here are some of ours, with screenshots to showcase them:

![pretty git log alias]({{ site.url }}/images/git_aliases/gitlog2.jpg)
`--pretty=format:'%h %ad | %s%d [%an]'`

![pretty git log alias]({{ site.url }}/images/git_aliases/gitlog1.jpg)
`--pretty=format:'%C(bold blue)%h%C(reset) - %C(bold green)(%ar)%C(reset) %C(white)%s%C(reset) %C(dim white)- %an%C(reset)%C(auto)%d%C(reset)'`

Extensive documentation on how to build your own `--pretty` line can be found [here](https://git-scm.com/docs/pretty-formats). An example of a finished `git log` alias could be:

`lg = git log --graph --abbrev-commit --decorate --pretty=format:'%h %ad | %s%d [%an]'`

# Pull requests

If you've installed the `hub` CLI ([and here's how to do that]({% post_url 2017-03-28-installing-the-hub-cli-on-linux %})), you can add the following alias to your toolbelt:

`pr = !hub pull-request`

In case you're wondering, that `!` at the start of the alias tells git that the command has to be run by the shell.
