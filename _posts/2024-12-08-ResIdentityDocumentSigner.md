---
layout: article
title: ResIdentity Document Signer
excerpt_separator: "<!-- excerpt end -->"
Author: Sachin Balasubramanyam, Kiranpal Singh, Ananya Pandey, Anish Kataria, Shyam Varahagiri
tags: ResilientDB ResIdentity
aside:
   toc: true
article_header:
   type: overlay
   theme: dark
   ackground_color: '#000000'
   background_image:
    gradient: 'linear-gradient(135deg, rgba(0, 204, 154 , .2), rgba(51, 154, 154, .2))'
    src: /assets/images/residentity/banner.gif
---

**Powered by the robust ResilientDB fabric, ResIdentity secures critical files with a tamper-proof signing process, ensuring heightened protection and verifiable integrity for crucial documents.**<!-- excerpt end -->

# Project Overview
## Background
**What ResIdentity Does** \
ResIdentity is a document signing application that allows users to securely login and upload PDF documents for signing. Once a document is signed, the application stores the hash of the signed PDF on the secure, blockchain-based ResilientDB fabric ensuring that each document's signature is verifiable and cannot be altered. Users can then download their signed PDFs. By directly recording each signature's details on the blockchain, ResIdentity provides a tamper-proof ledger of transactions, which enhances the verifiability and reliability of document signing.

## Motivation
**Why Did We Build ResIdentity?** \
Centralized document signing platforms are vulnerable to cyber-attacks, with hackers targeting central servers to steal or alter sensitive documents and credentials. Additionally, any failure in the central server can cause significant service disruptions and downtime. Such platforms also raise privacy concerns as they limit users' control over their own data, with all management and security protocols handled by the service provider.

## What Sets Us Apart
ResIdentity utilizes ResilientDB to store document hashes on a blockchain, making each signature tamper-proof and verifiable. This decentralized approach eliminates single points of failure, enhancing reliability and reducing risks of data manipulation. Blockchain's inherent transparency allows for secure, auditable records without compromising privacy, providing a dependable and user-empowered alternative to centralized document signing.

# Feature Set
## User Registration and Login
Faciliates simple and secure user onboarding and sign-in.

## Upload, Sign and Download PDFs
In just a few clicks, users can easily upload PDFs, apply their digital signatures and download signed PDFs.

## Verify Signed PDFs
Users can verify whether a particular signed PDF was signed via the ResIdentity application by matching the uploaded file hash with a corresponding signing record on the underlying ResilientDB blockchain.

# User Guide
## Setup
**Dependencies:**
[Docker](https://docs.docker.com/compose/install/), [ResilientDB]()

- To build, run:
```
docker compose build
```
- To launch ResIdentity, run:
```
docker compose up -d
```
- To launch ResilientDB's KV Service, run:
```
docker exec -it resid-core-backend bash
/bin/sh ./INSTALL.sh
/bin/sh ./service/tools/kv/server_tools/start_kv_service.sh
```
To verify the status of KV Service, run:
```
curl -X POST -d '{"id":"key1","value":"value1"}' 127.0.0.1:18000/v1/transactions/commit
curl 127.0.0.1:18000/v1/transactions/key1
```
- The server should be running on ```http://localhost:3000/```.\
Visit ```http://localhost:8000/api``` to verify the API.

## Usage
### Uploading and Signing PDFs
### Verifying PDFs

# Tech Specs
## Technology Stack
**Frontend Frameworks:**\
Next.js, Tailwind CSS\
**Backend Frameworks:**\
ResilientDB, Docker, Python FastAPI, MongoDB, AWS S3

## Architecture
![defaultview](/assets/images/residentity/architecture.png)

## Sequence Diagrams
### User Login
### PDF Upload, Signing and Download
### Signed PDF Verification

# About The Developers
- **Sachin Shankar Balasubramanyam:** A first-year MSCS student at UC Davis, Sachin earned his Computer Science degree from PES University, Bengaluru, in 2023. He was the lead engineer at a VC-funded AI startup, with expertise in software architecture, Python, and web development, essential for ResIdentity's client application.
- **Ananya Pandey:** Ananya is an MSCS student at UC Davis. She earned her Bachelor's in Computer Science from PES University, Bengaluru, in 2018. With over five years of software development experience at Zebra Technologies and Expedia Group, she specializes in Android/AOSP, 802.11 protocols, Node.js, Spring Boot, and AWS, focusing on backend services for ResIdentity.
- **Anish Kataria:** Anish is pursuing an MS in Computer Science at UC Davis, with a Bachelor's in Computer Science from VIIT, Pune (2023). Formerly an Associate Software Engineer at LogicMonitor, he focused on backend development with Golang, Docker, Kubernetes, and scalable microservices. Anish is dedicated to the client application for ResIdentity.
- **Shyam Varahagiri:** Shyam is a first-year MSCS student at UC Davis and graduated from Manipal University Jaipur in 2024 with a BTech in Information Technology. He worked 11 months at Tata Elxsi as an MLOps Developer, using Docker, Kubernetes, and Python, and has frontend experience with React and Typescript. Shyam works on ResIdentity’s backend.
- **Kiransingh Pal:** Kiran, an MSCS student at UC Davis with a Bachelor's in Electronics and Telecommunications from Pune Institute of Computer Technology, has two years at miniOrange, leading Cloud Access Security projects, building cross-browser extensions, and working with MERN, Golang, Docker, and Kubernetes. Skilled in blockchain and multi-wallet solutions, Kiran focuses on client app development for ResIdentity.