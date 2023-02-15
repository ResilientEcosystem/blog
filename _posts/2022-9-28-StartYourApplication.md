---
layout: article
title: Start your Application
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

Here we illustrate how to write applications using NexRes. We use our KV Server as an example to provide steps by steps tutorial. More details can be seen from the code base [kv_client](https://github.com/resilientdb/resilientdb/tree/nexres/kv_client), [kv_server](https://github.com/resilientdb/resilientdb/tree/nexres/kv_server), [proto](https://github.com/resilientdb/resilientdb/blob/nexres/proto/kv_server.proto), and [client sdk](https://github.com/resilientdb/resilientdb/blob/nexres/example/kv_server_tools.cpp).

### Write our own proto messages
The first step is to define the proto messages of your application. It includes what kind of interfaces you are going to provide and what the requests and responses are. Here we define two interfaces: Set and Get. For the request, we only need a key and a value that is used for Set request. We write the messages in [kv_server.proto](https://github.com/resilientdb/resilientdb/blob/nexres/proto/kv_server.proto).

```
syntax = "proto3";

package resdb;

message KVRequest {
    enum CMD {
        NONE = 0;
        SET = 1;
        GET = 2;
    }
    CMD cmd = 1;
    string key = 2;
    bytes value = 3;
}

message KVResponse {
    string key = 1;
    bytes value = 2;
}
```

### Write a BUILD file to compile the proto message
In order to compile the proto file and generate the c++ library, we need to provide a [BUILD file](https://github.com/resilientdb/resilientdb/blob/nexres/proto/BUILD#L95).
Inside the BUILD, there are two parts, first part is to compile the proto file which will generate some metadata information.
Then we need to indicate to generate the c++ library for the messages by specifying cc_proto_library. If you want to generate
a python library, py_proto_library can be used. 

```
proto_library(
    name = "kv_server_proto",
    srcs = ["kv_server.proto"],
)
cc_proto_library(
    name = "kv_server_cc_proto",
    deps = [":kv_server_proto"],
)
```

### Write Interface Implementations
Now we are ready to write your own server. Create a KVServerExecutor which is inherited from TransactionExecutorImpl.
TransactionExecutorImpl will deliver every transaction once it is committed. NexRes guarantees that the delivery
is running in one thread and all the transactions are in the same order as when they are committed and executed in the 
Consensus Layer. NexRes also guarantees all the nodes will have the same execution ordering.

Then over-write the ExecuteData function to provide your own execution for each request data. [Code](https://github.com/resilientdb/resilientdb/blob/nexres/kv_server/kv_server_executor.h).
```
#pragma once

#include <unordered_map>

#include "execution/transaction_executor_impl.h"

namespace resdb {

class KVServerExecutor : public TransactionExecutorImpl {
 public:
  KVServerExecutor(void);

  std::unique_ptr<std::string> ExecuteData(const std::string& request) override;

 private:
  void Set(const std::string& key, const std::string& value);
  std::string Get(const std::string& key);

 private:
  std::unordered_map<std::string, std::string> kv_map_;
};
```

Inside ExecuteData, the first thing is to deserialize the input data into our own request message.
Once we get the request message, we will be able to identify the request by its cmd type: Set or Get.
Depending on the request type, we will call different execution functions. Once we get the result if
we get a Get request, we set the value into KVResponse and serialize it into a string then 
return it. For the Set request, because we don't need to return a response, we return a nullptr instead.

[Code](https://github.com/resilientdb/resilientdb/blob/nexres/kv_server/kv_server_executor.cpp).

```
std::unique_ptr<std::string> KVServerExecutor::ExecuteData(
    const std::string& request) {
  KVRequest kv_request;
  KVResponse kv_response;

  if (!kv_request.ParseFromString(request)) {
    LOG(ERROR) << "parse data fail";
    return nullptr;
  }

  if (kv_request.cmd() == KVRequest::SET) {
    Set(kv_request.key(), kv_request.value());
  } else {
    kv_response.set_value(Get(kv_request.key()));
  }

  std::unique_ptr<std::string> resp_str = std::make_unique<std::string>();
  if (!kv_response.SerializeToString(resp_str.get())) {
    return nullptr;
  }

  return resp_str;
}
```

The execution function for Set and Get:
```
void KVServerExecutor::Set(const std::string& key, const std::string& value) {
  if (equip_rocksdb_) {
    r_storage_layer_.setDurable(key, value);
  } else if (equip_leveldb_) {
    l_storage_layer_.setDurable(key, value);
  } else {
    kv_map_[key] = value;
  }
}

std::string KVServerExecutor::Get(const std::string& key) {
  if (equip_rocksdb_)
    return r_storage_layer_.getDurable(key);
  else if (equip_leveldb_)
    return l_storage_layer_.getDurable(key);
  else
    return kv_map_[key];
}
```

Next, write a [BUILD file](https://github.com/resilientdb/resilientdb/blob/nexres/kv_server/BUILD#L4) to compile the code. Inside the BUILD file, we need to add the transaction_executor_impl dependency.
Because we use the proto message, we need to add the dependency as well.

```
cc_library(
    name = "kv_server_executor",
    srcs = ["kv_server_executor.cpp"],
    hdrs = ["kv_server_executor.h"],
    deps = [
        "//execution:transaction_executor_impl",
        "//proto:kv_server_cc_proto",
    ],
)
```

### Build the Server
Now the final step is to write the server main function. In the main function, we need to provide the server configuration,
the node's private key, and its certificate to create ResDBConfig. Then generate the server by passing our execution 
implementations. Finally, Run the server. Code can be seen from [here](https://github.com/resilientdb/resilientdb/blob/nexres/kv_server/kv_server.cpp).
```
int main(int argc, char** argv) {
  if (argc < 4) {
    ShowUsage();
    exit(0);
  }

  char* config_file = argv[1];
  char* private_key_file = argv[2];
  char* cert_file = argv[3];
  char* logging_dir = nullptr;

  std::unique_ptr<ResDBConfig> config =
      GenerateResDBConfig(config_file, private_key_file, cert_file);
  ResConfigData config_data = config->GetConfigData();

  auto server = GenerateResDBServer(
      config_file, private_key_file, cert_file,
      std::make_unique<KVServerExecutor>(config_data, cert_file), logging_dir);
  server->Run();
}
```

Then write the [BUILD file](https://github.com/resilientdb/resilientdb/blob/nexres/kv_server/BUILD#L27) to compile the binary. We put the server implementation in the same folder where the KVServerExecutor is.
```
cc_binary(
    name = "kv_server",
    srcs = ["kv_server.cpp"],
    deps = [
        ":kv_server_executor",
        "//application/utils:server_factory",
        "//config:resdb_config_utils",
    ],
)
```


### Build the Client
NexRes provides a basic client **ResDBUserClient** to help developers send out messages to the server nodes.
Now we create our own client class **ResDBKVClient** and implement the two interfaces we defined before. [Code](https://github.com/resilientdb/resilientdb/blob/nexres/kv_client/resdb_kv_client.h)
```
#include "client/resdb_user_client.h"

namespace resdb {

class ResDBKVClient : public ResDBUserClient {
 public:
  ResDBKVClient(const ResDBConfig& config);

  int Set(const std::string& key, const std::string& data);
  std::unique_ptr<std::string> Get(const std::string& key);
};
```

In the implementation, we just need to fill the Request proto and call the SendRequest function.
If we don't need the response, only need to pass the Request. [Code](https://github.com/resilientdb/resilientdb/blob/nexres/kv_client/resdb_kv_client.cpp)

```
int ResDBKVClient::Set(const std::string& key, const std::string& data) {
  KVRequest request;
  request.set_cmd(KVRequest::SET);
  request.set_key(key);
  request.set_value(data);
  return SendRequest(request);
}

std::unique_ptr<std::string> ResDBKVClient::Get(const std::string& key) {
  KVRequest request;
  request.set_cmd(KVRequest::GET);
  request.set_key(key);
  KVResponse response;
  int ret = SendRequest(request, &response);
  if (ret != 0) {
    LOG(ERROR) << "send request fail, ret:" << ret;
    return nullptr;
  }
  return std::make_unique<std::string>(response.value());
}
```

### Build Client Library
Next write a [BUILD](https://github.com/resilientdb/resilientdb/blob/nexres/kv_client/BUILD) to the client library. We also need to include the proto dependency we defined and the user client
NexRes has provided.

```
cc_library(
    name = "resdb_kv_client",
    srcs = ["resdb_kv_client.cpp"],
    hdrs = ["resdb_kv_client.h"],
    deps = [
        "//client:resdb_user_client",
        "//proto:kv_server_cc_proto",
    ],
)
```

### Write the Client SDK Binary
Now it is time to write the SDK. The SDK takes the client config which specifies the client node (not the server node).
Then call the interfaces from our ResDBKVClient. [Code](https://github.com/resilientdb/resilientdb/blob/nexres/example/kv_server_tools.cpp).
```
int main(int argc, char** argv) {
  if (argc < 4) {
    printf("<config path> <cmd>(set/get), key [value]\n");
    return 0;
  }
  std::string client_config_file = argv[1];
  std::string cmd = argv[2];
  std::string key = argv[3];
  std::string value;
  if (cmd == "set") {
    value = argv[4];
  }

  ResDBConfig config = GenerateResDBConfig(client_config_file);
  ResDBKVClient client(config);

  if (cmd == "set") {
    int ret = client.Set(key, value);
    printf("client set ret = %d\n", ret);
  } else {
    auto res = client.Get(key);
    if (res != nullptr) {
      printf("client get value = %s\n", res->c_str());
    } else {
      printf("client get value fail\n");
    }
  }
}
```
Last write a [BUILD file](https://github.com/resilientdb/resilientdb/blob/nexres/example/BUILD) for the SDK.
```
cc_binary(
    name = "kv_server_tools",
    srcs = ["kv_server_tools.cpp"],
    deps = [
        "//config:resdb_config_utils",
        "//kv_client:resdb_kv_client",
    ],
)
```

Then we can test it by following the steps [here](/blog/2022/09/28/GettingStartedNexRes.html#running-kv-server).



