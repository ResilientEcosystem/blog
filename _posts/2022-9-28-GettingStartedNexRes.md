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
  > chmod +x INSTALL_MAC.sh
  > ./INSTALL_MAC.sh

Otherwise, use INSTALL.sh:
  > chmod +x ./INSTALL.sh
  > ./INSTALL.sh

INSTALL.sh or INSTALL_MAC.sh will install some necessary software, like Bazel and Protobuf Buffer.

# Running KV Server
Once the installation is done, everything is ready. Now let's start our KV Server with 4 nodes locally.

Running the script in the example folder:
  > chmod +x ./example/start_kv_server.sh
  > ./example/start_kv_server.sh

When the script is done, you will see that 4 applications called kv_server have been launched locally. Now build the client sdk using Bazel:
  > bazel build example/kv_server_tools

Now everything is ready. It is time to test the kv server. First, get a key with an empty value:
  > bazel-bin/example/kv_server_tools example/kv_client_config.config get test

It should return an empty value for the key "test":
  ```
  test = 
  ```

Next, we set a value to the key "test":
  > bazel-bin/example/kv_server_tools example/kv_client_config.config set test value

Now it should return a value "value":
  ```
  test = value
  ```


### Behind the script example/start_kv_server.sh
We do a couple of things in this [script](https://github.com/resilientdb/resilientdb/blob/nexres/example/start_kv_server.sh).
1. Build the kv server under the kv_server folder:
  > bazel build kv_server:kv_server

2. Run 4 kv servers and write its screen output to a local log file in the background:
  > nohup bazel-bin/kv_server/kv_server example/kv_config.config cert/nodeX.key.pri cert/cert_X.cert > serverX.log &

3. Run a client-server to receive the client requests:
  > nohup bazel-bin/kv_server/kv_server example/kv_config.config cert/node5.key.pri cert/cert_5.cert > client.log &

In NexRes, each server should be assigned a certificate by its own private key. This certificate includes some node information, such as its node ip/port, its node type: server or client. If you are using the wrong certificate or running in the wrong node, the server could not be started. 

### Run nodes remotely
Before running more nodes, certificates have to be generated. Each certificate contains the information about the replica and is signed by the administrator. 

1. Go to the main folder where the WORKSPACE locates.

2. Generate the private/public key of AES type:
  >  bazel build tools:key_generator_tools <br>
  >  bazel-bin/tools/key_generator_tools cert/node_X AES

    key_generator_tools will generate a private-public key pair into cert and name them as node_X.key.pri and node_X.key.pub.
We support severial types : **RSA**, **AES**, and **ED25519**. You can select your favorite one.

    For example, we generate 5 keys with AES format and all keys will be put into cert/.
    >  bazel-bin/tools/key_generator_tools cert/node_1 AES
    ```
    save key to path cert/node_1.key.pri
    save key to path cert/node_1.key.pub
    ```
    > bazel-bin/tools/key_generator_tools cert/node_2 AES
    ```
    save key to path cert/node_2.key.pri
    save key to path cert/node_2.key.pub
    ```
    > bazel-bin/tools/key_generator_tools cert/node_3 AES
    ```
    save key to path cert/node_3.key.pri
    save key to path cert/node_3.key.pub
    ```
    > bazel-bin/tools/key_generator_tools cert/node_4 AES
    ```
    save key to path cert/node_4.key.pri
    save key to path cert/node_4.key.pub
    ```
    > bazel-bin/tools/key_generator_tools cert/node_5 AES
    ```
    save key to path cert/node_5.key.pri
    save key to path cert/node_5.key.pub
    ```

3. Generate the certificate for a server:
  >  bazel build tools:certificate_tools <br>
  >  bazel-bin/tools/certificate_tools save_path admin_private_key admin_public_key replica_pub_key node_index ip port replica_type

    | save_path | the path to put the resulting certificate |
    | admin_private_key | the private key from the administrator |
    | admin_public_key | the public key from the administrator |
    | replica_public_key | the public key for the replica |
    | idx | unique node id |
    | ip | replica ip |
    | port |  replica port |
    | replica_type | node type (replica, client) |

    The certificate is assigned by the administrator so we need to provide its private and public keys of it. We have generated the key pair of the administrator and put them in [cert folder](https://github.com/resilientdb/resilientdb/tree/nexres/cert).
    Once it is done, you will see cert_X.cert inside the cert folder.

    For example, we create 4 replicas and 1 client using the keys above and put the results under cert/.

    | id | ip | port | type |
    | 1 | 172.31.88.5 | 10001 | replica |
    | 2 | 172.31.91.225 | 10001 | replica |
    | 3 | 172.31.95.13 | 10001 | replica |
    | 4 | 172.31.85.139 | 10001 | replica |
    | 5 | 172.31.80.182 | 10001 | client |

    > bazel-bin/tools/certificate_tools cert/ cert/admin.key.pri cert/admin.key.pub cert/node_1.key.pub 1 172.31.88.5 10001 replica
    Get the resulting certificate:
    ```
    info:admin_public_key {
      key: "u>\215\254,\352\314\311\315Nb+R\345m\235\341\366\303.\343\004dQ|;V;\335\205\013\006"
      hash_type: ED25519
    }
    public_key {
      public_key_info {
        key {
          key: "E16E3E2AFDA33845AF0E8E2381F87DBE"
          hash_type: CMAC_AES
        }
        node_id: 1
        ip: "172.31.88.5"
        port: 10001
      }
      certificate {
        signature: "=\276m\371\234\374MC\241\036`\364\277\000\240\025\000k\316\216\300bP\345\252\232\002\235T\017\252\347\313\377p\375\327G\316\225l\255\211\'\334EhI\333cb\023V\323>\035\271`n\351\032\035\360\010"
      }
    }
    node_id: 1
    save key to path cert//cert_1.cert
    ```

    > bazel-bin/tools/certificate_tools cert/ cert/admin.key.pri cert/admin.key.pub cert/node_2.key.pub 2 172.31.91.225 10001 replica
    Get the resulting certificate:
    ```
    info:admin_public_key {
      key: "u>\215\254,\352\314\311\315Nb+R\345m\235\341\366\303.\343\004dQ|;V;\335\205\013\006"
      hash_type: ED25519
    }
    public_key {
      public_key_info {
        key {
          key: "C023647DCC67E082E522F3408402F5DE"
          hash_type: CMAC_AES
        }
        node_id: 2
        ip: "172.31.91.225"
        port: 10001
      }
      certificate {
        signature: "\213\366\331RY#\314c\250\211\275^\324$\007\000\336\n\013\332K\026*.X\275\230L*B\004\352\206\252\261\r\377\037&\274\001-=D\216,[\010\230\343\325\360\323v@P\037\267\362N_c\000\001"
      }
    }
    node_id: 2
    save key to path cert//cert_2.cert
    ```

    > bazel-bin/tools/certificate_tools cert/ cert/admin.key.pri cert/admin.key.pub cert/node_3.key.pub 3 172.31.95.13 10001 replica
    Get the resulting certificate:
    ```
    info:admin_public_key {
      key: "u>\215\254,\352\314\311\315Nb+R\345m\235\341\366\303.\343\004dQ|;V;\335\205\013\006"
      hash_type: ED25519
    }
    public_key {
      public_key_info {
        key {
          key: "E2CABB23B382A6B765E898A2DF6F9BDF"
          hash_type: CMAC_AES
        }
        node_id: 3
        ip: "172.31.95.13"
        port: 10001
      }
      certificate {
        signature: "?C\265\207\376\006\033\233\032\342\000\036A\313_B\327\332T\301-\263\264\"I\303\031\256_\330\177K6\320 \000B\203:;J6\276\302s\347K\311\315\371 \346\374\210\326\200\035v}\364\342Q3\014"
      }
    }
    node_id: 3
    save key to path cert//cert_3.cert
    ```

    > bazel-bin/tools/certificate_tools cert/ cert/admin.key.pri cert/admin.key.pub cert/node_4.key.pub 4 172.31.85.139 10001 replica
    Get the resulting certificate:
    ```
    info:admin_public_key {
      key: "u>\215\254,\352\314\311\315Nb+R\345m\235\341\366\303.\343\004dQ|;V;\335\205\013\006"
      hash_type: ED25519
    }
    public_key {
      public_key_info {
        key {
          key: "1C14404CB0314CAD2A89F68BD8D27E5D"
          hash_type: CMAC_AES
        }
        node_id: 4
        ip: "172.31.85.139"
        port: 10001
      }
      certificate {
        signature: "^\305\022\271\337\345\341c5\263\351\217\316\277\023x\260A\335\177e\007eZ<\364\243\nJ \004@\237\2670\2222Q\361\345H\260\340U`x\005\271\263\221\355\341\001\001\002\245\301sPGw\026f\t"
      }
    }
    node_id: 4
    save key to path cert//cert_4.cert
    ```

    > bazel-bin/tools/certificate_tools cert/ cert/admin.key.pri cert/admin.key.pub cert/node_5.key.pub 5 172.31.80.182 10001 client
    Get the resulting certificate:
    ```
    info:admin_public_key {
      key: "u>\215\254,\352\314\311\315Nb+R\345m\235\341\366\303.\343\004dQ|;V;\335\205\013\006"
      hash_type: ED25519
    }
    public_key {
      public_key_info {
        key {
          key: "DD9D69CAE794B27D9C65B7FB98FCDB95"
          hash_type: CMAC_AES
        }
        node_id: 5
        type: CLIENT
        ip: "172.31.80.182"
        port: 10001
      }
      certificate {
        signature: "\3321\3419!@\370\321\002\376\255\311J\304Ur\031\253\217\207\357\343\353y\301<\356C~\371\n\250\177\235MY\302\351\261\363\335-[;%\035:2F\250l\"\347\346V1r?\316{w\310\324\006"
      }
    }
    node_id: 5
    save key to path cert//cert_5.cert
    ```
4. Create a server config including the replica information above.

    Here is the example of **kv_config.config** including the 4 replicas.
    ```
    {
      region : {
        replica_info : {
          id:1,
          ip:"172.31.88.5",
          port: 10001,
        },
        replica_info : {
          id:2,
          ip:"172.31.91.225",
          port: 10001,
        },
        replica_info : {
          id:3,
          ip:"172.31.95.13",
          port: 10001,
        },
        replica_info : {
          id:4,
          ip:"172.31.85.139",
          port: 10001,
        },
        region_id: 1,
      },
      self_region_id:1,
    }
    
    ```

5. Build the Key-Value(KV) Service
    > bazel build kv_server/kv_server

6. Upload the server stuff
    We create an upload script to upload our stuff to each node listed above, including the client node.
    ``` 
    #upload.sh

    SSH_KEY=your ssh key
    iplist=(
      172.31.88.5
      172.31.91.225
      172.31.95.13
      172.31.85.139
      172.31.80.182
    )

    i=1
    for ip in ${iplist[@]}
    do
        echo upload to ${ip}
        ssh -i ${SSH_KEY} -n ubuntu@${ip} "rm -rf /home/ubuntu/*"
        scp -i ${SSH_KEY} bazel-bin/kv_server/kv_server ubuntu@${ip}:/home/ubuntu/
        scp -i ${SSH_KEY} kv_config.config ubuntu@${ip}:/home/ubuntu/
        scp -i ${SSH_KEY} cert/node_${i}.key.pri ubuntu@${ip}:/home/ubuntu/
        scp -i ${SSH_KEY} cert/cert_${i}.cert ubuntu@${ip}:/home/ubuntu/
        ((i++))
    done
    ```
    > ./upload.sh

7. Start the KV service
    We create a start script to start the KV service remotely.
    ```
    # start_service.sh

    SSH_KEY=your ssh key
    iplist=(
      172.31.88.5
      172.31.91.225
      172.31.95.13
      172.31.85.139
      172.31.80.182
    )

    for ip in ${iplist[@]}
    do
      echo kill the running replica on ${ip}
      ssh -i ${SSH_KEY} -n ubuntu@${ip} "killall -9 kv_server"
    done

    i=1
    for ip in ${iplist[@]}
    do
      cmd="cd /home/ubuntu/; nohup ./kv_server ./kv_config.config ./node_${i}.key.pri ./cert_${i}.cert > ./server_${i}.log 2>&1 &"
      echo start replica ${ip}
      ssh -i ${SSH_KEY} -n -o BatchMode=yes -o StrictHostKeyChecking=no ubuntu@${ip} "${cmd}" &
      ((i++))
    done

    while [ $i -gt 0 ]; do
      wait $pids
      ((i--))
    done
    ```

    > ./start_service.sh

7. Check the initial state of each replica

    Login to your replica and check the server log.
      > ssh -i ${your_ssh_key} ubuntu@172.31.91.225
      >
      > cat server_2.log

    Once you see the following records in the server_*.log, the server is ready.

    In the log below, *public size:5* means the replica has received 5 public keys from others so it can be able to verify each message sent from others..

    ```
    E20230215 22:35:18.262069  6288 consensus_service.cpp:271] ============ Server 2 is ready =====================
    I20230215 22:35:18.289825  6319 consensus_service.cpp:218] receive public size:5 primary:1 version:1 from region:1
    I20230215 22:35:18.290783  6304 consensus_service.cpp:218] receive public size:5 primary:0 version:0 from region:1
    I20230215 22:35:18.313995  6260 consensus_service.cpp:218] receive public size:5 primary:1 version:1 from region:1
    I20230215 22:35:19.273571  6307 consensus_service.cpp:218] receive public size:5 primary:1 version:1 from region:1
    I20230215 22:35:19.275673  6288 consensus_service.cpp:218] receive public size:5 primary:1 version:1 from region:1
    ```
8. Access the KV service
    
    Build the KV service tool.
    > bazel build example/kv_server_tools

    Create a client config
    ```
    # kv_client.config
    5 172.31.80.182 10001
    ```

    Run the tool to set a key-value pair value
    > bazel-bin/example/kv_server_tools kv_client.config set test 123

    You will see the following result if successful:
    ```
    client set ret = 0
    ```

    Run the tool to get back a key-value pair value
    > bazel-bin/example/kv_server_tools kv_client.config get test

    You will see the following result if successful:
    ```
    client get value = 123
    ```

