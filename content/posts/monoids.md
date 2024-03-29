---
title: "A design pattern for composition"
draft: true
---

Composition is the beating heart of software design.

From the beginning of our careers, we learn the key to good software design is thoughtful decomposition of problems, composing the solutions into a single, well-designed system.

And that's where monoids come in.

A monoid is a dead simple concept with a scary sounding name.

We use it everyday, without thinking.

And by using it consciously, it becomes a design pattern for composable units of code at multiple layers of abstraction.


## What is a monoid?

In a nutshell, a monoid is a protocol. For a thing to be a monoid, it must have:

- An associative operation, function or method that returns the same type as its inputs.
- An implementation or value that is 'null' or 'zero' in terms of the above, called the "identity element".

### Associativity

Associativity is a rule or a law about certain two-argument functions. For a two-arg function to be associative, we can rearrange the parentheses without changing the result. For example:

```js
(1 + 1) + 1 === 1 + (1 + 1)
```

### The identity element

An identity element is any value that is a no-operation when used with a function. For example, zero is the identity element when considering addition over integers. Or one for multiplication. Or the empty string for string concatentation.

## Simple examples

Monoids pervade every day programming. You're already using monoids when you:

- Add or multiply
- Concatenate strings
- Compound two Boolean expressions
- Merge hash tables
- Take the union of two sets

### Adding and multiplying integers

The `+` and `*` operators form a monoid over integers.

For `+`, `0` is the null value. For `*`, it's `1`.

Addition and multiplication are associative:

```ts
1 + 2 + 3 === 1 + (2 + 3)
1 * 2 * 3 === 1 * (2 * 3)
```

A pattern forms when we juxtapose operations on the identity element, showing how it behaves like a noop:

```ts
10 * 1 == 10
1 * 10 == 10
10 + 0 == 10
0 + 10 == 10
```

We know from early lessons on arithmetic that we can chain together, and simplify, addition and multiplication expressions:

```js
0 + 1 + 1 === 1 + 2 === 3
1 * 2 * 3 === 1 * 6 === 6
```

This is perhaps the simplest example of a monoid.

### Concatenating strings

String concatenation forms a monoid too.

Concatenation is both associative and closed over the string type:

```ts
("a" + "b") + "c" === "abc"
"a" + ("b" + "c") === "abc"
```

Our null value is the empty string: `""`:

```ts
"a" + "" === "a"
"" + "a" === "a"
```

This guarantees we can decompose strings into smaller strings. And compose them together to form larger strings. 

### Boolean OR/AND

There's a natural symmetry between the OR/AND operators and addition/multiplication.

It's both associative and closed over the Boolean type:

```python
a and (b and c) == (a and b) and c

a or (b or c) == (a or b) or c
```

And both have an identity element:

```python
assert (a or False) == (False or a)

assert (a and True) == (True and a)
```

Booleans form monoids too. The symmetry with addition and multiplication is striking.

### Merging hash tables

A function that merges two hash tables forms a monoid.

Associative? Check. Closed over hash tables? Check.

```ruby
x = {a: 1}.merge({b: 2}).merge({c: 3})
y = {a: 1}.merge({b: 2}.merge({c: 3})

x == y # => true
```

An empty hash table becomes our identity element:

```ruby
{}.merge({a: 1}) == {a: 1}
{a: 1}.merge({}) == {a: 1}
```

But another identity element exists too:

```ruby
{a: 1}.merge({a: 1}) == {a: 1}
```

Merging a hash table with itself always produces the same hash table. This too can be considered an identity element.


### Set Union / Intersection

The set union operator forms a monoid over sets, with the empty set as an element.

```python
a = {1, 2}
b = {2, 3}
c = {3, 4}

# Associativity
assert a | (b | c) == (a | b) | c

# Identity element
assert a | set() == set() | a
```

Intersection is an interesting case. The identity element is the "universal set". We can use `union` to create a pretend "universal set" out of the sets we're using.

```python
a = {1, 2}
b = {2, 3}
c = {3, 4}
u = a | b | c

assert a & (b & c) == (a & b) & c
```

There's that symmetry again.

## Real-world examples

So how do we use monoids to design composable software?

Let's examine real-world problems that come up when building commercial systems.

Each example uses a language where the type system cannot represent a useful, general monoid type.

Instead, we can take [Peter Norvig's excellent presentation](https://norvig.com/design-patterns/) and do the reverse: use design patterns to fill in for missing language features.

In doing so, we end up with composable designs expressed in terms that the language and its type system can support.

We examine each of these real-world examples:

1. The pure command value pattern
5. Role-based access control
7. The composite pattern
8. The decorator pattern

### The Command Pattern

Over the past few months, me and a colleague have been iterating on a SaaS product, working from a legacy codebase.

Through careful iteration and refactoring, a CQRS architecture began to emerge naturally.

Our initial iteration of command objects bundled the computation up with the command as a value.

```ts
class RegisterPatient {
  repository: PatientRepository;

  execute(): void {
    // ... do work in repository ...
  }
}
```

This worked well at first. But it soon became cumbersome to serialise when paired with dependency injection. 

So, instead, we decided to treat a command as an immutable value object, decoupling it from the handler, which executes the command.

```ts
interface RegisterPatient {
  // ... attributes ...
}

class RegisterPatientReceiver {
  repository: PatientRepository;

  execute(cmd: RegisterPatient): void {
    // ... execute command ...
  }
}
```

Problem solved - it was now far easier to transport the command.

But that wasn't the only benefit.

#### What a command value enables

By enforcing purity, we gain a number of advantages right out the gate.

It's now possible to express logic that determines the side-effect to perform, without actually performing it.

That makes it ridiculously easy to test and debug.

But what else can it do?

Although redundant in typescript due to structural typing, we are talking about a general `Cmd` type:

```ts
interface Cmd
{
}
```

Since `Cmd` types are values, a set of commands may contain commands themselves:

```ts
interface ChainCmd extends Cmd {
  head: Cmd;
  next: Cmd;
}
```

Or a collection of commands:

```ts
interface SeqCmd extends Cmd {
  sequence: Cmd[];
}
```

We can also implement a command that does nothing:

```ts
interface NullCmd extends Cmd {
}
```

We can also write pure functions that transform one command into one or more commands:

```ts
const seqToChain => (cmd: SeqCmd): ChainCmd {
  // ... transform ...
}
```

Since we decoupled side-effects from the command itself, we can use simple values to present individual computations.

By using general glue values, we can compose smaller units of computation into larger, more complex values.

Thanks to the `NullCmd`, our command values have the concept of zero.

#### Where's the monoid?

Sharp readers will notice that we already have all the ingredients for our first, real-world monoid.

To bring it together, we just need a binary function that takes two commands and produces a new command.

We could write a `sequence` function, that represents a set of commands that are performed asynchronously but one after the other:

```ts
const sequence = (...cmds: Cmd): SeqCmd => {
  // build sequence
}
```

We can compose any command using `sequence` and the rules hold:

```ts
// show how sequence composes
// give example output
```

That's great but it's a bit noisy. Since we know that all sequences must run in order, we can flatten our `SeqCmd`.

```ts
// show how we can flatten seq commands
```

In the same light, we can define a command that represents commands that can be performed in parallel:

```ts
interface ParallelCmd extends Cmd {
  commands: Cmd[]
}
```

And a simple parallel function, including the ability to flatten:

```ts
const parallel = (...cmds: Cmd): ParallelCmd => {
  // convert cmds to parallel cmd
}
```

Not only that, we can now use these functions together to design a simple notation for describing workflows of commands that can still be executed asynchronously:

```ts
sequence(
  createContact(...),
  parallel(
    createNote(...),
    createOpportunity(...)
  )
);
```

Inspired by the `Promise` type, we can extend our types to define a more readable API:

```ts
sequence()
  .then(createContact(...))
  .then(parallel(
    createNote(...),
    createOpportunity(...)
  ));
```

But I think the functional interface is less hassle.

Although the design and implementation is naive, it's not hard to see how we can design a pattern language to choreograph background workloads.

### Role-based access control

In many applications, we need to restrict functionality users have access to based on their job role(s).

For example, in a CRM application, you want your sales team to have full access to leads. But you don't want them changing automation rules or business configuration.

One simple solution is role-based access control.

With RBAC, each user has a set of roles. Each role has a set of policies. Both of these sets may be empty.

For brevity, this example does not consider explicit denial, using deny-all by default.

#### Understanding the data

[include a simple box and arrow diagram]

By understanding the data involved, we reduce the problem down to these types:

- Sets
- Users
- Roles
- Policies

Starting with the root of the tree, our user type looks like this:

```python
class User:
    roles: Set[Role]
```

And roles look like this:

```python
class Role:
    policies: Set[Policy]
```

And policies look like this:

```python
class Policy:
    key: str
    name: str
```

In both users and roles, we use a generic set type.

#### Understanding the operations

So how do we test if a user is permitted? There's a few ways to do it:

1. We can compose the policies of each role using a union operation and test that.
1. Or we delegate the test to each role, using `or` to compose results.

With the former, we expose that each role has a set of policies. With the second, this knowledge is hidden, exposing an interface that allows us to test each role.

Let's stub out the API:

```Python
class User:
    roles: Set[Role]

    def can(self, policy: Policy) -> bool:
        pass

class Role:
    roles: Set[Policy]

    def can(self, policy: Policy) -> bool:
        pass

class Policy:
    key: str
    name: str
```

#### Designing with monoids

Starting with roles, we can implement the `can` method using `or`, with `False` as the identity element, over the set of policies:

```python
class Role:
    def can(self, policy: Policy) -> bool:
        pass
```

We can implement the test on users in a similar way:

```python
class User:
    roles: Set[Role]

    def can(self, policy: Policy) -> bool:
        pass
```

What if we wanted to list a users set of policies?

We can use another monoid - union over sets:

```python
class User:
    @property
    def policies(self): -> Set[Policy]
        pass
```

We considered this as an initial solution but it wasn't necessary to implement the test operation.

But in any case, we have designed a composable access control API using monoids.

Because of this, the implementation is simple, easy to understand and it's easy to test.

And, since we used simple data structures, our data is easy to persist.

#### Persisting access control rules

Set theory forms the basis of relational databases. Since a number of set operations form a monoid, we can physically model our rules of composition.

Let's define some base tables:

```sql
CREATE TABLE user (
  name char
);

CREATE TABLE role (
  name char
);

CREATE TABLE policy (
  name char
);
```

### The composite pattern

> Compose objects into tree structures to represent part-whole hierarchies. Composite lets clients treat individual objects and compositions of objects uniformly.
>
> <cite>Gamma, Helm, Johnson, Vlissides</cite>

- What is the composite pattern?
  - Include a diagram of the composite pattern
  - Quote Gang of Four
  - The composite pattern describes a way to compose a set of values into a single tree-shaped value.
  - We can then do work on the composite as if it was an individual component.
  - Sounds a lot like a design pattern for composition.
  - What's the definition from GoF?
- What problem does it solve?
  - Whenever we need to operate on a set of  values of the same type as a single unit, the composite pattern applies.
  - Especially useful for grouped UI components
  - Or for grouping shapes in a vector drawing application.
  - The composite pattern is useful for anything that requires the composition of values in a tree-like form.
  - React, as a framework, provides an entire UI framework this way. Each Component has a render function. To render a UI, React renders each sub-component in turn.
  - Likewise, Redux suggests decomposing reducers into individual units, providing a simple helper to compose a tree of reducers into a single reducer.
- What does an example implementation look like?
  - Choose a suitable problem
  - Design a possible solution
  - Show example code
- Where are the monoids?
  - The composite pattern forms an inherent monoid.
  - How does it form a monoid?
  - Why is this important?
  - What does this enable that isn't typically mandated by the composite pattern?
  - What are some examples use cases?

[insert credit to source on monoids and composite pattern]

### The decorator pattern

> Attach additional responsibilities to an object dynamically. Decorators provide a flexible alternative to subclassing for extending functionality
>
> <cite>Gamma, Helm, Johnson, Vlissides</cite>

- What is the decorator pattern?
  - Include a diagram
  - Include a quote from GoF
  - Allows behaviour to be added to an object without changing the behavior of other objects of the same type
  - Instead, functionality can be decomposed into decoupled layers of decorators.
  - These layers can be composed into a single object as if a single class does all of the work.
- What problem does it solve?
  - Removes the need and limitations of complex inheritance hierarchies to extend objects with additional behavior
  - In Java land, this is done via wrapper classes, that accept the object to decorate
  - But in other languages, this extends to other first class constructs, like functions and methods.
  - Indeed, Python provides direct language support for decorators that may wrap both classes and functions.
  - In functional languages, a decorator would simply be a higher order function that accepts a function and returns a new function.
  - And in React, the idea of Higher-order Components provides a pattern to decorate React components with new behavior dynamically.
- What does an example look like?
  - Choose a suitable problem
  - Design it using decorators
- Where are the monoids?
  - Like the composite pattern, decorators form an inherent monoid.
  - How do they form a monoid?
  - Why is this important?
  - What does this enable to enhance the decorator pattern?

