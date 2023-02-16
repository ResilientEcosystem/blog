---
layout: article
title: NexRes Durability Layer
author: Glenn Chen, Julieta Duarte
tags: NexRes
aside:
    toc: true
article_header:
  type: overlay
  theme: dark
  background_color: '#000000'
  background_image:
    gradient: 'linear-gradient(135deg, rgba(0, 204, 154 , .2), rgba(51, 154, 154, .2))'
    src: /assets/images/resdb-gettingstarted/storage_close_up.jpg

---

We provide two databases to choose from to use for durable storage: LevelDB and RocksDB. Both are key-value stores based on log-structured merge (LSM) trees. LSM trees are optimized for write-heavy workloads, and consist of multiple levels, with the keys sorted at each level. The top level is called the memtable, where the most recently inserted data is kept in-memory. The two databases are similar, as RocksDB was initially built on top of LevelDB.

# Install
If you don't have NexRes installed already please check the [install tutorial](https://blog.resilientdb.com/2022/09/28/GettingStartedNexRes.html).

# LevelDB vs RocksDB

Through testing we have found that NexRes works up to ~35-40% faster with LevelDB enabled than with RocksDB. When batched writes are enabled in the durable layer, performance increases by up to 3x with with LevelDB and up to around 2x with RocksDB. RocksDB also provides multiple features not in LevelDB, including allowing the user to set the number of worker threads used.

# Configuration

We use Google's Protobuf to specify the configurations for NexRes. You can configure following settings for the durable layer in a config file such as the one provided below:

LevelDB and RocksDB
- enable_leveldb/enable_rocksdb
  - Make sure to only set one of these to true at the same time
- write_buffer_size_mb
- write_batch_size
  - Keep at 1 to use only individual writes. For batched writes, sizes of 100+ bring higher performance numbers
- path
- generate_unique_pathnames

RocksDB only
- num_threads

example/kv_config.config

    {
      region : {
        ...
      },
      ...
      rocksdb_info : {
        enable_rocksdb:false,
        num_threads:1,
        write_buffer_size_mb:32,
        write_batch_size:1,
        generate_unique_pathnames:true,
      },
      leveldb_info : {
        enable_leveldb:false,
        write_buffer_size_mb:128,
        write_batch_size:1,
        generate_unique_pathnames:true,
      },
    }

# Warning
Currently, reads are guaranteed to be correct when batched writes are enabled in the durable layer. This is because a read may access the old value of a key whose value is updated in a write that is in a batch that has not been committed yet.

If "path" is not set in the durability settings for RocksDB or LevelDB then default paths will be generated in the /tmp/ folder, which is cleared whenever the machine is shut down.

If you are testing NexRes on your local machine using localhost ip, then make sure to set generate_unique_pathnames to true, since multiple processes are not allowed to open up the same RocksDB/LevelDB directory at the same time. When generate_unique_pathnames is set to true, the durable layer uses the cert file names to generate separate directories for each port of the localhost ip.

For example, the process hosting a server with cert_1.cert will append "1" to its directory path. If the cert file name does not contain a number and generate_unique_pathnames is set, "0" will be appended to the path.