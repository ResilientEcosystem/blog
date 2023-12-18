---
layout: article
title: Using the ResilientDB TypeScript SDK
author: Christopher Chen, Jamie Hsi, Samuel Li, Shrey Gupta, Tyler Beer
tags: SDK ResilientDBGraphQL TypeScript Installation
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

Introduction to ResilientDB TypeScript SDK: Building Robust Applications

The digital era demands robust solutions that can handle data with precision, security, and flexibility. Blockchain technology is championing this revolution, and at the heart of many innovative applications is ResilientDB, a versatile distributed ledger that offers both trust and transparency.

In this in-depth look, we will explore the ResilientDB TypeScript SDK, an essential toolkit for developers seeking to leverage the ResilientDB platform in their JavaScript and TypeScript applications. Through the integration of GraphQL, the SDK simplifies interactions with the ResilientDB server, enabling easy queries and mutations to manage transactions effectively.

## Table of Contents
- [**What Is ResilientDB?**](#what-is-resilientdb)
- [**Prerequisites**](#prerequisites)
- [**Installation**](#installation)
- [**SDK**](#introducing-the-sdk)
- [**Clients**](#clients)
- [**ResilientDB Client Methods**](#resilientdb-client-methods)
- [**Types**](#types)
- [**SDK Demo Application**](#sdk-demo-application)
- [**Use the SDK in 6 easy steps**](#use-the-sdk-in-6-easy-steps)
- [**Real-World Value and Implications**](#real-world-value-and-implications)
- [**Conclusion**](#conclusion)

## What Is ResilientDB?

ResilientDB is a distributed ledger infrastructure that emphasizes resilience, scalability, and decentralized control. It allows applications to perform transactions in a distributed environment with trust and transparency being paramount.

## NPM Package: `resilientdb-javascript-sdk`

Before diving deep into the codebase, let's address how to get the SDK into your project. The `resilientdb-javascript-sdk` is readily available on the NPM registry and can be included in your project with ease.

## Prerequisites

### Setup Node v18
Ensure you have Node v18 installed
```sh
>> node --version
v18.17.0
```

### Setup ResilientDB-GraphQL (Run Locally)
If you would like to use your own ResilientDB replica set, you must run the ResilientDB stack along with ResilientDB-GraphQL.
Instructions can be found [here](https://blog.resilientdb.com/2023/09/21/ResVault.html)
Otherwise, point the client to `https://cloud.resilientdb.com`.


### Installation

To install the SDK, run the following command in your project directory:

```sh
npm install resilientdb-javascript-sdk
```

## Introducing the SDK

The SDK comes with three main modules that form its core functionality. Each plays a significant role in ensuring a streamlined interface for communicating with the ResilientDB server.

### Clients

The `FetchClient` and `AxiosClient` are two interchangeable network clients that ResilientDB utilizes to make HTTP requests to the ResilientDB server.

#### FetchClient (`FetchClient.ts`)

A lightweight client using the Fetch API. It’s optimal for environments where size and simplicity are key.

Example of initializing a `FetchClient`:

```typescript
import { ResilientDB, FetchClient } from 'resilientdb-javascript-sdk';

const fetchClient = new FetchClient();
```

#### AxiosClient (`AxiosClient.ts`)

A feature-rich client based on `axios`. This client is suited for those who may need interceptors, progress indicators, or other advanced features provided by `axios`.

Example of initializing an `AxiosClient`:

```typescript
import { ResilientDB, AxiosClient } from 'resilientdb-javascript-sdk';

const axiosClient = new AxiosClient();
```

### ResilientDB Client (`ResilientDB.ts`)

The SDK’s main class provides a user-friendly interface to the ResilientDB server's GraphQL API. It includes methods for retrieving, filtering, and posting transactions, as well as generating cryptographic key pairs.

### Core Types (`types.ts`)

Defines the interfaces and types used throughout the SDK, ensuring type safety and developer experience.

## ResilientDB Client Methods

Here's a detailed look at each method provided by the `ResilientDB` client.

### Transaction Retrieval Methods

| Method                       | Input Parameters                            | Output                                        | Description                                                        |
|------------------------------|---------------------------------------------|-----------------------------------------------|--------------------------------------------------------------------|
| `getTransaction`             | `requestId: string`                         | `Promise<RetrieveTransaction>`                | Retrieves a single transaction by its unique ID.                    |
| `getAllTransactions`         | -                                           | `Promise<RetrieveTransaction[]>`              | Fetches all transactions.                                           |
| `getFilteredTransactions`    | `filter?: FilterKeys`                       | `Promise<RetrieveTransaction[]>`              | Retrieves transactions that match the given filtering criteria.     |

- [**FilterKeys**](#filterkeys): A type that includes optional fields for filtering transactions by `ownerPublicKey` and `recipientPublicKey`.

### Transaction Mutation Methods

| Method                       | Input Parameters                            | Output                                        | Description                                                        |
|------------------------------|---------------------------------------------|-----------------------------------------------|--------------------------------------------------------------------|
| `postTransaction`            | `transaction: PrepareAsset`                 | `Promise<CommitTransaction>`                  | Posts a new transaction to the ledger.                              |
| `updateTransaction`          | `transaction: UpdateAsset`                  | `Promise<RetrieveTransaction>`                | Updates an existing transaction.                                    |
| `updateMultipleTransaction`  | `transactions: UpdateAsset[]`               | `Promise<RetrieveTransaction[]>`              | Updates multiple transactions at once.                              |

- [**PrepareAsset**](#prepareasset): A type representing the necessary information to prepare a transaction for posting.
- [**UpdateAsset**](#updateasset): A type used for detailing the specifications required to update a transaction.
- [**CommitTransaction**](#committransaction): Represents the output of posting a new transaction, which includes, at minimum, the `id` of the committed transaction.

### Key Generation Method

| Method                      | Input Parameters                            | Output                                    | Description                                                        |
|-----------------------------|---------------------------------------------|-------------------------------------------|--------------------------------------------------------------------|
| `static generateKeys()`     | -                                           | `{ publicKey: string; privateKey: string}`| Generates a pair of public and private keys for signing transactions.|

## Types

### `NetworkClient`

```typescript
export interface NetworkClient {
  request<TReturn extends object>(options: {
    url: string,
    headers?: Record<string, string>
  } & (
      | {
        method: 'GET',
      }
      | {
        method: 'POST',
        body: string | object
      }
    )): Promise<TReturn>
}
```

### `RetrieveTransaction`

```typescript
type RetrieveTransaction = {
  id: string;
  version: string;
  amount: number; // integer
  uri: string;
  type: string;
  publicKey: string;
  operation: string;
  metadata?: string | null;
  asset: string;
}
```

### `CommitTransaction`

```typescript
type CommitTransaction = {
  id: string;
}
```

### `PrepareAsset`

```typescript
type PrepareAsset = {
  operation: "CREATE" | string;
  amount: number;
  signerPublicKey: string;
  signerPrivateKey: string;
  recipientPublicKey: string;
  asset: object;
}
```

### `UpdateAsset`

```typescript
type UpdateAsset = {
  id: string;
  operation?: string | null;
  amount?: number | null; // int
  signerPublicKey: string;
  signerPrivateKey: string;
  recipientPublicKey?: string | null;
  asset?: string | null;
}
```

### `FilterKeys`

```typescript
type FilterKeys = {
  ownerPublicKey?: string | null;
  recipientPublicKey?: string | null;
}
```

### `Keys`

```typescript
type Keys = {
  publicKey: string
  privateKey: string
}
```

## SDK Demo Application

Included within the SDK repository is a demo React application that showcases the SDK's capabilities. The app allows users to filter transactions, post new transactions, and explore the core features of the ResilientDB TypeScript SDK.

## Use the SDK in 6 easy steps

### Step 1: Project Setup and SDK Installation

Set up a new TypeScript project and install the `resilientdb-javascript-sdk` package as well as TypeScript and the necessary types for Node.js.

```sh
mkdir resilientdb-ts-example
cd resilientdb-ts-example
npm init -y
npm install resilientdb-javascript-sdk
npm install typescript ts-node @types/node --save-dev
```

Create `tsconfig.json`:

```json
{
  "compilerOptions": {
    "target": "es2018",
    "module": "commonjs",
    "strict": true,
    "esModuleInterop": true
  }
}
```

### Step 2: Generating Keys

In the `index.ts` file:

```typescript
import { ResilientDB } from 'resilientdb-javascript-sdk';

// Generate public and private keys
const { publicKey, privateKey } = ResilientDB.generateKeys();

console.log(`Public Key: ${publicKey}`);
console.log(`Private Key: ${privateKey}`);
```

### Step 3: Initializing the Client

Continue editing the `index.ts` to initialize the client:

```typescript
import { ResilientDB, FetchClient } from 'resilientdb-javascript-sdk';

// Initialize the client
const resilientDBClient = new ResilientDB("https://cloud.resilientdb.com", new FetchClient());
```

### Step 4: Fetching and Filtering Transactions

Add functions to fetch all transactions and filter based on certain criteria.

```typescript
// Fetch all transactions
async function getAllTransactions() {
  const transactions = await resilientDBClient.getAllTransactions();
  console.log('All Transactions:', transactions);
}

// Fetch transactions with filters
async function getFilteredTransactions() {
  const filter = {
    ownerPublicKey: publicKey,
    // recipientPublicKey can also be specified here.
  };
  const transactions = await resilientDBClient.getFilteredTransactions(filter);
  console.log('Filtered Transactions:', transactions);
}

getAllTransactions();
getFilteredTransactions();
```

### Step 5: Posting and Updating Transactions

Add functions to post a new transaction and then update it.

```typescript
// Post a new transaction
async function createTransaction() {
  const transactionData = {
    operation: "CREATE",
    amount: 100,
    signerPublicKey: publicKey,
    signerPrivateKey: privateKey,
    recipientPublicKey: publicKey, // For the sake of example, sending to self
    asset: { message: "Initial transaction" }
  };
  
  const transaction = await resilientDBClient.postTransaction(transactionData);
  console.log('Transaction posted:', transaction);
  
  return transaction.id; // We'll need the transaction ID to update it next
}

// Update the created transaction
async function updateTransaction(transactionId: string) {
  const updateData = {
    id: transactionId,
    amount: 150, // Updated amount
    signerPublicKey: publicKey,
    signerPrivateKey: privateKey,
    recipientPublicKey: publicKey, // Still sending to self
    asset: { message: "Updated transaction data" }
  };
  
  const updatedTransaction = await resilientDBClient.updateTransaction(updateData);
  console.log('Transaction updated:', updatedTransaction);
}

async function runDemo() {
  const transactionId = await createTransaction();
  await updateTransaction(transactionId);
}

runDemo();
```

### Step 6: Running the Project

You can now run the project from the terminal:

```sh
npx ts-node index.ts
```
## Real-World Value and Implications

By simplifying the integration and interaction with the ResilientDB server, the SDK opens the door to a plethora of applications. Whether you're developing a finance app that requires ledger capabilities or seeking the immutability of blockchain for asset tracking, the ResilientDB TypeScript SDK is a capable starting point.

The versatility in choosing between the `FetchClient` and the `AxiosClient` ensures developers have the liberty to pick a client that best suits their specific needs, whether they prioritize speed and minimalism or extensive features.

## Conclusion

The ResilientDB TypeScript SDK stands as a testament to the possibilities when modern web technologies meet blockchain principles. Through its approachable interface and adaptable architecture, the SDK demonstrates how applications can be built with resilience at their core, ready to scale and secure transactions across a distributed network.

Whether you're an experienced blockchain developer or a newcomer to distributed ledger technology, the SDK provides the necessary tools to interface with the ResilientDB platform with confidence and ease.
