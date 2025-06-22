---
title: "Design and Theory"
---

> I love diagrams and pictures. I think you have those on your blog? Maybe we
> can make a simple version here.

## Core components

Primarily, `mgmt` consists of two main parts: the `engine` and the `language`.

**The Engine** manages your system
[resources](https://mgmtconfig.com/docs/resources/) (files, packages, services)
by running them as a connected
[graph](https://en.wikipedia.org/wiki/Directed_acyclic_graph). It understands
which tasks depend on others and can run independent tasks simultaneously.

**The Language** is how you tell mgmt what you want your system to look like.
You write `.mcl` files that describe your desired state, and the language
converts this into instructions the engine can execute.

The clever part: when you change your `.mcl` files, mgmt smoothly transitions
from your old configuration to the new one without stopping.

## What's unique about the engine?

### Parallel graph execution:

Resources run simultaneously instead of waiting for non-dependent resources. The
programs you build are much more akin to real "software", rather than
"application of configuration".

### Event-driven execution:

In addition to being able to check state and apply changes to a running system,
`mgmt` has an event mechanism to watch (without polling) for changes per
resource. This leverages inotify, dbus, and other types of user and kernel space
events to detect drift, and avoid the need for the redundant, constant
re-checking that is the norm with the currently available legacy tooling.

### Distributed topology:

Unlike traditional configuration management tools that use a client-server
model, `mgmt` operates as a true distributed system. Here's how it works:

The `mgmt` tool differs significantly from traditional client-server or
orchestrator topologies. We instead, built it to work as a more general
distributed system, which offers many compelling benefits.

- **Traditional approach:** Central server → Agent pulls config → Agent applies
  changes
- **mgmt approach:** Distributed consensus (etcd) ← → mgmt agents collaborate

The most common implementation is one in which you first run an `etcd` cluster,
and then run an `mgmt` agent on each machine which you'd like to manage
resources from. Each agent connects to one or more of the `etcd` peers.

There are also facilties available for running `mgmt` in an agentless mode, over
`SSH`, and even in standalone mode. No central control server is ever used for
coordinating operations. This does not mean that you can't model centralized
control over your desired states.

This architecture eliminates single points of failure and enables true
peer-to-peer coordination.

## What's unique about the language?

### Safe:

The language ([DSL](https://en.wikipedia.org/wiki/Domain-specific_language)) is
designed to be incredibly safe. This is because you don't want something like an
off-by-one error in an imperative language to destroy an entire data centre.

To that end, the language is `immutable`, `strongly typed`, and `functional`.

The language itself is implemented in the memory-safe language `golang`.

### Powerful:

Traditionally, DSL's lack power. This makes it difficult to build complex
systems simply. Our DSL is quite powerful because it is `reactive`, and has
special primitives for features such as `exported resources`, `send->recv`, and
more!

### Easy to reason about:

Understanding your code, particularly when you're building distributed systems,
is an important goal for a successful language. As a result, we've built a
domain-specific-language, which can compress a large amount of functionality
into a small number of lines of code. It's very easy to express logic for state
machines and complex distributed systems.
