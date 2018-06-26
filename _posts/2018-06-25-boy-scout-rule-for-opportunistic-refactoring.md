---
date: 2018-06-14
categories:
- Work Ethics
- Refactoring
- Boy Scout rule
title: Boy scout rule for opportunistic refactoring
author_staff_member: 06_sooraj
image: /images/boyscout.jpg
---

![Boytscout- image courtesy: Pixar.]({{ site.url }}/images/boyscout.jpg)

This article takes a deep dive into store2be's philosophy of code refactoring.

We always try to foster a culture that values code quality. Constantly staying away from refactoring and accumulation of tech debts can lead to the [Broken Window](https://en.wikipedia.org/wiki/Broken_windows_theory) Effect - which simply states, if you are surrounded by bad code, the chance of you writing bad code is higher. This would lead to a downward spiral of the quality of the codebase or in other words - the number of WTF's per minute. 


Refactoring code can be tricky when it is being applied to a wider process of software development, especially in an organisation. It can trigger a domino effect leading from one refactor to the other.

![Code refactoring.]({{ site.url }}/images/regret.jpg)

---

Deciding how much time and resources should be allocated to refactoring or resolving technical debts is always a challenging engineering decision. This is where the Boy Scout Rule for [opportunistic refactoring](https://martinfowler.com/bliki/OpportunisticRefactoring.html) comes into play.

## The rule

> Leave your code better than you found it.

Many of you would already be familiar with the Boy Scout Rule in programming. Everybody talks about it, but not everyone does it.

At store2be, we follow the Boy Scout Rule and have made tremendous improvements to our code base. We always try to be informed about the latest technologies and use only what makes sense - always trying to strike a balance between what is stable and what is cool.

Making the codebase better could be done in several ways - from writing a better comment, writing tests for an untested module, or renaming a function to refactoring an entire file or module.

## Real life example

A while ago, we introduced TypeScript in our code base and never looked back. However, for obvious reasons, we didn't move all the Javascript files to TypeScript in one sitting. At this point, we have both JavaScript and TypeScript coexisting in our codebase. Our goal obviously, is to have TypeScript alone. 

One of the situations in which we strictly follow the Boy Scout Rule at store2be is when converting Javascript to TypeScript.

Whenever someone changes a Javascript file, even if it's a small change, we take time to convert it into TypeScript. We even have step by step documentation of how to convert Javascript to TypeScript. 

Here is how it looks.

![Typescript to Javacript conversion documentation.]({{ site.url }}/images/techhandbook.png)

Harnessing the power of opportunistic refactoring can make sure you always adhere to high standards of code quality, while shipping the features on time. I will leave this quote from the book Clean Code by [Robert Martin](https://en.wikipedia.org/wiki/Robert_C._Martin) here for you to think about how much writing clean code matters.

>"The ratio of time spent reading (code) versus writing is well over 10 to 1 … (therefore) making it easy to read makes it easier to write."
