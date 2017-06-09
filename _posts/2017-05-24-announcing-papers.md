---
date: 2017-05-24
title: Announcing Papers, a PDF generation service based on LaTeX, written in Rust
categories:
- Rust
- Open source
- Latex
author_staff_member: 03_tom
medium_link: https://medium.com/store2be-tech/releasing-papers-a-pdf-generation-service-based-on-latex-written-in-rust-e224dc68ba8
practical_dev_link: https://dev.to/tomhoule/releasing-papers-a-pdf-generation-service-based-on-latex-written--in-rust
---

The store2be team is happy to announce the [first public release](https://github.com/store2be/pape-rs/releases/tag/0.1.0) of [Papers](https://github.com/store2be/pape-rs), an HTTP service that receives LaTeX templates, variables and assets needed for the generated LaTex file, and generates good looking PDFs. It is designed to be stateless, safe, reliable, easy to use and to deploy. We even have a second binary to test your templates locally before deploying them.

## What? Why?

We needed a PDF generation service at store2be for a new feature in our advertising space manager. We investigated four options:

- Use a pure Ruby PDF library like [prawn](http://prawnpdf.org/)
- Generate PDFs from HTML and CSS with something like [wkhtmltopdf](https://wkhtmltopdf.org/)
- Use a paying service like [DocRaptor](https://docraptor.com/)
- Write it ourselves

Writing templates as code takes too long for the number of templates we are likely to write, and generating from HTML ourselves did not yield very nice documents, so the remaining solutions were an external service or Papers. Given the small scope of the project, what we would learn from the experiment of using a different programming ecosystem (we only used Ruby in the backend before), and the much better looking PDFs produced by TeX, we went ahead and wrote Papers.

## How do I use it?

You will need three things:

- Have the Papers service running somewhere so you can make HTTP calls to it
- Have the ([Tera](https://github.com/Keats/tera)) templates and assets hosted somewhere (we use S3/CloudFront)
- Have an HTTP endpoint in your app where the PDF or errors will be posted

Generating a PDF is then a simple matter of posting a JSON object describing your wish to the `/submit` endpoint of your Papers service. For example:

```json
{
  "assets_urls": [
    "https://static.cook.example.com/spaghetti.jpg",
    "https://static.cook.example.com/pesto.jpg"
  ],
  "callback_url": "https://cook.example.com/illustrated-recipes/generated&token=18e8d495ac34c541b5a5167bffcbac96",
  "template_url": "https://static.cook.example.com/templates/pesto_spaghetti.tex"
}
```

In order to deploy Papers easily, we prepared a [docker image](https://hub.docker.com/r/store2be/pape-rs/). There is also a [Kubernetes example](https://github.com/store2be/pape-rs/tree/master/examples/kubernetes) in the repository.

## Our experience with Rust

Overall, Rust was a very good fit for the goals we had.

- We wanted a crash-free service, and the explicit error handling made that possible with a high level of confidence.

- We wanted the service to be stable and not leak memory, disk space or file descriptors. Once again, it was very helpful that Rust gives you the tools to apply the same logic that makes memory safety without garbage collection possible for other resources, like files or temporary directories. And we get fine-grained concurrency control with tokio as well.

- Since most of the overhead of the service comes from LaTeX itself, we wanted it to be as lightweight as possible, mostly memory-wise, and there Rust is ideal again compared to Ruby.

- We wanted easy deployment and administration, and having a compiled binary makes it very easy.

Writing a small service was also a way for our team to assess Rust as a tool for other projects in the future. Our perceptions have adjusted in more than a few ways from this experience.

We set out with a perception of Rust as the cool new thing, and expected to get high level abstractions from it. The previous experience with Rust in the team ranged from having read the Book to having completed small programs, and learning curve was a bigger deal than we thought. Rust *does* force you to think at a lower level and handle more details than something like Ruby or Python: you have to make decisions about errors, even if you want to ignore them, and even if there are ways to simplify memory management problems (`Rc` for example), they are opt-in.

It was hard to accept that you sometimes have to write code to make the program compile and does not contribute at all towards solving the business problem (in our minds as programmers used to high-level languages), for example:

```rust
fn make_pasta(&self) -> Box<Future<Item=Satisfaction, Error=OvercookedError>> {
  let logger = self.logger.clone();
  Box::new(cook_pasta.and_then(move |pasta| {
    warn!(logger, "Pasta is very hot, be careful");
    eat_pasta(pasta)
  }))
}
```

The line where we clone the logger so it can be moved into the Future is purely there to satisfy the borrow checker, and while it makes complete sense once you "get" the concept ownership, it still feels like clutter in the context of high level code.

The cargo ecosystem, and especially cargo doc, is fantastic. Period.

Library-wise, we found good quality libraries for everything. The elephant in the room now for server programming is of course the work-in-progress conversion of the ecosystem to asynchronous APIs based on [futures](https://github.com/alexcrichton/futures-rs) and [tokio](https://tokio.rs/), which was inconvenient but not a show stopper. We went for hyper with the master branch, as there was no higher level alternative that worked with futures.

In general, futures were hard. There is a clear rightward drift, and error handling differs from synchronous code (in a bad way). The interaction with the event loop also has a learning curve of its own. To be fair, things are going to improve a lot with time and a few language features that are already planned, in particular [impl Trait](https://github.com/rust-lang/rfcs/pull/1951), [trait aliases](https://github.com/rust-lang/rfcs/pull/1733) and, albeit more distant, async/await syntax. The current need to box everything or write complicated types nudged us in the direction of big, monolithic functions when we have to return a Future.

Compile times are long. Coming from interpreted languages, this is the most frustrating part of developing in Rust. They are getting better though, and incremental compilation has yielded promising results for us.

Our conclusion is not very original: Rust is a good tool for the use-cases where it makes sense, but we wouldn't use it as a language to write a classical web app (yet).

## What's next?

The service is usable in production for small volumes of PDFs (depending on the size, a dozen per minute seems reasonable) and behind a firewall. We do plan to make it faster, and the first obvious step in that direction would be caching the assets. We also need a smarter way to manage the latex processes, and benchmarks to measure and improve performance.

There are already many other features that go in the direction of more configurability and better debugging experience planned for the next releases.

## Epilogue

If you read this far, we'd be really interested in feedback on our endeavour and on the code if you have some. Just shoot us an email at tech@store2be.com :)

For comprehensive documentation, you can head to the [README](https://github.com/store2be/pape-rs/README.md).

This is of course all open source, released under the MIT license, and [available on Github](https://github.com/store2be/pape-rs). We are very much open to contributions.
