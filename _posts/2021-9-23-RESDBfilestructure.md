---
layout: article
title: ResilientDB - File Structure
author: alex
tags: ResilentDB
aside:
    toc: true

---

This is an installment of the ResilientDB tutorial series. ResilientDB is a permissioned blockchain fabric developed by the developers at the Exploratory Systems Lab at UC Davis. We are a group of researchers on a mission led by Prof. Mohammad Sadoghi to pioneer a resilient data platform at scale. 
{:.info}

### File Structure 

In your linux command line interface, `tree resilientdb -L 2` provides us the following output:


```rust
resilientdb
├── CHANGELOG.md
├── CODE_OF_CONDUCT.md
├── LICENSE.md
├── Makefile
├── README.md
├── benchmarks
│   ├── YCSB_schema.txt
│   ├── ycsb.h
│   ├── ycsb_query.cpp
│   ├── ycsb_query.h
│   ├── ycsb_txn.cpp
│   └── ycsb_wl.cpp
├── blockchain
│   ├── chain.cpp
│   ├── chain.h
│   └── crypto.h
├── client
│   ├── client_main.cpp
│   ├── client_query.cpp
│   ├── client_query.h
│   ├── client_txn.cpp
│   └── client_txn.h
├── config.cpp
├── config.h
├── dashboard
│   ├── Readme.md
│   ├── dashboard.tgz
│   └── nginx-site.conf
├── data_structures
│   ├── hash_map.h
│   ├── hash_set.h
│   └── shared_int.h
├── db
│   ├── database.h
│   ├── in_memory_db.cpp
│   └── sqlite.cpp
├── deps
│   ├── boost.tgz
│   ├── boost_1_67_0
│   ├── crypto
│   ├── crypto.tgz
│   ├── jemalloc-5.1.0
│   ├── jemalloc.tgz
│   ├── nng-1.3.2
│   ├── nng.tgz
│   ├── sqlite-autoconf-3290000
│   └── sqlite.tgz
├── docker-compose.yml
├── ifconfig.txt
├── obj
│   ├── banking_sc.o
│   ├── chain.o
│   ├── client_main.o
│   ├── client_query.o
│   ├── client_thread.o
│   ├── client_txn.o
│   ├── config.o
│   ├── global.o
│   ├── helper.o
│   ├── in_memory_db.o
│   ├── io_thread.o
│   ├── lock_free_queue.o
│   ├── main.o
│   ├── mem_alloc.o
│   ├── message.o
│   ├── msg_queue.o
│   ├── msg_thread.o
│   ├── parser.o
│   ├── pool.o
│   ├── query.o
│   ├── sim_manager.o
│   ├── sqlite.o
│   ├── stats.o
│   ├── stats_array.o
│   ├── thread.o
│   ├── timer.o
│   ├── transport.o
│   ├── txn.o
│   ├── txn_table.o
│   ├── txn_test.o
│   ├── wl.o
│   ├── work_queue.o
│   ├── work_queue_nopipeline.o
│   ├── worker_thread.o
│   ├── worker_thread_pbft.o
│   ├── ycsb_query.o
│   ├── ycsb_txn.o
│   └── ycsb_wl.o
├── res.out
├── resilientDB-docker
├── results
│   ├── s4_c1_results_PBFT_b100_run0_node0.out
│   ├── s4_c1_results_PBFT_b100_run0_node1.out
│   ├── s4_c1_results_PBFT_b100_run0_node2.out
│   ├── s4_c1_results_PBFT_b100_run0_node3.out
│   └── s4_c1_results_PBFT_b100_run0_node4.out
├── runcl
├── rundb
├── scripts
│   ├── docker-ifconfig.sh
│   ├── make_config.sh
│   ├── monitorResults.sh
│   ├── result.sh
│   ├── result_colorized.sh
│   ├── scp_binaries.sh
│   ├── scp_results.sh
│   ├── setUlimit.py
│   ├── set_ulimit.sh
│   ├── settings
│   ├── simRun.py
│   ├── startResilientDB.sh
│   ├── vcloud_cmd.sh
│   ├── vcloud_deploy.sh
│   └── vcloud_monitor.sh
├── smart_contracts
│   ├── banking_sc.cpp
│   ├── smart_contract.h
│   └── smart_contract_txn.h
├── statistics
│   ├── stats.cpp
│   ├── stats.h
│   ├── stats_array.cpp
│   └── stats_array.h
├── system
│   ├── array.h
│   ├── client_thread.cpp
│   ├── client_thread.h
│   ├── global.cpp
│   ├── global.h
│   ├── helper.cpp
│   ├── helper.h
│   ├── io_thread.cpp
│   ├── io_thread.h
│   ├── lock_free_queue.cpp
│   ├── lock_free_queue.h
│   ├── main.cpp
│   ├── mem_alloc.cpp
│   ├── mem_alloc.h
│   ├── msg_queue.cpp
│   ├── msg_queue.h
│   ├── parser.cpp
│   ├── pool.cpp
│   ├── pool.h
│   ├── query.cpp
│   ├── query.h
│   ├── sim_manager.cpp
│   ├── sim_manager.h
│   ├── thread.cpp
│   ├── thread.h
│   ├── timer.cpp
│   ├── timer.h
│   ├── txn.cpp
│   ├── txn.h
│   ├── txn_table.cpp
│   ├── txn_table.h
│   ├── txn_test.cpp
│   ├── wl.cpp
│   ├── wl.h
│   ├── work_queue.cpp
│   ├── work_queue.h
│   ├── work_queue_nopipeline.cpp
│   ├── worker_thread.cpp
│   ├── worker_thread.h
│   └── worker_thread_pbft.cpp
└── transport
    ├── message.cpp
    ├── message.h
    ├── msg_thread.cpp
    ├── msg_thread.h
    ├── nn.hpp
    ├── transport.cpp
    └── transport.h
```


### File Overview


Let's present an overview of what each file is responsible for. We are going to exclude some files neccessary for the github repository; namely `CHANGELOG.md`, `CODE_OF_CONDUCT.md`, `LICENSE.md`, `README.md`:

| File/Directory | Description | Lines of Code |
| -------------- | ----------- | -- |
| :floppy_disk: `resilientDB-docker`  | A bash script responsible for bootstrapping the programming environment. Since we wish to use a dockerized deployement of several nodes, a multi-container Docker application and its services are configured through the generation of a Docker Compose file `docker-compose.yml`. These containers are subsequently started (via `docker-compose up -d`).          | 149 |
| :floppy_disk: `runcl`               | Client node executable. These generate and request transaction to be executed. | N/A |
| :floppy_disk: `rundb`               | Replica node executable. These perform consensus. | N/A |
| :scroll:      `res.out`             | The standard out from `scripts/result.sh` is piped into this file via `resilientDB-docker`.   | N/A |
| :scroll:      `Makefile`            | ResilientDB is compiled by invoking `make` in the main working directory (`resilientdb/`) | 85 |
| :scroll:      `config.h`            | Configuration file. Comprised of preprocessor constants where the generation of which are embedded via metaprogramming from the `scripts/make_config.sh` bash script. `config.cpp` merely imports all these symbols so that an object file can be compiled later. | 201 |
| :scroll:      `docker-compose.yml`  | Docker Compose file and configuration for its respective services.        | N/A |
| :scroll:      `ifconfig.txt`        | Created by `scripts/docker-ifconfig.sh` during the `resilientDB-docker` script.         | N/A |
| :file_folder: `benchmarks/`         | Class definitions for abstractions relating to transactions (TXs) (`ycsb_request` and `YCSBQuery`) as well as the generation of synthetic TXs (`YCSBQueryGenerator`) in `yscb_query.h` and their corresponding workload managers found in `ycsb.h`.         | 331 |
| :file_folder: `blockchain/`         | Class definitions for blocks (`BChainStruct`) and the chain of blocks (`BChain`) in `chain.h` as well as encryption constructs for signing messages in `crypto.h`.        | 583 |
| :file_folder: `client/`             | Client entry point `client_main.cpp` as well as class definitions for `Client_query_queue`.  | 528 |
| :file_folder: `data_structures/`    | Class definitions for data structures with concurrent access via locks.      | 190 |
| :file_folder: `db/`                 | Class definitions for the `DataBase` interface, which connects to an InMemoryDB or sqlite instance. | 338 |
| :file_folder: `deps/`               | Import location for statically built libraries, which consists of routines that are compiled and linked directly into `runcl` and `rundb` via the `Makefile`.         | N/A |
| :file_folder: `obj/`                | Directory where all the object files are compiled and organized into for later linking. | N/A |
| :file_folder: `results/`            | Results from the workload are piped into `.out` files into this directory. | N/A |
| :file_folder: `scripts/`            | Bash scripts for things such as bootstrapping programming environment ( `docker-ifconfig.sh`, `make_config.sh`), collecting workload statistics (latency, memory usage, transaction throughput, idle times) from pipe indirection operator (via stdout and `result.sh` script), as well as others relating to running experiments on cloud compute (`scp_results.sh`, `set_ulimit.sh`, `vcloud_cmd.sh`, and `vcloud_deploy.sh`)        | 1484 |
| :file_folder: `smart_contracts/`    | Class definitions for types inherited from `SmartContract` for smart contract workloads. | 206 |
| :file_folder: `statistics/`         | Class definition for structures relating to accumulation of system statistics (Transactions, execution, worker threads, I/O).         | 1484 |
| :file_folder: `system/`             | **Bulk** of class definitions of ResilientDB are found here such as `WorkerThread` and `ClientThread` which relate to performing consensus.         | 7585 |
| :file_folder: `transport/`          | Class definitions for structures relating to the infrastructure of message communication such as `Socket`, `Message`, `MessageThread`.         | 4258 |
