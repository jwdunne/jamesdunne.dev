---
title: "MEAN Stack Considered Harmful"
date: 2020-09-13T16:00:39+01:00
---

The MEAN stack is everywhere.

Proponents promise getting started is **easy**.

Hundreds of boot camps provide crash courses promising a MEAN stack job. You
don't owe them until you get hired. No win, no fee.

Developers
[write](https://www.packtpub.com/product/mean-web-development/9781783983285)
[thousands of
words](https://www.manning.com/books/getting-mean-with-mongo-express-angular-and-node)
teaching the stack, preaching the benefits. Become a "full stack developer" in
as little as twelve weeks.

It's adored by greenfield developers. They can build new projects fast. And move
on even faster.

The MEAN stack is flawed. It is a poor combination of technologies and I
consider it harmful.

**Why?**

The ideas underpinning each technology in the stack are fundamentally
incompatible, with potential for genuine harm.

## The MEAN stack

The acronym used to mean:

1. MongoDB, a popular NoSQL database
2. Express, a micro framework for NodeJS
3. AngularJS, the now-obsolete front end framework
4. NodeJS, the JS runtime.

Thanks to the JS ecosystem, the A is now a wildcard. Bring your own frontend
framework. MERN and MEVN are popular acronyms.

To make life easier, by MEAN I mean any frontend framework. There's nothing
specific to AngularJS here.

## MongoDB

> "A chain is no stronger than its weakest link" - **Thomas Reid**

A stack is only as strong as it's weakest component. MongoDB is that weakest component.

[So
much](http://cryto.net/~joepie91/blog/2015/07/19/why-you-should-never-ever-ever-use-mongodb/)
[has been](https://news.ycombinator.com/item?id=18366385)
[written about](https://dev.to/ankush981/mongodb-has-no-use-case-58ob)
[what is wrong](http://www.sarahmei.com/blog/2013/11/11/why-you-should-never-use-mongodb/)
 with MongoDB as a technology. But
I'll focus on what makes it especially dangerous in combination with other MEAN
technologies:

1. It's schema-less
2. It's unnecessary

### The schema-less fallacy

With MongoDB, there is no schema. You're free to structure your data in response
to change. No more heavy migrations. You're free to iterate fast.

And it works. At first.

You can get up and running with ease. You can change your data structures on the
fly, as you build, as requirements change.

**It's a fallacy.**

Sure. MongoDB refuses to enforce a schema. But you still get a schema. It's just
on you to enforce it at the application level.

That's fine. We just need _discipline_. We just need more unit tests. It's
_our_ fault for being undisciplined.

Bob Martin made this [argument for dynamic
typing](https://blog.cleancoder.com/uncle-bob/2017/10/04/CodeIsNotTheAnswer.html).
And it just doesn't wash.

You still need to design a schema. Otherwise, you end up with a mess of
documents with no clear structure. In other words, chaos.

You need to enforce your schema at the application level too. You end up with a
patchwork of conditionals, accommodating the two schemas. Or you write a
migration to migrate to the new schema.

Someone will screw up eventually. It's Sod's law. Instead of failing at the
point of error, MongoDB will carry on as normal, letting your application deal
with the mess.

It’s no coincidence why popular ODMs come with schema validation. Unfortunately,
it's still at the application level.

If you're doing all this on a new project with few to no users, what's the
point?

Just use SQL with good integrity constraints. Errors become far more obvious.
Data corruptions less common. And that's crucial in debugging - the last 10%
that takes 90% of the time.

### It's unnecessary

Don't start a project with NoSQL. These solutions solve problems at scale. That
was the disclaimer when NoSQL came onto the scene.

MongoDB doesn't market itself that way. Instead, it pushes itself as solution
for problems big and small, as a database for the modern world.

**It renders itself unnecessary.**

Most applications don't operate at scale. Your new project, right now,
definitely doesn't.

It's not just me. The Guardian, despite higher volumes of traffic than most
systems, [switched to Postgres](https://www.theguardian.com/info/2018/nov/30/bye-bye-mongo-hello-postgres) whilst keeping the car moving (and my news
flowing).

### Why it's so prevalent

MongoDB attracts developers because it makes it easy to get something up and
running. You have no schema, you don't need migrations.

That’s great for prototyping. _If you can throw it away._

Prototypes often stick around. You're often forced to shape it up and live
with it. That means living with Mongo.

Unfortunately, there's a subset of developers who only accept new projects. Get
something done fast. Move on. They don't live with their decisions and they
never learn from their mistakes.

And there's another subset, perhaps overlapping, who have been taught the MEAN
stack as if it is _the_ way to build software. Twelve week boot camps that
promise a development job at the end of it are the culprit. The boot camps don't
have to live with developers trained in a rush.

### Why it's so harmful

MongoDB offers no guarantees on your data. Not even the shape. You're free to
diverge as much as you like. Even nullify whole sub-documents.

_Where are those guarantees?_

Express doesn't help. Neither does Node. And if it's getting to your frontend,
your API is broken. End of story.

Nothing in the stack helps guarantee the shape and integrity of the most
important part of your application: the data.

Each of these technologies have a number of flaws that make them especially
dangerous when combined with MongoDB.

## Node + Express + Mongo = quicksand

I remember when Node took the world by storm. There was a huge rush of
development in JS, with frameworks, libraries and new ways of building web
applications.

Express was one of those frameworks. Express is a lightweight, no frills web
framework promising zippy web apps thanks to Node’s non-blocking nature.

Both NodeJS and Express were built before:

1. Promises entered the mainstream
2. Static types were seen as a positive

Despite best efforts, it still shows.

There's no real guarantee that your values are what you expect them to be. Or
_when_ you expect them to be, for that matter.

We are then combing that with a DB that offers no remedy and has the same
problems. It's such a great idea, we give it a clever sounding acronym.

You end up with an API built out of quicksand.

Now we've got thousand of developers starting new projects thinking this is the
standard in full stack JS development. We've got thousands more coming in
thinking this is the _only_ way.

There are thousands of businesses that depend on the results. And many of them,
after the developer has moved on, are left to live with an unreliable and
unmaintainable API.

That plus a frontend that's built on top of quicksand.

## Quicksand + SPA = trauma

A frontend framework is apparently a key component of the MEAN stack. It isn't
really. It can be any frontend framework or nothing at all.

On coining, the A stood for “Angular”, which is now “AngularJS”, which was
superseded by “Angular”, but could mean anything from React, Vue or just Vanilla
JS.

The idea is that the A means an Single Page Application, which consumes the MEN
part as an API. So now you have two applications. One made of quicksand and
another built on top of it. With a nice wedge of network complexity in the
middle.

An SPA adds a layer of complexity and possible failure. They decompose an
application into two applications that interact over the network. The complexity
makes it harder to debug and maintain the system.

The stack makes debugging *harder*.

Now you must live with a system that has the complexity of two pieces of
software and one of them is a maintainability nightmare.

**Is this a good way to build new applications?**

Of course not.

## Sparing the pain

**Step one:** stop giving false legitimacy to immature stacks with
official-sounding, easy to remember acronyms like MEAN.

Joking. We can't stop doing this.

### Choose a better database

Don't use Mongo. It's a disaster.

Instead, stick with boring old tech. You will go a long way with just Postgres
or MySQL.

There are good JS solutions for working with SQL databases. Start with those.

### Consider static typing

Static types eliminate a whole class of problems by catching invalid use of your
data structures at compile time.

That's exactly what's wrong with the stack.

Static types are not a replacement for a solid set of tests. But they do stop us
from the worst of our stupid errors. Invaluable in a language with two
separate null values. [Programming is a loser's game](https://tomgamon.com/posts/a-losers-game/).

Since TypeScript supports gradual typing, you can move over a piece at a time.
While you do that, you can start pushing up the strictness.

There are also tools to make the transition easier. AirBnb released one such
tool, [ts-migrate](https://github.com/airbnb/ts-migrate). But don't quote me - I
haven't tried it yet.

### Implement a full suite of tests

Don't just rely on unit tests. Test the integration of your subsystems. Test the
entire system and the application as a whole.

If you insist on any part of this stack, you need a lot more of this than usual.
There is simply more that can go wrong.

There's no easy way to do this. Writing a good suite of tests means having the
right attitude towards testing: tests are mandatory.

Test everything. Use white and black box methods. Start with a good acceptance
test and work downwards as you need to.

Integrate components bottom up and test those too. You'll find more errors at
the boundary where components meet.

I recommend reading [The Art of Testing](https://www.amazon.co.uk/Art-Software-Testing-Glenford-Myers/dp/1118031962).

### Consider a better framework

There are now solid alternatives to Express. One of those is Nest, built using
TypeScript.

Using a framework that was built Promise-first will save you a lot of headache.

### Consider a better stack

NodeJS has its problems. Even the guy who built it thinks so.

[Deno](https://deno.land/) is a great example of a developer reflecting on their decisions and
learning from them.

### Don't use a frontend framework

Unless you need to. Stick with a good old server-rendered templating engine to
start with. You'll get more done faster. And the faster you release, the better
the outcomes.

## Closing thoughts

Ditching Mongo is the best thing you can do for a project. Everything is an
order of magnitude easier once you do that.

A stack is only as strong as it's weakest component. Even if it's part of an
easy to remember acronym. Mongo is the weakest link.

Stop using Mongo, start living.
