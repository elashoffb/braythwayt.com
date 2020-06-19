---
title: "Application developers, we need to have a talk"
layout: default
---

[![Howl's Moving Castle](/assets/images/real-howl.jpg)](https://www.flickr.com/photos/schoukah/11972966156)

You may have encountered this variation on [Greenspun's Tenth Rule](https://en.wikipedia.org/wiki/Greenspun's_tenth_rule):

> Every sufficiently complicated application contains an ad hoc, informally-specified, bug-ridden, slow implementation of half of a framework.

Meaning, all applications of a certain size inevitably contain a framework. Some are explicitly built on a robust, high-performance and feature-complete framework, and the rest end up with crud.

This leads people to reason that if we're setting out to write a non-trivial application, we'll wind up with a framework anyways, so why not explicitly pick a good one up front? Why accidentally create our own ad hoc, informally-specified, bug-ridden, slow implementation of half of a framework in an effort to do without?

But I believe that while writing an application without a framework sometimes does result in writing our own framework of indifferent (or worse) quality, it needn't. We just need to be aware of what a framework provides, and if we choose not to use a famework, we should consciously choose to do without some of what it provides.

### what is a framework?

In contemporary programming terms, a framework contains--at a minimum--a collection of libraries that interoperate well, an architecture for organizing the libraries and the code we write, and a mechanism for wiring up our code to the libraries provided by the framework. Other tools such as build pipelines may also be provided.

The benefits of choosing a good framework include an assurance that the libraries provided all work together seamlessly, and a standard way to do many things. When a large number of people embrace the framework, we benefit to some degree from having a consistent way to solve certain problems.[^folder]

There is merit to this. Let's forget about the effort of writing the code for a moment. There are decisions that matter, and decisions that don't. One of the benefits of a framework is that it makes a lot of decisions for you. If those decisions don't matter, you've saved yourself a tremendous amount of time by not having to think about them. If you're on a team, you've saved the team a tremendous amount of time by not debating them.[^python]

[^python]: This is the rationale behind [Python] having significant whitespace: It is never a good idea to spend time arguing about how to indent code, and whether the braces go on one line or the next. So having a language enforce one right way to do it means you spend all your time thinking about things that matter.

[Python]: https://en.wikipedia.org/wiki/Python_(programming_language)

[^folder]: In [Rails][Ruby on Rails], controller class files go in the "controllers" folder, which is inside the "app" folder by default. Since almost all Rails apps are organized this way, when a Rails programmer starts work on a new app, they know exactly where to find controller files.

To summarize, the important things about a framework are that it is:

- A collection of libraries;
- That interoperate well;
- And an architecture for organizing code.

### how does a framework organize responsibilities?

In addition to these things we can observe, framework architectures typically are organized around the framework doing most of the work, and calling out to our code where necessary. In [Ruby on Rails](https://en.wikipedia.org/wiki/Ruby_on_Rails), we write controllers, and Rails invokes our controllers. Everything we write conforms to an API provided by the framework and is, in essence, subservient to its design.

[Ruby on Rails]: https://en.wikipedia.org/wiki/Ruby_on_Rails

This is an important point, because by design, frameworks are intended to be used by a staggeringly diverse number of different applications, each with its own unique requirements and behaviour.[^diverse]

The intersection between the design of the framework being in control, and the framework needing to support a widely diverse set of applications drives an important characteristic of frameworks: They introduce [abstraction layers](https://en.wikipedia.org/wiki/Abstraction_layer) and [indirection](https://en.wikipedia.org/wiki/Indirection).

> Frameworks introduce abstraction layers and indirection.

[^diverse]: [Ruby on Rails](https://en.wikipedia.org/wiki/Ruby_on_Rails) was created to power [Basecamp], an application to help teams coördinate their work on projects. But it also powers applications as diverse as [GitHub] and [PagerDuty], code bases with completely different needs.

[Basecamp]: https://basecamp.com
[GitHub]: https://github.com
[Ruby on Rails]: https://en.wikipedia.org/wiki/Ruby_on_Rails\
[PagerDuty]: https://pagerduty.com

Let's consider what happens when a framework like Rails calls one of our controller methods. Rails is fixed code, it is the same for every application from a mom-and-pop site that registers children for summer camp to [GitHub]. But Rails must work for every one of our controller methods and for every controller method in every controller class in every Rails application.

{% highlight ruby %}
class SomeController < ApplicationController

  def index
    # ...
  end

end
{% endhighlight %}

By design, Rails must handle every possible error, every possible thing a controller method might want to return. Rails must provide every possible piece of data and environment our particular controller method might need to decide what to do and/or what to return. Rails must provide a way for we to chain our response to another handler, return redirects, everything.

Controller methods are complex, obviously. But the way in which Rails is wired up to controller methods in applications must, by design, handle all of the complexity of every application that could ever be written in Rails.

That flexibility comes at a cost: The framework must introduce "magic," a/k/a indirection and abstraction. And this happens everywhere that the framework calls our code. It's not just how it calls a controller method: It's in how models have lifecycle methods, how there are multiple ways to observe changes, how you can decorate methods, everything.

> "All problems in computer science can be solved by another level of indirection."—[David Wheeler](https://en.wikipedia.org/wiki/David_Wheeler_(computer_scientist))
>
> "...except for the problem of too many layers of indirection."—[Kevlin Henney](https://en.wikipedia.org/wiki/Kevlin_Henney)

Notice that this is not the case with libraries. Our code calls library functions and methods, and since we are in charge, the complexity of dealing with our particular needs is in our code. Thus, library code is considerably simpler than the complexity of framework code.

In sum, when a framework calls our code, it must handle the complexity of handling all application needs by introducing abstraction and indirection into the API between the framework and our code. When a library is called by our code, it provides a simpler, more direct API, and we are responsible for handling our particular needs.

### what if we don't use a framework?

If we don't use a framework, don't we always end up with a framework? Won't we wind up building "An ad hoc, informally-specified, bug-ridden, slow implementation of half of a framework?"

Don't all applications have a collection of libraries that interoperate well, an architecture, and mechanism for wiring our code to the libraries? So therefore, don't we always get a framework, but sometimes we get the pain and suffering of libraries that don't interoperate well, or a poor architecture?

No.

When we set out to write an application without a framework, we certainly need libraries (It's quite rare to build most of an application entirely bespoke). And yes, we may need to do some research to discover which ones work best in concert. That being said, there are certain well-known collections of complementary libraries, and some libraries consist of components that are all written to work together, like [Backbone.js](https://en.wikipedia.org/wiki/Backbone.js).

We'll also need an architecture, but it's not like we need to invent one from scratch, and that nobody else will understand it. If we have Models, Views, Controllers, and View Models, everyone knows what you're doing, why, and how to work with it.

We've had variations on [MVC] since at least 1982. The same goes for [hexagonal], [naked objects], or other architectures: There are well-known patterns of architecture for organizing applications, and choosing one that programmers will understand is hardly the most daunting challenge we will face.

[hexagonal]: http://www.c2.com/cgi/wiki?HexagonalArchitecture
[MVC]: https://en.wikipedia.org/wiki/Model–view–controller
[naked objects]: http://www.c2.com/cgi/wiki?TheNakedObjectsFramework

And naturally, our code interoperates cleanly with the libraries and everything else. After all, you're writing it! By definition it works with everything.

But have we wound up writing our own framework? Or not?

The answer can be found by looking at the way our code is hooked up within our architecture. If it is full of abstractions and indirections designed to facilitate behaviours we do not use and do not need, we have created our own framework.

If our architecture is economical with its abstractions and indirections, if it uses them to create cohesion without coupling and to separate responsibilities without introducing extra code to handle phantom responsibilities, we have not created a framework.

### so, what's better? framework? or no framework?

It's a tradeoff: *We win if we build an app using a framework*, for all the reasons noted: a curated set of libraries, a standard way to do things understood by a community, and we need never waste time worrying about making decisions that don't matter.

*We also win if we write an application using good libraries, well-known architectural patterns, and without introducing unnecessary indirection and abstraction*. We win if we make it do one thing and do that one thing well. Our app can be simpler and cleaner without a framework.

### the dark side of application development

So why the warnings about writing ad hoc, informally-specified, bug-ridden, slow implementation of half of a framework? *Because of the [inner platform effect]*. Whether intentionally or through overzealously copying the abstractions and APIs of frameworks, many applications end up with a framework inside them.

The point of a framework is to be extremely flexible and to minimize the amount of code you write. Although few people set out to write a framework for many different applications written by many different teams, most people know that the needs of their application will change over time, so they try to accomodate at least two people: Themselves now, and themselves in the future.

If they also try to minimize the amount of code they will write in the future, they will organize their architecture around relatively static code that calls out to smaller pieces of code to be written later. Now they are writing a framework.

And they get all the disadvantages of a framework--like extra levels of indirection and unnecessary customizability--coupled with all the disadvantages of writing their own app--like needing to write more code, make more decisions, and not have a community standard way to do anything.

[inner platform effect]: https://en.wikipedia.org/wiki/Inner-platform_effect

### success in application development

The key to success in writing an application without a framework *is to write an application without a framework*.

Have libraries. Have an architecture. Wire things together. But strive to keep it lean and focused. Practice YAGNI ruthlessly and view with deep prejudice any extra abstraction or customizability "just in case we need it."

Do not embrace wild convention-over-configuration in a custom app. Do not build elaborate class hierarchies just so that a new widget can be written in one line of code.

Just write the app.

---

### other reading

0. [The Wrong Abstraction](http://www.sandimetz.com/blog/2016/1/20/the-wrong-abstraction)
0. [Why I No Longer Use MVC Frameworks](http://www.infoq.com/articles/no-more-mvc-frameworks)
0. [The Vietnam of Computer Science](http://blogs.tedneward.com/post/the-vietnam-of-computer-science/)
0. [Don't Let Architecture Astronauts Scare You](http://www.joelonsoftware.com/articles/fog0000000018.html)

### notes
