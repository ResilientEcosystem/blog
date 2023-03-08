---
layout: article
title: Using the NexRes Python SDK
author: Glenn Chen, Julieta Duarte, Arindaam Roy
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

We provide a Python SDK for committing transactions to create and transfer assets. Despite NexRes being written in C++, the code for validating transactions is written in Python so that we can use the cryptoconditions library, which provides functionalities not available in any widely distributed C++ libraries currently.

# Install
Please check the [install tutorial](https://blog.resilientdb.com/2022/09/28/GettingStartedNexRes.html) to install NexRes.

After this you will need some additional steps for [pybind11](https://github.com/pybind/pybind11) to work, as we are embedding the Python interpreter in the C++ code.

Make sure your Python version is 3.9+. Check the version with
  > python3 --version

In the .bazelrc file in your nexres directory, have the PYTHON_BIN_PATH reference the location of your python executable. For example, if you have Python installed at /home/ubuntu/.linuxbrew/bin/python3, then it will look like
  > build --action_env=PYTHON_BIN_PATH="/home/ubuntu/.linuxbrew/bin/python3"

Then install the Python dev library for your corresponding Python version. These are used when the C++ binary is run.
  > sudo apt-get install python3.10-dev

If apt cannot find the dev library, you might not have deadsnakes added as a source. You can use the following commands to check your sources and add if needed:
  > ls /etc/apt/sources.list.d

  > sudo add-apt-repository ppa:deadsnakes/ppa

# Setting Up Virtual Environment
It is heavily advised to set up a virtual Python environment so you do not disturb your system's Python settings.

  > python3 -m venv venv

  > source venv/bin/activate

Then install the Python dependencies

  > pip install -r sdk_validator/requirements.txt

If you wish to deactivate the virtual environment you can enter
  > deactivate

# Running NexRes KV Servers
NexRes needs to be running first for the SDK endpoints to connect to. To enable the Python transaction validation, go to example/kv_config.config and set the flag require_txn_validation to true. The file should look something like:

    {
      region : {
        ...
      },
      self_region_id:1,
      rocksdb_info : {
        ...
      },
      leveldb_info : {
        ...
      },
      require_txn_validation:true,
    }
Make sure the Python virtual environment is activated if it is not already. You will see a (venv) on the left of your command line if it is active.
  > source venv/bin/activate

Start the KV servers with the example script. This script uses the example/kv_config.config file.
  > sh example/start_kv_server.sh

# Running Crow Service
We use [Crow](https://github.com/CrowCpp/Crow), a C++ framework for creating HTTP or Websocket web services to connect our SDK to NexRes.

In another terminal shell after starting KV Server, build the crow service: 
  > bazel build sdk_client/crow_service_main

Run the binary to start the service:
  > bazel-bin/sdk_client/crow_service_main example/kv_client_config.config sdk_client/server_config.config

You will see this if successful: 
  ```
  (2022-12-19 06:12:02) [INFO    ] Crow/master server is running at http://0.0.0.0:18000 using 16 threads
  (2022-12-19 06:12:02) [INFO    ] Call `app.loglevel(crow::LogLevel::Warning)` to hide Info level logs
  ```

# Running the SDK
The SDK is another repository outside NexRes and can be run on another machine if you wish. To get started:

## Check Python is up-to-date (3.9+)
`$ python3 --version`

If your Python version number is too low you may encounter type hinting issues when attempting to run the code

## Installing Dependencies
`$ python3 -m venv venv`

`$ source venv/bin/activate`

`$ pip install -r requirements.txt`

## Running the Driver

Examples of using the driver can be seen in test_driver.py

Replace the db_root_url with the url of the NexRes SDK endpoints e.g `127.0.0.1:18000`


# Validation
## Entrypoint
`validator.py`
    call the `is_valid_tx(tx_dict)` function with the transaction json (tx_dict) as the argument.

## Transaction Validation Rules
A transaction is said to be valid if it satisfies certain condtions or rules.

We employ a simpler version of the transaction spec and validation rules specified by [BigchainDB](https://github.com/bigchaindb/BEPs/tree/master/13#transaction-validation)

#### JSON Schema Validation
The json structure of a transaction should be the transaction spec v2 of BigchainDB 

#### The output.amount Rule
For all output.amount must be an integer between 1 and 9×10^18, inclusive. The reason for the upper bound is to keep amount within what a server can comfortably represent using a 64-bit signed integer, i.e. 9×10^18 is less than 2^63.

#### The Duplicate Transaction Rule
If a transaction is a duplicate of a previous transaction, then it’s invalid. A quick way to check that is by checking to see if a transaction with the same transaction ID is already stored.
A transaction ID is the hash of the transaction.

####  The TRANSFER Transaction Rules
if a transaction is a `TRANSFER` transaction:

- TODO: If an input attempts to fulfill an output that has already been fulfilled (i.e. spent or transferred) by a previous valid transaction, then the transaction is invalid. (You don’t have to check if the fulfillment string is valid.)
- If two or more inputs (in the transaction being validated) attempt to fulfill the same output, then the transaction is invalid. (You don’t have to check any fulfillment strings.)
-  The sum of the amounts on the inputs must equal the sum of the amounts on the outputs. In other words, a TRANSFER transaction can’t create or destroy asset shares.
    
For all inputs, if input.fulfills points to:
- a transaction that doesn’t exist, then it’s invalid.
- a transaction that’s invalid, then it’s invalid. (This check may be skipped if invalid transactions are never kept.)
- a transaction output that doesn’t exist, then it’s invalid.
- a transaction with an asset ID that’s different from this transaction’s asset ID, then this transaction is invalid. (The asset ID of a CREATE transaction is the same as the transaction ID. The asset ID of a TRANSFER transaction is asset.id.)
Note: The first two rules prevent double spending.

#### The input.fulfillment Rule
Regardless of whether the transaction is a CREATE or TRANSFER transaction: For all inputs, input.fulfillment must be valid.

# Transactions

Transaction structure:

```bash
{
    "id": id,
    "version": version,
    "inputs": inputs,
    "outputs": outputs,
    "operation": operation,
    "asset": asset,
    "metadata": metadata
}
```

- Tx ID: The ID of a transaction is the SHA3-256 hash of the transaction, loosely speaking. It’s a string. An example is:

`"0e7a9a9047fdf39eb5ead7170ec412c6bffdbe8d7888966584b4014863e03518"`

- Version: 2.0 (TODO: remove)
- Inputs
    
    _List_ of tx inputs
    
    - Each tx input spends a previous tx output
    - a CREATE tx must have exactly one input
    - a TRANSFER tx should have at least one input
    
    - Transaction inputs and outputs are the mechanism by which control or ownership of an asset
    - Amounts of an asset are encoded in the outputs of a transaction, and each output may be spent separately
    - To spend an output, the output’s condition must be met by an input that provides a corresponding fulfillment
    - Each output may be spent at most once, by a single input
    
    Example of a structure of an element in the input list
    ```bash
    {
        "fulfills": {
            "transaction_id": transaction_id,
            "output_index": output_index
        },
        "owners_before": [public_key_1, public_key_2, etc.],
        "fulfillment": fulfillment
    }
    ```
    
    > For create tx, the value for fulfills is `None`
    
    - Owners_before: public keys 
    
    fulfillment: a str as per crypto conditions spec
    
    - The basic steps to compute a fulfillment string are:
    
      1. Construct the fulfillment as per the crypto-conditions spec.
      2. Encode the fulfillment to bytes using the [ASN.1 Distinguished Encoding Rules (DER)](http://www.itu.int/ITU-T/recommendations/rec.aspx?rec=12483&lang=en)
      3. Encode the resulting bytes using “base64url” (*not* typical base64) as per [RFC 4648, Section 5](https://tools.ietf.org/html/rfc4648#section-5)
  
- Outputs
    
    list of Tx outputs 
    
    Each output indicates the crypto-conditions which must be satisfied by anyone wishing to spend/transfer that output. It also indicates the number of shares of the asset tied to that output.
    
    output eg:
    
    ```bash
    {
        "condition": condition,
        "public_keys": [public_key_1, public_key_2, etc.],
        "amount": amount
    }
    ```
    
    Condition:  its a list or array 
    
    ```bash
    {
        "details": subcondition,
        "uri": uri
    }
    ```
    
    subconditions:
    
    1. ED25519-SHA-256 (We only care about this for NFTs!) 
    2. THRESHOLD-SHA-256
    
    ```bash
    {
        "type": "ed25519-sha-256",
        "public_key": "HFp773FH21sPFrn4y8wX3Ddrkzhqy4La4cQLfePT2vz7"
    }
    ```
    
    uri: [cost](https://datatracker.ietf.org/doc/html/draft-thomas-crypto-conditions-03#section-7.2.2)
    
    ```bash
    "uri": "ni:///sha-256;at0MY6Ye8yvidsgL9FrnKmsVzX0XrNNXFmuAPF4bQeU?fpt=ed25519-sha-256&cost=131072"
    ```
    
    Code to compute the uri
    
    ```bash
    import base58
    from cryptoconditions import Ed25519Sha256
    
    pubkey = 'HFp773FH21sPFrn4y8wX3Ddrkzhqy4La4cQLfePT2vz7'
    
    # Convert pubkey to a bytes representation (a Python 3 bytes object)
    pubkey_bytes = base58.b58decode(pubkey)
    
    # Construct the condition object
    ed25519 = Ed25519Sha256(public_key=pubkey_bytes)
    
    # Compute the condition uri (string)
    uri = ed25519.condition_uri
    # uri should be:
    # 'ni:///sha-256;at0MY6Ye8yvidsgL9FrnKmsVzX0XrNNXFmuAPF4bQeU?fpt=ed25519-sha-256&cost=131072'
    ```
    
- Asset
    
    For a CREATE Tx
    
    ```bash
    {
        "data": {
            "desc": "Laundromat Fantastique",
            "address": "461B Grand Palace Road",
            "international_laundromat_identifier": "bx45-am-333",
            "known_issues": "No known issues. It's fantastique!"
        }
    }
    ```
    
    it should just have the `data` key
    
    For TRANSFER tx 
    
    the asset key will have: The id of the tx which has the asset
    
    ```bash
    {
        "id": "38100137cea87fb9bd751e2372abb2c73e7d5bcf39d940a5516a324d9c7fb88d"
    }
    ```
    
- metadata
    
    Bust be any valid associative array or dict (in python) or Null.
    
    ```bash
    {
        "timestamp": "1510850314",
        "weather_conditions": "So hot that our crayons melted.",
        "location": {
            "name": "Death Valley, California",
            "latitude": "36.457N",
            "longitude": "116.865W"
        }
    }
    ```

> **_NOTE:_**  We use the bigchainDB transaction spec v2.

## Constructing a Transaction

1. Set a variable named `version` to a [valid version value](#). (We need to remove this) 
2. Set a variable named `operation` to a [valid operation value](#).
3. Set a variable named `asset` to a [valid asset value](#).
4. Set a variable named `metadata` to a [valid metadata value](#).
5. Generate or get all the required [public keys](#) 
6. Construct a [list](#) named `outputs` of all the [outputs](#) that should be in the transaction. (Note: Each output includes a [condition](https://github.com/bigchaindb/BEPs/tree/master/13#transaction-components-conditions).)
7. Construct a [list](#) named `unfulfilled_inputs` of all the [inputs](#) that should be in the transaction. All *fulfillment* strings should be set to  `None`  (We’re building an “unfulfilled transaction” first.)
8. Construct an [associative array](#)  (dict) named `unfulfilled_tx` of the form:
    
    `{
        "id": null,
        "version": version,
        "inputs": unfulfilled_inputs,
        "outputs": outputs,
        "operation": operation,
        "asset": asset,
        "metadata": metadata
    }`
    
    Note how `unfulfilled_tx` includes a key-value pair for the `"id"` key. The value must be your [ctnull](#) (e.g. `None` in Python).
    
9. Convert unfulfilled_tx to a serialized json named `utx_json`.
    1. unicode, sorted keys
    
    ```bash
    import rapidjson
    
    # input_dict is a dictionary
    json_str = rapidjson.dumps(input_dict,
                                skipkeys=False,
                                ensure_ascii=False,
                                sort_keys=True)
    ```
    
10. Create `inputs` as a deep copy of `unfulfilled_inputs`.
11. For each input in `inputs`:
    1. If `fulfills` is `None` (because this is a CREATE transaction, for example), then let `string1 = utx_json`, otherwise let `string1 = utx_json + output_tx_id + output_index` where `output_tx_id` is the transaction ID of the output that this input fulfills and `+` means concatenate the strings.
    2. [Convert string1 to bytes](#) and call the result `bytes1`.
    3. [Compute the SHA3-256 hash](#) of `bytes1` and leave the result as bytes (i.e. don’t convert to a hex string). Call the result `bytes_to_sign`.
    4. fulfill the associated crypto-condition [using an implementation of crypto-conditions](https://github.com/rfcs/crypto-conditions#implementations). You will need `bytes_to_sign` and one or more private keys (which are used to sign `bytes_to_sign`). The end result is usually some kind of fulfilled condition object. Compute the fulfillment string of that fulfilled condition object, and put that as the value of `"fulfillment"` for the input in question.
12. Construct a new [associative array](#) `tx` by making a deep copy of `unfulfilled_tx`.
13. In `tx`, change the value of `"inputs"` to the just-computed `inputs` (an array of fulfilled inputs).
14. [Compute the transaction ID of tx](#). Call it `id`.

The final result (`tx`) is a valid fulfilled transaction (in the form of an associative array). To put it in the body of an HTTP POST request, you’ll have to [convert it to a JSON string](#).