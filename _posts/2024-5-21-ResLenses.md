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
    src: /assets/images/resdb-gettingstarted/code_close_up.jpeg
---

## Project Overview: 
The Res-Lenses page is an in browser UTXO heatmap visualization of the Resilient DB network and Ethereum network. 
It utilizes the three.js 3D graphics library for the browser to visualize transaction activity across all possible pairs of addresses collected.
It allows for the viewing of transaction activity in various configurations to best find patterns in UTXO activity.

##### Browsers this is known to work in (others may have difficulty): Microsoft Edge and Google Chrome

[Link to Site](https://res-lenses.resilientdb.com)

## Front End Overview

### Getting Started on Page Launch
You will initially view the top left corner of a very large grid representing transactions on the Resilient DB network. As you mouse around, you will notice green and blue lines pinpointing the block you are hovering. That block represents the transaction activity between two address as shown on the top left tab. As you will notice, addressing implicity line the X and Y axis of the grid, and each block represents activity between two addresses. If the activity between two addresses is non-grey, then there is some activity, and double clicking on that block will pull up a tab on the left. In that tab, you can see all UTXO sent from one address to the other in a From and To relationship.

![defaultview](/assets/images/res-lenses/reslenses default getting started.png)

### Scene Controls

The basis of the visualization is with the Three.js library, which allows for the creation of 3D scenes in the browser. The camera is fixed to face down on the grid and can be navigated using purely mouse controls.

**Movement:**
Hold click and drag aronud to move through the 2D Grid. Clicking the *Return to Origin* button will return you to the top left corner of the grid.
Chunks of blocks will load and unload as you move around, you may see chunks load in if moving quickly enough. Mouse dragging does not work when the mouse is hovering over UI.

**Selection:**
Blocks that are moused over will display their From and To addresses in the top left tab with a summation of the transactions between the two addresses.
Double clicking a block will focus and highlight it and open a tab to the left that displays all the transactions that occurred. Regular clicking anywhere will de-focus the block.

### Bottom Bar Buttons

At the bottom of the page is a bar with five buttons. These allow for various configurations for visualizing the same data. Though the data can also be swapped between Resilient DB and the Ethereum Mainnet.

![bottombar](/assets/images/res-lenses/reslenses bottom bar.png)

**Return to Origin:**
Reset the position of the camera to the top left corner. If view mode is in bar mode, simply moves the camera back to the far left.

**Data:** 
Selects the source of the data to view.
- `ResDB`: Resilient DB data. All transactions that have occurred on the Resilient DB network with amounts involved.
- `Ethereum`: Sampled data collected on the ethereum network. A sample of 48,000 transactions from the last 24 hours and all the addresses involved. Typically will have more addresses than transactions. It is recommnded that this data is viewed with the *Largest Transaction* sorting scheme as the data is very sparse. Note that this will take a second to load.

**Sort:** 
Selects the sorting method for the order in which the adddresses appear from left/top to right/bottom
- `Transactions Total`: Addresses are sorted by the sum total of UTXO sent out.
- `Number of Transaction`: Addresses are sorted by the total number of transactions they've sent out
- `Largest Transaction`: The address to address activity blocks are sorted first, then the addresses pertaining to the largest activity are added first. This typically results in a diagonal of blocks that slowly shrink from top left to bottom right. 

**View:** 
Selects the viewing method of transaction activity.
- `Grid`: This is the default and primary way to view transactions. Displays all pairwise activity between addresses as a 2D grid, forming a physical adjacency of addresses to addresses with blocks as the activity between them.
- `Bar`: This flattens all the transactions into a single block along the y axis, changing the view into a bar graph along a single axis. This gives the total activity of each address, with details on all transactions still available on the focus tab to the left when bars are selected.

**Symmetry:**
Toggle for whether or not to combine transactions sent out and transactions recieved when computing blocks.
- `False`: This is the default and displays transactions as asymmetrical, the transactions sent from and to and address are distinct.
- `True`: Transactions are symmetrical, as in there is no distinction between transactions sent from or to and address, they are the same and just considered transactions between addresses.

## Back End Overview

### Server
The backend is an Node.js Express app that runs on the ResilientDB cloud. It procures data from the Resilient DB and Ethereum Mainnet networks at regular intervals, processes and stores the data, and provides endpoints for Res-Leneses to access that processed data.

### Data collection

**Resilient DB:**
Data is collected at a one hour interval from the `https://crow.resilientdb.com/v1/transactions` endpoint. This contains all transactions that have ever occurred on the ResilientDB network and is then processed for only From/To transactions with UTXO amounts. A single file holds the ResilientDB data.

**Ethereum Mainnet:**
Data is collected at a 30 minute inverval using the [Bit Query API](https://bitquery.io). Using this, the server collects 1000 transactions at a time per interval containing only transactions from that interval. Each interval correlates to a file holding transactions for that interval. Transactions 24 hours old are discarded every interval. This leaves the newest 48,000 transactions available at all times. Due to API limitations, this is the most data that can be collected per day.

### Endpoints
There are two endpoints provided by the backend, each to supply data for the Resilient DB network and the Ethereum Mainnet. The format of the data is listed as a list of all addresses involved and then all transactions. All configuration and reorganization of data is done client side on the front end.

- `https://localhost:8080/getData_RESDB`: This provides the processed Resilient DB file contraiing all transactions with transactions times set as DateTime numbers.
- `https://localhost:8080/getData_ETH`: This compiles all the data from all the Ethereum data files and send them. Transaction times are set as DateTime strings.

Transaction data is relatively simple, they contain the following:
- `From`: Address UTXO is sent from
- `To`: Address UTXO is sent to
- `Amount`: Amount  of UTXO sent
- `Timestamp`: Time of transaction (Number for RESDB and String for ETH)
