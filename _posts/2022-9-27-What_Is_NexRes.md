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

[NexRes](https://github.com/resilientdb/resilientdb/tree/nexres) is the next generation of ResilientDB, an open source high performance Remote Procedure Call (RPC) framework running in a byzantine environment. It provides a simplier system structure and kindly APIs so that developers can be able to explore their research or implement their applications in a more convenient way.

NexRes is still Written entirely in C++, which offers compatible interfaces to most of any other programming language.
NexRes introduces [Bazel](https://bazel.build/about/intro) which is a compiling tool that helps us to manage the compling environment. Bazel provides a strong capability to make the compling easier and the library dependency more clear, comparing to Makefile.

NexRes also adopts [Protobuf Buffer](https://developers.google.com/protocol-buffers/docs/cpptutorial) to serialize and deserialize structured data.

# System Architecture
NexRes contains three layers: Application, Consensus and Network. Developers can explore any layer they are interested in without accessing others.

## Application
Application addresses the transactions publiced by the users. NexRes guarantees that once the application receives a transactions, this transaction has been committed by some consensus protocol handled by Consensus layer. Developers do not need to consider how the transactions are committed.

## Consensus
Consensus tackles the consensus algorithm to commit the transactions. Normally, it contains two phases: ordering and exection.Ordering helps to order each transaction among the system by runing some consensus protocol, like PBFT. Once the transactions are commited with a sequence id, they will be executed in sequence in the exection phase by calling the Application.

## Network
Network handles message delivering. It helps the system to receive messages from other machines and deliver messages to them.
NexRes uses coroutine to improve the system performance. Moreover, each machine will keep connecting with each other in a cluster to reduce the connection cost.

