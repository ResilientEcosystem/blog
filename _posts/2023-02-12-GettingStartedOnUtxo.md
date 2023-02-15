---
layout: article
title: Getting Started On UTXO
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
    src: /assets/images/nexres/coin.jpg

---

Here we illustrate how to run the UTXO service on NexRes. We provide steps by steps tutorial to set up the UTXO service locally with 4 nodes and 1 client. We also provide tools to access the UTXO transactions and the user wallets.


## Create a Wallet

Beginning using UTXO transactions on NexRes, we need to create a wallet by generating a private key, a public key, 
and an address corresponding to your wallet.
Wallet addresses are encoded by bech32 which is also used for Bitcoin.

These are some steps to build your private-public keys and your address:

1. Go to the source folder which contains the WORKSPACE file.

   Make sure  bech32 has been installed which is used for the address encoding. 
   > pip install bech32

2. Create your own private key. It will return a private key and a public key, encoded by ECDSA.
    In the example here, we will create two key pairs and two addresses.
  > bazel run //application/utxo/wallet_tool/py:keys
  ```
  private key:303E020100301006072A8648CE3D020106052B8104000A0427302502010104202CB99BBB2AFEB7F48A574064091B34F24781C93AD8181A511C8DCFB2A111AD82
  public key:3056301006072A8648CE3D020106052B8104000A034200049C8FBD86EA4E38FD607CD3AC49FEB75E364B0C694EFB2E6DDD33ABED0BB1017575A79CC53EC6A052F839B4876E96FF9E4B08ECF23EC9CD495B82ECF9D95303BD
  ```
  > bazel run //application/utxo/wallet_tool/py:keys
  ```
  private key:303E020100301006072A8648CE3D020106052B8104000A0427302502010104202BDCC4974026EC852F95D481AE8FEC3AC31E130AB6C78A32EB8410CCDCA4B337
  public key:3056301006072A8648CE3D020106052B8104000A03420004F838F3253A5224411D8951AA6EF2BB474EDD283EC088CD13D5404956C0A88079ECF539D9669A3D639A35BF9FD0F67ECBB3D332733C59B0272EB844405B6568D3
  ```

3. Create your wallet address based on the public key.
  This will return an bech32 address encoded from ripemd160(sha256(public_key))
  > bazel run //application/utxo/wallet_tool/py:addr \-\- 3056301006072A8648CE3D020106052B8104000A034200049C8FBD86EA4E38FD607CD3AC49FEB75E364B0C694EFB2E6DDD33ABED0BB1017575A79CC53EC6A052F839B4876E96FF9E4B08ECF23EC9CD495B82ECF9D95303BD
  ```
  address: bc1q09tk54hqfz5muzn9rgalfkdjfey8qpuhmzs5zn
  ```
  bazel run //application/utxo/wallet_tool/py:addr \-\- 3056301006072A8648CE3D020106052B8104000A03420004F838F3253A5224411D8951AA6EF2BB474EDD283EC088CD13D5404956C0A88079ECF539D9669A3D639A35BF9FD0F67ECBB3D332733C59B0272EB844405B6568D3
  ```
  address: bc1qd5ftrxa3vlsff5dl04nxg06ku6p4w6enk0cna9
  ```


## Start UTXO Service

Before starting the service, you need to assign a genesis coin to some addresses.

Modify the UTXO config to add coins.

> application/utxo/server/config/utxo_config.config

the field 'out' indicates the output of a UTXO. 

```
{
  genesis_transactions: {
    transactions: {
        out:  {
            address : "bc1q09tk54hqfz5muzn9rgalfkdjfey8qpuhmzs5zn",
            value : 1000
            pub_key: "3056301006072A8648CE3D020106052B8104000A034200049C8FBD86EA4E38FD607CD3AC49FEB75E364B0C694EFB2E6DDD33ABED    0BB1017575A79CC53EC6A052F839B4876E96FF9E4B08ECF23EC9CD495B82ECF9D95303BD"
        }
    }
  }
}
``` 

Once you added the genesis coins, start the UTXO service.
> ./application/utxo/server/start_contract_server.sh


## Transfer your coins

First, we need to build the tools

> bazel build application/utxo/wallet_tool/cpp/utxo_client_tools

Then, run the tools to transfer the coins
> bazel-bin/application/utxo/wallet_tool/cpp/utxo_client_tools -c application/utxo/wallet_tool/cpp/client_config.config -m transfer -t bc1qd5ftrxa3vlsff5dl04nxg06ku6p4w6enk0cna9 -d bc1q09tk54hqfz5muzn9rgalfkdjfey8qpuhmzs5zn -x 0 -v 100 -p 303E020100301006072A8648CE3D020106052B8104000A0427302502010104202CB99BBB2AFEB7F48A574064091B34F24781C93AD8181A511C8DCFB2A111AD82 -b 3056301006072A8648CE3D020106052B8104000A03420004F838F3253A5224411D8951AA6EF2BB474EDD283EC088CD13D5404956C0A88079ECF539D9669A3D639A35BF9FD0F67ECBB3D332733C59B0272EB844405B6568D3

  | -c | server config points to the client node |
  | -m | function |
  | -d | owner address |
  | -t | address to transfer |
  | -x | input transaction id |
  | -v | transfered values of coins |
  | -p | private key of the owner |
  | -b | public key of the delivered address |

  If it runs successfully, it returns the transaction id.
  ```
  E20230214 17:55:23.280972 39813 utxo_client_tools.cpp:61] execute result:
  1
  ```

## Get the transaction list

Obtain the transaction list from one of the replica nodes. The server config is different from the one used in the transfer, which uses the client one.

> bazel-bin/application/utxo/wallet_tool/cpp/utxo_client_tools -c application/utxo/wallet_tool/cpp/server_config0.config -m list -e -1 -n 5

  | -c | server config points to the server node |
  | -m | function |
  | -e | last transaction id, or -1 points to the last one |
  | -n | the number of utxos needed to be returned |

  ```
  {"in":[{}],"out":[{"address":"bc1qd5ftrxa3vlsff5dl04nxg06ku6p4w6enk0cna9","value":"100"}],"address":"bc1q09tk54hqfz5muzn9rgalfkdjfey8qpuhmzs5zn","transactionId":"1"}
  {"out":[{"address":"bc1q09tk54hqfz5muzn9rgalfkdjfey8qpuhmzs5zn","value":"1000","spent":true}]}
  ```

## Get the wallet value
> bazel-bin/application/utxo/wallet_tool/cpp/utxo_client_tools -c application/utxo/wallet_tool/cpp/server_config0.config -m wallet -t bc1qd5ftrxa3vlsff5dl04nxg06ku6p4w6enk0cna9

  | -c | server config points to the server node |
  | -m | function |
  | -t | wallet address to be fetched |

  ```
  E20230214 18:02:26.936648 41575 utxo_client_tools.cpp:76] address:bc1qd5ftrxa3vlsff5dl04nxg06ku6p4w6enk0cna9 get wallet value:100
  ```
