---
layout: post
title:  "Distributed time: lamport-clocks and ordering of events"
date:   2022-09-21 09:06:43 +0300
categories: clock time lamport-clocks
---

# Time
Time and ordering of events is important in all systems as it allow users to serialize events into a log that has all the events as they occured.
Although this is important and is trivial in a single threaded application, in distributed systems global ordering is difficult to achieve.

Why is this? Supose we have two processes A and B. 

It is easy to order all the events that happen in A such that 
a1, a2, a3 ... an will be their times and those in B will have the same ordering where b1, b2 ...bn is the order relative to each other. 

However with this information we can not come up with an ordering for events within both A and B. 

## Ordering
If we are given more information say a3 is the event that sends a message from A to B.

<div class='mermaid'>
sequenceDiagram
	participant A
	participant B
	A->>A: a1
	B->>B: b1
	B->>B: b2
	A->>A: a2
    A->>B: a3
    note right of B: b3
    A->>A: a4
    B->>B: b4
</div>

From the above diagram we can say that the following must be true and we will use the arrow to show the happens before relationship.

a1,a2 -> b3
a1 -> a2 
a2 -> a3
a3 -> a4

This is the ordering we can comfortably deduce from the diagram above.

What about a1 and b1 and b2? Is there any relationship?
Well we can deduce that these events in the processes can be termed as concurrent events. For concurrent events we can not deduce any happens before relationship.

## Relationships.
As you can see even given the above chart we cannot order any nodes that are not due to the consequence of sending and recieving messages. 

a3 and b3 can be ordered only due to them having an explicit happens before relationship. b3 can never happen before a3 as a3 has to have happened before.

In the process it is easy to order items as they do have explicit ordering information. This ordering can then be used for asyncronous systems through the use of casual ordering.

## Casual ordering.
This is defined as the ordering between two item such that we know the first item (a) causes (b). This means the clock C at C(a) < C(b) since the transmission time can never be less than 0 hence a->b.

## Clock types.
1. [Lamport clocks](https://en.wikipedia.org/wiki/Lamport_timestamp).
2. [Vector clocks](https://en.wikipedia.org/wiki/Vector_clock)
3. [Matrix clocks](https://arxiv.org/abs/cs/0309042).