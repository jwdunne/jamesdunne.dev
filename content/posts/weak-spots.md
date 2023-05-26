---
title: Finding your weak spots
date: 2023-05-26 13:44:08+00:00
---
In 2018, I was coasting. I didn't like it.

Life was good, I was steadily employed as a senior developer. We had a good team. We were building valuable software for our customers. Everything seemed great.

But I'd been coasting for a few years. I believed I could coast for a few years more (I was wrong) but I was uncomfortable. I needed to fix it.

I went through books, articles, podcasts and more books. But things never quite connected with software development.

So I polled other programmers by asking HackerNews: ["Finding your weak spots?"](https://news.ycombinator.com/item?id=16226294)

The responses were pure gold and... I did **nothing** with them.

## Rock bottom

Soon after, my personal life fell apart. I moved back in with my parents and I forgot all about the HackerNews thread. I'd sunk to rock bottom.

Getting back on my feet was slow. I took a round of antidepressants, meeting new people and adjusting to my new normal. But once I felt able enough, I returned to that Ask HN and looked at my biggest professional weaknesses.

I sucked at leading people. Our software development life cycle was Jurassic. We wrote few tests and often had to delete them. And we ran a script from our local machine to deploy our software.

## Un-rutting myself

I took on two remote developers. I _had_ to practise leading a team. We _had_ to improve our SDLC. We _had_ to improve coverage with higher quality tests and implement a controlled, automated CI/CD pipeline. Otherwise, chaos.

I read books like The Phoenix Project, Continuous Integration, Continuous Delivery and The Art of Testing. And I practised.

It worked beyond anything I could have imagined - our work rate accelerated. We delivered milestone after milestone.

## Disaster

And then the pandemic hit. Without the practices, processes and compatibility with remote workers, we couldn't have transitioned to remote work ourselves. It would have been a disaster.

But the opposite happened. We took to remote working well. So well, in fact, we ditched the office and became 100% remote.

Had I not taken heed of that small HackerNews thread from five years ago, my life would be  much worse today. I'd still be living with my mum and dad. I'd probably be jobless.

Here's what I learned.

## How to identify your weaknesses

### Define what 'weakness' means

A weakness is only a weakness if _it's actually getting in your way_.

I once found Haskell code difficult to understand. Its powerful type system was alluring. So I spent ages wrapping my head around monads, monoids, functors and what not. I learned little bits of alien syntax.

I made progress! Some Haskell code started making sense. Most of it didn't. **And it still doesn't make sense.** I don't use Haskell and I won't for the foreseeable future. It wasn't getting in my way so I didn't have a reason to retain what I learned.

So, first, define what "weakness" means to you. If you're a web developer, you might struggle with embedded systems, but that's probably not getting in your way.

### Take on a "stretch" project

A "stretch" project sits just outside of your skillset. By attempting it, you will learn in short order where your weak spots are. By completing it, you will have expanded your abilities. They _stretch_ your skillset.

A "study" in art is a similar idea. In a study, an artist focuses on one area, like applying textures to a set of spheres. Or sketching body parts ahead of a portrait.

Likewise, stretch projects are like a study. Small, focused tasks that feel around the edges of a topic and uncover areas of confusion and potential improvement.

Stretch projects don't need to have real-world value but often that helps connect the theoretical to the practical.

For example, the recommended course of action to learn game development is by a series of well known stretch projects.

Start out with Pong. Polish it, with menus, music and sound. Then do snake, to the same standard. And then break out. And then asteroids. After those, you could stretch to a simple platformer. Each project stretches your skills whilst teaching you what goes into a finished game. The last thing you do is start developing an MMORPG as your first game.

Likewise, with frontend development, you could start with a todo app. Then an address book. And then a simple social media feed. Or you could clone a series of popular interfaces that grow in complexity of interaction.

Stretch projects are an excellent way to find your weak spots, and _practice_ on them at the same time.

### Accept code reviews

Last year, I worked on a project using Scala and ZIO. ZIO provides a type-safe effect system for Scala. A bit like a super-charged, all-encompassing IO monad.

I had _zero_ idea what I was doing.

But, fortunately, I had colleagues that did. The feedback was harsh, direct but _useful_.

I learned how not to use JDBC data sources with ZIO. I learned how to sequence and manipulate DB queries using Quill. And I learned how Scala projects are organised.

I was soon able to contribute useful features.

Feedback from knowledgeable colleagues is worth its weight in gold. Be humble. Accept and _apply_ the feedback.

### Give code reviews

In January 2023, we took on an intern. Since then, I've been providing as much useful feedback through code reviews as I can (see above).

In one code review, I spotted something odd.

The way the intern wrote a conditional `ORDER BY` clause in Postgres looked wrong:

```sql
ORDER BY
	unread DESC,
	CASE WHEN :sort_direction = 'asc' THEN last_message_at END ASC,
	CASE WHEN :sort_direction = 'desc' THEN last_message_at END DESC
```

I thought I knew how it should be done. But I stopped, looked it up and it turns out I was wrong. This is how you do it. I was humbled.

Giving feedback is a two way relationship. You're not just imparting knowledge. You're solidifying your own knowledge **and** opening up the possibility of learning something new.

### Go down the rabbit hole

Earlier, I mentioned Haskell and how its powerful type-system captivated me. I read how you could do magical things like model nulls, exceptions, IO and whole DSLs using types alone.

I read how a monad is a "programmable semicolon". And I read that a monad is like a burrito. Or a box. Or a container. Or an execution context. I also read a whole bunch of formalisms with laws and theorems and proofs.

It took bit of head bashing but functors made sense first. Then with a bit more reading and playing, applicative functors started making sense. Then monads started making sense.

No single paper, page or book provided full understanding. I had to go down a long rabbit hole with many stops and starts until it started making sense.

Each new step along the way illuminated a weak spot. I couldn't understand the proofs. So I learned to read and write rudimentary proofs, which isn't something I learned in school. I could, at least, begin reading the journal articles.

Finding your weak spots is like feeling your way around in the dark. By going down the rabbit hole, you navigate the darkness. With small steps, you find out where you went wrong. When you reverse, you find some darkness has given way. By the end of it, you've felt your way around the edges of a topic. You know what to practise next.

### Learn new languages

> A language that doesn't affect the way you think about programming is not worth knowing
> **Alan J. Perlis**

Learning new programming languages reveals weak spots.

Many programmers say you should focus on different _programming paradigms_, not languages. I agree but it's not the whole story either.

#### Learning within your paradigm

By learning new languages within the same paradigm as what you know, you will uncover what can and can't be generalised. You'll uncover weaknesses and faulty assumptions.

For example, if you only know JavaScript, you might expect that every array type has a property or method called `length`:

```javascript
[1, 2, 3].length === 3
```

And we find that it's true in Ruby:

```ruby
[1, 2, 3].length == 3
```

And in Java:

```java
int[] arr = {1, 2, 3}
arr.length == 3
```

But it isn't true in PHP:

```php
count([1, 2, 3]) === 3
```

Nor Python:

```python
len([1, 2, 3]) == 3
```

I'll admit, these examples are small potatoes. But you could go up and down the tree of concepts. You would find similarities and differences every step of the way. All the way up to module systems, package management and the ecosystem as a whole.

Each could be a hidden weak spot if you knew only a single language. If you only
knew PHP, you would never know that namespaces and autoloading is a poor
substitute for a module system. If you only knew Python, you'd never know module systems
can be much worse.

But, eventually, you should leave the comfort of your paradigm.

#### Venturing out of your paradigm

The biggest differences _do_ lie between paradigms.

My stories around monads and Scala are an example. I had never programmed that way for work before. I had tried to write code in a functional style before. I'd tinkered many times. But I'd never done _real work_ with type-safe effect systems.

So learn Clojure. Learn how powerful data-oriented programming is, with a powerful suite of persistent data structures, a great, functional standard library and a language built with immutability and concurrency at heart. Learn how all that makes things tougher unless you go all in on the whole stack.

And learn Prolog. Learn how to solve problems using rules, relationships and queries over for-loops and mutation of state. But also learn that searching a solution space isn't always the right tool for the job. Or maybe learn that performance isn't the be all and end all when it just _works_.

And learn Ruby or Smalltalk. Learn how everything being an object brings about a style of programming that just is not possible in other object-oriented languages like PHP, Java and C#. But also learn that pure object-oriented programming without good static type checking can lead to proliferation of null-pointer exceptions.

But, most importantly, keep in mind how it will improve your skills day to day. What fundamental "truths" of programming have you annihilated by experiencing this paradigm? How will this change how you program? And what hidden weaknesses has this paradigm uncovered?

### Learn new technologies

When learning Scala and ZIO, I had to learn the JVM. I had to learn how to work with the ecosystem, how to deploy code and run it in production.

Until that point, this was a blindspot. Again, I had no idea what I was doing.

Deploying a JAR with Docker is simple enough. Much simpler than dockerising and deploying a Rails or Laravel app.

Other things were trickier, like learning the SBT build tool, its plugin ecosystem and its DSL. The Maven repository was weird compared to what I'd used before.

But after a month or two, I got the hang of it and I had a better idea of what I didn't know. My confidence and skill set expanded.

Learning new technologies is a wonderful way of finding your weak spots. And it goes hand in hand with learning new languages. That's because learning to _use_ a language is not the same as learning a language. It often means learning new technologies, such as:

- Build tools
- Testing frameworks
- Linters
- Deployment
- Database migrations
- Database abstraction layers

This is deeper than reading and writing code in the language.

### Use StackOverflow

I'm going to take an educated guess: StackOverflow is a key part of your toolkit. To save time, you sometimes use it when you could work out the solution yourself. I do too!

It's great for giving back by answering questions too (I don't do this often enough).

But it's also a gold mine for identifying your weaknesses:

1. Look up a topic that you want to assess
2. Can you answer more than 80-90% of the questions?
3. If yes, you're on track
4. If not, you have some learning to do

This technique does take a bit of nuance though. Looking at the tag for TypeScript, there are many questions about Angular. If you've never used Angular and have no intention of using it, these questions tell you nothing. Refine things down. Exclude tags that you're not interested in. As always, a weakness is only a weakness if it's getting in your way.

### Communicating with non-developers

> If you cannot explain something in simple terms, you don't understand it.
> **Richard Feynman**

Over the years, I've had to explain technical topics to my non-technical boss to explain why I'm doing what I'm doing. It took _a lot_ of practice.

At first, his eyes glazed over. And they still do when I don't understand a topic well enough.

But I can explain the topics I _do_ understand well. When I can do that, we're able to talk about what I'm working on, even if it's deeply technical. It is a valuable skill.

Whenever I cannot explain a topic well, it's a sign I need to _learn more_. Once I can explain it to my non-technical boss, I know I've got a handle on it.

Non-technical people dislike things getting technical, surprise surprise. If you get too technical, some will outright refuse to absorb what you're saying. Their eyes, like my boss's, glaze over.

And if you can explain a technical topic to someone in non-technical terms, it's a good sign you have a grasp on things.

But, if not, it _usually_ means you have more learning to do.

## Get started

These techniques turn your unknown-unknowns into known-unknowns by shining a light on your weak spots. Knowing what you don't know is half the battle. The other half involves focus, practice and more practice.

And it doesn't just apply to software development.

Stack Exchange has a site for everything, professional or not. There's one for server admin. There are others for _home brewing_, physical fitness and world building.

Instead of going down the type system rabbit hole, you can go down a rabbit hole with maths, knitting or anything at all.

Instead of accepting and giving code reviews, you could accept and give feedback on sketches, paintings or creative writing.

So what next?

Find your weak spots. Decide which will change your game. And then focus and practice and practice even more.

_Thank you to all the wise and wonderful people that took the time to give me
advice in my Ask HN post five years ago. Your wisdom has improved my life,
professional and otherwise. I now hope to pay it forward and share it as widely
as I can._
