# UPI Offline Mesh

A Spring Boot backend that simulates **offline UPI payments routed through a mesh network**.

This project demonstrates how encrypted payment packets can travel through nearby devices without internet connectivity and later settle once a bridge device regains access to the internet.

The system focuses on three core distributed-system problems:

* Secure end-to-end encrypted payments through untrusted devices
* Idempotent settlement under duplicate packet delivery
* Replay and tampering protection

---

# Demo Scenario

Imagine a basement with no internet.

1. Alice sends Bob ₹500.
2. The payment is encrypted locally.
3. Nearby phones relay the encrypted packet through a simulated Bluetooth mesh.
4. A bridge device later reconnects to the internet.
5. The packet is uploaded to the backend.
6. The backend decrypts, validates, deduplicates, and settles the payment exactly once.

This repository contains:

* Spring Boot backend
* Cryptography pipeline
* Mesh network simulator
* Transaction settlement engine
* Interactive dashboard
* Concurrency and tamper tests

---

# Features

## Secure Hybrid Encryption

Uses:

* RSA-OAEP for key exchange
* AES-256-GCM for payload encryption and authentication

Intermediate mesh devices cannot:

* Read transaction data
* Modify packets
* Forge valid ciphertext

---

## Idempotent Settlement

Duplicate uploads are safely rejected using:

* SHA-256 ciphertext hashing
* Atomic `putIfAbsent()` deduplication
* Database-level unique constraints

Even if multiple bridge nodes upload the same packet simultaneously, settlement occurs exactly once.

---

## Replay Protection

Each payment contains:

* Unique nonce
* Timestamp validation

Old or replayed packets are rejected automatically.

---

# Tech Stack

| Layer       | Technology                             |
| ----------- | -------------------------------------- |
| Backend     | Spring Boot 3                          |
| Language    | Java 17                                |
| Database    | H2 (in-memory)                         |
| ORM         | Spring Data JPA                        |
| Build Tool  | Maven Wrapper                          |
| Encryption  | RSA-OAEP + AES-GCM                     |
| Concurrency | ConcurrentHashMap + optimistic locking |
| Frontend    | Thymeleaf Dashboard                    |

---

# Project Architecture

```text
Sender Phone (Offline)
        │
        ▼
Encrypt Payment Packet
        │
        ▼
Mesh Network Gossip
        │
        ▼
Bridge Device Gets Internet
        │
        ▼
POST /api/bridge/ingest
        │
        ▼
Hash → Deduplicate → Decrypt → Validate → Settle
```

---

# Running the Project

## Prerequisites

* JDK 17+

Verify installation:

```bash
java -version
```

---

## Run on Windows

```cmd
.\mvnw.cmd spring-boot:run
```

---

## Run on Mac/Linux

```bash
./mvnw spring-boot:run
```

---

# Open Dashboard

After startup:

```text
http://localhost:8080
```

The dashboard allows you to:

* Inject encrypted payments into the mesh
* Run gossip rounds
* Upload packets through bridge nodes
* Observe balances and transaction settlement

---

# Available APIs

| Method | Endpoint             | Description                        |
| ------ | -------------------- | ---------------------------------- |
| GET    | `/`                  | Dashboard UI                       |
| GET    | `/api/accounts`      | Account balances                   |
| GET    | `/api/transactions`  | Transaction ledger                 |
| GET    | `/api/mesh/state`    | Mesh device state                  |
| POST   | `/api/demo/send`     | Inject encrypted payment           |
| POST   | `/api/mesh/gossip`   | Run mesh gossip round              |
| POST   | `/api/mesh/flush`    | Upload packets from bridge devices |
| POST   | `/api/bridge/ingest` | Production ingestion endpoint      |

---

# Important Components

## Cryptography

### `HybridCryptoService`

Implements:

* RSA key wrapping
* AES-GCM encryption/decryption
* Ciphertext hashing
* Tamper detection

---

## Mesh Simulator

### `MeshSimulatorService`

Simulates:

* Virtual phones
* Packet gossip propagation
* TTL-based packet routing
* Bridge nodes with internet access

---

## Settlement Engine

### `SettlementService`

Responsible for:

* Atomic debit/credit
* Transaction persistence
* Optimistic locking
* Ledger consistency

---

## Idempotency Layer

### `IdempotencyService`

Prevents duplicate settlement using:

```java
seen.putIfAbsent(packetHash, now)
```

Only the first thread claiming a packet proceeds to settlement.

---

# Testing

Run all tests:

```cmd
.\mvnw.cmd test
```

Key test cases:

* Encryption/decryption round trip
* Tampered ciphertext rejection
* Concurrent duplicate settlement prevention

The concurrency test simulates multiple bridge nodes uploading the same packet simultaneously.

---

# Folder Structure

```text
src/main/java/com/demo/upimesh/
├── config/
├── controller/
├── crypto/
├── model/
├── service/
└── UpiMeshApplication.java
```

---

# Production Improvements

This project is a simulation/demo environment.

For production-scale deployment:

| Demo               | Production               |
| ------------------ | ------------------------ |
| H2 Database        | PostgreSQL / MySQL       |
| ConcurrentHashMap  | Redis SETNX              |
| Simulated Mesh     | BLE / Wi-Fi Direct       |
| Local RSA Keys     | HSM / KMS                |
| No Auth            | Mutual TLS               |
| In-memory Accounts | Real banking integration |

---

# Known Limitations

* Offline payments cannot guarantee sender balance availability
* Double-spending is possible before settlement
* Real Bluetooth mesh networking on mobile devices is difficult
* This is deferred settlement, not true instant offline UPI

---

# Learning Outcomes

This project demonstrates concepts from:

* Distributed systems
* Applied cryptography
* Concurrency control
* Idempotent API design
* Transaction processing
* Mesh networking simulation
* Spring Boot backend engineering

---

<!-- # Screenshots

Add dashboard screenshots here after deployment.

-->


# Author

Chirag Chahar

---

# License

Open for learning and educational purposes.
