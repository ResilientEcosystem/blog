---
layout: article
title: PetChain 
author: Ansha Prashanth, Neha Pradeep, Sakshi Singh, Sarika Dinesh
tags: PetChain ResilientDb Full-Stack SmartContracts
aside:
    toc: true
article_header:
  type: overlay
  theme: dark
  background_color: '#000000'
  background_image:
    gradient: 'linear-gradient(135deg, rgba(0, 204, 154 , .2), rgba(51, 154, 154, .2))'
    src: /assets/images/petchain/blog-bg.jpg

---

Built on the ResilientDB fabric, PetChain provides secure lost-and-found tracking, seamless health record management, streamlined ownership transfer, and automated pet insurance processing, ensuring reliability in pet care.

## Project Overview
### Motivation

The pet industry, valued at around $150 billion in the U.S. alone, faces critical challenges in pet identification, health record management, and delays in insurance processing. As a team of pet owners and technology enthusiasts, we’ve experienced various challenges in pet ownership—ranging from the frustration of delayed insurance disbursements during emergencies to difficulties accessing reliable information on pet ownership transfers. These shared experiences underscored the need for a unified, secure system that offers pet owners faster and better transparent solutions. This need drove us to develop PetChain, a blockchain-based platform designed to holistically address these issues and enhance the overall pet ownership experience.

### Why PetChain?
**Scalable and Reliable**: PetChain, powered by ResilientDB, delivers a robust platform designed to scale effortlessly with increasing data and users, maintaining high performance and fault tolerance.<br>
**Streamlined Lost-and-Found and Ownership Transfers**: Leveraging blockchain technology, PetChain enables swift and secure reunification of lost pets with their owners while simplifying the transfer of pet ownership.<br>
**Automated Insurance Processing**: PetChain uses smart contracts to automate insurance claims, ensuring faster, more efficient, and error-free processes.<br>
**Secure and Transparent**: Blockchain-based storage ensures pet identification and ownership data remain tamper-proof, fostering trust and transparency in pet care.

## Feature Set
### User and Pet Registration
Pet registration event logged in ResDB:
```
(2024-12-09 11:05:19) [INFO    ] Request: 127.0.0.1:45068 0x7fccdc002870 HTTP/1.1 POST /v1/transactions/commit
I20241209 03:05:19.347833 652360 crow_service.cpp:158] body: {"id":"OWNER_1733742092244","value":{"pet_id":"PET_1733742319324","ownershipHash":"a44f3f6cb0c0efc4671f254ffe1eeb28a09ae7dfc820f5be21e85c38c2eef74c","timeStamp":"2024-12-09T11:05:19.326Z","status":"active","event":"register pet"}}
I20241209 03:05:19.349356 652360 crow_service.cpp:180] Set OWNER_1733742092244 to {"id":"OWNER_1733742092244","value":{"pet_id":"PET_1733742319324","ownershipHash":"a44f3f6cb0c0efc4671f254ffe1eeb28a09ae7dfc820f5be21e85c38c2eef74c","timeStamp":"2024-12-09T11:05:19.326Z","status":"active","event":"register pet"}}
```
### Lost and Found
Lost Pet event stored in ResDB:
```
(2024-12-09 11:05:19) [INFO    ] Response: 0x7fccdc002870 /v1/transactions/commit 201 0
(2024-12-09 11:06:38) [INFO    ] Request: 127.0.0.1:39276 0x7fccdc000c20 HTTP/1.1 POST /v1/transactions/commit
I20241209 03:06:38.358037 652359 crow_service.cpp:158] body: {"id":"OWNER_1733742092244","value":{"pet_id":"PET_1733742319324","lostHash":"0d753f137b9725f0bc7b8e29d376f4ff9ae67ff946a8fea7a301695dae293de9","timeStamp":"2024-12-09T11:06:38.354Z","status":"active","event":"lost pet"}}
I20241209 03:06:38.358719 652359 crow_service.cpp:180] Set OWNER_1733742092244 to {"id":"OWNER_1733742092244","value":{"pet_id":"PET_1733742319324","lostHash":"0d753f137b9725f0bc7b8e29d376f4ff9ae67ff946a8fea7a301695dae293de9","timeStamp":"2024-12-09T11:06:38.354Z","status":"active","event":"lost pet"}}
```
Found Pet event stored in ResDB:
```
(2024-12-09 11:08:41) [INFO    ] Request: 127.0.0.1:35466 0x7fccdc002870 HTTP/1.1 POST /v1/transactions/commit
I20241209 03:08:41.755368 652360 crow_service.cpp:158] body: {"id":"OWNER_1733742092244","value":{"pet_id":"PET_1733742319324","foundHash":"8c85e9e98417656cdbc5172bfac80aa7bde53deda0fd44f8d8d519646cfeb240","timeStamp":"2024-12-09T11:08:41.752Z","status":"active","event":"found pet"}}
I20241209 03:08:41.755961 652360 crow_service.cpp:180] Set OWNER_1733742092244 to {"id":"OWNER_1733742092244","value":{"pet_id":"PET_1733742319324","foundHash":"8c85e9e98417656cdbc5172bfac80aa7bde53deda0fd44f8d8d519646cfeb240","timeStamp":"2024-12-09T11:08:41.752Z","status":"active","event":"found pet"}}
```
### Pet-Health Management
### Ownership Transfer
Ownership Transfer event logged in ResDB:
```
(2024-12-09 11:25:14) [INFO    ] Request: 127.0.0.1:38158 0x7fccdc002870 HTTP/1.1 POST /v1/transactions/commit
I20241209 03:25:14.258247 652360 crow_service.cpp:158] body: {"id":"OWNER_1733743246179","value":{"pet_id":"PET_1733742319324","ownershipTransfer":{"oldOwnerId":"OWNER_1733742092244","newOwnerId":"OWNER_1733743246179"},"transferHash":"0e3da49f70790cc3d2a761319c8e2432507d36c312f390ef0850f8803701ff99","timeStamp":"2024-12-09T11:25:14.255Z","status":"completed","event":"ownership transfer"}}
I20241209 03:25:14.258463 652360 crow_service.cpp:180] Set OWNER_1733743246179 to {"id":"OWNER_1733743246179","value":{"pet_id":"PET_1733742319324","ownershipTransfer":{"oldOwnerId":"OWNER_1733742092244","newOwnerId":"OWNER_1733743246179"},"transferHash":"0e3da49f70790cc3d2a761319c8e2432507d36c312f390ef0850f8803701ff99","timeStamp":"2024-12-09T11:25:14.255Z","status":"completed","event":"ownership transfer"}}
```
### Smart Contract Based Insurance Processing

## Technical Specifications
### System Architecture
<p align="center">
    <img src="/assets/images/petchain/System-Architecture.jpg" alt="Logo"/>
    <br>
    <em> Figure 1. Architecture Diagram
    </em>
</p>

### Technology Stack
- Frontend: React, MaterialUI
- Backend: Node.js, Express.js, HardHat, ethers.js, Solidity, Nodemailer
- Databases: ResilientDB, MongoDB


### WorkFlow and Sequence Diagrams
The sequence diagrams below outline the key workflows for Lost and Found Service, Ownership Transfer, and Insurance Processing, illustrating the interactions between system components and the triggers for each process. These visualizations provide a clear understanding of the underlying mechanisms that ensure secure and efficient operations within the PetChain system.
<p align="center">
    <img src="/assets/images/petchain/LostFoundSequence.png" alt="Logo" width="800"/>
    <br>
    <em> Figure 2. Sequence Diagram for Lost and Service
    </em>
</p>

<p align="center">
    <img src="/assets/images/petchain/OwnerTransferSequence.png" alt="Logo" width="800"/>
    <br>
    <em> Figure 3. Sequence Diagram for Ownership Transfer
    </em>
</p>

<p align="center">
    <img src="/assets/images/petchain/InsuranceSequence.png" alt="Logo" width="800"/>
    <br>
    <em> Figure 4. Sequence Diagram for Insurance Processing
    </em>
</p>

### Steps to Run the System
#### Prerequisites
- Download NodeJS from [here](https://nodejs.org/en/download) and ensure that it’s added to PATH.
- ##### Setup ResilientDB and start KV Service
```sh
git clone https://github.com/apache/incubator-resilientdb.git resilientdb
cd resilientdb
./INSTALL.sh
./service/tools/kv/server_tools/start_kv_service.sh
```
- ##### Setup Crow HTTP Service
```sh
git clone https://github.com/ResilientApp/ResilientDB-GraphQL.git
cd ResilientDB-GraphQL
bazel build service/http_server/crow_service_main
bazel-bin/service/http_server/crow_service_main service/tools/config/interface/client.config service/http_server/server_config.config
```


### Links
- Code repository:
[Frontend](https://github.com/nehapradeep/PetChainPlus), 
[Backend](https://github.com/sakshisingh301/PetChainBackend/tree/master)
<br>
- [Demo Video]()
<br>
- [Presentation Slides](https://docs.google.com/presentation/d/190cyr4yLdWzSKJSrkdzT-m8rJX4C3bKX/edit?usp=sharing&ouid=116836227048374057417&rtpof=true&sd=true)


## About Team

- Sakshi Singh
- Ansha Prashanth
- Neha Pradeep
- Sarika Dinesh