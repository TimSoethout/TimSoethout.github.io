---
title: "Leverage Domain Knowledge for Faster Distributed Transactions"
layout: post
category: null
tagline: null
tags:
  - distributed systems
  - atomic commit
  - scalability
  - two-phase commit
  - distributed transactions
---

{% include JB/setup %}

> Blog post about paper: Path-Sensitive Atomic Commit: Local Coordination Avoidance for Distributed Transactions @ [https://doi.org/10.22152/programming-journal.org/2021/5/](https://doi.org/10.22152/programming-journal.org/2021/5/3)

<!-- > Motivation: Abstractions, such as DSLs, can help business users to grasp IT trade-offs between performance and functional requirements. Automatically generating the implementation, picking the best-performing implementation, helps achieving this goal. -->

<!-- > TLDR: There is ample opportunity for cleverly leveraging high-level models when generating code for better performance, scalability and specialized synchronization. -->

Many tools and libraries in software try to make the work of engineers easier, both to speed up development, but also to close the gap between IT and business.
These tools provide abstractions that focus on writing business logic.
Within ING Bank this is no different. We use and create tools and abstractions that are closer to the business and abstract away implementation details: [Baker](https://medium.com/ing-blog/baker-a-microservice-orchestration-library-e2d162be3d71), [Cucumber](https://medium.com/ing-blog/cucumber-ing-making-it-part-of-the-agile-workflow-4b53926fbd6), [front-end libraries](https://medium.com/ing-blog/micro-front-end-architecture-rapid-development-in-a-startup-environment-10270dca1d5b) and [components](https://github.com/ing-bank/lion), [query creators](https://github.com/ing-bank/scruid), [cryptography primitives](https://github.com/ing-bank/zkkrypto), [security layers](https://github.com/ing-bank/rokku), our internal API SDK and a lot more.

The premise of this blog is not different. We want to describe high-level business logic without being bothered by low-level implementation details. However, creating a performant implementation of said logic is non-trivial.
This blog describes an approach on how this high-level domain knowledge encoded in a model can be used to optimize distributed transactions.
This even gives us an advantage over general purpose transaction mechanism that can not depend on this extra domain knowledge and can be used for optimizing transactions between micro-services.

<!-- This effectively splits up developers in two categories, tool users and tool developers. -->

<!-- I spend a lot of time the last years devising an algorithm that leverages semantically rich models of objects to speed up coordination between objects: Path-Sensitive Atomic Commit or Local Coordination Avoidance.  -->
In other words: We want to make transactions faster, automatically.
Our algorithm, Path-Sensitive Atomic Commit (PSAC), provides a more performant synchronization implementation for automatically generated implementations. This enables writing of high-level business logic or functional requirements, and letting the algorithm take care of performance at run time.

PSAC's main idea is to use the explicit domain knowledge to improve concurrency where safely possible, e.g. multiple concurrent withdrawals on a bank account are safe when there is enough balance available for both. However, determining this can be more expensive computationally.

Of course this algorithm's performance has to be evaluated.
Does it really perform better than a base-line implementation?
Here is a sneak preview of the performance results. PSAC performs up to 1.8 times better than 2PL/2PC in a high-contention scenario.

![Throughput of 2PL/2PC and PSAC]({{ site.url }}/assets/images/psac2pc-sync1000-1.svg)

## Background: Distributed Transactions
 
Transactions are a mechanism to limit the complexities inherent to concurrent and distributed systems, such as dealing with hardware failure, application crashes, network interruptions, multiple clients writing to same resource, reading of partial updates and data and race conditions.
ACID transactions are the standard in databases. [ACID](https://en.wikipedia.org/wiki/ACID) stands for Atomic, Consistent, Isolated and Durable. 

_Atomic Commit & Two-Phase Commit (2PC):_
[2PC](https://en.wikipedia.org/wiki/Two-phase_commit_protocol) is a well-studied atomic commitment protocol. Atomic Commit requires that multiple resources agree on a action: all should do it or non should do it. This also hold in case of failure of one of the resources.
Resources in this case can be distributed over multiple server nodes, or can even be different applications (see [XA](https://en.wikipedia.org/wiki/X/Open_XA)).

2PC works with a transaction manager and multiple transaction resources. 
The manager asks the resources to vote on an action. Only when it receives a vote commit from all, it tells all to globally commit and apply the decision. 
<!-- If any resource votes to abort, it globally aborts the transaction.  -->
<!-- When a resource votes to commit it promises to durably store and accept a later global commit. This makes sure it continue in case of failure. -->

<!-- The biggest drawback of 2PC is blocking behaviour when the transaction manager crashes between receiving votes and globally committing or aborting. Now the resources are left dangling, because they promised to wait on commit on the manager, and can not continue without it. -->

_Concurrency Control & Two-Phase Locking (2PL)_:
[2PL](https://en.wikipedia.org/wiki/Two-phase_locking) is a concurrency control mechanism that uses locking to make sure that no concurrent changes are made to a resource.

_Distributed Transactions_: 
2PL and 2PC can be combined to implement ACID distributed transactions.
The locks are on the level of the 2PC resources. When a resource has voted, it is considered locked. Only after handling a global commit or abort it is unlocked again. This makes sure no other transactions can change the data in the mean time.

## Path-sensitive Atomic Commit

### Models in [Rebel](https://github.com/cwi-swat/rebel)

Let's first look at an example of such semantically rich models.
We use [Rebel](https://github.com/cwi-swat/rebel) ([paper](https://dl.acm.org/doi/10.1145/2998407.2998413)), a domain specific language for financial products, based on state machines. The concept of leveraging model knowledge is not limited to Rebel.
Our example:
A bank account system example consisting of money transfers and accounts with balances, which should never go below 0, visualized as state charts:

![Rebel State Charts]({{ site.url }}/assets/images/progamming-state-charts.svg)

In the textual representation, we see different classes, with some internal data, representing the account balance and identities. On each of the states events are defined with pre- and postconditions, e.g. `Withdraw` is only valid when the account has enough balance available. 
The `MoneyTransfer` class has a special construct `sync` which represents an atomic synchronized event, where money is `Withdraw`n from one account and `Deposit`ed from another. Either both should happen or none.

```scala
class Account
  accountNumber: Iban @identity
  balance: Money

  initial init   
    on Open(initialDeposit: Money): opened
      pre:  initialDeposit >= €0
      post: this.balance == initialDeposit
      
  opened
    on Withdraw(amount: Money): opened
      pre:  amount > €0, balance - amount >= €0
      post: this.balance == balance - amount
    on Deposit(amount: Money): opened
      pre:  amount > €0
      post: this.balance == balance + amount
    on Close(): closed
    
  final closed

class MoneyTransfer
  initial init
    on Book(amount: Money, to: Account, from: Account): booked
      sync:
        from.Withdraw(amount)
        to.Deposit(amount)
  final booked  
```

We can see how these kinds of models can represent different business logic on a relatively high level.

### Rebel with 2PC/2PL

If we want to implement these models in a scalable systems, we can represent all instances of these objects as 2PC resources. This means that can be interacted with separately, until synchronization (using `sync`) is requested. Locally each resource does 2PL, making sure that data is not changed concurrently, and 2PC is used to coordinate the sync.

![2PL/2PC example]({{ site.url }}/assets/images/programming-PSAC-2pc.svg) | ![PSAC example]({{ site.url }}/assets/images/programming-PSAC-psac.svg)

Above on the left illustration describes what happens for such a resource (Account Entity). Vertically time is represented and the arrow represent messages send and received.

First (1) a vote request is received from a 2PC manager, preconditions are checked and the resource is locked. When another event (2) arrives it is delayed. Now when the 2PC manager signals the commit later (3), the event's effects are applied to the resource's internal state and the resource is unlocked.
Now the delayed event can start as well. 
We see that in this way all events are nicely serialized for this resource and no preconditions are done on possibly invalid (partial) state. Event do have to wait on each other in this case, which can become a problem for busy resources.

### PSAC

When looking at the account model above, the most interesting precondition is `balance - amount >= €0` of `Withdraw`, denoting that there should be enough balance available for the `Withdraw` to be allowed.
2PL only allows a single `Withdraw` to be in progress at the same time by locking the Account resource. When we naively allow multiple concurrent `Withdraw`s on an account, precondition checks could interleave each other, resulting in a balance below zero.
Enter Path-Sensitive Atomic Commit:

PSAC enables multiple concurrent events in-progress at the same time, resulting in lower latency for individual events, because of no locking and no delaying for events.

_But how does it keep that safe?:_
PSAC makes multiple concurrent `Withdraw`s safe by keeping track of all in-progress events. It effectively tracks all possible outcome states of in-progress events, and when a concurrent event arrives the preconditions can be checked against all outcomes. If preconditions hold in all states, an event can already be accepted for processing (and the 2PC commit vote send). Same for abort, if the preconditions fail in all states.
If the preconditions hold in some states, but not all, PSAC falls back to 2PL/2PC behavior and delays the events.
For our `Withdraw` example, multiple `Withdraw`s can be in progress concurrently when there is enough balance available for all.
Other examples such as `Deposit`s can also run concurrently, because adding money to an account is always allowed by its preconditions.

The PSAC diagram (above on the right) tries to explain in more detail how this works and represents the internal decisions of above sequence diagrams:
1. The `Withdraw` arrives and since there are no events in progress, the preconditions are checked against the account state of €100. Now internally there are two possible outcome states, represented by the arrows: €100, when the `Withdraw` is eventually aborted by the transaction manager, and €70 when the `Withdraw` is committed. `+` and `-` respectively representing the global commit and global abort.
2. Now when another `Withdraw` arrives the possible outcomes tree is split again for the existing possible states. 
3. When a `Withdraw` of €60 arrives, it is delayed, because in some of the outcome states its preconditions are valid and in some not.
4. As -€50 commits, the outcome tree can be pruned, and the leafs where it had aborted are cut off.
5. Now -€60 can be retried and can be directly rejected (`Fail`), because in no possible outcome state its preconditions hold.
6. When the first event commits the last open branch is pruned as well and the state can be calculated by applying the postconditions of all events in order of original arrival.

This blog post's goal is to intuitively sketch the algorithm. Please see [the paper](https://doi.org/10.22152/programming-journal.org/2021/5/3) for more details.

## Does it perform?

We implemented code generators for the Rebel specifications to both 2PL/2PC and PSAC on the Akka actor toolkit. 

Experiments using the bank account system example above are scaled over an increasing number of nodes (and Cassandra database nodes) on Amazon AWS virtual machines. In this case there are as many money transfers as possible done picked uniformly from 1000 bank accounts. This artificially increases the contention.
The graph below contains a [violin plots](https://en.wikipedia.org/wiki/Violin_plot) that show all captured throughput numbers and a fit through it. The transparent line is the linear scalability upper bound.
In this graph we see both algorithms and their throughput numbers. PSAC outperforms 2PL/2PC, which is explained by the increased concurrency.

![Throughput of 2PL/2PC and PSAC]({{ site.url }}/assets/images/psac2pc-sync1000-amdahl-plot-1.svg)

## Concluding

We thus see that there PSAC performs up to 1.8 times better than 2PL/2PC in this high-contention scenario. This promises good results in other situations with other models. We expect to improve the throughput (and latency) even more when more contention is happening, such as when a bank has to execute a lot of transactions involving a single bank account, e.g. when a tax office pays out benefits to citizens.

This shows that higher level semantically-rich models, such as Rebel, give possibilities in bridging the gap between declarative high-level models and optimized implementations. 


> This paper is part of my PhD project in the ongoing collaboration between ING Bank and Centrum Wiskunde & Informatica (CWI) on managing complexity of enterprise software ecosystems.