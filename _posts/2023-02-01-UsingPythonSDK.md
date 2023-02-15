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