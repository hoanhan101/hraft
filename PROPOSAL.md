# Implementing a distributed key-value store

In this paper, I will describe my plan to build a distributed key-value store.
The goal of this project to gain a better understanding of some specific topics
in distributed systems, such as: distributed hash table, consensus protocol,
RPC, CAP theorem.

There are 3 big components that it has: a key-value storage, consensus
protocol and routing/service discovery.

### A key-value storage

The most straightforward way is to use a hash table to store key-value pairs.
It allows user to read and write in constant time. It's also very easy to use.
Particularly in Python, a dicionary is a hash table with supported CRUD 
(Create, Read, Update, Delete) operations.

However, using a hash table means I need to store everything in memory, which
is not great when the data get big. One way to solve it is to store them in
disk and use some kind of a cache system (LRU). Frequently visited data is kept in
memory and the rest is on a disk.

> Other caching systems to learn from? Redis, Memcached.

### Consensus Protocol

A consensus protocol is critical in a distributed system because it allows a
collection of machines to work as a coherent group that can survice failures of
some its members. Paxos is undeniably the most popular algorithm but is also
known for its complexity. Understanding Paxos is hard. Last term I attempted to
implement Paxos but it did not turn out very well. There were still a lot of 
aspects that I was uncertain about. Therefore, I want to try something new this
term.

Raft seems like a good fit. It is made to solve Paxos's understandability
problem. It has been used by etcd, HashiCorp's Consul and continued to gain its
popularity.

### Routing/Service Discovery

The last piece of the system is routing/service discovery. At the momement,
I am not sure how to do this yet. I know that HashiCorp's Consul achieve this 
by DNS routing mechanism but I am not familiar that.

In Apache's Cassandra, a Distributed Hash Table is used, which maps key to
specific node in the rings structure. However, that is not the same as service
discovery.


In the sections below, I will provide more details about the
design/architecture and outline of the project.


# Design

> TODO

# Timeline

> TODO

Here is the format that every week should follow:
- Task
- Approach
- Deliverables


# Final Product

### CLI
```
Usage: python3 mgmt.py -r <role> -c <command> [-h <host>] [-k <key>] [-v <value>]
Help:  Run the management system to control and execute tasks.
Options:
      -r                Role to be defined.
      -c                Command to be executed.
      -h                Host name, including address and port.
      -k                Key to be stored.
      -v                Value corresponding to the key.
```

Here are the lists of arguments with descriptions:

Arguments | Description
-- | --
`-r server -c start -h <host>` | Start a seed node
`-r server -c join -h <host>` | Join a node
`-r server -c list` | List all available nodes
`-r server -c kill -h <host>` | Kill a node
`-r server -c stop -h <host>` | Stop a node
`-r server -c restart -h <host>` | Restart a node
`-r client -c read -h <host>` | Get all the keys and values
`-r client -c read -h <host> -k <key>` | Read a value for a given key
`-r client -c write -h <host> -k <key> -v <value>` | Write a value to a key
`-r client -c update -h <host> -k <key> -v <value>` | Update a value for a key
`-r client -c delete -h <host> -k <key>` | Delete a key

**Step:**
- You first start with a seed node.
- You join the seed node with as many as you want.
- When there are 3 nodes, leader election will occur.
- One is the leader, the rest are followers.
- As a client, you can read, write, update, or delete a key-value in any of these
nodes.
- The key-value should be replicated among themselves, no matter where you put it
(it doesn't have to be the master)
- If you stop a node or mutiple nodes, the system should still work.
- If you stop the master, it will start the leader election again and everything
should still work.


### APIs

List of exposed APIs for each node.

Endpoint | Description
-- | --
`/read` | Read all keys and values
`/read/<key>` | Read a value for a given key in a node
`/write` | Write a value to a key in a node
`/update` | Update a value for a key in a node
`/delete/<key>` | Delete a key in a node


# Testing

There should be automated tests for the system.


# Monitoring

> TODO


# Docker

Everything should be dockerized.
