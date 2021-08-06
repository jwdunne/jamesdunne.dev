---
title: "Finite-state automata"
date: 2020-09-13T14:48:49+01:00
draft: true
---

- What is a finite state automata?
- What are the typical problems that arise with them?
- What are the existing solutions?
- How can we design simple, composable finite-state automata?

FSA are an age-old technique for representing complex state change logic.

And they too find use *everywhere*, including:

- HTTP routing
- Regular expressions
- Promises
- UI state
- Instant message status

#### What is a finite state automata

At its heart, a finite state automata is composed of these things:

1. Inputs
2. States
3. The initial state
4. A state transition table
5. Optional final states

As a working example, let's define a simple FSA that represents the status of an entity in a write model.

Our entities can be in one of the following states:

- **Transient** when the entity is ephemeral, in-memory
- **Invalid** when the entity's validation rules fail
- **Persisting** when the entity is in the process of writing to storage
- **Erroring** when the entity fails to persist
- **Persisted** when the entity is now permanent in its current form

With **Transient** as the initial state, our FSA accepts inputs:

- **Invalidate** when the entity fails validation rules
- **Persist** when the entity is ready to persist
- **Fail** when the entity fails to persist
- **Done** when the entity is persisted

Bringing these together, we can define simple table of rules that define transitions:

```ts
const enum State {
  TRANSIENT = 'transient',
  INVALID = 'invalid',
  PERSISTING = 'persisting',
  ERRORED = 'errored',
  PERSISTED = 'persisted'
}

const enum Input {
  INVALIDATE = 'invalidate',
  PERSIST = 'persist',
  FAIL = 'fail',
  DONE = 'done'
};

const INITIAL = State.TRANSIENT;

const Transitions = {
  transient: {
    invalidate: State.INVALID,
    persist: State.PERSISTING
  },

  persisting: {
    fail: State.ERRORED,
    done: State.PERSISTED
  }
}
```

This, in a nutshell, models the states an entity can be in and how they get that way as a value.

We can define a simple transition function for this FSA:

```
const next = (state: State, input: Input): State => {
  // ... calculate next state
}
```

If you've encountered state machines before, these simple values lie at the heart of them.

For an FSA to do something useful, we need to read from it. We can use a finite-state transducer to do just that.

A finite-state transducer is a finite-state automata with the addition of:

- Outputs
- An output table

One set of outputs could be an event to publish, for example:

- `EntityPersisted`
- `EntityErrored`

With an output function, where we assume `entity` is within scope:

```
interface EntityEvent {
  kind: 'persisted' | 'errored';
  entity: Entity;
}

// assuming a Contact entity

const event = (state: State): EntityEvent => {
  // ... convert state to event
}
```
