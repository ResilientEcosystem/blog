---
layout: article
title: Running NexRes With KV Server 
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

Here we illustrate how to run NexRes with its builtin application: KV Server. We provide steps by steps tutorial to set up NexRes locally with 4 nodes and use the builtin KV Server to set and get a key-value pair.

# Before you begin
NexRes is only able to be run in Ubuntu 20.04 with the ubuntu host. Please make sure you installed the right version.

### Linux
If you are using Linux, please install Ubuntu 20.04. The default user will be ubuntu.

### MaxOS
If you are using MacOS, please install a docker with Ubuntu 20.04. If you are not familiar with docker, we provide a docker file to help, see [here](https://github.com/resilientdb/resilientdb/tree/nexres/docker).

Install the docker image from the docker file named by nexres:
  > docker build . -f DockerfileForMac -t=nexres

Run the docker image with bash command:
  > docker run -i --name nexres nexres 

Get your container **ID**:
  > docker ps -a (find your container id ID)

If the container is not started, start the container:
  > docker start **ID**

Login to the container and swith to the ubuntu user:
  > docker exec -it **ID** /bin/bash <br>
  > su - ubuntu

# Installation
Now it is time to install NexRes.
Clone the repo from github and go to its folder:
  > git clone https://github.com/resilientdb/resilientdb.git
  > cd resilientdb/

If you are using MacOS with arm64 processor (CPU) architecture, like MacOS M1, using INSTALL_MAC.sh to install the environment:
  > sh INSTALL_MAC.sh

Otherwise, use INSTALL.sh:
  > sh INSTALL.sh

INSTALL.sh or INSTALL_MAC.sh will install some necessary software, like Bazel and Protobuf Buffer.

# Running KV Server
Once the installation is done, everything is ready. Now let's start our KV Server with 4 nodes locally.

Running the script in the example folder:
  > sh example/start_kv_server.sh

When the script is done, you will see that 4 applications called kv_server have been launched locally. Now build the client sdk using Bazel:
  > bazel build example/kv_server_tools

Now everything is ready. It is time to test the kv server. First, get a key with an empty value:
  > bazel-bin/example/kv_server_tools example/kv_client_config.config get test

It should return an empty value for the key "test":
  > test = 

Next, we set a value to the key "test":
  > bazel-bin/example/kv_server_tools example/kv_client_config.config set test value

Now it should return a value "value":
  > test = value


### Behind the script example/start_kv_server.sh
We do a couple of things in this [script](https://github.com/resilientdb/resilientdb/blob/nexres/example/start_kv_server.sh).
1. Build the kv server under the kv_server folder:
  > bazel build kv_server:kv_server

2. Run 4 kv servers and write its screen output to a local log file in the background:
  > nohup bazel-bin/kv_server/kv_server example/kv_config.config cert/nodeX.key.pri cert/cert_X.cert > serverX.log &

3. Run a client-server to receive the client requests:
  > nohup bazel-bin/kv_server/kv_server example/kv_config.config cert/node5.key.pri cert/cert_5.cert > client.log &

In NexRes, each server should be assigned a certificate by its own private key. This certificate includes some node information, such as its node ip/port, its node type: server or client. If you are using the wrong certificate or running in the wrong node, the server could not be started. 

### Run more nodes
Before running more nodes, you need to generate some new certificates first because we only provide 5 certificates with 4 server types and 1 client type. We have already provided tools to help you generate these certificates.

1. Go to the workspace folder where you clone the repo.

2. Generate the private/public key of AES type:
  >  bazel build tools:key_generator_tools <br>
  >  bazel-bin/tools/key_generator_tools cert/node_X AES

  key_generator_tools will generate a private-public key pair into cert and name them as node_X.key.pri and node_X.key.pub.
We support several types: RSA, AES, and ED25519. You can select your favorite one.

3. Generate the certificate for a server:
  >  bazel build tools:certificate_tools <br>
  >  bazel-bin/tools/certificate_tools cert/admin.key.pri cert/admin.key.pub node_X.key.pub idx ip port replica

  The certificate is assigned by the administrator so we need to provide its private and public keys of it. We have generated the key pair of administrators and put them in [cert folder](https://github.com/resilientdb/resilientdb/tree/nexres/cert).
  Once it is done, you will see cert_X.cert inside the cert folder.

4. Generate the certificate for a client (optional if you want to change the node where the client is):
  >  bazel-bin/tools/certificate_tools cert/admin.key.pri cert/admin.key.pub node_X.key.pub idx ip port client

5. Add more start commands in the script, like:
  >  nohup bazel-bin/kv_server/kv_server example/kv_config.config cert/nodeX.key.pri cert/cert_X.cert > serverX.log &
