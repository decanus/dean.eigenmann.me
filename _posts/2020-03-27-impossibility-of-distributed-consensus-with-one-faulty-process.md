---
layout:     post
title:      Impossibility of Distributed Consensus with One Faulty Process
date:       2020-03-27 18:00:00
summary:    A look at asynchronous algorithms and their inability to solve the consensus problem.
categories: blog
tags:       distributed-systems newsletter
---

> A look at consensus, asynchronous algorithms and the consensus problem.

First things first, this post took me way longer to write than I thought it would. Reason being the subtlety of what I decided to write about along with other factors. Hopefully my next posts will be on a more regular schedule again!

Let's talk consensus. Consensus is the process of reaching agreement -- a fundamental issue of distributed systems laying at the core of countless algorithms. Agreement upon some value or state is a common requirement.

Reaching agreement is not that hard. Well, at least it isn't if all nodes as well as the network they communicate over are reliable and all operate under the same assumptions. And as highlighted by both of my previous posts, ["Time, clocks, and order"](https://dean.eigenmann.me/blog/2020/01/06/time-clocks-and-order/) and ["The CAP Theorem"](https://dean.eigenmann.me/blog/2020/02/17/cap-theorem/), this is pretty much never the case in practice.

["Impossibility of Distributed Consensus with One Faulty Process"](https://groups.csail.mit.edu/tds/papers/Lynch/jacm85.pdf) by Fischer, Lynch and Patterson proves solving the problem of consensus is impossible in an asynchronous system where even only one node may crash. The paper won the [Dijkstra award](http://eatcs.org/index.php/dijkstra-prize), a prize for outstanding papers on the principles of distributed computing. But why is it the case that asynchronous systems can't solve the consensus problem?

## The Problem

Let's first look at the **consensus problem**. What kind of properties would an algortihm need to satisfy in order to solve it?

 - **Safety** - Nodes will agree on the same value, which was proposed by one of the nodes.
 - **Liveness** - Eventually, every non-faulty node should decide upon some value.
 - **Fault tolerance** - All non-faulty nodes must agree on the same value.

In the paper, the above are referred to as **agreement**, **termination** and **validity**.

The above property definitions use the term **fault** several times. What exactly is a fault? In distributed systems, we have several types of faults. Explaining them could probably be a post of its own, but I will do my best to summarize them concisely. 

As stated in the paper, we can experience failures such as process crashes, network partitioning, lost, distorted, or duplicated messages or even byzantine failures. Byzantine failures are ones where nodes fail by deviating arbitrarily from the algorithm they are executing.

## A look at consensus

This section will provide a brief overview of consensus that will be useful in understanding the rest of the consensus problem along with how to circumvent it. It's a short summary of Preethi Kasireddy's ["Let’s take a crack at understanding distributed consensus"](https://www.preethikasireddy.com/post/lets-take-a-crack-at-understanding-distributed-consensus). I suggest you read it if you're interested in a deeper dive on consensus. 

Traditionally, consensus algorithms have three actors:
 - **Leaders:** This is the node usually responsible for proposing values.
 - **Acceptors:** Listen to proposed values and accept them.
 - **Learners:** Nodes that learn about the decided upon values.

These three actors traditionally participate in 3 rounds. These are:

 - **Election:** A leader is elected, and then proposes the next value.
 - **Vote:** Acceptors listen to the leader's proposed value, validate it and then propose it as the next valid value.
 - **Decide:** If enough Acceptors proposed the new value, then the value becomes decided upon, else we restart.

So this is a very brief look at traditional consenus, now lets continue.

## The System

Let's look at the type of system we are dealing with and the assumptions the paper makes. 

As previously stated, we are dealing with an **asynchronous** system which is different from a synchronous one in that there is no upper-bound in the delivery or the processing of messages. This means that faults are impossible to detect. Are messages taking long to deliver? Is a node taking its sweet-ass time or has the other node failed?

As we see with the above described system, we cannot detect all faults therefore we cannot state that all non-faulty nodes agreed on the same value. We were not aware of which nodes are non-faulty, so our consensus algorithm is not fault tolerant. This is a simple argument to make informally, so let's jump into the more formal definitions as stated in the paper. 

## The Proof

The paper contains two seperate lemmas that together show that a consensus algorithm can remain forever undecided in an asynchronous enviroment. 

The first lemma (which is actually the second in the paper), states that the algorithm can't simply query the network for the initial state due to the fact that failures would prevent this from completing. Assuming that the states of nodes are seperate, then the value nodes reach agreement upon depends on the order in which messages are received. This is due to the fact that asynchronous systems are non-deterministic.

And finally, the second lemma (the third discussed lemma in the paper), states if you start with an undecided state, then it is always possible to reach another undecided state by delaying pending messages. This can be done perpetually even if all messages are eventually delivered. That means that no protocol can guarantee that something will eventually be decided upon.

## Overcoming the impossibility

There are two methods to overcome the impossibility we have previously presented: either introduce **synchrony assumptions** or create a **non-deterministic** consensus algorithm.

So let's look at **synchrony assumptions**, what does this mean and how do we achieve this? It basically means that we need to somehow guarantee **termination** so that we have every non-faulty node decide upon the same value eventually. We can do this by implementing timeouts. Famous consensus algorithms like [Raft](https://raft.github.io/raft.pdf) and [Paxos](https://lamport.azurewebsites.net/pubs/lamport-paxos.pdf) implement timeouts for example. Essentially what we do is we allow nodes to attempt to reach consensus until a certain timeout is reached. If the timeout is reached, we restart the process and try again starting from the beginning.

Raft nodes wait for a leader to propose a value. If a specific node doesn't hear a proposed value in a certain amount of time, it will propose itself to become the new leader. If other nodes also haven't heard a proposed value, they will agree and elect the node a new leader. The leader then proposes new values, and tries to reach consensus.

Now how about **non-deterministic** consensus, what does that even mean? Algorithms that use **synchrony assumptions** require that all nodes know eachother and are able to communicate with eachother, but this is impossible for a distributed system like Bitcoin. Consensus algorithms that are non-determinisitic are refered to often as **probabilistic**, meaning that instead of agreeing whether a value is correct. They agree on the probability that some value is correct. 

This is how **Nakomoto consensus** works, it does away with the traditional methods of achieving consenus and instead opts for a more simplified and scalable (with respect to participant inclusion) algorithm. It does away with electing leaders. Instead all nodes try to solve a known computationally hard mathematical problem. When the problem has been solved, the next value can be added. This forms a chain of values. Nodes choose to continue building on the chain of values that have the most combined computational power, preventing from branches with invalid values to be selected.

*For a more accurate and longer explanation of Nakomoto consensus I recommend reading ["The Nakamoto Consensus Algorithm"](https://medium.com/nakamo-to/nakamoto-consensus-21cd304f96ff) by William Nester.*

This was quite a challenging topic to summarize in such a brief post, I hope that it made sense and taught you about various aspects of consensus, however. It's a fundamental issue in computer science and will probably be important in a lot of my next posts on distributed systems!

## References

1. Fischer, Lynch & Paterson. ["Impossibility of Distributed Consensus with One Faulty Process"](https://groups.csail.mit.edu/tds/papers/Lynch/jacm85.pdf). Journal of the Assccktion for Computing Machinery. 1985.
2. Robinson. ["A Brief Tour of FLP Impossibility"](https://www.the-paper-trail.org/post/2008-08-13-a-brief-tour-of-flp-impossibility/). The Paper Trail. 2008
3. Janik ["Connecting the Dots: FLP, BFT & Consensus Algorithms"](https://hackernoon.com/connecting-the-dots-flp-bft-and-consensus-algorithms-m9r62bs1). Hackernoon. 2019
4. Kasireddy. ["Let’s take a crack at understanding distributed consensus"](https://www.preethikasireddy.com/post/lets-take-a-crack-at-understanding-distributed-consensus). [preethikasireddy.com](https://www.preethikasireddy.com/). 2018
