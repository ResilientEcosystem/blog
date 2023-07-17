---
layout: article
title: How to setup the NexRes Wallet
author: Apratim Shukla
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

The NexRes wallet allows you to commit and retrieve data from the NexRes blockchain and provides user account creation and deletion. It communicates with the NexRes backend using the NexRes GraphQL server and enables transaction logging.

# Install
Go to our [install tutorial](https://blog.resilientdb.com/2022/09/28/GettingStartedNexRes.html) for instructions on how to install NexRes.

# Set Up 

You will need to clone two repos to get started: [NexRes GraphQL Server](https://github.com/ExpoDB/NexresGraphQL) for the GraphQL server, and [NexRes Wallet](https://github.com/ResilientApp/NexresWallet) for the wallet. 

### NexRes GraphQL server
Install dependencies.
  > python3 -m pip install -r requirements.txt

Activate the virtual environment
  > source venv/bin/activate

Start the GraphQL server.
  > python3 app.py

In case of any errors follow the detailed debugging [README](https://github.com/ResilientApp/NexresGraphQL/blob/main/README.md)

### NexRes Wallet
The NexRes Wallet requires NodeJs, so make sure you have it installed before proceeding.

In another terminal shell after starting KV Server and NexRes GraphQL server, install the wallet dependencies by opening the terminal in the NexRes Wallet directory: 
  > npm install

Build the wallet:
  > npm run build

You will see the following message if build was successful: 
  ```
    The build folder is ready to be deployed.
    You may serve it with a static server:

    serve -s build

    Find out more about deployment here:

    https://cra.link/deployment
  ```

And the build will be available in `build` inside the NexRes Wallet directory

### Add wallet to chrome
Open chrome and navigate to `chrome://extensions/`

Make sure developer mode is enabled using the toggle button.

Finally, load the extension by clicking on load unpacked button and then select the `build` folder that was created in the previous step.

Now you can open the wallet from the extension button and start using it!

The next tutorial will elaborate on how to develop web applications to communicate with the NexRes wallet.