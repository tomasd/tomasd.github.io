---
layout: post
title: An introduction to distributed systems by Kyle Kingsbury
description: "Notes on outline"
category:
tags: [distributed-system]
---

{{ page.title }}
================

<p class="meta">12 Aug 2016</p>

I've found very interesting lecture notes on distributed systems written by
Kyle Kingsbury https://github.com/aphyr/distsys-class.

Some of the best points:

* TCP works. Use it.
* Clocks
  * Wall Clocks
      In theory, the operating system clock gives you a partial order on system events
    Caveat: NTP is probably not as good as you think
    Caveat: Definitely not well-synced between nodes
    Caveat: Hardware can drift
    Caveat: By centuries
    Caveat: POSIX time is not monotonic by definition
    Caveat: The timescales you want to measure may not be attainable
    Caveat: Threads can sleep
    Caveat: Runtimes can sleep
    Caveat: OS's can sleep
    Caveat: "Hardware" can sleep
    Just don't.
  * Lamport Clocks
    * One clock per process
    * Increments monotonically with each state transition: t' = t + 1
    * Included with every message sent
    * t' = max(t, t_msg + 1)
  * Vector Clocks
    * Generalizes Lamport clocks to a vector of all process clocks
    * t_i' = max(t_i, t_msg_i)
    * For every operation, increment that process' clock in the vector
* Availability
  * Total availability - every operation succeeds
  * Sticky availability - Every operation against a non-failing node succeeds
  * High availability - tolerant of up to f failures, but no more
  * Majority available - Operations succeed if they occur on a node which can communicate with a majority of the cluster
* Consistency
  * A consistency model is the set of "safe" histories of events in the system
  * Tradeoffs
    *  Ideally, we want total availability and linearizability
    *  Consistency requires coordination
      *  If every order is allowed, we don't need to do any work!
      *  If we want to disallow some orders of events, we have to exchange messages
    *  Coordinating comes (generally) with costs
      * More consistency is slower
      * More consistency is more intuitive
      * More consistency is less available
* CRDT
  * Associative: m(x1, m(x2, x3)) = m(m(x1, x2), x3)
  * Commutative: m(x1, x2) = m(x2, x1)
  * Idempotent: m(x1, x1) = m(x1)

* Patterns
  * Rule 1: don't distribute where you don't have to
  * Use an existing distributed system
    * If we have to distribute, can we push the work onto some other software?
      *  distributed database or log
      * Can we pay Amazon to do this for us?
  * Never fail
    * Buy really expensive hardware
  * Accept failure
    * What's our SLA anyway?
    * Can we recover by hand?
    * Can we pay someone to fix it?
    * Could insurance cover the damage?
    * Could we just call the customer and apologize?

  * Immutable values
    * Data that never changes is trivial to store
    * Never requires coordination
    * Cheap replication and recovery
    * Minimal repacking on disk
    * Cassandra, Riak, any LSM-tree DB, Kafka
  * Mutable identities
    * Pointers to immutable values
    * Pointers are small! Only metadata!
    * Can fit huge numbers of pointers on a small DB
    * Good candidate for consensus services or relational DBs
* Services for domain models
    * Divide your system into logical services for discete parts of the domain model
    * OO approach: each noun is a service
      * User service
      * Video service
      * Index service
    * Functional approach: each verb is a service
      * Auth service
      * Search service
      * Dispatch/routing service
    * Most big systems I know of use a hybrid
      * Services for nouns is a good way to enforce datatype invariants
      * Services for verbs is a good way to enforce transformation invariants
      * So have a basic User service, which is used by an Auth service
* Structure Follows Social Spaces
  * Services can be libraries
  * Initially, all your services should be libraries
  * Perfectly OK to depend on a user library in multiple services
  * Libraries with well-defined boundaries are easy to extract into services later

When possible, try to use a single node instead of a distributed system. Accept that some failures are unavoidable: SLAs and apologies can be cost-effective. To handle catastrophic failure, we use backups. To improve reliability, we introduce redundancy. To scale to large problems, we divide the problem into shards. Immutable values are easy to store and cache, and can be referenced by mutable identies, allowing us to build strongly consistent systems at large scale. As software grows, different components must scale independently, and we break out libraries into distinct services. Service structure goes hand-in-hand with teams.


* Production
  * Test everything
    * Quick example-based tests that run in a few seconds
    * More thorough property-based tests that can run overnight
    * Be able to simulate an entire cluster in-process
    * Control concurrent interleavings with simulated networks
    * Automated hardware faults
  * Instrument everything
    * we need a way to understand what the system is doing in prod
    * good monitoring is like continuous testing, but not a replacement: these are distinct domains
    * Instrumentation should be tightly coupled to the app
      * Measure only what matters
        * Responding to requests is important
        * Node CPU doesn't matter as much
        * Key metrics for most systems
          * Apdex: successful response WITHIN latency SLA
          * Latency profiles: 0, 0.5, 0.95, 0.99, 1
          * Overall throughput
          * Queue statistics
          * Subjective experience of other systems latency/throughput
            * The DB might think it's healthy, but clients could see it as slow
            * Combinatorial explosion--best to use this when drilling into a failure
          * You probably have to write this instrumentation yourself
            * Invest in a metrics library
    * Out-of-the-box monitoring usually doesn't measure what really matters: your app's behavior
      * But it can be really useful in tracking down causes of problems
      * Host metrics like CPU, disk, etc
  * Versioning
    * Do include a version tag with all messages
    * Do include compatibility logic
    * Inform clients when their request can't be honored
      * And instrument this so you know which systems have to be upgraded
  * Rollouts
    * Rollouts are often how you fix problems
  * Feature flags
    * When a feature is problematic, disable it
    * Use a highly available coordination service to decide which codepaths to enable
      * This service should have minimal dependencies
        * Don't use the primary DB
    * When things go wrong, you can tune the system's behavior
      * When coordination service is down, fail safe!
  * Queues
    * No node has unbounded memory. Your queues must be bounded
    * But how big? Nobody knows
    * Instrument your queues in prod to find out
    * Queues exist to smooth out fluctuations in load
      * If your load is higher than capacity, no queue will save you
      * Instrument queue depths
