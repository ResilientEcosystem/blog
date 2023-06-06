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
Start the KV servers with the example script. This script uses the example/kv_config.config file.
  > sh example/start_kv_server.sh

You will observe that four kv_server replicas have locally launched. 

### nexres_sdk
We use [Crow](https://github.com/CrowCpp/Crow), a C++ framework for creating HTTP or Websocket web services to connect NexRes to the Explorer.

In another terminal shell after starting KV Server, build the crow service: 
  > bazel build sdk_client/crow_service_main

Run the binary to start the service:
  > bazel-bin/sdk_client/crow_service_main example/kv_client_config.config sdk_client/server_config.config

You will see this if successful: 
  ```
  (2022-12-19 06:12:02) [INFO    ] Crow/master server is running at http://0.0.0.0:18000 using 16 threads
  (2022-12-19 06:12:02) [INFO    ] Call `app.loglevel(crow::LogLevel::Warning)` to hide Info level logs
  ```

### resilientdb.github.io
Project Setup
  > npm install

Compile and Hot-Reload for Development
  > npm run dev

Type-Check, Compile and Minify for Production
  > npm run build

Lint with [ESLint](https://eslint.org/)
  > npm run lint