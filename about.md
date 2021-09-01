---
layout: article
title: About Us
key: page-about
---


<p style="text-align:center;">
    <img src="/assets/expolab-icon.png" width="70%" height="70%" alt="Functions" style="background-color:#000;"/>
    <br>
</p>

> The Exploratory System Laboratory aims at Making Permissioned Blockchain Systems *Fast Again*.

ResilientDB makes system-centric design decisions by adopting a multi-thread architecture that encompasses deep-pipelines. Further, we separate the ordering of client transactions from their execution, which allows us to perform out-of-order processing of messages.

>
> 1. ResilientDB supports a Dockerized implementation, which allows specifying the number of clients and replicas.
> 
> 2. PBFT [Castro and Liskov, 1998] protocol is used to achieve consensus among the replicas.
> 
> 3. ResilientDB expects minimum 3f+1 replicas, where f is the maximum number of byzantine (or malicious) replicas.
> 
> 4. ReslientDB designates one of its replicas as the primary (replicas with identifier 0), which is also responsible for initiating the consensus.
> 
> 5. At present, each client only sends YCSB-style transactions for processing, to the primary.
> 
> 6. Each client transaction has an associated transaction manager, which stores all the data related to the transaction.
> 
> 7. Depending on the type of replica (primary or non-primary), we associate different a number of threads and queues with each replica.
> 
> 8. ResilientDB allows easy implementation of Smart Contracts. At present, we provide a comprehensive implementation of Banking Smart Contracts.
> 
> 9. To facilitate data storage and persistence, ResilientDB provides support for an in-memory key-value store. Further, users can take advantage of SQL query execution through > the fully-integrated APIs for SQLite.
> 
> 10. With ResilientDB we also provide a seamless GUI display. This display generates a status log and also accesses Grafana to plot the results. Further details regarding the > setup 1. of GUI display are available in the dashboard folder.