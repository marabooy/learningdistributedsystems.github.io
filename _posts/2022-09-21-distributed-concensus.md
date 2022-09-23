---
layout: post
title: "Distributed consensus: paxos and friends"
date: 2022-09-21 09:40:43 +0300
categories: FLP paxos raft pbft
---

## Distributed Computing

Imagine trying to upload an infinite number of gigabytes of data to the internet over your wifi router using your personal laptop. How much storage and memory would you need for a task like that? The answer is a lot more than your laptop or router could ever handle. It requires a lot of computers coming together in a network to attempt to solve this. We need consensus algorithms for all of the computers in the network to agree on what parts of the data will be sent when and in what order.

Theres a couple of ways to do this.

# Leader-Replica

This method involves picking a leader to collect votes from other computers on the network. The computers are voting on if a piece of data should become the consensus, or the general agreement, among everyone on the network. If the leader is able to get a majority of votes, it will make this piece of data and its timestamp part of the official record. This record is then kept on the local memory of all the computers.

If the leader is to ever go offline or otherwise lose its connection to the network, a new leader must be elected, again, by consensus.

# Peer-to-peer

In the absence of a leader, all computers are equals and must find a way to agree. Any computer can request that data is added to the record. It will do this by sending requests to other computers and maintain agreement constantly.

## Paxos

Paxos algorithm uses both of these schema to establish agreement across networks. First it defines the roles of proposer, acceptor, and learner. Computers can be any, or all, of these roles.

<div class='mermaid'>
sequenceDiagram
	participant Proposer1
    participant Proposer2
	participant Acceptor
    participant Learner
</div>

Proposers act as a leader and collect votes on different pieces of data. It sends a timestamp and the data to a majority of the acceptors trying to get them to vote yes. An acceptor will vote yes if it isn't considering other proposals.

<div class='mermaid'>
sequenceDiagram
	participant Proposer1
    participant Proposer2
	participant Acceptor
    participant Learner
	Proposer1->>Acceptor: 5:31PM, "data1"
	Acceptor->>Proposer1: i vote yes, 5:31PM, "data1"
</div>

Once it votes yes, it will no longer consider any proposals with an older timestamp than what it just agreed to. This makes sure old data that may have just taken longer to travel across the network isn't accepted as the latest data.

<div class='mermaid'>
sequenceDiagram
	participant Proposer1
    participant Proposer2
	participant Acceptor
    participant Learner
	Proposer1->>Acceptor: 5:31PM, "data1"
	Acceptor->>Proposer1: i vote yes, 5:31PM, "data1"
	Proposer2->>Acceptor: 4:06PM, "data2"
	Acceptor->>Proposer2: i vote no, 4:06PM, "data2"
</div>

After the proposer has collected a majority of votes, it will ask those acceptors that voted yes to consider the data and its timestamp as "law". If the timestamp is **_still_** the latest, freshest data, it will accept it as law and send the law out to learners.

<div class='mermaid'>
sequenceDiagram
	participant Proposer1
    participant Proposer2
	participant Acceptor
    participant Learner
	Proposer1->>Acceptor: 5:31PM, "data1"
	Acceptor->>Proposer1: i vote yes, 5:31PM, "data1"
	Proposer2->>Acceptor: 4:06PM, "data2"
	Acceptor->>Proposer2: i vote no, 4:06PM, "data2"
    Proposer1->>Acceptor: accept as law, 5:31PM, "data1
    Acceptor->>Learner: 5:31PM, "data1"
</div>

Now the whole network is in agreement!
