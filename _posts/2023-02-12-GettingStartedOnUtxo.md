---
layout: article
title: UtxoOnNexres
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

## Create Wallet

Begining using UTXO transactions on Nexres, we need to create a wallet by generating a private key, a public key, 
and an address corresponding to your wallet.
Wallet addresses are encoded by bech32 which is also used for Bitcoin.

These are some steps to build your private-public keys and your address:

1. Go to work space folder which contains the WORKSPACE file
   Because the addresses are encoded by bech32, make sure bech32 has been installed.
   > pip install bech32

2. Create your own private key. It will return a private key and a public key, encoded by ECDSA.
  > bazel run //application/utxo/wallet_tool/py:keys

  ```
  private key:303E020100301006072A8648CE3D020106052B8104000A0427302502010104202CB99BBB2AFEB7F48A574064091B34F24781C93AD8181A511C8DCFB2A111AD82
  public key:3056301006072A8648CE3D020106052B8104000A034200049C8FBD86EA4E38FD607CD3AC49FEB75E364B0C694EFB2E6DDD33ABED0BB1017575A79CC53EC6A052F839B4876E96FF9E4B08ECF23EC9CD495B82ECF9D95303BD
  ```

3. Create your wallet address based on your public key
  > bazel run //application/utxo/wallet_tool/py:addr -- 3056301006072A8648CE3D020106052B8104000A034200049C8FBD86EA4E38FD607CD3AC49FEB75E364B0C694EFB2E6DDD33ABED0BB1017575A79CC53EC6A052F839B4876E96FF9E4B08ECF23EC9CD495B82ECF9D95303BD

  This will return an bech32 address encoded from ripemd160(sha256(public_key))

  ```
  address: bc1q09tk54hqfz5muzn9rgalfkdjfey8qpuhmzs5zn
  ```


## Start UTXO Service

Before start the service, you need to assign a genesis coin to some address.
Modify the utxo config to add coins.

application/utxo/server/config/utxo_config.config

the field 'out' indicates the output of a utxo. 

'''
{
  genesis_transactions: {
    transactions: {
        out:  {
            address : "0001",
            value : 1000
        }
    }
  }
}
'''

Once you added the genesis coins, start the utxo service.
> ./application/utxo/server/start_contract_server.sh


## Transfer your coins

First we need build the tools

> bazel build application/utxo/wallet_tool/cpp/utxo_client_tools

Then, run the tools to transfer the coins
> bazel-bin/application/utxo/wallet_tool/cpp/utxo_client_tools -c application/utxo/wallet_tool/cpp/client_config.config -m transfer -d 0002 -t "0002" -x 1 -v 100

``
  c server config points to the client node
  m function
  d wallet address
  t address to transfer
  x input transaction id
  v transfer values of coins
``

## Get the transaction list

build the tools

> bazel build application/utxo/wallet_tool/cpp/utxo_client_tools

Obtain the transaction list from one of the replica node. The server config is different from the one used in transfer, which uses the client one.

> bazel-bin/application/utxo/wallet_tool/cpp/utxo_client_tools -c application/utxo/wallet_tool/cpp/server_config0.config -m list -e -1 -n 5

## Get the wallet value
> bazel-bin/application/utxo/wallet_tool/cpp/utxo_client_tools -c application/utxo/wallet_tool/cpp/server_config0.config -m wallet -t 0002
