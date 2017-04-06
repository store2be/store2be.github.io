---
date: 2017-04-06
title: Introduction to Metabase
categories:
  - Metabase
  - BI
  - PostgreSQL
author_staff_member: 01_peter
medium_link:
practical_dev_link:
---

When you start a company, setting up a business intelligence (BI) solution is not the first thing most founding teams
implement, I guess. Now that I'm past the step of implementing such a solution, I think it is something that should
be implemented in the very early days to build a culture of measuring everything.

The main reason why it wasn't implemented directly from the beginning at my company was that no one had the
experience with a good solution yet. Therefore, reports were conducted manually against the database in raw SQL
or implemented into the internal admin area to make them a little bit more accessible (but without graphs of course
because it would have cost too much time). As you might guess, this was some work and couldn't be changed or extended fast.

In the fall of 2015, [AWS QuickSight was announced](https://aws.amazon.com/blogs/aws/amazon-quicksight-fast-easy-to-use-business-intelligence-for-big-data-at-110th-the-cost-of-traditional-solutions/).
It seemed to be a very good solution for us as our databases are hosted on AWS. In August 2016 we got access to
the preview version. It wasn't really easy to use and the user management was not really good. Reports and datasources
weren't shared accross users or I didn't find the right way to do it. In most cases we use UUIDs as our primary keys and
AWS QuickSight had no support for UUIDs back then. This lead to the situation that I didn't put more time and effort in
using AWS QuickSight. When [AWS QuickSight was finally released](https://aws.amazon.com/blogs/aws/amazon-quicksight-now-generally-available-fast-easy-to-use-business-analytics-for-big-data/),
I took another look at it but I still wasn't satisfied with the look and feel.
(This is very subjective and might be different if I gave it another try today!)

At the same time of the release of AWS QuickSight, my former CTO at [fotograf.de](https://www.fotograf.de),
[Marco Beinbrech](https://github.com/beinbm), showed me their BI solution, an open source software called [Metabase](http://www.metabase.com/).
He told me that it is super simple to setup and easy to use for all team members (especially non-technical team members 😉).

And indeed, he is right. [Metabase provides different ways to deploy it anywhere.](http://www.metabase.com/start/) Most
of our servers were at Heroku then and Metabase provides a one click deployment to Heroku. Pretty neat! But at that
time we were also undertaking our first steps with Kubernetes and Metabase seemed to a be a perfect fit to play around with
Kubernetes. So we wrote the Kubernetes objects for it and voilà, it was up and running. No hassle at all!


## Further setup

_A collection of things you should consider when setting up Metabase_

Using the production PostgreSQL user creates a security risk because this user can change data in the database. As
Metabase is only for reporting, and therefore reading data, we created a read-only PostgreSQL user for our
production database.

It is not a good idea to give Metabase direct access to your production database. When you create huge and slow queries,
it will directly affect your production servers. Fortunately, [AWS RDS makes it super easy to create a read replica
of your production database.](https://aws.amazon.com/rds/details/read-replicas/)

Metabase needs a database it can read and write from for itself in order to manage data about the users, reports and so
on. To make the database of Metabase persistent across Kubernetes deployments, we created another database on our
RDS instance. This way, the Metabase data is also backed up by our current backup procedure. You just have to
provide the PostgreSQL connection details to the Metabase deployment in Kubernetes and it works out of the box.
Then you can create and rebuild the cluster without loosing any Metabase internal data.

We use Google Apps at my company and Metabase has a Google OAuth login built in. [So you just have to follow the guide](http://www.metabase.com/docs/v0.23.1/administration-guide/10-single-sign-on.html)
and all your colleagues can register with their Google Apps account.

User management can be handled with user groups and collections (folders for your reports in Metabase). Marco gave me
the tip to set up collections and and user groups according to the department structure. This way you can implement
user access on a collection level very easily.


## First steps

I really recommend [reading the docs of Metabase](http://www.metabase.com/docs/v0.23.1/). They are really good,
especially for the first steps.

In Metabase reports are called questions. So in order to create the first report you click on `New Question`, select
the database, select the table, add filters, specify what you want to see (eg. count of users) and add a grouping
(eg. created_at by month). And all of that in a simple click interface that can be used by a lot of users and not only the
SQL pros in the company. (It still gives you the possibility to create custom SQL queries).

_Me encanta!_ 😊

Finally, you can put multiple questions on a dashboard and have a nice overview of what's going on in the company.


## Conclusion

All in all, Metabase was really easy to setup for the tech team. If we had used Heroku, it would have been even easier.
And most importantly, it is really easy and fun to use for all team members. 😊


_Thanks Marco, for all the tips here 😊 In further posts I will go into detail on how we create advanced reports at
[store2be](https://www.store2be.com)._
