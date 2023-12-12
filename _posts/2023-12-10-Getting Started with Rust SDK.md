---
layout: article
title: Getting Started with Rust SDK 
author: Dhruv Sangamwar
tags: Rust SDK ResilientDBGraphQL
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

We provide a Rust SDK for committing transactions to create and transfer assets. Despite NexRes being written in C++, users can interface with ResilientDB through Rust. This article assumes you have setup a ResilientDB instance that is accessible through HTTP requests and are familiar with Rust, we will walk you through basics of using ResilientDB in your first Rust project.

# Prerequisites
### Install Rust and Cargo 
The easiest way to get Cargo is to install the current stable release of Rust by using rustup. Installing Rust using rustup will also install cargo.
On Linux and macOS systems, this is done as follows:

> `curl https://sh.rustup.rs -sSf | sh`

It will download a script, and start the installation. If everything goes well, you’ll see this appear:

> `Rust is installed now. Great!`

On Windows, download and run `rustup-init.exe`. It will start the installation in a console and present the above message on success.

After this, you can use the rustup command to also install beta or nightly channels for Rust and Cargo.

You can find more informatin on rust on [The Cargo Book](https://doc.rust-lang.org/cargo/getting-started/installation.html).

### Starting a new Rust Project and installing the SDK
You can start a new rust project through cargo with the following command: 
> `cargo new resilientDB_test`

You should now see a new folder generated named `resilientDB_test`.
```
$ cd resilientDB_test
$ tree .
.
├── Cargo.toml
└── src
    └── main.rs

1 directory, 2 files
``` 
You can get started with the Rust SDK in one of two ways: 
1. You can simply run the following command:
   ```
   cargo add resilientdb_rust_sdk
   ```
2. You can also add the following line to your `Cargo.toml`
   ```
   resilientdb_rust_sdk = "0.1.1"
   ```

It's that easy! You are now good to go with our SDK. Make sure you have a ResilientDB instance setup, for more info on that follow this [blog](https://blog.resilientdb.com/2023/11/22/Deploying-ResilientDB.html). This tutorial also makes a graphql query to resdb, if you don't have graphql setup take a look at this other [blog]().

### Dependencies

You may need to install the following dependencies as our library has `future` return types, meaning some returns from our APIs would need to be unwrapped within an `async` function. 

This is by design as we encourage users to handle errors on their end as many of the calls are made asynchrounously. Following good error handling practices would be beneficial to developing a secure application with ResilientDB. You can run obtain the dependencies through: 

1. Running the following command
   > `cargo add tokio serde serde_json reqwest`
   
2. Adding this to your Cargo.toml file:
   ```rust
    tokio = { version = "1", features = ["full"] }
    serde = { version = "1.0", features = ["derive"] }
    serde_json = "1.0"
    reqwest = {version = "0.11.22", features = ["blocking", "json"]}
   ```

# Developement
### Import the SDK

Let's build a sample program that makes a transactions and then fetches transactions to and from your ResilientDB instance. In your `main.rs` file import the SDK like so: 
```rust
use resilientdb_rust_sdk::ResDB;
``` 

### Initializing a ResDB object

Our first step to using the functions within the SDK is to create a ResDB object in main like so:
```rust
fn main() {
    let res_db = ResDB::new();

}
```

### Creating Keypairs

Generating public/private keypairs is as easy as calling the `generate_kepair()` function provided in the crypto module of our sdk. The Key pair generation is done using the Ed25519 algorithm. Here's how you would use this API:

```rust
let keypair = res_db.generate_keypair();
println!("Public Key: {:?}", keypair.0);
println!("Private Key: {:?}", keypair.1);
```

### Transaction API - Committing

Currently the SDK support transaction commits through the graphql format. In our case, we will use the public and private key pairs that we generated using `generate_keypair()` function above. In the future we plan to add support for Struct and Hash map based transaction commits. Here's how you would commit a transaction.
```rust

let data = format!("""r#"
{{}}{{
    {{}}"{{}}query{{}}"{{}}: "mutation { postTransaction(data: {{\noperation: \"CREATE\"\namount: 173812\nsignerPublicKey: \"{{}}\"\nsignerPrivateKey: \"{{}}\"\nrecipientPublicKey: \"ECJksQuF9UWi3DPCYvQqJPjF6BqSbXrnDiXUjdiVvkyH\"\nasset: \"\"\"{{\n            \"data\": {{ \"time\": 173812\n            }},\n          }}\"\"\"\n      }}) {{\n  id\n  }}\n}}\n"
}}
"#""", keypair.0, keypair.1);


let endpoint = "http://YouResDBInstance.com";
match res_db.post_transaction_string(&data, endpoint).await {
    Ok(body) => println!("{}", body),
    Err(err) => eprintln!("Error: {}", err),
}
```


### Transaction API - Fetching

We can fetch the transaction we just posted either using a struct to guide are JSON schema or a untyped Hash map. You would need to define a struct like so to pass into the `get_all_transactions()` functions:
```rust
use serde::Deserialize;
/** User Defined Struct for transaction endpoints **/ 

#[derive(Debug, serde::Deserialize)]
pub struct Transaction {
    pub inputs: Vec<Input>,
    pub outputs: Vec<Output>,
    pub operation: String,
    pub metadata: Option<serde_json::Value>,
    pub asset: Asset,
    pub version: String,
    pub id: String,
}

impl Default for Transaction {
    fn default() -> Self {
        Transaction {
            inputs: Vec::new(),
            outputs: Vec::new(),
            operation: String::new(),
            metadata: None,
            asset: Asset::default(),
            version: String::new(),
            id: String::new(),
        }
    }
}

#[derive(Debug, serde::Deserialize)]
pub struct Input {
    pub owners_before: Vec<String>,
    pub fulfills: Option<serde_json::Value>,
    pub fulfillment: String,
}

impl Default for Input {
    fn default() -> Self {
        Input {
            owners_before: Vec::new(),
            fulfills: None,
            fulfillment: String::new(),
        }
    }
}

#[derive(Debug, serde::Deserialize)]
pub struct Output {
    pub public_keys: Vec<String>,
    pub condition: Condition,
    pub amount: String,
}

impl Default for Output {
    fn default() -> Self {
        Output {
            public_keys: Vec::new(),
            condition: Condition::default(),
            amount: String::new(),
        }
    }
}

#[derive(Debug, serde::Deserialize)]
pub struct Condition {
    pub details: ConditionDetails,
    pub uri: String,
}

impl Default for Condition {
    fn default() -> Self {
        Condition {
            details: ConditionDetails::default(),
            uri: String::new(),
        }
    }
}

#[derive(Debug, serde::Deserialize)]
pub struct ConditionDetails {
    #[serde(rename = "type")]
    pub condition_type: String,
    pub public_key: String,
}

impl Default for ConditionDetails {
    fn default() -> Self {
        ConditionDetails {
            condition_type: String::new(),
            public_key: String::new(),
        }
    }
}

#[derive(Debug, serde::Deserialize)]
pub struct Asset {
    pub data: serde_json::Value,
}

impl Default for Asset {
    fn default() -> Self {
        Asset {
            data: serde_json::Value::Null,
        }
    }
}
```
Now that we have defined the JSON through structs, we can pass them to the `get_all_transactions()` function, along with that it will take our endpoint URL - this will be the local or remote URL to your ResilientDB instance.
```rust
async fn fetch_transactions(res_db: &ResDB) {
match res_db.get_all_transactions::<Transaction>("http://YouResDBInstance.com").await {
    Ok(transactions) => {
        for transaction in transactions {
            // Access and process each transaction
            println!("{:?}", transaction);
        }
    }
    Err(error) => {
        // Handle the error
        eprintln!("Error fetching transactions: {}", error);
    }
}
}
```

The struct defined may seem verbose, this is by design as we want to ensure that users are building type safe and fault tolerant applications using ResilientDB. While the struct defines the JSON schema it also comes with `impl` that constructors for the stucts. For those who want to interface with ResilientDB in an unstructured manner we also provide the Hash map varient of the same function.

```rust
    let data_map = HashMap::new();
    let res_db = ResDB::new();
    
    match res_db.get_all_transactions_map(
        "http://YouResDBInstance.com",
        data_map,
    )
    .await
    {
        Ok(map) => {
            // Access fields of the transaction (replace with the desired field)
            println!("{:?}", map[10]);
        }
        Err(error) => {
            // Handle the error
            eprintln!("Error: {}", error);
        }
    }
```

All the retrival functions provided by the SDK come in both Struct and Hash Map varients to provide the you with the choice between safety, type strictness and flexibility. 

### Blocks API

Similarly, the SDK offers APIs for interacting with blocks. You can fetch blocks by range, group them, or retrieve all blocks from a specified API endpoint. Below is an example of fetching all blocks:

```rust
let res_db = ResDB::new();
let data_map = HashMap::new();

// Call the asynchronous function to get blocks
match res_db
    .get_all_blocks_map("http://YouResDBInstance.com", data_map)
    .await
{
    Ok(blocks) => {
        if let Some(first_block) = blocks.first() {
            // Access fields of the first transaction (replace with the desired field)
            println!("{:?}", first_block);
        } else {
            println!("No transactions available");
        }
    }
    Err(error) => {
        // Handle the error
        eprintln!("Error: {}", error);
    }
}
```

### Crypto operations

Our Crypto API also provides a convinient function to hash strings. Here is how you would use the function:

```rust
let data = "Hello, ResilientDB!";
let hashed_data = res_db.hash_data(data);
println!("Hashed Data: {}", hashed_data);
```

# Conclusion

This guide provides a glimpse into the powerful features our ResilientDB Rust SDK offers for seamless integration into your Rust projects. Explore the full potential of ResilientDB, experiment with different APIs, and leverage the SDK's capabilities to build robust and secure applications.

For more in-depth information and advanced features, refer to the official [ResilientDB Rust SDK documentation](https://docs.rs/resilientdb_rust_sdk/0.1.1/resilientdb_rust_sdk/).

Happy coding with ResilientDB and Rust!