---
layout: post
title: State reducer
description: "Library for CQRS and command processing"
category:
tags: [java cqrs]
---

{{ page.title }}
================

<p class="meta">24 Jan 2016</p>

Recently I've read a lot about CQRS. I find it very useful, but havent found implementation details about business logic. How to process commands, how to generate events and how to apply events to the state?

In this article I want to propose formal definition of the problem.

At the beginning we have 2 sets from the business requirements

* commands
* events

and their corresponding mapping. 

Each command can generate several events based on the command parameters and current state. Command can also be rejected if can't be applied to the current state.

Application of the event to the model state generates new model state.

Commands and events should be processed in multiple steps. E.g. validate command, process command, recompute derived values, validate invariants etc...

There are 2 layers of functions. Pure and side effecting. Side effecting functions changes persistent state in the store. In application business logic is represented as 1 function. All the routing is created by functional composition.

Example
```
EsStateReducer<Person, State> sr = EsStateReducer.of(
                Dispatch.cmd(Match
                        .whenType(CreatePerson.class)
                        .then(pc(StateReducerTest::createPerson, State.class))

                        .whenType(ChangeFirstName.class)
                        .then(pc(StateReducerTest::changeFirstNameEvent))),
                Dispatch.event(Match
                        .whenType(PersonCreated.class)
                        .then(p(StateReducerTest::personCreated, State.class))

                        .whenType(FirstNameChanged.class)
                        .then(p(StateReducerTest::firstNameChanged))),
                1);
sr.apply(eventStore, ctx, 1, new CreatePerson("Jane", "Doe"))
sr.apply(eventStore, ctx, 1, new ChangeFirstName("John"));
```

Implementation is on my github https://github.com/tomasd/state-reducer

## Command function

One command can be processed in multiple steps, each generating 0..n events. Each step should introspect initial and current state.

Header:
f(command, initialState, currentState, ctx, player) -> tuple(events, newState)

As this function is very complex we can use several command adapters which can be used in different situations. Wrapper internally calls provided function and replays generated events on current state. It uses player for replaying events. Player is event function described lower.

### Process command function

f(command, currentState) -> events

Processes command in context of actual state. Useful in:

* command validation
* command process

### Invariant command function

f(currentState) -> events

Processes actual state. Useful in:

* generating derived values
* validation of state invariants

### Thread command function

f(commandFunction1, commandFunction2, ...commandFunctionN)
 
Represent multiple command functions as 1.

### Dispatcher command function

f(
   predicate1, commandFunction1,
   predicate2, commandFunction2
   .
   .
   predicateN, commandFunctionN
   )

Business logic should be represented as 1 function. As there are multiple commands, they need to be processes separately.


## Event function

Generated events should be processed. Event function changes current state. Event can be processed in multiple steps, each advancing state into new one.

Header:
f(event, initialState, currentState, ctx) -> new state

This function is also complex, so we can use adapters.

### Process event function

Process event in context of current state.

f(event, state) -> new state

### Invariant event function

Validate or recalculate derived values from current state.

f(state) -> new state

### Thread event function

Represent multiple event functions as 1.

f(eventFunction1, eventFunction2,...eventFunctionN) -> new state

### Dispatcher event function

Dispatch events to multiple functions.

f(
   predicate1, eventFunction1,
   predicate2, eventFunction2
   .
   .
   predicateN, eventFunctionN
   )
   
   
## State reducer

State reducer is side effecting function applying command, generating and storing events into event store.

Header:
f(eventStore, ctx, id, command) -> new state

