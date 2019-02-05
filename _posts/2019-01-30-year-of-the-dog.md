---
date: 2019-01-30
categories:
- Stack
- GraphQL
- Typescript
- API
title: What changed in our stack in 2018
author_staff_member: 03_tom
---

As our team and codebase grow, we sometimes find the need to cast a look back
at how things changed, and remember fondly the code we wrote and deleted, the
abstractions we built and scrapped, how our code used to be, and how much fun
it was to get it to where it is now.

We are building a platform to make live marketing scalable, measurable and
targeted — as such we have multiple apps for the various actors of the platform
and one API.  We want to make sure we can keep growing and adapting the
platform to changing needs, maintaining agility and stability in the process.
This is our high-level mindset when making decisions regarding our tech stack.
This post is a recap of how it has worked for us in 2018.

![store2be developer, early 2018]({{ site.url }}/images/2018_retrospective/grumpy_pallas_cat_c_tambako_the_jaguar_flickr.jpg)
_store2be developer, early 2018 (photo © Tambako The Jaguar, flickr)_

## Frontend - the React apps

We introduced Typescript in our React apps at the end of 2017, and we are only
now getting close to completely typed frontend apps. Before Typescript, we had
suffered from churn in our JS architecture as we experimented with more
scalable approaches to state management and asynchronous actions - we had our
own abstractions on top of
[redux-saga](https://github.com/redux-saga/redux-saga) and even open sourced
our way to deal with boilerplate in redux,
[redux-belt](https://github.com/store2be/redux-belt/). These abstractions were
constantly evolving, and it was a challenge to evolve core abstractions in very
stateful apps without introducing type errors which translated directly to
crashes for our users. There are enough Typescript success stories on the
internet, so I will not elaborate too much on this.

Good tooling is something we appreciate, so in addition to Typescript we
adopted [prettier](https://prettier.io/),
[TSLint](https://github.com/palantir/tslint/) (in addition to ESLint),
[jest](https://jestjs.io/) (replacing jasmine), and [snapshot
testing](https://jestjs.io/docs/en/snapshot-testing.html). All of these lifted
a lot of mental burden: no more worrying about or manually fiddling with code
formatting, automatically enforced conventions, more certainty that the UI does
not change by surprise, etc. Linting, testing and a lot more is enforced in CI so
we can focus on the hard problems during code reviews.

We liked the experience so much we implemented a backend service for email templating
in typescript and [wrote about
it](https://tech.store2be.com/email/sendwithus/mjml/typescript/react/2018/06/14/email-templates-at-store2be-and-gdpr/).

Another notable and much-appreciated addition to our toolbox was
[react-storybook](https://github.com/storybooks/storybook) for our shared UI
components.

## An unexpected journey - GraphQL in our apps

Despite the numerous changes to the frontend workflow, architectural issues
were still making our lives difficult.

1. Our RESTful API structure had not evolved much: most of our endpoints were
   mapping directly to database tables, with little additional logic.  With more
   resources and more complex business logic, that meant **a lot of business
   logic had to be implemented in the frontend** and merely validated by the
   API.
2. **State management and synchronization in the React apps was getting too
   complex** - some operations and queries required several network calls and
   dispatching multiple redux actions. Some critical operations were not
   atomic, when they should have been.
3. **Onboarding new developers to our redux-saga based abstractions** and all
   the related conventions **proved challenging**, as documentation was not as
   comprehensive as it should have been and old conventions coexisted with new
   ones.
4. With most of the data in our apps coming from the API, we felt that manually
   writing Typescript interfaces for all these resources was a lot of error
   prone manual work. We thought about generating them from our
   [OpenAPI/Swagger](https://swagger.io/specification/) documentation, but the
   experiment wasn't very conclusive. Coupled with the type information loss
   from the redux-saga generators, we felt that we were **not realizing
   Typescript's potential**.
5. **Documentation was maintained manually as a Swagger specification**. This
   was also a lot of error-prone, manual work.

Faced with these problems, we decided to try and address them by progressively
introducing a new GraphQL API, in the same Rails codebase as the existing
RESTful API. Here is what we observed:

1. The additional layer gave us more room and structure to **move business
   logic back to the backend**.
2. **Data fetching in the frontend is now a lot less complex**, with queries
   mapping more or less 1-to-1 with views, and mutations to forms and buttons.
   We are not using any redux-based abstraction for CRUD anymore, and the
   overall amount of state management code has decreased. State is (or appears)
   also more localized to where it belongs in the component tree. We found this
   easier to reason about.
3. The simplicity, but also the adherence to a well-adopted and documented
   standard for data fetching made **onboarding easier**. There are a lot of
   good GraphQL resources for frontend developers ([the official
   guide](https://graphql.org/learn/), to mention just one). In our experience
   picking up the technology was easy for everyone.
4. The tooling is amazing, it lets us **leverage Typescript's strengths by
   generating precise types for our API data**. We use apollo-client and
   apollo-codegen, they are well-documented, worked fine and the caching layer
   is the cherry on top.
5. **Our API documentation is now in much better shape**, because most of it is
   automated, keeping it in sync with the implementation is trivial, and
   [GraphiQL][graphiql] offers a good interface to explore it.

It took some time to have a viable minimum implementation, but once this was
deployed, it became clear very quickly that we wanted to go in that direction.

Two more pain-points that we did not identify before migrating, but improved as
a result:

- **Debuggability**. [GraphiQL][graphiql] has been a boon to identify where
  errors come from.  With multiple requests that cannot be reproduced easily,
  it was sometimes hard to distinguish between backend and frontend issues (the
  network tools do not let you tweak and reproduce the requests easily). With
  queries you can share directly as a link — [GraphiQL][graphiql] can take
  queries from URL query parameters — we saved a lot of debugging time.
- **Evolvability**. The way fields can be added and deprecated progressively
  has radically decreased the frequency of necessary breaking changes and made
  deployment smoother as a result. We use
  [eslint-plugin-graphql](https://github.com/apollographql/eslint-plugin-graphql)
  to warn us when an app is using a field that has been deprecated.

![store2be developer discovering GraphiQL]({{ site.url }}/images/2018_retrospective/developer_discovering_graphql.jpg)
_store2be developer discovering GraphiQL (photo © unknown)_

## GraphQL in our Rails API

Adopting GraphQL was, in retrospect, an easy win in the frontend. It did
require some changes, but towards a more standard workflow with a wide array of
tools to make our lives easier. It was not the same in the API. GraphQL APIs
pose a different set of problems and require different solutions, compared to
purely RESTful APIs. To list a few of the challenges we faced:

- **Code organization**. We naturally started by separating the API we expose
  to users and the one we use for our internal tools, since we had separate
  sets of endpoints. It seemed like the easiest way to separate what admins and
  normal users should see and be able to do, and code-sharing would still be
  possible. For a variety of reasons, this turned out to be a bad decision and
  we ended up merging the two APIs after a few months. This is only one example
  of the difficulties we had with proper code organization and reuse in the
  GraphQL API. We now have a good set of conventions and abstractions but it
  took a fair amount of exploration.
- **Testing**. Some of the abstractions in
  [graphql-ruby](http://graphql-ruby.org/) are not very friendly to test, so we
  had to come up with a test-friendly code structure.
- **Limiting access and visibility**. Nothing particularly challenging here,
  but this also needs to be learned.
- **Proper use of more "advanced" features like interfaces**
- **Avoiding the n+1 query problem** - there is one well-supported,
  well-documented and straightforward solution,
  [graphql-batch](https://github.com/Shopify/graphql-batch) (by Shopify), but
  this is one more thing to learn and implement.

It took us a few months to settle on satisfactory abstractions. Resources like
the [Shopify GraphQL design
tutorial](https://github.com/Shopify/graphql-design-tutorial) were helpful. In
general, having someone with prior knowledge or significant interest/experience
advocating GraphQL good practices and tools helps.

Now that the abstractions are well defined, we find it easier to onboard
developers to the API, and we work more with plain ruby classes instead of
Rails-specific methods in models and controllers.

In short, the change was received enthusiastically by the whole team once the
initial hurdles were behind us, and we are very happy with it.

## Backend and infrastructure - events, k8s

We started implementing an event log for our platform, for auditing, debugging
and in the future for decoupling tasks. The primary feature at the moment is a
timeline in our internal app, but it opens up a lot of perspectives and
challenges for 2019.

Another change we have been working on is the migration to Kubernetes. We
strictly adhere to _infrastructure as code_ and have all of our infrastructure
defined as terraform scripts and helm charts. We want to complete the migration
this year. We discovered a lot of tools on the way, but one fun tool that bears
mentioning is [click](https://github.com/databricks/click); it acts as a kind
of REPL/terminal UI for your kubernetes cluster.


## The Tech Handbook

At the end of 2017 our team was still at a scale where every developer had a
relatively complete knowledge of the codebase in their head, but this could not
last. We started and grew our internal development team documentation, what we
call the Tech Handbook. It takes the shape of a GitHub repository with simple
markdown files — with indexes and ToCs — so the editing workflow is comfortable
for everyone. We discovered that as the codebase grows, this is not a
nice-to-have but a necessity.

## Sign-off

![store2be developer, early 2019]({{ site.url
}}/images/2018_retrospective/hopeful_pallas_cat.jpg) _store2be developer, early
2019 (photo © HuffingtonPost)_

We made rather dramatic changes to our stack in 2018 and did not compromise the
delivery of actual features in the process. There was no hard, breaking change,
migration or rewrite, only improvements. Let us hope the same goes for 2019.

If you would like to know more, feel free to reach out to us on
[twitter](https://twitter.com/store2be_tech).

[graphiql]: https://github.com/graphql/graphiql

