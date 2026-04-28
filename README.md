<p align="center">
  <img src="https://alexandrebarros.com/global/hyperledger/hero_image.jpg?alt=hyperledger-supply-chain" alt="Hyperledger Food Supply Chain" width="100%" />
</p>

<h1 align="center">Hyperledger Food Supply Chain</h1>

<p align="center">
  <strong>A permissioned blockchain network for end-to-end food traceability — from farm to fork.</strong>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Hyperledger_Fabric-v2.2-2F3134?style=for-the-badge&logo=hyperledger&logoColor=white" alt="Fabric v2.2" />
  <img src="https://img.shields.io/badge/TypeScript-Chaincode-3178C6?style=for-the-badge&logo=typescript&logoColor=white" alt="TypeScript" />
  <img src="https://img.shields.io/badge/Node.js-Express_API-339933?style=for-the-badge&logo=node.js&logoColor=white" alt="Node.js" />
  <img src="https://img.shields.io/badge/Docker-Compose-2496ED?style=for-the-badge&logo=docker&logoColor=white" alt="Docker" />
  <img src="https://img.shields.io/badge/License-Apache_2.0-D22128?style=for-the-badge" alt="License" />
</p>

---

## Table of Contents

- [Overview](#overview)
- [Why Blockchain for Food Supply Chain?](#why-blockchain-for-food-supply-chain)
- [System Architecture](#system-architecture)
  - [High-Level Architecture Diagram](#high-level-architecture-diagram)
  - [Network Topology](#network-topology)
  - [On-Chain vs Off-Chain Components](#on-chain-vs-off-chain-components)
- [Project Structure](#project-structure)
- [Smart Contract (Chaincode)](#smart-contract-chaincode)
  - [Contract Functions](#contract-functions)
  - [Data Models](#data-models)
  - [Validation Rules](#validation-rules)
- [REST API Reference](#rest-api-reference)
- [Transaction Lifecycle](#transaction-lifecycle)
- [Identity & Access Control](#identity--access-control)
- [Network Configuration Deep Dive](#network-configuration-deep-dive)
  - [Consensus Mechanism](#consensus-mechanism)
  - [Channel & Policy Configuration](#channel--policy-configuration)
  - [Docker Services](#docker-services)
  - [State Database Options](#state-database-options)
- [Prerequisites](#prerequisites)
- [Getting Started](#getting-started)
  - [Step 1 — Start the Network](#step-1--start-the-network)
  - [Step 2 — Create the Channel](#step-2--create-the-channel)
  - [Step 3 — Deploy the Chaincode](#step-3--deploy-the-chaincode)
  - [Step 4 — Set Up the Web Application](#step-4--set-up-the-web-application)
  - [Step 5 — Enroll Identities](#step-5--enroll-identities)
  - [Step 6 — Start the API Server](#step-6--start-the-api-server)
- [Usage Examples](#usage-examples)
- [Testing](#testing)
- [Technology Stack](#technology-stack)
- [Known Limitations & Future Improvements](#known-limitations--future-improvements)
- [Authors](#authors)
- [License](#license)
- [References](#references)

---

## Overview

**Hyperledger Food Supply Chain** is a permissioned blockchain application built on **Hyperledger Fabric v2.2** that connects participants across the food supply through a permanent, shared, and tamper-proof record of food system data.

The system tracks food products from their point of origin through every step of the distribution chain. Each product carries a complete history of where it has been, when it arrived, and what component ingredients it contains — all recorded immutably on the blockchain. This enables:

- **Full traceability** — trace any product back to its farm of origin in seconds
- **Recall efficiency** — identify affected products instantly during food safety incidents
- **Provenance verification** — prove authenticity and origin for premium or organic products
- **Multi-party trust** — all supply chain participants share a single source of truth without relying on a central authority


---

## Why Blockchain for Food Supply Chain?

Traditional food supply chains suffer from fragmented record-keeping. Each participant — farmers, processors, distributors, retailers — maintains their own siloed databases. When a contamination event occurs, tracing the source can take **days or weeks**.

| Challenge | Traditional Approach | Blockchain Approach |
|-----------|---------------------|---------------------|
| **Data Integrity** | Records can be altered or lost | Immutable ledger — once written, data cannot be changed |
| **Traceability** | Manual, paper-based, takes days | Instant lookup of full product history |
| **Trust** | Requires intermediaries to verify claims | Cryptographic proof — every transaction is signed |
| **Transparency** | Each party sees only their own data | Shared ledger visible to all authorized participants |
| **Recall Speed** | Days to weeks to identify affected products | Seconds to trace origin and distribution path |

This project uses **Hyperledger Fabric** specifically because it is a **permissioned** blockchain — only authorized organizations can participate, and transaction data is only visible to channel members. This is critical for commercial supply chains where competitive data must remain confidential.

---

## System Architecture

### High-Level Architecture Diagram

```
┌──────────────────────────────────────────────────────────────────────────┐
│                          CLIENT LAYER (Off-Chain)                        │
│                                                                          │
│   ┌──────────────┐         ┌──────────────────────────────────────┐     │
│   │  HTTP Client  │────────►│  Express.js REST API (port 3003)     │     │
│   │  (Browser /   │         │                                      │     │
│   │   Postman)    │         │  Endpoints:                          │     │
│   └──────────────┘         │   POST /createProduct                │     │
│                             │   POST /shipProduct                  │     │
│                             │   GET  /getProduct                   │     │
│                             │   GET  /getProductWithHistory        │     │
│                             │   GET  /productExists                │     │
│                             └──────────────┬───────────────────────┘     │
│                                            │                             │
│                             ┌──────────────▼───────────────────────┐     │
│                             │  Fabric SDK (fabric-network v2.2.7)  │     │
│                             │  Gateway → Channel → Contract        │     │
│                             │  Identity: "manager" (Org1MSP)       │     │
│                             └──────────────┬───────────────────────┘     │
└────────────────────────────────────────────┼─────────────────────────────┘
                                             │
┌────────────────────────────────────────────┼─────────────────────────────┐
│                     BLOCKCHAIN LAYER (On-Chain)                          │
│                                                                          │
│   ┌────────────────────┐       ┌────────────────────┐                   │
│   │   peer0.org1        │       │   peer0.org2        │                   │
│   │   (port 7051)       │       │   (port 9051)       │                   │
│   │   Org1MSP           │       │   Org2MSP           │                   │
│   │                     │       │                     │                   │
│   │  ┌───────────────┐ │       │  ┌───────────────┐ │                   │
│   │  │  Chaincode     │ │       │  │  Chaincode     │ │                   │
│   │  │  (TypeScript)  │ │       │  │  (TypeScript)  │ │                   │
│   │  └───────────────┘ │       │  └───────────────┘ │                   │
│   │  ┌───────────────┐ │       │  ┌───────────────┐ │                   │
│   │  │  World State   │ │       │  │  World State   │ │                   │
│   │  │  (LevelDB /    │ │       │  │  (LevelDB /    │ │                   │
│   │  │   CouchDB)     │ │       │  │   CouchDB)     │ │                   │
│   │  └───────────────┘ │       │  └───────────────┘ │                   │
│   └─────────┬──────────┘       └──────────┬─────────┘                   │
│             │                              │                             │
│             └──────────┬───────────────────┘                             │
│                        ▼                                                 │
│              ┌─────────────────────┐                                     │
│              │  Orderer (Raft)      │                                     │
│              │  orderer.example.com │                                     │
│              │  (port 7050)         │                                     │
│              │  OrdererMSP          │                                     │
│              └─────────────────────┘                                     │
│                                                                          │
│   ┌──────────────┐  ┌──────────────┐  ┌──────────────┐                  │
│   │ CA Org1       │  │ CA Org2       │  │ CA Orderer   │                  │
│   │ (port 7054)   │  │ (port 8054)   │  │ (port 9054)  │                  │
│   └──────────────┘  └──────────────┘  └──────────────┘                  │
└──────────────────────────────────────────────────────────────────────────┘
```

### Network Topology

| Component | Count | Description |
|-----------|-------|-------------|
| **Peer Organizations** | 2 | Org1 and Org2 — each operates one peer node |
| **Orderer Organization** | 1 | Single Raft orderer node |
| **Certificate Authorities** | 3 | One per organization (Org1, Org2, Orderer) |
| **Channel** | 1 | `mychannel` — shared application channel |
| **Chaincode** | 1 | `basic` — ProductSupplyChain contract |

### On-Chain vs Off-Chain Components

| Layer | Component | Location | Technology |
|-------|-----------|----------|------------|
| **On-Chain** | Smart Contract | `chaincode/src/` | TypeScript → compiled to JS |
| **On-Chain** | Data Models | `chaincode/src/models/` | Fabric Contract API decorators |
| **On-Chain** | Ledger State | Peer nodes | LevelDB (default) or CouchDB |
| **Off-Chain** | REST API | `web-app/server/app.js` | Express.js |
| **Off-Chain** | SDK Integration | `web-app/server/fabric/` | fabric-network v2.2.7 |
| **Off-Chain** | Identity Management | `web-app/server/fabric/` | fabric-ca-client v2.2.7 |
| **Infrastructure** | Network Config | `network/fabric-network/` | Docker Compose, shell scripts |


---

## Project Structure

```
hyperledger-supply-chain/
│
├── chaincode/                          # Smart Contract (On-Chain Logic)
│   ├── src/
│   │   ├── index.ts                    # Contract export & registration
│   │   ├── product-supply-chain-contract.ts  # Core business logic (5 public functions)
│   │   ├── product-supply-chain-contract.spec.ts  # Unit tests (Mocha + Chai + Sinon)
│   │   ├── roles.ts                    # Role constants (manager, employee, client)
│   │   └── models/
│   │       ├── product.ts              # Product entity (14 fields)
│   │       ├── product-location-data.ts    # Location tracking container
│   │       ├── product-location-entry.ts   # Single location record
│   │       └── product-with-history.ts     # Product + resolved component products
│   ├── transaction_data/
│   │   └── product-transactions.txdata # Sample transaction definitions
│   ├── package.json                    # Dependencies: fabric-contract-api, fabric-shim
│   ├── tsconfig.json                   # TypeScript compiler configuration
│   └── tslint.json                     # Linting rules
│
├── network/                            # Blockchain Network Infrastructure
│   ├── bin/                            # Fabric CLI binaries (peer, configtxgen, etc.)
│   ├── config/                         # Runtime configuration
│   │   ├── configtx.yaml              # Channel transaction config
│   │   ├── core.yaml                  # Peer runtime config
│   │   └── orderer.yaml              # Orderer runtime config
│   └── fabric-network/
│       ├── network.sh                  # Main network orchestrator script
│       ├── configtx/
│       │   └── configtx.yaml          # Organization, channel & policy definitions
│       ├── docker/
│       │   ├── docker-compose-test-net.yaml  # Peers + Orderer + CLI
│       │   ├── docker-compose-ca.yaml        # Certificate Authorities
│       │   └── docker-compose-couch.yaml     # CouchDB state databases
│       ├── organizations/
│       │   ├── cryptogen/             # Crypto material generation configs
│       │   ├── fabric-ca/             # CA enrollment scripts
│       │   ├── ccp-generate.sh        # Connection profile generator
│       │   ├── ccp-template.json      # Connection profile template (JSON)
│       │   └── ccp-template.yaml      # Connection profile template (YAML)
│       ├── scripts/
│       │   ├── deployCC.sh            # Chaincode lifecycle deployment
│       │   ├── createChannel.sh       # Channel creation & peer joining
│       │   ├── envVar.sh             # Environment variable setup per org
│       │   ├── configUpdate.sh       # Channel config update utilities
│       │   ├── setAnchorPeer.sh      # Anchor peer configuration
│       │   └── utils.sh              # Shared utility functions
│       └── addOrg3/                   # Scripts for dynamic org addition
│
├── web-app/                            # Client Application (Off-Chain)
│   └── server/
│       ├── app.js                      # Express.js REST API (5 endpoints, port 3003)
│       ├── fabric/
│       │   ├── network.js             # Fabric Gateway connection middleware
│       │   ├── enrollAdmin.js         # CA admin enrollment script
│       │   ├── registerUser.js        # Single user registration (appUser)
│       │   └── registerUsers.js       # Batch registration (manager + employee)
│       └── package.json               # Dependencies: fabric-network, express, cors
│
├── docs/                               # Documentation (GitHub Pages)
├── README.md                           # This file
├── LICENSE                             # Apache 2.0
└── _config.yml                         # GitHub Pages config
```

---

## Smart Contract (Chaincode)

The smart contract is written in **TypeScript** and compiled to JavaScript before deployment. It lives in `chaincode/src/product-supply-chain-contract.ts` and is registered as the `ProductSupplyChainContract` class extending Fabric's `Contract` base class.

### Contract Functions

#### `createProduct(ctx, productJson)` — Submit Transaction

Creates a new product on the ledger with full metadata and initial location.

```typescript
// Input: JSON string containing all product fields
// State Change: putState(product.id, serializedProduct)
// Validation: 12 required fields checked, duplicate ID rejected
```

**Required Fields:** `id`, `name`, `barcode`, `placeOfOrigin`, `productionDate`, `expirationDate`, `unitQuantity`, `unitQuantityType`, `unitPrice`, `category`, `locationData.current.location`, `locationData.current.arrivalDate`

---

#### `shipProductTo(ctx, productId, newLocation, arrivalDate)` — Submit Transaction

Updates a product's location, preserving the full movement history.

```typescript
// 1. Reads current product from ledger
// 2. Pushes current location → previous[] array
// 3. Sets new current location and arrival date
// 4. Writes updated product back to ledger
```

**How location history works:**
```
Before shipProductTo("P1", "Toronto Warehouse", "2024-01-15"):
  current:  { location: "Montreal Farm", arrivalDate: "2024-01-01" }
  previous: []

After:
  current:  { location: "Toronto Warehouse", arrivalDate: "2024-01-15" }
  previous: [{ location: "Montreal Farm", arrivalDate: "2024-01-01" }]
```

---

#### `getProduct(ctx, productId)` — Query Transaction

Retrieves a single product by ID. Executes only on the queried peer (no ordering/commit).

---

#### `getProductWithHistory(ctx, productId)` — Query Transaction

Retrieves a product along with all its **component products** (resolved from `componentProductIds`). This enables tracing composite food products (e.g., a jam) back to their raw ingredients (e.g., apples, sugar).

```
Product: "Apple Jam" (id: 1003)
  └── componentProductIds: ["1001", "1002"]
       ├── Product: "Apples" (id: 1001) — with full location history
       └── Product: "Sugar"  (id: 1002) — with full location history
```

---

#### `productExists(ctx, productId)` — Query Transaction

Returns `true` if a product with the given ID exists on the ledger, `false` otherwise.

---

### Data Models

All models use Fabric's `@Object()` and `@Property()` decorators for metadata generation.

#### Product (`chaincode/src/models/product.ts`)

| Field | Type | Description |
|-------|------|-------------|
| `id` | `string` | Unique product identifier (ledger key) |
| `componentProductIds` | `string[]` | IDs of ingredient/component products |
| `barcode` | `string` | Product barcode (UPC/EAN) |
| `name` | `string` | Product name |
| `placeOfOrigin` | `string` | Where the product was produced |
| `productionDate` | `string` | ISO 8601 production timestamp |
| `expirationDate` | `string` | ISO 8601 expiration timestamp |
| `unitQuantity` | `number` | Quantity per unit |
| `unitQuantityType` | `string` | Unit of measure (e.g., "mg", "kg", "L") |
| `batchQuantity` | `number` | Number of units in the batch |
| `unitPrice` | `string` | Price per unit (e.g., "$5.00") |
| `category` | `string` | Product category (e.g., "Fruits", "Dairy") |
| `variety` | `string` | Product variety/sub-type |
| `misc` | `string` | Additional metadata (JSON string) |
| `locationData` | `ProductLocationData` | Current and historical locations |

#### ProductLocationData (`chaincode/src/models/product-location-data.ts`)

| Field | Type | Description |
|-------|------|-------------|
| `current` | `ProductLocationEntry` | Where the product is right now |
| `previous` | `ProductLocationEntry[]` | Ordered history of all past locations |

#### ProductLocationEntry (`chaincode/src/models/product-location-entry.ts`)

| Field | Type | Description |
|-------|------|-------------|
| `location` | `string` | Location name/address |
| `arrivalDate` | `string` | ISO 8601 timestamp of arrival |

#### ProductWithHistory (`chaincode/src/models/product-with-history.ts`)

Extends `Product` with one additional field:

| Field | Type | Description |
|-------|------|-------------|
| `componentProducts` | `Product[]` | Fully resolved component product objects |

### Validation Rules

The `requireField()` private method enforces that all critical fields are non-empty:

```typescript
private requireField(value: string | number, fieldName: string) {
    if (!value) {
        throw new Error(`The '${fieldName}' field is required.`);
    }
}
```

- `createProduct` — validates 12 fields before writing to ledger
- `shipProductTo` — validates `newLocation` and `arrivalDate`
- Both write operations check for product existence before proceeding


---

## REST API Reference

The Express.js server (`web-app/server/app.js`) exposes five endpoints on **port 3003**. Every endpoint uses the `connectToNetwork` middleware to establish a Fabric Gateway connection before handling the request.

### Endpoints

#### `POST /createProduct`

Create a new product on the blockchain.

```bash
curl -X POST http://localhost:3003/createProduct \
  -H "Content-Type: application/json" \
  -d '{
    "id": "P001",
    "name": "Organic Apples",
    "barcode": "1234567890128",
    "placeOfOrigin": "Markham Farm, Ontario, Canada",
    "productionDate": "2024-01-10T08:00:00.000Z",
    "expirationDate": "2024-03-10T08:00:00.000Z",
    "unitQuantity": 500,
    "unitQuantityType": "g",
    "batchQuantity": 2000,
    "unitPrice": "$3.50",
    "category": "Fruits",
    "variety": "Honeycrisp",
    "misc": "{}",
    "componentProductIds": [],
    "locationData": {
      "current": {
        "location": "Markham Farm, Ontario, Canada",
        "arrivalDate": "2024-01-10T08:00:00.000Z"
      },
      "previous": []
    }
  }'
```

**Response:** `{ "result": "" }`

---

#### `POST /shipProduct`

Update a product's location (records the movement on-chain).

```bash
curl -X POST http://localhost:3003/shipProduct \
  -H "Content-Type: application/json" \
  -d '{
    "productId": "P001",
    "newLocation": "Toronto Distribution Center, ON",
    "arrivalDate": "2024-01-12T14:30:00.000Z"
  }'
```

**Response:** `{ "status": "Transaction submitted.", "txId": "" }`

---

#### `GET /getProduct?id={productId}`

Retrieve a product's current state from the ledger.

```bash
curl http://localhost:3003/getProduct?id=P001
```

**Response:**
```json
{
  "result": {
    "id": "P001",
    "name": "Organic Apples",
    "barcode": "1234567890128",
    "locationData": {
      "current": {
        "location": "Toronto Distribution Center, ON",
        "arrivalDate": "2024-01-12T14:30:00.000Z"
      },
      "previous": [
        {
          "location": "Markham Farm, Ontario, Canada",
          "arrivalDate": "2024-01-10T08:00:00.000Z"
        }
      ]
    }
  }
}
```

---

#### `GET /getProductWithHistory?id={productId}`

Retrieve a product with all its component products fully resolved.

```bash
curl http://localhost:3003/getProductWithHistory?id=P001
```

**Response:** Same as `getProduct` but includes a `componentProducts` array with full product objects for each ID in `componentProductIds`.

---

#### `GET /productExists?id={productId}`

Check whether a product exists on the ledger.

```bash
curl http://localhost:3003/productExists?id=P001
```

**Response:** `{ "exists": "true" }`

---

### SDK Connection Flow

The `connectToNetwork` middleware (`web-app/server/fabric/network.js`) runs before every endpoint:

```
1. Load connection profile (connection-org1.json)
2. Open file-system wallet → retrieve "manager" identity
3. Create Fabric Gateway → connect to peer0.org1
4. Get "mychannel" network → get "basic" contract
5. Attach contract to req.contract → call next()
```

Key configuration constants:
```javascript
const IDENTITY = 'manager';     // Wallet identity used for all transactions
const CHANNEL  = 'mychannel';   // Application channel name
const CONTRACT = 'basic';       // Chaincode name as deployed
```

---

## Transaction Lifecycle

A complete write transaction (e.g., `shipProductTo`) follows the Fabric v2.2 transaction flow:

```
Step 1: PROPOSAL
  Client SDK constructs a transaction proposal
  ├── Signed with the "manager" identity (X.509 certificate)
  └── Sent to endorsing peers (Org1 + Org2)

Step 2: ENDORSEMENT
  Each peer independently:
  ├── Verifies client identity and permissions
  ├── Executes chaincode in isolated Docker container
  │   ├── getState("P001") → reads current product
  │   ├── Business logic (update location, push history)
  │   └── putState("P001") → proposes state change
  ├── Generates read-write set (what was read, what would change)
  └── Signs the result and returns to client

Step 3: ORDERING
  Client SDK collects endorsements from both orgs
  ├── Validates endorsement policy is satisfied (MAJORITY = both orgs)
  └── Sends endorsed transaction to orderer (orderer.example.com:7050)

Step 4: BLOCK CREATION
  Raft orderer:
  ├── Batches transactions (max 10 per block or 2s timeout)
  ├── Creates new block
  └── Distributes block to all peers on the channel

Step 5: VALIDATION & COMMIT
  Each peer:
  ├── Validates transaction signatures
  ├── Runs MVCC check (ensures read-set versions haven't changed)
  ├── Marks transaction as valid or invalid
  └── Commits block to ledger and updates world state

Step 6: EVENT NOTIFICATION
  SDK receives commit event → returns success to Express → HTTP 200
```

**Query transactions** (`evaluateTransaction`) skip steps 3–6 entirely. They execute on a single peer and return the result directly.


---

## Identity & Access Control

### Membership Service Providers (MSPs)

The network uses three MSPs to manage organizational identities:

| MSP | Organization | Role |
|-----|-------------|------|
| `OrdererMSP` | Orderer Org | Manages the ordering service identity |
| `Org1MSP` | Organization 1 | First peer organization (runs the web app) |
| `Org2MSP` | Organization 2 | Second peer organization |

### Certificate Authorities

Each organization has a dedicated Fabric CA for issuing X.509 certificates:

| CA | Port | Organization |
|----|------|-------------|
| `ca_org1` | 7054 | Org1 — issues peer and client certificates |
| `ca_org2` | 8054 | Org2 — issues peer and client certificates |
| `ca_orderer` | 9054 | Orderer — issues orderer node certificates |

### Identity Enrollment Flow

```
1. enrollAdmin.js
   └── Enrolls CA admin (admin/adminpw) → stores X.509 cert in wallet

2. registerUsers.js
   ├── Registers "manager" identity with CA → stores in wallet
   └── Registers "employee" identity with CA → stores in wallet

3. network.js (runtime)
   └── Loads "manager" identity from wallet → connects Gateway
```

### Defined Roles

The project defines three role constants in `chaincode/src/roles.ts`:

```typescript
export const ORG_MANAGER_ROLE = 'manager';   // Full access (create, ship, query)
export const ORG_EMPLOYEE_ROLE = 'employee'; // Intended for limited access
export const CLIENT = 'client';              // External client role
```

### TLS Security

All network communication is encrypted with TLS:
- **Peer-to-peer:** TLS enabled via `CORE_PEER_TLS_ENABLED=true`
- **Client-to-peer:** TLS certificates loaded from organization MSP
- **Orderer:** TLS enabled via `ORDERER_GENERAL_TLS_ENABLED=true`
- **Certificate Authorities:** TLS enabled via `FABRIC_CA_SERVER_TLS_ENABLED=true`

### Endorsement Policy

The default endorsement policy requires **MAJORITY** approval:

```yaml
# From configtx.yaml
Endorsement:
    Type: ImplicitMeta
    Rule: "MAJORITY Endorsement"
```

This means both Org1 **and** Org2 must endorse every transaction before it can be committed. This prevents any single organization from unilaterally modifying the ledger.

---

## Network Configuration Deep Dive

### Consensus Mechanism

The network uses **etcdRaft** — a Raft-based crash fault tolerant (CFT) ordering service.

```yaml
# From configtx.yaml
Orderer: &OrdererDefaults
    OrdererType: etcdraft
    EtcdRaft:
        Consenters:
        - Host: orderer.example.com
          Port: 7050
          ClientTLSCert: .../tls/server.crt
          ServerTLSCert: .../tls/server.crt
    BatchTimeout: 2s
    BatchSize:
        MaxMessageCount: 10
        AbsoluteMaxBytes: 99 MB
        PreferredMaxBytes: 512 KB
```

**Why Raft?** Raft is the recommended ordering service for Fabric v2.x. It provides leader-based consensus with automatic leader election, making it suitable for production deployments. The single-node configuration here is for development; production would use 3 or 5 orderer nodes for fault tolerance.

### Channel & Policy Configuration

**Channel:** `mychannel` (defined via `TwoOrgsChannel` profile)

**Organization-Level Policies:**

| Policy | Org1 / Org2 Rule | Meaning |
|--------|-----------------|---------|
| Readers | `OR('OrgMSP.admin', 'OrgMSP.peer', 'OrgMSP.client')` | Admins, peers, and clients can read |
| Writers | `OR('OrgMSP.admin', 'OrgMSP.client')` | Only admins and clients can write |
| Admins | `OR('OrgMSP.admin')` | Only admins can perform admin operations |
| Endorsement | `OR('OrgMSP.peer')` | Peers endorse transactions |

**Channel-Level Policies:**

| Policy | Rule | Meaning |
|--------|------|---------|
| Readers | `ANY Readers` | Any org's readers can read |
| Writers | `ANY Writers` | Any org's writers can write |
| Admins | `MAJORITY Admins` | Majority of org admins needed for admin ops |
| LifecycleEndorsement | `MAJORITY Endorsement` | Both orgs must approve chaincode lifecycle |
| Endorsement | `MAJORITY Endorsement` | Both orgs must endorse transactions |

### Docker Services

The network runs as a set of Docker containers orchestrated by Docker Compose:

| Container | Image | Port | Purpose |
|-----------|-------|------|---------|
| `orderer.example.com` | `hyperledger/fabric-orderer` | 7050 | Raft ordering service |
| `peer0.org1.example.com` | `hyperledger/fabric-peer` | 7051 | Org1 endorsing/committing peer |
| `peer0.org2.example.com` | `hyperledger/fabric-peer` | 9051 | Org2 endorsing/committing peer |
| `cli` | `hyperledger/fabric-tools` | — | Admin CLI for channel/chaincode ops |
| `ca_org1` | `hyperledger/fabric-ca` | 7054 | Org1 Certificate Authority |
| `ca_org2` | `hyperledger/fabric-ca` | 8054 | Org2 Certificate Authority |
| `ca_orderer` | `hyperledger/fabric-ca` | 9054 | Orderer Certificate Authority |
| `couchdb0` | `couchdb:3.1.1` | 5984 | Org1 state database (optional) |
| `couchdb1` | `couchdb:3.1.1` | 7984 | Org2 state database (optional) |

### State Database Options

| Option | Default | Rich Queries | Performance |
|--------|---------|-------------|-------------|
| **LevelDB** | ✅ Yes | ❌ Key-based only | Faster for simple lookups |
| **CouchDB** | Optional (`-s couchdb`) | ✅ JSON queries | Supports complex queries |

To enable CouchDB, start the network with:
```bash
./network.sh up -s couchdb
```


---

## Prerequisites

Before running this project, ensure you have the following installed:

| Requirement | Version | Purpose |
|-------------|---------|---------|
| **Docker** | 20.10+ | Container runtime for all network components |
| **Docker Compose** | 1.29+ | Multi-container orchestration |
| **Node.js** | 12+ | Chaincode runtime and web application |
| **npm** | 6+ | Package management |
| **Hyperledger Fabric Binaries** | 2.2.x | `peer`, `configtxgen`, `cryptogen`, `fabric-ca-client` |
| **Go** | 1.14+ | Required by some Fabric tools |

### Installing Fabric Binaries

```bash
# Download Fabric binaries and Docker images
curl -sSL https://bit.ly/2ysbOFE | bash -s -- 2.2.0 1.4.9
```

This places binaries in a `bin/` directory and pulls the required Docker images.

---

## Getting Started

### Step 1 — Start the Network

```bash
cd network/fabric-network

# Start with LevelDB (default)
./network.sh up

# OR start with CouchDB for rich queries
./network.sh up -s couchdb

# OR start with Certificate Authorities (instead of cryptogen)
./network.sh up -ca
```

This will:
1. Generate cryptographic material for all organizations
2. Create the orderer genesis block
3. Start all Docker containers (orderer, peers, CAs)

### Step 2 — Create the Channel

```bash
./network.sh createChannel -c mychannel
```

This will:
1. Generate the channel creation transaction
2. Create the `mychannel` application channel
3. Join both peers to the channel
4. Set anchor peers for cross-org gossip communication

### Step 3 — Deploy the Chaincode

```bash
./network.sh deployCC -ccn basic -ccp ../../chaincode -ccl typescript
```

This will:
1. Compile TypeScript chaincode to JavaScript (`npm install && npm run build`)
2. Package the chaincode into a `.tar.gz` archive
3. Install the package on `peer0.org1` and `peer0.org2`
4. Approve the chaincode definition for both organizations
5. Commit the chaincode definition to the channel
6. The chaincode is now live and ready to accept transactions

### Step 4 — Set Up the Web Application

```bash
cd ../../web-app/server
npm install
```

### Step 5 — Enroll Identities

These scripts register blockchain identities with the Certificate Authority and store them in a local wallet.

```bash
# 1. Enroll the CA admin
node fabric/enrollAdmin.js

# 2. Register application users (manager + employee)
node fabric/registerUsers.js
```

After this step, your wallet directory will contain:
```
web-app/server/fabric/wallet/
├── admin.id       # CA admin identity
├── manager.id     # Application user (used by the API)
└── employee.id    # Application user (for future role-based access)
```

### Step 6 — Start the API Server

```bash
node app.js
```

The server starts on **http://localhost:3003**. You can now interact with the blockchain through the REST API.

### Shutting Down

```bash
cd network/fabric-network
./network.sh down
```

This stops all containers, removes volumes, and cleans up generated crypto material.

---

## Usage Examples

### Complete Workflow: Track a Product from Farm to Store

```bash
# 1. Register a new product at the farm
curl -X POST http://localhost:3003/createProduct \
  -H "Content-Type: application/json" \
  -d '{
    "id": "APPLE-2024-001",
    "name": "Organic Honeycrisp Apples",
    "barcode": "0123456789012",
    "placeOfOrigin": "Green Valley Farm, Markham, ON",
    "productionDate": "2024-01-10T06:00:00.000Z",
    "expirationDate": "2024-04-10T06:00:00.000Z",
    "unitQuantity": 1,
    "unitQuantityType": "kg",
    "batchQuantity": 5000,
    "unitPrice": "$4.99",
    "category": "Fruits",
    "variety": "Honeycrisp",
    "misc": "{\"organic\": true, \"certification\": \"USDA-NOP\"}",
    "componentProductIds": [],
    "locationData": {
      "current": {
        "location": "Green Valley Farm, Markham, ON",
        "arrivalDate": "2024-01-10T06:00:00.000Z"
      },
      "previous": []
    }
  }'

# 2. Ship to distribution center
curl -X POST http://localhost:3003/shipProduct \
  -H "Content-Type: application/json" \
  -d '{
    "productId": "APPLE-2024-001",
    "newLocation": "FreshCo Distribution Center, Brampton, ON",
    "arrivalDate": "2024-01-11T14:00:00.000Z"
  }'

# 3. Ship to retail store
curl -X POST http://localhost:3003/shipProduct \
  -H "Content-Type: application/json" \
  -d '{
    "productId": "APPLE-2024-001",
    "newLocation": "Walmart Supercentre, 900 Dufferin St, Toronto, ON",
    "arrivalDate": "2024-01-12T09:30:00.000Z"
  }'

# 4. Query the full product with location history
curl http://localhost:3003/getProduct?id=APPLE-2024-001
```

**Result:** The product now shows its complete journey:
```
Green Valley Farm → FreshCo Distribution Center → Walmart Supercentre
```

---

## Testing

The chaincode includes a comprehensive unit test suite using **Mocha**, **Chai**, and **Sinon**.

### Running Tests

```bash
cd chaincode
npm install
npm test
```

### Test Coverage

The project enforces **100% code coverage** across all metrics:

```json
{
  "nyc": {
    "check-coverage": true,
    "statements": 100,
    "branches": 100,
    "functions": 100,
    "lines": 100
  }
}
```

### Test Scenarios

| Contract Function | Test Cases |
|-------------------|------------|
| `productExists` | Returns `true` for existing product; returns `false` for non-existent product |
| `createProduct` | Creates product successfully; rejects duplicate ID; rejects missing `id`; rejects missing `name` |
| `shipProductTo` | Updates current location; moves old location to history; rejects non-existent product; rejects empty location; rejects empty arrival date |
| `getProduct` | Returns correct product data; rejects non-existent product |
| `getProductWithHistory` | Returns product with resolved component products; rejects non-existent product |

### Test Architecture

Tests use a `TestContext` class that stubs the Fabric `ChaincodeStub` and `ClientIdentity`:

```typescript
class TestContext implements Context {
    public stub = sinon.createStubInstance(ChaincodeStub);
    public clientIdentity = sinon.createStubInstance(ClientIdentity);
}
```

Pre-configured stub responses simulate two products (`1001` and `1002`) in the ledger state.


---

## Technology Stack

### Blockchain Layer

| Technology | Version | Role |
|-----------|---------|------|
| [Hyperledger Fabric](https://www.hyperledger.org/use/fabric) | 2.2.x | Permissioned blockchain framework |
| [Fabric Contract API](https://www.npmjs.com/package/fabric-contract-api) | ^2.2.0 | Smart contract development SDK |
| [Fabric Shim](https://www.npmjs.com/package/fabric-shim) | ^2.2.0 | Low-level chaincode interface |
| [Fabric CA](https://hyperledger-fabric-ca.readthedocs.io/) | 1.4.x | Certificate Authority for identity management |

### Application Layer

| Technology | Version | Role |
|-----------|---------|------|
| [Node.js](https://nodejs.org/) | 12+ | Server runtime |
| [Express.js](https://expressjs.com/) | ^4.17.1 | REST API framework |
| [Fabric Network SDK](https://www.npmjs.com/package/fabric-network) | ^2.2.7 | High-level blockchain client SDK |
| [Fabric CA Client](https://www.npmjs.com/package/fabric-ca-client) | ^2.2.7 | CA enrollment and registration |

### Development & Testing

| Technology | Version | Role |
|-----------|---------|------|
| [TypeScript](https://www.typescriptlang.org/) | ^4.3.4 | Chaincode language |
| [Mocha](https://mochajs.org/) | ^7.1.1 | Test framework |
| [Chai](https://www.chaijs.com/) | ^4.2.0 | Assertion library |
| [Sinon](https://sinonjs.org/) | ^9.0.1 | Mocking and stubbing |
| [nyc](https://istanbul.js.org/) | ^15.0.0 | Code coverage |
| [TSLint](https://palantir.github.io/tslint/) | ^6.1.3 | TypeScript linting |

### Infrastructure

| Technology | Role |
|-----------|------|
| [Docker](https://www.docker.com/) | Container runtime |
| [Docker Compose](https://docs.docker.com/compose/) | Multi-container orchestration |
| [CouchDB](https://couchdb.apache.org/) | Optional rich-query state database |
| [LevelDB](https://github.com/google/leveldb) | Default key-value state database |

---

## Known Limitations & Future Improvements

### Current Limitations

| Area | Limitation | Impact |
|------|-----------|--------|
| **Access Control** | `roles.ts` defines roles but they are not enforced in chaincode | Any authenticated user can create/ship products |
| **Gateway Lifecycle** | `network.js` creates a new Gateway per request without disconnecting | Connection resource leak under load |
| **Query Capability** | No `getAllProducts` or search-by-field functions | Products can only be retrieved by exact ID |
| **Single-Org Client** | Web app only connects through Org1 | Org2 has no client application |
| **Transaction Data** | `product-transactions.txdata` references non-existent functions (`readProduct`, `updateProduct`, `deleteProduct`) | Sample data is stale |

### Recommended Improvements

1. **Implement Role-Based Access Control in Chaincode**
   - Use `ctx.clientIdentity.getAttributeValue('role')` to enforce permissions
   - Restrict `createProduct` and `shipProductTo` to `manager` role
   - Allow `employee` and `client` roles for query-only access

2. **Add Rich Query Support**
   - Implement `getProductsByCategory()`, `getProductsByLocation()`, and `getAllProducts()` using CouchDB selectors
   - Add pagination support for large result sets

3. **Fix Gateway Connection Management**
   - Implement a singleton Gateway pattern or add proper cleanup
   - Disconnect the gateway after each request completes

4. **Add Product Update and Delete Operations**
   - Implement `updateProduct()` for modifying product metadata
   - Implement `deleteProduct()` with proper authorization checks

5. **Enhance Security**
   - Replace hardcoded CA credentials with environment variables
   - Enable CA TLS certificate verification
   - Add input sanitization for all chaincode parameters

---

## Authors

| Name | GitHub | Role |
|------|--------|------|
| Alexandre Rapchan B. Barros | [@AleRapchan](https://www.github.com/AleRapchan) | Project Lead, Architecture |
| Alexei Pancratov | [@AlexeiPancratov](https://github.com/alexeipancratov) | TypeScript Chaincode |
| Michael Francis Jerome Victor | [@Mike-64](https://github.com/Mike-64) | Certificate Authorities, Review |
| Dhruvam Patel | [@DhruvamPatel](https://github.com/dhruvampatel) | SDK & API Integration |

---

## License

This project is licensed under the **Apache License 2.0** — see the [LICENSE](LICENSE) file for details.

---

## References

### Hyperledger Fabric Documentation
- [What is Hyperledger Fabric?](https://hyperledger-fabric.readthedocs.io/en/latest/whatis.html)
- [Fabric v2.2 Documentation](https://hyperledger-fabric.readthedocs.io/en/release-2.2/)
- [Writing Your First Chaincode](https://hyperledger-fabric.readthedocs.io/en/release-2.2/chaincode4ade.html)
- [Private Data Collections](https://hyperledger-fabric.readthedocs.io/en/release-2.2/private-data/private-data.html)
- [Fabric CA User Guide](https://hyperledger-fabric-ca.readthedocs.io/en/latest/)

### Related Projects
- [Hyperledger Fabric GitHub](https://github.com/hyperledger/fabric)
- [Hyperledger Blockchain Explorer](https://github.com/hyperledger/blockchain-explorer)
- [IBM Food Trust](https://www.ibm.com/blockchain/solutions/food-trust)
- [Fair Trade International](https://www.fairtrade.net/)

### Additional Resources
- [Hyperledger Wiki](https://wiki.hyperledger.org/display/fabric)
- [Blockchain Governance Considerations](https://www.blockchain.ae/articles/blockchain-governance-considerations)
- [Private Data Collections on Fabric (IBM)](https://github.com/IBM/private-data-collections-on-fabric)

---

<p align="center">
  <strong>Built with Hyperledger Fabric — Connecting the food supply chain through trust and transparency.</strong>
</p>
