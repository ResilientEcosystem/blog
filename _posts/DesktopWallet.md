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

**This is a blog to help get started with the Desktop Wallet for Tx/Rx built on the Resilient DB network**

## Table of Contents
- [**Connecting to Resilient DB**](#connecting-to-resilientdb)
- [**Building the Frontend**](#building-the-frontend)
- [**Securing User Accounts**](#securing-user-accounts)
- [**Familiarizing Yourself with Your Wallet**](#familiarizing-yourself-with-your-wallet)
- [**Future Plans for ResilientDB Desktop Wallet**](#familiarizing-yourself-with-your-wallet)

## Connecting to Resilient DB

There are a host of ways we can connect to the Resilient DB decentralized system, including

- Crow HTTP server
- Python SDK
- GraphQL
  
We chose the last option, GraphQL, as it it offers a high level and easy to implement API for a fast development cycle.

**Mini SDK and Third Party Libraries**

It’s hard to use raw HTTP requests for accessing GraphQL. It’s difficult to read and complicated to understand GraphQL documents.

To facilitate our development, we built a mini sdk using typescript and the open source library [graphql-request](https://www.npmjs.com/package/graphql-request).

**Mutations**

The primary mutation we implemented is `postTransaction`.

```typescript
postTransation = async (
    amount: number,
    recipientPublicKey: string,
    data?: object,
    metadata?: string
): Promise<string> => {
    data = data || {};

    data = {
        ...data,
        time: Date.now(),
    };

    data = {
        ...data,
        client: {
            name: appName,
            version: appVersion,
        },
    };
    const doc = gql`
        mutation { postTransaction(data: {
            operation: "CREATE"
            amount: ${amount}
            signerPublicKey: "${this.user.publicKey}"
            signerPrivateKey: "${this.user.privateKey}"
            recipientPublicKey: "${recipientPublicKey}"
            asset: """{
                        "data":  ${JSON.stringify(data)},
                    }"""
            }) 
            {
                id
            }
        }
    `;

    const resp: { postTransaction: PostTransactionResult } =
        await this.client.request(doc);
    return resp.postTransaction.id;
};
```
`postTransaction` is responsible for sending RoKs between user accounts.

Here we allow for the developer to pass in arguments related relevant for sending tokens.

- `amount` – the total number to tokens to send
- `recipientPublicKey` – the public key of the recipient
- `metadata` – optional metadata to pass in the transaction
- `data` – other data to pass into the asset field
  
**Queries**

While we did implement quries such as `getTransaction`, the primary method we use to retrieve transactions data is the `getFilteredTransactions` query.

```typescript
getFilteredTransactions = async (
    ownerPublicKey?: string,
    recipientPublicKey?: string
): Promise<FilteredTransaction[]> => {
    const doc = gql`
        query {getFilteredTransactions(filter: {
            ownerPublicKey: "${ownerPublicKey || ""}",
            recipientPublicKey: "${recipientPublicKey || ""}"
        }) {
            id
            version
            amount
            metadata
            operation
            asset
            publicKey
            uri
            type
        }}
    `;

    const resp: GetFilteredTransactionsResult = 
        await this.client.request(
            doc
        );

    return resp.getFilteredTransactions;
};
```
`getFilteredTransactions` allow us to retrieve the list of transactions either sent or received by our user.

We retrieve sent transactions by passing in the user’s public key as the `ownerPublicKey`.

Alternatively we retrieve transactions the user received by passing in their public key as the `recipientPublicKey`.

**Processing**

The GraphQL queries and transactions do not directly allow us to view the user’s wallet contents.

To do so we need additional processing.

```typescript
getPastTransactions = async (): Promise<WalletGetTransactionsResult> => {
    const transactionsSent =
        await this.graphqlClient.getFilteredTransactions(
            this.user.publicKey
        );
    const transactionsReceived =
        await this.graphqlClient.getFilteredTransactions(
            null,
            this.user.publicKey
        );
    return {
        transactionsSent,
        transactionsReceived,
    };
};
```
This utility function allows us to retrieve all of the users transactions.

We pass in the user’s public key as the `ownerPublicKey` and the `recipientPublicKey` separately to receive both transactions sent and received.

```typescript
getWalletContent = async () => {
    const { transactionsSent, transactionsReceived } =
        await this.getPastTransactions();

    return (
        transactionsReceived.reduce(
            (a, b) => ({ amount: a.amount + b.amount }),
            { amount: 0 }
        ).amount -
        transactionsSent.reduce(
            (a, b) => ({ amount: a.amount + b.amount }),
            { amount: 0 }
        ).amount
    );
};
```
To actually calculate the amount of RoKs in the user’s wallet:

We first add up the amount of RoKs received in all transactions, then subtract the amount of RoKs sent from the wallet to calculate the final amount.

**Summary**

We choose to use the GraphQL API in connecting to the Resilient DB instance due to it’s simplicity and use of use.

We created a Mini-SDK implementing functions for querying and mutating transactions.

We established a method for calculating the total amount of RoKs in the user’s wallet.

**Further Resources**

- [ResDB API](https://blog.resilientdb.com/2023/09/21/ResVault.html)

- [GraphQL](https://graphql.org/)

- [Playground](https://cloud.resilientdb.com/graphql)
  

## Building the Frontend

**Technologies Used**

- React.js

- Electron.js

- GraphQL

- Mantine

- Theme Forest Template (Hexadash)

**Why React.js?**

Given the time constraint that we had, we had to make sure that we chose frameworks that all of our members were comfortable with! 
As all of us had experience with React before, we decided to go ahead with React, as opposed to other JavaScript frameworks like Angular or Vue.

We were also debating as to whether we should use React.js or React Native. 
However, once we eventually settled on the fact that we were going to make a Desktop app, and not a mobile app, React.js was an easy choice.

**Why Electron.js**

React.js is normally used for locally hosting a website that follows HTTP/HTTPS standards. 
However, we were aiming for a Desktop app, to make it easier to use for users. 
Similar to how Discord uses Electron to convert their web app into a desktop app, we used Electron.js on top of our React.js code to develop a desktop app instead of a web app.

**Communicating between the frontend and backend**

Given that we wanted to make our app functional, it was vital that there was a clear path for the frontend to talk to the backend. 
We did this using GraphQL. We called the GraphQL from the frontend, which then executed our queries on the backend, and efficiently returned the data back to the frontend.

**Sample backend call for sending a transaction**

```javascript
if (amount !== "" && publicKey != "") {
  await api.transactions.postTransaction(
    parseInt(amount),
    publicKey,
    metaData ? { metadata: metaData } : undefined
  );
  open();
}
```

**CSS templates/components**

Once again, given the time constraint, we wanted to focus on the functionality of our app more than the design.
As a result, we went on [Theme Forest](https://themeforest.net/), and searched for potential templates we could use. 
We were looking for templates that had built-in designs for a dashboard and a sign in page, as that was the main part of our app.

We also used [React components](https://mantine.dev/) to make it easier to write CSS. 
Mantine has built in components for common things used in websites such as cards, checkboxes, or confirmation modals.

Overall, using these external libraries had both positives and negatives. 
As the theme forest library we chose had a huge number of files, it was hard to simply integrate their code into our repository. 
The library had a lot of dependencies and functions in other files that they relied on.
As a result, we ended up only using the Theme Forest library for our sign in page, and wrote the CSS for the dashboard ourselves, with help from Mantine as well.

**Example of a Mantine `TextInput` component**

```javascript
<TextInput
    placeholder="Enter amount"
    value={amount}
    onChange={(event) =>
        setAmount(
            event.currentTarget.value
        )
    }
    rightSectionPointerEvents="all"
    rightSection={
        <CloseButton aria-label="Clear input" onClick={() => setAmount("")} style={{ display: amount ? undefined : "none" }} />
    }
    withErrorStyles={false}
    required
    error={
        amountError
            ? "Please fill out this field"
            : null
    }
/>
```

## Securing User Accounts

If you’re wondering how we made our passwords secured, you’ve come to the right place! Our code was written in Nodejs.

You can access the 4 open source libraries that we implemented by importing the following: 

**`nacl`: Able to generate a new pair of public and private keys:**

```typescript
import nacl from "tweetnacl";
```

**`base58`: Encodes or Decodes our password and returns a string**

`import base58 from "bs58";`

**`crypto`: Used to implement SHA512 and AES-256-CBC to hash and cipher/decipher passwords**

`import crypto from "crypto";`

**`keytar`: Acts like a keychain where we can store passwords and access them safe and easily**

`import keytar from "keytar";`

Once you’ve imported all the necessary libraries, let’s dive into the functions that we’ve created to understand how we did it.

**Generating Keys**

Recalling from before, we were able to create a key pair with the use of `nacl`. 

With the help of `nacl`, we will have two outputs with our variable pair, one being a public key and the other being a secret key. 

Accessing them through the pair will get us our public and private keys.

```typescript
static createKeyValPair = () => {
  const pair = nacl.sign.keyPair(); // keypair
  const publicKey = base58.encode(pair.publicKey); //public key
  const privateKey = base58.encode(pair.secretKey.slice(0, 32)); // private key
  return { publicKey, privateKey };
};
```

**Hashing Password**

With the help of the crypto library, we are able to use SHA512 to create a hash object and update the password and lastly putting it into hexadecimal format.

```typescript
static hashPassword = (password: string) => {
  return crypto.createHash("sha512").update(password).digest("hex"); // hashes the password
};
```

**Cipher and Decipher Password**

Similarly like SHA512, we used the crypto library to use AES-256-CBC.
- We first converted the password to a buffer out of ASCII.

- We make sure that the password is exactly 32 bytes so it doesn’t throw any errors.
  
- We create the cipher object using AES-256-CBC along with our paddedkeybytes and cipherIV.
  
- Lastly, we update the cipher with the private key in UTF-8 encoding and produce the encrypted private key in hexadecimal format.

```typescript
encryptPrivateKey = (password: string) => {
    const keybytes = Buffer.from(password, "ascii");
    if (keybytes.length > 32 || keybytes.length < 1) {
        throw new Error("Password must be between 1 and 32 bytes long");
    }
    const paddedKeyBytes = Buffer.concat([keybytes], 32);
    const cipher = crypto.createCipheriv(
        "aes-256-cbc",
        paddedKeyBytes,
        cipherIV
    );
    this.encryptedPrivateKey =
        cipher.update(this.privateKey, "utf8", "hex") + cipher.final("hex");
};
```

The Decipher functions reverses the ciphering process:

```typescript
decryptPrivateKey = (password: string) => {
    const keybytes = Buffer.from(password, "ascii");
    const paddedKeyBytes = Buffer.concat([keybytes], 32);
    const decipher = crypto.createDecipheriv(
        "aes-256-cbc",
        paddedKeyBytes,
        cipherIV
    );
    this.privateKey =
        decipher.update(this.encryptedPrivateKey, "hex", "utf8") +
        decipher.final("utf8");
};
```

**Keytar**

After the user creates the account, the keys get saved into the keychain using keytar

```typescript
writeUserAccountToFile = async () => {
    await keytar.setPassword(
        appName,
        this.username,
        JSON.stringify({
            passwordHash: this.passwordHash,
            publicKey: this.publicKey,
            encryptedPrivateKey: this.encryptedPrivateKey,
        })
    );
};
```


## Familiarizing Yourself with Your Wallet

When you first launch our desktop wallet, things might look a little bit confusing.
But don’t worry! We’ve made our wallet simple to understand and navigate. However, if you’re still lost, you’ve come to the right place to learn more.

**Sign in / Register**

When launched, the application will display the following:

![Login/Signup Page](https://drive.google.com/file/d/1WgptYKE2sdNOLPpamPdG8dq7JfeMuXqM/view?usp=sharing)

As you can see, you have the option to register a new account with us, or log-in to an existing account in our database.

We’ll assume that you are a new user that has just registered an account. Once your account is created, you’ll be directed to your Dashboard.

**Dashboard**

Your dashboard is the main hub of your wallet. It’s where all of the wallet’s functionality can be accessed.

Here’s what the dashboard of a newly registered user looks like:

![Initial Dashboard](https://drive.google.com/file/d/1RfeU4Ozq3sFGNzNeFvRjh0J6UrMddRFw/view?usp=sharing)

There are 3 important sections of your dashboard (circled in red above)

- Your Past Transactions
  
- Send a Transaction
  
- View Keys
  
**Your Past Transactions**

This section of the application displays all previous transactions that your account has made. 
Naturally, for a new account, this section would be empty. However, as you accumulate transactions, they will start to display here, and you can scroll through all of them horizontally.

For example, this is what the dashboard for a highly-active account may look like:

![Older Dashboard](https://drive.google.com/file/d/1DU0CugQzmPNNmaLJuK4rLPBFdiFcDAsU/view?usp=sharing)

**Send a Transaction**

There are 2 things you must have for a successful transaction:

- Your recipient’s public key
  
- The amount that you want to transfer
  
There is a third (optional) spot, reserved for metadata. This could include a brief description of the transaction, ex. Groceries or Pizza.

To send a transaction, you’ll have to ask your recipient for their public key first. Once you have that, do the following:

- Paste the recipients public key

- Enter the transfer amount

- Add a description if you’d like
  
Then hit ‘Send’ and you’ll see a pop-up indicating a successful transfer.

To view the details of the transaction, just refresh the page, and it will pop up under ‘Your Past Transactions’.

**View Your Account’s Keys**

The ‘View Keys’ section displays some important information about your account. It holds sensitive information such as:

- Account Balance
  
- Public Key
  
- Private Key
  
The account balance shows the money your account holds. It updates every time you make a transaction.

Your public key is what you give to others so they can pay you. Only give someone your public key if you know them and if they want to initiate a transfer to your account.

Your private key is sensitive information that is unique to your account and your account only.

**NEVER** give someone your private key. To ensure that your private key isn’t easily viewable, we’ve added a button that, when clicked, either shows or hides the private key for extra security.

**Logging Out**

To log out, simply click the button at the top right. This will securely save your information in our database which can easily be re-accessed by logging in.

**Wrapping Up**

Once again, thanks for choosing to use our Desktop Wallet for your financial needs. We hope this post gave you a better understanding of the features of your wallet to help you use it to its maximum potential.

If you have any further questions, don’t hesitate to contact us!


## Future Plans for ResilientDB Desktop Wallet

In the dynamic world of digital currencies, 
the ResilientDB Desktop Wallet stands as a beacon of innovation. 
As we envision the future, our mission is to enhance accessibility, security, and user experience. 
Let’s delve into the exciting potential plans that await our desktop application.

**App Store Deployment**

Gone are the days of complex installations! Soon, users will find our desktop wallet on the app store, simplifying access for everyone. 
No more command lines or dependencies—just a seamless experience for all. We hope to soon have our application displayed right on the front page of the App Store.

**Going Mobile**

Imagine the freedom to manage your digital assets anytime, anywhere. 
Introducing the mobile version of our ResilientDB Desktop Wallet—a sleek and intuitive design tailored for on-the-go users.
The mobile app mirrors the desktop features with enhanced mobile-friendly interfaces.

**Currency Exchange at Your Fingertips**

We’re breaking barriers by introducing a currency exchange option directly on the dashboard. 
Users can effortlessly convert or trade their funds, opening the door to financial flexibility. 
Registering your bank account seamlessly links your financial world with our wallet.

**Fortifying Security**

Protecting your assets is paramount. 
Our enhanced security features include a authentication via email or phone, ensuring each login is secure. 
Additionally, receive instant transaction alerts via email, offering real-time updates on your wallet activity.

As we embark on this journey, the ResilientDB Desktop Wallet is poised to revolutionize the digital currency landscape.
Join us in this exciting chapter—try our wallet, share your thoughts, and be a part of the future we’re crafting together. 
The possibilities are endless, and the future is now.
