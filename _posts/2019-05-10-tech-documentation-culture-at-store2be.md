---
date: 2019-05-09
categories:
- Documentation
- Culture
- Engineering
title: Tech documentation culture at store2be
author_staff_member: 01_peter
medium_link: https://medium.com/store2be-tech/tech-documentation-culture-at-store2be-10230b7000e3
practical_dev_link: https://dev.to/peterfication/tech-documentation-culture-at-store2be-11m9
---
![Our documenation setup]({{ site.url }}/images/tech_documentation/library.jpg)
_Our documentation setup (photo by Michael D Beckwith, flickr)_

<!---
https://docs.google.com/document/d/1lxqjRedXeRYqzOrL-Lr1KyUHWO2xzi1Br5qM0xqoYZA/edit#
-->

## Intro

> Ink is better than the best memory. 🎓

This is the subtitle of our tech documentation!

---

If you haven’t thought about documentation in a small team, with a growing team you eventually realise how important documentation is. Even for one person alone, it takes a lot of cognitive load to remember all the knowledge when it is not written down. In a team, documentation is even more important and acts as a communication tool and ensures that everyone has the same understanding of certain topics. Documentation also helps a lot in conducting non-automated regular tasks that occur in a low frequency.

In this article, we want to share how we approach documentation at store2be. Documentation happens on different levels:

- Company-wide documentation
- Tech documentation
- [Infrastructure as code](https://en.wikipedia.org/wiki/Infrastructure_as_code)
- App-specific documentation
- Code documentation

Everything that is of importance for all departments needs to go into our company-wide documentation. For example, if something technical is important for the product team as well, we will put it in the company-wide documentation.

The engineering wide documentation is for all the technical things that are only relevant for the engineering and the devops team. In this part of the documentation, there is also the frontend and the backend related documentation that is shared across all our apps, so we don’t have to repeat it in these repositories.

Our engineering wide documentation is a Git repository with markdown files. This has the advantage that we can write it easily offline and use our favourite tools to edit and search the documentation. Also, everyone has always a local copy of the whole documentation present.

Infrastructure as code happens, when you use tools to describe your infrastructure in a declarative way and you don’t manage your infrastructure manually. Some tools that we use in that context are [Terraform](https://www.terraform.io/) and [Kubernetes](https://kubernetes.io/).

If something specific to an app needs to be documented, we will mostly put it into the README.md of the respective Git repository. Examples here are eg. app specific scripts/commands and what they are there for.

We won’t go into details about code documentation, because this is something a whole blog post on its own can talk about. We want to mention it for completeness here anyway.

## Tech Documentation

![The Tech Handbook]({{ site.url }}/images/tech_documentation/tech_handbook_intl.png)
_The Tech Handbook_

The main topic of this blog post is our tech documentation. In this part of the documentation, we collect the most knowledge of the engineering team and this is what helps us scale the engineering team. We call it: _The Tech Handbook_.

Looking back into 2017, we had nearly no tech documentation at all. We were still just a team of three and we hadn’t thought too much about documentation at that point. In hindsight, this is super bad and something we will avoid in the future, because now we know how valuable proper documentation is.

By the end of 2017, we had three people joining in the beginning of 2018 and we knew, we had to share a lot of knowledge with them. That’s when we began the groundwork of our Tech Handbook.

In the beginning, we focused on the areas that we needed to share with the new developers from day one, eg. processes and tools. The infrastructure sections followed quickly, as infrastructure work is mostly done once and then looked at again a lot later when you forgot a lot of it probably.

Since then we continuously add new things along the way, and documentation has become part of our tech culture.

Keeping documentation up to date is a whole different challenge. At store2be, it’s a collective effort. As soon as we discuss for example process changes, we assign someone with the task of updating the relevant part in the tech documentation via our project management tool. This helps us organizing documentation changes together with our normal development workflow. Thankfully, we use Git and GitHub for our Tech Handbook, everyone else in the team can review the documentation changes and comment on them. So in the end, we’re all on the same page again.

We divided our Tech Handbook into the following areas:

- General
- Processes and tools
- Infrastructure
- Backend
- Frontend
- Post mortems

These sections should be all pretty self-explanatory. The last section, post mortems, is the newest section that we added in autumn 2018. We realized that most of the time the same people worked on finding and fixing critical issues in our systems without much transparency to the rest of the team. We occasionally had small presentations of what caused the problem, but if a similar problem occurred again, the person working on it had to remember it. This approach doesn’t scale well, and even if the same person worked on a similar problem again, this person might have forgotten parts of the previous solution.

With the post mortems in our Tech Handbook, we can now share the steps on how we were able to debug an issue easily, and because everyone reviews it, everyone gets a feeling for possible issues and can account for it while coding.

## Conclusion

All in all, we are very happy with our choice of building up our Tech Handbook and can really recommend it to everyone. It gives us a lot of safety regarding not losing knowledge, especially if someone goes on holidays. Also, onboarding new people has become way easier and faster.
