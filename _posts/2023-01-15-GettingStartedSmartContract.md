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
    src: /assets/images/resdb-gettingstarted/diving-start.jpg

---

Here we illustrate how to run smart contract on Nexres locally. We provide steps by steps tutorial to set up locally with 4 nodes and using the builtin smart contract service.

# Install
Please check the [install tutorial](https://blog.resilientdb.com/2022/09/28/GettingStartedNexRes.html) for Nexres to install.

# Running Contract Service Locally
Running the set up script to start the server:
  > sh application/contract/server/start_contract_server.sh

When the script is done, you will see that 4 applications called contract_server have been lauched locally.Now build the contract tool to help you access the server:
  > bazel build application/contract/tools/contract_tools

Now everything is ready. It is time to test the kv server. First get a key with empty value:
  > bazel-bin/example/kv_server_tools example/kv_client_config.config get test

# Smart Contract Account

You have to provide an account address when you deploy a contract or execute your contract.
Using the contract tools to create an accout first:
  > bazel-bin/application/contract/tools/contract_tools create -c application/contract/tools/client_config.config

# Contract
Nexres only handles the json description on the contract source code. Using solc, a tool from Solidity, to obtain the json file.
We provide [token.sol](application/contract/tools/example_contract) as an example below:
  > solc --evm-version homestead --combined-json bin,hashes --pretty-json --optimize token.sol > token.json

Once get your json contract, you can find the contract name("token.sol:Token") and its function hashes under the contract name section in the file.

# Deploy Contract
Once you obtain the json contract and its contract name, deploy the contract using contract_tools.
  > bazel-bin/application/contract/tools/contract_tools deploy -c application/contract/tools/client_config.config -p application/contract/client/test_data/token.json -n token.sol:Token -a 1000 -m 0x67c6697351ff4aec29cdbaabf2fbe3467cc254f8

Some parameters:
  > -c the client configuation path
  > -p contract path
  > -n contract name
  > -a parameters to construct the contract object
  > -m the contract owner address

Parameters in the example above:
  > "token.sol:Token" is the contract name obtained from the json description. 
  > "0x67c6697351ff4aec29cdbaabf2fbe3467cc254f8" is your account address. 
  > "1000" is the parameter to construct the contract object.

Once it is done, you will see the output including the contract address which you need to provide when you want to execute the functions:
>
> owner_address: "0x67c6697351ff4aec29cdbaabf2fbe3467cc254f8"
> contract_address: "0xfc08e5bfebdcf7bb4cf5aafc29be03c1d53898f1"
> contract_name: "token.sol:Token"

# Execute Contract
When executing the contract functions, you need to provide the caller address, contract address, function name and its parameters.
Running Creat Account to get more address if needed.
The follow command runs a transfer function to transfer 100 token to an account with the address "0x1be8e78d765a2e63339fc99a663".
> bazel-bin/application/contract/tools/contract_tools ecute -c application/contract/tools/client_config.config -m 0x67c6697351ff4aec29cdbaabf2fbe3467cc254f8 -s 0xfc08e5bfebdcf7bb4cf5aafc29be03c1d53898f1 -f "transfer(address,uint256)" -a 0x1be8e78d765a2e63339fc99a66320db73158a35a,100

Once it is done, you will see the result:
> 0x0000000000000000000000000000000000000000000000000000000000000001

Some parameters:
  > -c the client configuation path
  > -m the contract owner address
  > -a parameters to construct the contract object
  > -s the contract address
  > -f contract function name(obtain from the json file)

