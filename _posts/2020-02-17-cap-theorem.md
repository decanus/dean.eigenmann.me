---
layout:     post
title:      The CAP Theorem
date:       2020-02-17 16:00:00
summary:    On consistency, availability and partition tolerance and why we can't build distributed systems that satisfy all these properties.
categories: blog
image: /images/cap-theorem/cap-hat.png
newsletter: https://distsys.substack.com/p/the-cap-theorem
tags: distributed-systems newsletter
permalink: /blog/cap-theorem/
redirect_from: /blog/2020/02/17/cap-theorem/
---

In this post, we will look at the CAP theorem. If you’ve read almost anything about distributed systems, you’ve probably encountered it. But what is it really and where did it come from? In this article we will discuss what it is, its origins and some common misconceptions around the theorem itself.

Before we get started, if you haven't read my previous post ["Time, clocks, and order."](https://dean.eigenmann.me/blog/2020/01/06/time-clocks-and-order/), make sure to check it out. It contains a primer on distributed systems. However, reading it is not necessary for understanding this post.

The CAP theorem itself originates from a presentation by Dr. Eric A. Brewer titled ["Towards Robust Distributed Systems"](https://people.eecs.berkeley.edu/~brewer/cs262b-2004/PODC-keynote.pdf). Later Seth Gilbert and Professor Nancy Lynch formalized the conjecture in a paper called ["Brewer’s conjecture and the feasibility of consistent, available, partition-tolerant web services"](https://users.ece.cmu.edu/~adrian/731-sp04/readings/GL-cap.pdf).

In its most basic definition, the CAP theorem states that it is impossible to build a read/write storage in an asynchronous network that is **available**, **consistent** and **partition tolerant**. Let us dive into this..

Firstly, we need to define the meaning of these 3 properties:

- **C**onsistency - While there are many different consistency models -- [jepsen provides a nice overview](https://jepsen.io/consistency) -- the CAP theorem concerns itself with linearizability (also known as atomic consistency). Essentially meaning that write requests should appear to be instantaneous, additionally a read from our storage should always return the value of the latest write.[^1]
- **A**vailability - This property means that all requests eventually receive a response. This does not include errors, a system could be trivially available all the time by just returning errors.
- **P**artition Tolerance - This means that a system can still continue to function even in the presence of a partition. We define a **partition** as the failure to deliver messages to one or more nodes by losing them. When 2 sets of nodes no longer deliver messages to each other, we call it a **total partition**. The CAP theorem relies on this more restrictive failure mode.

So now back to our most basic definition, let's go over it again as we have now defined our properties: 

Consider an event where we have a **partition**, and we receive a write on our side of the partition and a read is received on the other. Our side cannot know about the write we have previously received. This means that we must now decide to return stale data to the read request, or choose to wait to hear from the other side (potentially forever) of the partition, compromising availability.

If you've ever encountered distributed systems, you know how common a total partition can be. For example,  network equipment can fail and does so often. 

Therefore in the events of failure, we have to think about which we shall sacrifice: consistency or availability?

If you decide to choose consistency, the outcome is very simple: atomicity[^2] is guaranteed by refusing to respond to some or maybe even all requests.

However, if you decide to choose availability, all reads and writes will be accepted. This means that in the case of a read, stale data could be returned, and in the case of a write, there could be conflicts which later need to be resolved.

There are lots of applications where being unavailable can cost more than being inconsistent. I will go into a bit more detail here with my personal experience working at [brack.ch](https://brack.ch), one of Switzerlands largest e-commerce websites. 

With e-commerce, the most obvious question we face is "do we keep our customers shopping carts consistent or do we keep them available?" For example if two users are adding to a cart on the same account but their requests hit different servers, do we strive for consistency or resolve conflicts when a customer reaches the checkout stage? In scenarios like this, the answer is to choose **availability**. For those more interested Amazon's cart is another example of [this](http://s3.amazonaws.com/AllThingsDistributed/sosp/amazon-dynamo-sosp2007.pdf).

In this context, being **unavailable** often results in more lost money than being **inconsistent** does in the end. If a shopping cart is unavailable customers can't add items to buy, if it is inconsistent however a few items might be missing. It's quite easy to think of applications that are more likely to sacrifice consistency than availability. Social media is another big one.

Now that we understand the CAP theorem, let's discuss some misconceptions and inaccuracies.

By calling it the **CAP Theorem**, it leads people to believe that you can choose between either **consistency**, **availability** or **partition tolerance**. This is not true, however. The CAP theorem is only really relevant when your system can actually partition, and the second your system is distributed, it can partition. 

Although this post was shorter than my previous one, I hope it helped you understand a fundamental concept of distributed systems. The CAP theorem is a relatively simple, yet sometimes misunderstood concept.

---

Thanks again to [@ricburton](https://twitter.com/ricburton) for the drawing & [@renelubov](https://twitter.com/renelubov) for the help!  ["You Can’t Sacrifice Partition Tolerance"](https://codahale.com/you-cant-sacrifice-partition-tolerance/) & ["The CAP FAQ"](https://www.the-paper-trail.org/page/cap-faq/) are 2 really good posts that were very helpful while writing this.

[^1]: Bailis. [Linearizability versus Serializability](http://www.bailis.org/blog/linearizability-versus-serializability/). Highly Available, Seldom Consistent. 2014.
[^2]: [Atomicity (Database)](https://en.wikipedia.org/wiki/Atomicity_(database_systems)). Wikipedia.
