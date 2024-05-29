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
The Res-Lenses page is an in browser UTXO heatmap visualization of the resilient DB network and Ethereum network. 
It utilizes the three.js 3D graphics library for the browser to visualize transaction activity across all possible pairs of addresses collected.
It allows for the viewing of transaction activity in various configurations to best find patterns in UTXO activity.


## Front End Overview

### Scene Controls

**Movement**
Hold click and drag aronud to move through the 2D Grid. Clicking the *Return to Origin* button will return you to the top left corner of the grid.
Chunks of blocks will load and unload as you move around, you may see chunks load in if moving quickly enough.

**Selection**
Blocks that are moused over will display their From and To addresses in the top left tab with a summation of the transactions between the two addresses.
Double clicking a block will open a tab beneath that displays all the transactions that occurred.

### Bottom Bar Buttons

**Return to Origin:**
Reset the position of the camera to the top left corner. If view mode is in bar mode, simply moves the camera back to the far left.

**Data:** 
Selects the source of the data to view.
- `ResDB`: Resilient DB data. All transactions that have occurred on the Resilient DB network with amounts involved.
- `Ethereum`: Sampled data collected on the ethereum network. A sample of 48,000 transactions from the last 24 hours and all the addresses involved. Typically will have more addresses than transactions. It is recommnded that this data is viewed with the *Largest Transaction* sorting scheme as the data is very sparse.

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

### Data collection

### Endpoints
