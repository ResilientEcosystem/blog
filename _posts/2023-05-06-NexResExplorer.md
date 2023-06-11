---
layout: article
title: Using the NexRes Explorer
author: Glenn Chen, Julieta Duarte, Divjeet Singh Jas
tags: NexRes
aside:
    toc: true
article_header:
  type: overlay
  theme: dark
  background_color: '#000000'
  background_image:
    gradient: 'linear-gradient(135deg, rgba(0, 204, 154 , .2), rgba(51, 154, 154, .2))'
    src: /assets/images/resdb-gettingstarted/code_close_up.jpeg

---

Our Explorer page is a tool for visualizing the NexRes blockchain. The Explorer displays specific blocks on the blockchain, transactions within the blocks, ledger configuration data, and a chart exposing transaction history. 

# Install
Go to our [install tutorial](https://blog.resilientdb.com/2022/09/28/GettingStartedNexRes.html) for instructions on how to install NexRes.

# Set Up 

You will need to clone three repos to get started: [resilientdb](https://github.com/resilientdb/resilientdb.github.io.git), [nexres_sdk](https://github.com/msadoghi/nexres_sdk.git), and [resilientdb.github.io](https://github.com/resilientdb/resilientdb.github.io.git). 

### resilientdb
Install dependencies.
  > ./INSTALL.sh

Start the KV servers with the example script. This script uses the example/kv_config.config file.
  > ./service/tools/kv_service/service_tools/start_kv_service.sh

You will observe that four kv_server replicas have locally launched. 

### nexres_sdk
We use [Crow](https://github.com/CrowCpp/Crow), a C++ framework for creating HTTP or Websocket web services to connect NexRes to the Explorer.

In another terminal shell after starting KV Server, build the crow service from the nexres_sdk repo: 
  > bazel build service/http_server/crow_service_main

Run the binary to start the service:
  > bazel-bin/service/http_server/crow_service_main service/tools/config/interface/service.config service/http_server/server_config.config

You will see this if successful: 
  ```
  (2022-12-19 06:12:02) [INFO    ] Crow/master server is running at http://0.0.0.0:18000 using 16 threads
  (2022-12-19 06:12:02) [INFO    ] Call `app.loglevel(crow::LogLevel::Warning)` to hide Info level logs
  ```

### resilientdb.github.io
In another terminal open the resilientdb.github.io repo and do the steps below. 

Project Setup
  > npm install

Compile and Hot-Reload for Development
  > npm run dev

You will now be able to access a development server for [resilientddb.com](https://resilientdb.com) running at http://localhost:3000. Click on the explorer icon in the top right corner to migrate to http://localhost:3000/explorer. 

__NOTE: You must disable cross-origin restrictions in the browser where you open the development server.__

## Populating the Blockchain 
At this point you will be able to see the ledger configuration data, but the transaction history chart and block table will be empty. The next step is populating the NexRes blockchain to view blocks and transactions using the Explorer. 

You can use an API tester tool such as [Talend API Tester](https://chrome.google.com/webstore/detail/talend-api-tester-free-ed/aejoelaoggembcahagimdiliamlcdmfm/reviews) for committing and getting transactions using the routes defined by our [crow service](https://github.com/msadoghi/nexres_sdk/blob/main/service/http_server/crow_service.cpp). 

You can also use curl commands to transfer data to and from the server. Here are some examples of the routes available. 

For committing a transaction:
  > curl -X POST  -d '{"id":"samplekey","value":"samplevalue"}' localhost:18000/v1/transactions/commit

For getting the transaction:
  > curl localhost:18000/v1/transactions/samplekey

For getting __all__ transactions:
  > curl localhost:18000/v1/transactions

Once you have set and get some transactions you will be able to see the blocks and the transactions within them in the table and chart. 


