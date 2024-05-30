---
layout: article
title: ResLenses
Author: Gia-Phong Ngyuen
tags: ResilientDB, ResLenses
aside:
   toc: true
article_header:
   type: overlay
   theme: dark
   ackground_color: '#000000'
   background_image:
    gradient: 'linear-gradient(135deg, rgba(0, 204, 154 , .2), rgba(51, 154, 154, .2))'
    src: /assets/images/res-lenses/reslenses_main_image2.png
---

## Project Overview: 
The Res-Lenses page is an in browser UTXO heatmap visualization of the ResilientDB network and the Ethereum network. 
It utilizes the three.js 3D graphics library for the browser to visualize transaction activity across all possible pairs of addresses collected, allowing for the viewing of transaction activity in various configurations to best find patterns in UTXO movement.

##### Browsers this is known to work in (others may have difficulty): Microsoft Edge and Google Chrome

You may access the ResLenses app [here](https://res-lenses.resilientdb.com) and view the demo [here](https://www.youtube.com/watch?v=IzGxG4_WkDQ).

## Front End Overview

### Getting Started on Page Launch
Opening the page up, you will initially view the top left corner of a very large grid representing transactions on the ResilientDB network. As you mouse around, you will notice green and blue lines pinpointing the block you are hovering. That block represents the transaction activity between two address as shown on the top left tab. As you will also notice, addresses implicity line the X and Y axis of the grid, and each block represents activity of the intersecting addresses. If the activity between two addresses is non-grey, then there is some activity, and double clicking on that block will pull up a tab on the left. In that tab, you can see all UTXO sent from one address to the other in a *From* and *To* relationship.

![defaultview](/assets/images/res-lenses/reslenses default getting started.png)

### Scene Controls

The basis of the visualization is with the Three.js library, which allows for the creation and viewing of 3D scenes in the browser. The camera is fixed to face down on the grid and can be navigated using purely mouse controls.

**Movement:**
Hold click and drag aronud to move through the 2D Grid. Clicking the *Return to Origin* button in the bottom left will return you to the top left corner of the grid.
Chunks of blocks will load and unload as you move around, you may even see some chunks load in if moving quickly enough. Mouse dragging does not work when the mouse is hovering over UI.

**Selection:**
Blocks that are moused over will display their From and To addresses in the top left tab with a summation of the transactions between the two addresses.
Double clicking a block will focus and highlight it and open a tab to the left that displays all the transactions that occurred. Regular clicking anywhere will de-focus the block.

### Bottom Bar Buttons

At the bottom of the page is a bar with five buttons. These allow for various configurations for visualizing the same data. Though the data can also be swapped between ResilientDB and the Ethereum Mainnet. Aside from the data selection button, all configurations are computer client side.

![bottombar](/assets/images/res-lenses/reslenses_bottom_bar.png)

**Return to Origin:**
Reset the position of the camera to the top left corner. If view mode is in *bar* mode, this simply moves the camera back to the far left.

**Data:** 
Selects the source of the data to view. Default is *ResDB*.
- `ResDB`: ResilientDB data. All transactions that have occurred on the ResilientDB network with amounts involved.
- `Ethereum`: Sampled data collected on the ethereum network. A sample of 48,000 transactions from the last 24 hours and all the addresses involved. Typically will have more addresses than transactions. Note that this will take a second to load.

Note: With Ethereum, there are typically more addresses than transactions, resulting in very sparse looking data. It is recommnded that this data is viewed with the *Largest Transaction* sorting scheme and or with the *Symmetry* setting set to `true` to bring forth more activity to the top left.

**Sort:** 
Selects the sorting method for the order in which the adddresses appear from left/top to right/bottom. Default is *Transactions Total*
- `Transactions Total`: Addresses are sorted by the sum total of UTXO sent out.
- `Number of Transaction`: Addresses are sorted by the total number of transactions they've sent out.
- `Largest Transaction`: The address-to-address activity blocks are sorted first, then the addresses belonging to the largest activity are added first. This typically results in a diagonal pattern of blocks that slowly shrink from top left to bottom right. 

For Ethererum, data would typically look like this with default settings:
![ethdata](/assets/images/res-lenses/reslenses_eth.png)

There is basically nothing for a far range. You would need to search extensively to find the activity. But with *Sort by Largest Transaction*, you will get something like this:
![ethdata_sorted](/assets/images/res-lenses/reslenses_eth_sort_by_largest.png)

**View:** 
Selects the viewing method of transaction activity. Default is *Grid*
- `Grid`: This is the primary way to view transactions. Displays all pairwise activity between addresses as a 2D grid, forming a physical adjacency matrix of addresses to addresses with blocks as the activity between them.
- `Bar`: This flattens all the transactions into a single block along the y axis, changing the view into a bar graph along a single axis. This gives the total activity of each address, with details on all transactions still available on the focus tab to the left when bars are selected.

This is the sparse Ethereum data with bar view. We can more aptly view the activity of the addresses.
![barview](/assets/images/res-lenses/reslenses_bar_view.png)

**Symmetry:**
Toggle for whether or not to combine transactions sent out and transactions recieved when computing blocks. Default it *False*
- `False`: Transactions as asymmetrical, the transactions sent from and to and address are distinct.
- `True`: Transactions are symmetrical, as in there is no distinction between transactions sent from or to an address, they are the same and just considered transactions between addresses.

Here is how the ResilientDB data looks with *Sort by Largest Transaction* with *Symmetry* set to `false`
![asym](/assets/images/res-lenses/reslenses_asymmetric.png)

Here is the same configuration except with *Symmetry* set to `true`. Notice how seemingly more activity shows up as transactions sent to and transactions recieved from addresses are compounded together to form the blocks. This is an alternative way to reduce visual sparsity in the data as it increases the value of many blocks while also giving new blocks to addresses that only receive transactions.
![sym](/assets/images/res-lenses/reslenses_symmetric.png)

## Back End Overview

### Server
The backend is a Node.js Express app that runs on the ResilientDB cloud. It procures data from the ResilientDB and Ethereum Mainnet networks at regular intervals, processes and stores the data on the server, and provides endpoints for Res-Leneses to access that processed data.

### Data collection

**ResilientDB:**
Data is collected at a one hour interval from the `https://crow.resilientdb.com/v1/transactions` endpoint. This contains all transactions that have ever occurred on the ResilientDB network and is then processed for only From/To transactions with UTXO amounts. Processing cleans out the data of invalid JSON and bad transactions down to a set of addresses and transactions. A single file holds the processed ResilientDB data.

**Ethereum Mainnet:**
Data is collected at a 30 minute inverval using the [Bit Query API](https://bitquery.io). Using this, the server collects 1000 transactions at a time per interval containing only transactions from that interval. Each interval correlates to a file holding transactions for that interval. Transactions 24 hours old are discarded every interval. This leaves the newest 48,000 transactions available at all times. Due to API limitations, this is the most data that can be collected per day. Processing is very simple with some naming tweaks as the API already provides relatively clean data.

### Endpoints
There are two endpoints provided by the backend which the Res-Lenese front end utilizes, each to supply data for the ResilientDB network and the Ethereum Mainnet. The format of the data is listed as a list of all addresses involved and then all transactions. All configuration and reorganization of data is done client side on the front end.

- `https://res-lenses-backend.resilientdb.com/getData_RESDB`: This provides the processed ResilientDB file containing all transactions with transactions times set as DateTime numbers.
- `https://res-lenses-backend.resilientdb.com/getData_ETH`: This compiles all the data from all the Ethereum data files and send them. Transaction times are set as DateTime strings.

Transaction data is relatively simple, they contain the following:
- `From`: Address UTXO is sent from
- `To`: Address UTXO is sent to
- `Amount`: Amount  of UTXO sent
- `Timestamp`: Time of transaction (Number for RESDB and String for ETH)

This is what you should see if you try to directly access [https://res-lenses-backend.resilientdb.com/getData_RESDB](https://res-lenses-backend.resilientdb.com/getData_RESDB).
![backend](/assets/images/res-lenses/reslenses_backend.png)
