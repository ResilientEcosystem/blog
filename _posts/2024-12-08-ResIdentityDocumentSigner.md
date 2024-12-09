---
layout: article
title: ResIdentity Document Signer
excerpt_separator: "<!-- excerpt end -->"
Author: Sachin Shankar Balasubramanyam, Ananya Pandey, Anish Kataria, Shyam Varahagiri, Kiransingh Pal
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

**Powered by the robust ResilientDB fabric, ResIdentity secures crucial documents with a tamper-proof signing process that ensures verifiable integrity and heightened protection.**<!-- excerpt end -->

# Project Overview
## Background
**What ResIdentity Does** \
ResIdentity is a document signing application that allows users to securely authenticate and upload PDFs for signing. The digests of these signed PDFs are committed to the ResilientDB blockchain, which ensures that each document’s signature remains unaltered. Users can then download their signed PDFs as needed.

Additionally, ResIdentity enables users to upload previously signed PDFs for verification. During this process, the application checks the ResilientDB blockchain for a matching digest of the signed document to verify its authenticity.

You can view our application [here](https://residentity.resilientdb.com/) and our demo [here](https://residentity.resilientdb.com/)
## Motivation
**Why We Built ResIdentity** \
The need for secure and reliable document signing cannot be overstated. Traditional platforms, while prevalent, often fail to meet the necessary standards for handling sensitive documents like insurance contracts, loan paperwork, employment offers, etc. These platforms typically rely on centralized servers, which, if compromised, can expose personal information at the risk of theft or alteration. Such centralized systems are vulnerable to service disruptions owing to a single point of failure. This can impact the availability of critical services when they are most needed, causing operational delays, user frustration and reduced trust. Furthermore, privacy is another concern with conventional document signing solutions. Users must entrust their sensitive information to third-party providers, who manage and control all aspects of data security, leaving them withwith little control over their own data.

ResIdentity was built to address these vulnerabilities by leveraging the power of blockchain technology.

## What Sets Us Apart
By using the ResilientDB blockchain to store document digests, ResIdentity ensures that each document's signature is immutable and verifiable. This decentralized approach eliminates single points of failure, enhances data security, and gives users full control over their documents. The blockchain's inherent transparency also means that every transaction is auditable, providing a layer of accountability and trust that traditional platforms cannot match.

Through ResIdentity, we aim to restore confidence in digital transactions and empower users with a tool that upholds the integrity and privacy of their documents, redefining the standards for document security in the digital world.

# Feature Set
## Sign Up
A user's interaction with ResIdentity begins at Sign Up, which features a straightforward interface that requires only an email address and a secure password for registration. 

## Login
Returning users can quickly enter their email address and password to continue their secure journey on our platform.

## Upload, Sign and Download PDFs
With just a few clicks, users can upload PDFs, securely add their signatures, and download the signed documents.

## Verify Signed PDFs
Users can quickly verify the authenticity of signed PDFs with by simply uploading their signed documents to ensure that they are genuine and unaltered.

# User Guide
## Setup
**Dependencies:**
[Docker](https://docs.docker.com/compose/install/), [ResilientDB](https://resilientdb.incubator.apache.org/)

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
### Uploading, Signing and Downloading PDFs
### Viewing the Signature
### Verifying Signature Authenticity

# Tech Specs
## Technology Stack
**Frontend Frameworks:**\
Next.js, Tailwind CSS\
**Backend Frameworks:**\
ResilientDB, Docker, Python FastAPI, MongoDB, AWS S3

## Architecture
![defaultview](/assets/images/residentity/architecture.png)

## Sequence Diagrams
### Sign Up
### PDF Upload, Signing and Download
### Signature Verification
![defaultview](/assets/images/residentity/Signature Verification.png)

# About The Developers
- **Sachin Shankar Balasubramanyam:** A first-year MSCS student at UC Davis, Sachin earned his Computer Science degree from PES University, Bengaluru, in 2023. He was the lead engineer at a VC-funded AI startup, with expertise in software architecture, Python, and web development, essential for ResIdentity's client application.
- **Ananya Pandey:** Ananya is an MSCS student at UC Davis. She earned her Bachelor's in Computer Science from PES University, Bengaluru, in 2018. With over five years of software development experience at Zebra Technologies and Expedia Group, she specializes in Android/AOSP, 802.11 protocols, Node.js, Spring Boot, and AWS, focusing on backend services for ResIdentity.
- **Anish Kataria:** Anish is pursuing an MS in Computer Science at UC Davis, with a Bachelor's in Computer Science from VIIT, Pune (2023). Formerly an Associate Software Engineer at LogicMonitor, he focused on backend development with Golang, Docker, Kubernetes, and scalable microservices. Anish is dedicated to the client application for ResIdentity.
- **Shyam Varahagiri:** Shyam is a first-year MSCS student at UC Davis and graduated from Manipal University Jaipur in 2024 with a BTech in Information Technology. He worked 11 months at Tata Elxsi as an MLOps Developer, using Docker, Kubernetes, and Python, and has frontend experience with React and Typescript. Shyam works on ResIdentity’s backend.
- **Kiransingh Pal:** Kiran, an MSCS student at UC Davis with a Bachelor's in Electronics and Telecommunications from Pune Institute of Computer Technology, has two years at miniOrange, leading Cloud Access Security projects, building cross-browser extensions, and working with MERN, Golang, Docker, and Kubernetes. Skilled in blockchain and multi-wallet solutions, Kiran focuses on client app development for ResIdentity.