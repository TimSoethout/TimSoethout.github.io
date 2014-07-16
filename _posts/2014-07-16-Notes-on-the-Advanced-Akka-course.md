---
published: false
---

---
layout: post
title: "Notes on the Advanced Akka course"
date: 2014-07-15 12:00
comments: true
categories: Programming, Scala
tags:
  - Scala
  - Akka
---
The Advanced Akka course is provided by Typesafe and is aimed at teaching advanced usages of Akka. The course covers the basics of Akka, Remoting, Clustering, Routers, CRDTs, Cluster Sharding and Akka Persistance. The following post starts with a general introduction to Akka and presents the takeaways from the course as we experienced them.

## A general overview of Akka

The reader which is already familiar with Akka can skip this section.

According to the Akka site this is Akka:

> Akka is a toolkit and runtime for building highly 
> concurrent, distributed, and fault tolerant event-driven
> applications on the JVM.

Akka achieves this by using Actors.

> Actors are very lightweight concurrent entities. 

Each Actor has a corresponding mailbox stored separately from the Actor. The Actors together with their mailboxes reside in an ActorSystem. Additionally, the ActorSystem contains the Dispatcher which executes the handling of a message by an actor. Each Actor only handles a single message at a time.

In Akka everything is remote by design and philosophy. In practice this means that each Actor is identified by its `ActorRef`. This is a reference to the actor which provides *Location Transparency*.

Actors communicate with each other by sending messages to an another Actor through an `ActorRef`. This sending of the message takes virtually no time.

In addition to `ActorRef` there exists also an `ActorSelection` which contains a path to one or more actors. Upon each sending of the message the path is traversed until the actor is found or when not. No message is send back when the actor is not found however.

States: Started - Stopped - Terminated
If an actor enters the `Stopped` state it first stops its child actors before entering the `Terminated` state.

### Best-practices

Import the `context.dispatcher` instead of the global Scala ExecutionContext. It is the ExecutionContext managed by Akka. Using the global context causes the Actors to be run in the global Thread pool.

You should not use `PoisonPill` as it will be removed from future versions of Akka since it is not specific enough. Roll your own message to make sure the appropriate actions for graceful shutdown are done. Use `context.stop` to stop your actor.

Place your business logic in a separate trait and mix it in to the actor. This increases testability as you can easily unit test the trait containing the business logic. Also, you should put the creation of any child actors inside a separate method so the creation can be overridden from tests.

## Remoting
With the Remoting extension it is possible to communicate with other Actor Systems. This communication is often done through `ActorSelection`s instead of `ActorRef`.

Remoting uses Java serialisation by default which is slow and fragile in light of changing definitions. It is possible and recommended to use another mechanism such as Google Protobuf.

## Clustering
Akka has a simple perspective on cluster management with regards to split-brain scenarios. Nodes become dead when they are observed as dead and they cannot resurrect. The only way a node can come up again is if it registers itself again.

When a net split happens the other nodes are marked as *unreachable*. When using a Singleton, this means that only the nodes that can reach the singleton will access it. The others will not decide on a new Singleton in order to prevent a split-brain scenario.

Another measure against split-brain is contacting the seed nodes in order. The first seed node is required to be up.

The seed nodes are tried in order.

## FSM
There is an library for writing finite state machines called FSM. For larger actors it can be useful to use the FSM. Otherwise stick to pure `become` and `unbecome`.

FSM also has an interval timer for scheduling messages. However, the use of `stay()` resets the interval timer therefore you could have issues with never executing what is at the end of the timer.

## Routers

There are two different kinds of routers: Pools and Groups. Pools are in charge of their own children and they are created and killed by the pool. Groups are configured with an `ActorSelection` that defines the actors to which the group should sent its messages. There are several implementations: Consistent Hash, Random, Round Robin, BroadCast, Scatter - Gather First, and Smallest Mailbox. The names are self-explanatory.

## Synchronisation of data with CRDTs
Synchronising data between multiple nodes can be done by choosing your datatype so that If the timestamps and events are generated in one place no duplicate entries occur. Therefore merging a map from a different node in your map is easily done by copying entries you don't already have to your own data.

This can be implemented by letting each member node broadcast which data-points they have. Each node can then detect which information is lacking and request the specific data from the node that claimed to have the data. At some future point in time all nodes will be in sync. This is called *eventual consistency*.

## Singleton
If you have a singleton cluster manager proxy it only starts when the cluster is formed. A cluster is formed if a member connects. The proxy will then pass on the buffered messages.

## Cluster Sharding
Sharding is a way to split up a group of actors in a cluster. This can be useful if the group is too large to fit in the memory of a single machine. The Cluster Sharding feature takes care of the partitioning of the actors using a hash you have to define with a function `shardResolver`. The sharded actors can be messaged with an unique identifier using `ClusterSharding(system).shardRegion("Counter")` which proxies the message to the correct actor.
`ClusterSharding.start` is what the Manager is to Singletons.

It is recommended to put the sharding functions into a singleton object for easy re-use of your shards, containing the functions to start the sharding extension and proxy to the shard etc. It is also convenient to adds `tell` and `initialise` helper functions to respectively send a message and initialise the actor by its unique id.

## Akka Persistence

Akka persistence uses a Journal to store which messages were processed. One of the supported storage mechanisms is Cassandra. It is also possible to use a file-based journal which, of course, is not recommended.

In the current version of Akka there are two approaches to persistence: command sourcing and event sourcing. Simply but, in command storing each message is first persisted and then offered to the actor to do as it pleases whereas in event sourcing only the results of actions are persisted. The latter is preferred and will be the only remaining method in following versions.

Both methods support storing a snapshot of the current state and recovering from it.

### Command Sourcing
The main problem with command sourcing lies in that *all* messages are replayed. This includes requests for information from dead actors which wastes resources for nothing. Moreover, in case of errors, the last message that killed the actor is also replayed and probably killing the actor again in the proces.


### Event Sourcing

With event sourcing one only stores state changing events. Events are received by the `receiveRecover` method. *External* side-effects should be performed in the `receive` method. The code for the internal side-effect of the event should be the same in both the `receive` and `receiveRecover` methods. The actor or trait for this will be named `PersistentActor`. 

### Actor offloading

One can use Akka Persistence to "pause" long living actors, e.g. actors that have seen no activity lately. This frees up memory. When the actor is needed again it can be safely restored from the persistence layer.

## Tidbits

Akka 3 is to be released "not super soon". It will contain typed actors. The consequence of this is that the sender field will be removed from the actor. Therefore, for request-response, the `ActorRef` should be added to the request itself.

## Concluding

The Advanced Akka course gives a lot of insights and concrete examples of how to use the advanced Akka features of clustering, sharding and persisting data across multiple nodes in order to create a system that really is highly available, resilient and scalable. It also touches on the bleeding edge functionalities, the ideas and concepts around it and what to expect next in this growing ecosystem.
