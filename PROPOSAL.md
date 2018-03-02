# Implementing a distributed key-value store


## Introduction

A distributed key-value store is a computer network where information is
stored on more than one node, often in a replicated fashion. Examples of
existing systems are Google's Bigtable, Amazon's Dynamo, etcd...

It is used in production by many companies that need to solve big data
problem. It can also be found in different part of a distributed system acting
as a configuration control center. More interestingly, there are a lot of
implementations that take the core idea of a distributed key-value store to add more
functionalities and features to build a better DevOps workflow such as
HashiCorp's Consul or develop a complete solution for NoSQL database such as
MongoDB, Apache's Cassandra or in-memory database data structure store such as Redis.

My goal for this project to be able to implement a distributed key-value store
from scratch as well as to gain a better understanding of some specific topics in
distributed systems, such as: consensus algorithm, distributed hash table,
RPC, CAP theorem. 

In many sections below, I will talk about the design/architecture of the
project, final product, testing and monitoring. I will also provide a timeline 
with tasks, approaches and deliverables in a bi-weekly basis.


## Design

There are 3 big components of the system: a key-value storage, consensus
algorithm and routing/service discovery.

### A key-value storage

The most straightforward way is to use a hash table to store key-value pairs.
It allows user to read and write in constant time. It's also very easy to use.
Particularly in Python, a dictionary is a hash table with supported CRUD 
(Create, Read, Update, Delete) operations.

However, using a hash table means that I need to store everything in memory, which
is not great when the data get big. One way to solve it is to store them in
disk and use a cache system (something like LRU). Frequently visited data is kept in
memory and the rest is on disk.

> There might be other caching systems to learn from such as Redis and Memcached.

### Consensus algorithm

A consensus algorithm is critical in a distributed system because it allows a
collection of machines to work as a coherent group that can survive failures of
some its members. Paxos is undeniably the most popular algorithm. However, it is also
known for its complexity. Understanding Paxos is hard. Last term I attempted to
implement Paxos but it did not turn out very well. There were still a lot of 
aspects that I was uncertain about. Therefore, I want to try something new this
term.

Raft seems like a good fit. It is made to solve Paxos's understandability
problem. It has been used by a lot of organizations, such as etcd, 
HashiCorp's Consul, Docker Swarm,... and continued to gain its popularity.

> HashiCorp even provides a nice implementation of the Raft and it is
> imported by [many systems](https://godoc.org/github.com/hashicorp/raft?importers).

### Routing/Service Discovery

The last piece of the system is routing/service discovery. At the moment,
I am not sure how to do this yet. I know that HashiCorp's Consul achieve this 
by using a DNS routing mechanism but I am not familiar with its implementation.

In Apache's Cassandra, a Distributed Hash Table is used, which maps key to
specific node in the rings structure. Similar with Amazon DynamoDB, consistent
hashing is used. However, it is not the same as service discovery.
In the [final product section](#final-product), I will give an example of
a service discovery's behavior that I want.

> It would be helpful if you can provide any pointers for this.

### Flow

> There should be some images in this section for the reader to visualize the
> system easily. It also helps forming the flow of the system. I don't have one
> yet and I can't think of any at the moment. Below is my assumption and excuse
> for not having one. Will update this as I take a closer look at Raft as well
> as other documents.

For the initial design, after looking at some similar systems such as etcd,
Amazon's DynamoDB, Consul,... I realize that getting the consensus algorithm right
is the most important job, which is Raft in this case. As long as I have all
the nodes perform resiliently using the protocol, building a key-value
store on top seems much more natural. In other word, Raft does most of the
heavy lifting for the system.


## Timeline

### Week 1-2
- Task:
  - Finish first draft of the proposal.
- Approach:
  - Read about similar systems and learn how do they implement it.
  - Come up with a solution myself that fits the scope of the project.
- Deliverables:
  - A reasonable well-written first draft to start coding in the following week.

### Week 3-4
- Task:
  - Implement a minimum version of Raft.
  - Continue building up the proposal as I spend more time understanding Raft.
- Approach:
  - Start adopting pseudocode in Raft's original white paper.
  - Use [MIT's lab](https://pdos.csail.mit.edu/6.824/labs/lab-raft.html) as a reference.

    > The great thing about this MIT's Distributed System course is that their
    > labs are built on top of Raft. They go through its implementation,
    > add a distributed key-value service on top of it and eventually exploring
    > the idea of sharding. This is exactly what I want to do too. The only
    > difference is that they have the idea of sharding while I have the idea
    > for a service discovery feature (Well I guess I can do both if I have
    > time). Do you think this is a good hands-on lab to start with and maybe
    > follow throughout the term?
  
    > One more thing that I want to point out about this course is that, they
    > write everything in Go. I am new to Go and haven't had a chance to try it
    > out. Do you think is it good to go with Go, instead of Python?
    > If the answer is "Yes", I need to modify the final product section below
    > because I planned to implement the project in Python as well as the
    > timeline?

- Deliverables:
  - A minimum working version of Raft.

    > I am not sure how long it's gonna take for a minimum version of Raft so I
    > am assuming that it will take at least 2-3 weeks. After that, it will be
    > the improvement and testing phase. For now, I am leaving the all the
    > status for the rest of these weeks as *TODO*. However, I will update these
    > as long as I make progress with Raft's and have a better picture of how
    > things work.

### Week 5-6
- Task:
- Approach:
- Deliverables:

### Week 7-8
- Task:
- Approach:
- Deliverables:

### Week 9-10
- Task:
- Approach:
- Deliverables:

### Week 11-12
- Task:
- Approach:
- Deliverables:

### Week 13-14
- Task:
- Approach:
- Deliverables:


## Final Product

This is how I see it working as the final product.

### CLI
```
Usage: python3 mgmt.py -r <role> -c <command> [-h <host>] [-k <key>] [-v <value>]
Help:  Run the management system to control and execute tasks.
Options:
      -n                Name of a node.
      -r                Role to be defined.
      -c                Command to be executed.
      -h                Host name, including address and port.
      -k                Key to be stored.
      -v                Value corresponding to the key.
```

Here are the lists of arguments with descriptions:

Arguments | Description
-- | --
`-r server -c start -h <host>` | Start a seed node with a given host and prompt user into the shell.
`-r server -c join [-h <host>]` | Join a node to the cluster and prompt user into the shell.
`-r server -c list` | List all available nodes showing their name, address, health status and type.
`-r server -c kill -h <host>` | Kill a node with a given host.
`-r server -c stop -h <host>` | Stop a node with a given host.
`-r server -c restart -h <host>` | Restart a node with a given host.
`-r client -c read -h <host>` | Get all the keys and values for a given host.
`-r client -c read -h <host> -k <key>` | Read a value for a given key, for a given host.
`-r client -c write -h <host> -k <key> -v <value>` | Write a value to a key for a given host.
`-r client -c update -h <host> -k <key> -v <value>` | Update a value for a key for a given host.
`-r client -c delete -h <host> -k <key>` | Delete a key for a given host.

> Can also use the name of a node, instead of its host?

> How to make configurations dynamic? For example: name, type, health checking
> interval?

> Instead of having one file to manage among roles and execute tasks, it would
> be great if I can execute `hstore-server <command>` to
> start/stop/kill/restart the server, `hstore-client -h <host>`
> to get inside the shell and do `set foo bar` or `get foo`. 
> However, I am not sure how to do this yet.

#### Steps

- I first start with a seed node as a server.
- I use other node to join the seed node or I can choose to join other node
  that are available in the system if I know its host. However, if I am able to
  implement the service discovery feature, I just need to tell it to `join` 
  without explicitly specifying the host. It automatically know how to route 
  to the right cluster. 

  > I think this is how a service discovery should work. However I am not sure.

- When there are 3 nodes, leader election will occur. One is the leader, the
  rest are followers.
- Using the client machine, I can read, write, update or delete key-values in any
  of these nodes.
- The key-values should be replicated among themselves, no matter where I put
  them. It means that I don't have to put a key-value in the master node in
  order for it to be propagated.

  > I feel like this behavior somewhat conflicts with the idea of Raft's leader
  > election. Because in Raft, there are one master at a time and the rest are
  > followers. A configuration first go through the master then if the consensus is
  > reached, it is replicated to other nodes in the cluster. However, in this
  > situation that I am proposing, it seems like there is no master 
  > and every one has equal role. I can put a key-value anywhere in the
  > cluster.
  
  > If I follow the masterless architecture, is it better to put a load
  > balancer in front of these nodes?

- If I choose to stop a node or multiple nodes, the system must still work.
- If I stop the master, it will start the leader election again if and only if
  the quorum size is big enough. Everything should behave the same way.

### APIs

List of exposed APIs for each node.

Method | Endpoint | Description
-- | -- | --
`GET` | `/read` | Read all keys and values.
`GET` | `/read/<key>` | Read a value for a given key in a node.
`POST` | `/write` | Write a value to a key in a node.
`POST` | `/update` | Update a value for a key in a node.
`GET` | `/delete/<key>` | Delete a key in a node.


## Testing

> **TODO:** There should be automated tests for the system.

> Other than unit test, integration test (if needed), end to end test (if
> needed), how to introduce failure injections/exercises for the system,
> exploring its behavior in the face of crashes and network partitioning?


## Monitoring

> **TODO:** Having a dashboard to view all the statistics is a good idea. Can
> be a simple web page or just a CLI.
