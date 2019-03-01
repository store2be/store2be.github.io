---
title: Never Forgit
description: A collection of our blog posts about git, git aliases and everything git.
---

# Rebasing

For the following aliases we're going to define shell functions inside of an alias and call them directly afterwards.

## rbb

`rbb = "!f() { git rebase -i HEAD~$1; }; f"`

Interactive rebase by the amount of commits passed through the first argument. Using `git rbb 3` in a project with at least three commits will leave you in your editor of choice with something like this:

![interactive rebase alias]({{ site.url }}/images/git_aliases/rbb.jpg)

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
