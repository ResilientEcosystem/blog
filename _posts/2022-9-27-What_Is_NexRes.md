---
layout: article
title: NexRes - What is NexRes
author: Junchao Chen
tags: NexRes
aside:
    toc: true
article_header:
  type: overlay
  theme: dark
  background_color: '#000000'
  background_image:
    gradient: 'linear-gradient(135deg, rgba(0, 204, 154 , .2), rgba(51, 154, 154, .2))'
    src: /assets/images/resdb-gettingstarted/codecover.jpg

---

[NexRes](https://github.com/resilientdb/resilientdb/tree/nexres) is the next generation of ResilientDB, an open-source high-performance Remote Procedure Call (RPC) framework running in a byzantine environment. It provides a simpler system structure and kind APIs so that developers can be able to explore their research or implement their applications in a more convenient way.

NexRes is still Written entirely in C++, which offers compatible interfaces to most any other programming languages.
NexRes introduces [Bazel](https://bazel.build/about/intro) which is a compiling tool that helps us to manage the compiling environment. Bazel provides a strong capability to make the compiling easier and the library dependency clear, comparing to Makefile.

NexRes also adopts [Protobuf Buffer](https://developers.google.com/protocol-buffers/docs/cpptutorial) to serialize and deserialize structured data.

# System Architecture
NexRes contains three layers: Application, Consensus, and Network. Developers can explore any layer they are interested in without accessing others.

## Application
The application addresses the transactions published by the users. NexRes guarantees that once the application receives a transaction, this transaction has been committed by some consensus protocol handled by the Consensus layer. Developers do not need to consider how the transactions are committed.

## Consensus
Consensus tackles the consensus algorithm to commit the transactions. Normally, it contains two phases: ordering and execution. Ordering helps to order each transaction in the system by running some consensus protocol, like PBFT. Once the transactions are committed with a sequence id, they will be executed in sequence in the execution phase by calling the Application.

## Network
The network handles message delivery. It helps the system to receive messages from other machines and deliver messages to them.
NexRes uses a coroutine to improve the system's performance. Moreover, each machine will keep connecting with each others in a cluster to reduce the connection cost.

