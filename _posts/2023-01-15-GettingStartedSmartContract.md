---
layout: article
title: Getting Started with Smart Contract on Nexres
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

Here we illustrate how to run a smart contract on Nexres locally. We provide steps by steps tutorials to set up locally with 4 nodes and use the builtin smart contract service.

# Install
Please check the [install tutorial](https://blog.resilientdb.com/2022/09/28/GettingStartedNexRes.html) for Nexres to install.

# Running Contract Service Locally
Running the setup script to start the server:
  > sh application/contract/server/start_contract_server.sh

When the script is done, you will see that 4 applications called contract_server have been launched locally. Now build the contract tool to help you access the server:
  > bazel build application/contract/tools/contract_tools

# Smart Contract Account

You have to provide an account address when you deploy a contract or execute your contract.
Using the contract tools to create an account first:
  > bazel-bin/application/contract/tools/contract_tools create -c application/contract/tools/client_config.config

# Contract
Nexres only handles the JSON description of the contract source code. We use solc, a tool from Solidity, to obtain the JSON file.
We provide [token.sol](https://github.com/msadoghi/nexres/blob/master/application/contract/tools/example_contract/token.sol) as an example below:
  > solc --evm-version homestead --combined-json bin,hashes --pretty-json --optimize token.sol > token.json

Once get your [json contract](https://github.com/msadoghi/nexres/blob/master/application/contract/tools/example_contract/token.json), you can find the contract name("token.sol:Token") and its function hashes under the contract name section in the file.

token.sol：
```
// Transfer tokens from the contract owner
contract Token {
  mapping (address => uint256) balances;

  event Transfer(address indexed _from, address indexed _to, uint256 _value);

  constructor(uint256 s) public {
    balances[msg.sender] = s;
  }

  // Get the account balance of another account with address _owner
  function balanceOf(address _owner) public view returns (uint256) {
    return balances[_owner];
  }

  // Send _value amount of tokens to address _to
  function transfer(address _to, uint256 _value) public returns (bool) {
    if (balances[msg.sender] >= _value) {
      balances[msg.sender] -= _value;
      balances[_to] += _value;
      emit Transfer(msg.sender, _to, _value);
      return true;
    }
    else
    {
      return false;
    }
  }
}
```

token.json
```
{
  "contracts":
  {
    "token.sol:Token":
    {
      "bin": "608060405234801561001057600080fd5b506040516101fe3803806101fe8339818101604052602081101561003357600080fd5b5051336000908152602081905260409020556101aa806100546000396000f3fe608060405234801561001057600080fd5b5060043610610052577c0100000000000000000000000000000000000000000000000000000000600035046370a082318114610057578063a9059cbb1461008f575b600080fd5b61007d6004803603602081101561006d57600080fd5b5035600160a060020a03166100cf565b60408051918252519081900360200190f35b6100bb600480360360408110156100a557600080fd5b50600160a060020a0381351690602001356100ea565b604080519115158252519081900360200190f35b600160a060020a031660009081526020819052604090205490565b33600090815260208190526040812054821161016b573360008181526020818152604080832080548790039055600160a060020a03871680845292819020805487019055805186815290519293927fddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef929181900390910190a350600161016f565b5060005b9291505056fea265627a7a72315820600579dee5d66ea3489085c4cb41116045ba977cb4804caac8d5f24e248e337064736f6c63430005100032",
      "hashes":
      {
        "balanceOf(address)": "70a08231",
        "transfer(address,uint256)": "a9059cbb"
      }
    }
  },
  "version": "0.5.16+commit.9c3226ce.Linux.g++"
}
```

# Deploy Contract
Once you obtain the JSON contract and its contract name, deploy the contract using contract_tools.
  > bazel-bin/application/contract/tools/contract_tools deploy -c application/contract/tools/client_config.config -p application/contract/client/test_data/token.json -n token.sol:Token -a 1000 -m 0x67c6697351ff4aec29cdbaabf2fbe3467cc254f8

Some parameters:
  > -c the client configuration path  
  > -p contract path   
  > -n contract name   
  > -a parameters to construct the contract object  
  > -m the contract owner's address

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
Running Create Account to get more addresses if needed.
The following command runs a transfer function to transfer 100 tokens to an account with the address "0x1be8e78d765a2e63339fc99a663".
> bazel-bin/application/contract/tools/contract_tools ecute -c application/contract/tools/client_config.config -m 0x67c6697351ff4aec29cdbaabf2fbe3467cc254f8 -s 0xfc08e5bfebdcf7bb4cf5aafc29be03c1d53898f1 -f "transfer(address,uint256)" -a 0x1be8e78d765a2e63339fc99a66320db73158a35a,100

Once it is done, you will see the result:
> 0x0000000000000000000000000000000000000000000000000000000000000001

Some parameters:
  > -c the client configuration path  
  > -m the contract owner's address  
  > -a parameters to construct the contract object  
  > -s the contract address  
  > -f contract function name(obtain from the JSON file)  


