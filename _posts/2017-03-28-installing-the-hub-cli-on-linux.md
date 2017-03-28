---
date: 2017-03-28
title: Installing the hub CLI on Linux
categories:
  - CLI
  - Linux
  - Hub
author_staff_member: 03_tom
---

[Hub](https://github.com/github/hub) is a command line tool to interact with the Github-specific features of a repository,
one of them being pull requests. We use it extensively at store2be and it is the natural way to create PRs for us.
It has a Homebrew package for OS X users, so it’s completely straightforward there, but it is not packaged even for
the most popular Linux distributions.

Here’s how to install it on your distribution of choice:

1. Download the latest release on the [release page](https://github.com/github/hub/releases) (“hub ${version} for Linux 64-bit”)
2. Go to the directory where it was downloaded and decompress it: `tar xzf [name of the file]`
3. Enter the decompressed directory with the same name as the archive you downloaded (with `cd`)
4. Run the `install` script inside this directory: `sudo ./install`
5. Voilà!

You can now create pull requests by working on your branch, pushing it and running `hub pull-request`.

Tip: `hub browse` is a convenient way to open a browser tab for the current repository on Github.
