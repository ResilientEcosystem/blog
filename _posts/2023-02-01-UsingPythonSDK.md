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
    src: /assets/images/resdb-gettingstarted/diving-start.jpg

---


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

# Validation

The code for validating transactions is written in Python so that we can use the cryptoconditions library, which provides functionalities not available in any widely distributed C++ libraries currently.