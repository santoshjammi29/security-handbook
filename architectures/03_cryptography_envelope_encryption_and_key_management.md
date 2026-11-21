# 🏛️ Architecture Deep Dive 03: Cryptography, Envelope Encryption & Key Lifecycle

![Delta Lake vs. Apache Iceberg Storage & Metadata Architecture](../assets/whiteboards/delta_lake_vs_iceberg_architecture.webp)
> *Architectural Blueprint: [Delta Lake vs. Apache Iceberg Storage & Metadata Architecture](../docs/whiteboard_diagrams/README.md#delta-vs-iceberg-architecture). View full multi-persona breakdown and engine mechanisms in the master index.*

---

## 1. Cryptographic Primitives for Data Platforms

Modern cloud data platforms process terabytes of data daily. Ensuring confidentiality and integrity requires selecting appropriate cryptographic primitives:

```mermaid
flowchart TD
    subgraph CryptoTaxonomy["Data Platform Cryptographic Primitives"]
        Sym["Symmetric Encryption (AES-256-GCM)"] --> Sym_Use["High-throughput bulk data encryption (Parquet files, temp spills). Fast, hardware-accelerated (AES-NI)."]
        Asym["Asymmetric Encryption (RSA-4096 / ECDSA)"] --> Asym_Use["Identity assertions, JWT signing, TLS handshakes, key exchange."]
        Hash["Cryptographic Hashing & MAC (HMAC-SHA256, SHA-3)"] --> Hash_Use["Deterministic tokenization, surrogate key generation, data integrity checks."]
    end
```

### Why AES-256-GCM (Galois/Counter Mode)?
Traditional AES cipher block chaining (CBC) modes require separate HMAC computations to detect tampering and are vulnerable to padding oracle attacks. 

**AES-256-GCM is an Authenticated Encryption with Associated Data (AEAD) algorithm**:
- Encrypts plaintext into ciphertext using a counter mode stream.
- Computes an authentication tag ($\text{AuthTag}$) across ciphertext and optional unencrypted metadata (Associated Data) via Galois field polynomial multiplication.
- Guarantees both **confidentiality** (data cannot be read without key) and **authenticity** (tampering with a single bit in the ciphertext causes immediate decryption failure).

---

## 2. Envelope Encryption Architecture (DEK + KEK)

Encrypting multi-terabyte datasets directly with centralized Cloud Key Management Services (AWS KMS, Azure Key Vault, Google Cloud KMS) introduces prohibitive API latency, throttled request limits (e.g. 10,000 requests/sec limit), and severe costs.

**Envelope Encryption** resolves this by encrypting data with a local data key and encrypting that data key with a remote master key:

```mermaid
sequenceDiagram
    autonumber
    participant App as ETL Engine (Spark / Python Worker)
    participant KMS as Cloud KMS / HSM (Azure Key Vault / AWS KMS)
    participant Disk as Persistent Object Storage (Delta Lake Parquet)

    App->>KMS: Request new Data Encryption Key (GenerateDataKey API)
    Note over KMS: KMS uses Key Encryption Key (KEK / Root Key)<br>inside FIPS 140-2 Level 3 HSM
    KMS-->>App: Return Plaintext DEK (256-bit) + Encrypted DEK
    App->>App: Encrypt local Parquet row groups using Plaintext DEK (AES-256-GCM)
    App->>App: Securely zero-out (scrub) Plaintext DEK from RAM
    App->>Disk: Write Ciphertext Data + Encrypted DEK header
```

### Mathematical Formulation of Envelope Encryption:
$$\text{Ciphertext Data} = \text{AES-GCM-Encrypt}(\text{Plaintext Data}, \text{DEK}, \text{Nonce})$$
$$\text{Encrypted DEK} = \text{KMS-Encrypt}(\text{Plaintext DEK}, \text{KEK})$$

When reading data back:
1. The engine extracts the **Encrypted DEK** from the file header.
2. The engine calls `KMS-Decrypt(Encrypted DEK, KEK)` to retrieve the **Plaintext DEK**.
3. The engine decrypts the ciphertext data locally using hardware AES-NI CPU instructions.

---

## 3. Customer-Managed Keys (CMK / BYOK) vs. Service-Managed Keys

| Architecture Tier | Key Owner | Rotation Management | Revocation Control | Threat Model Mitigated |
| :--- | :--- | :--- | :--- | :--- |
| **Service-Managed (Default)** | Cloud Provider (AWS / Azure) | Automated by Cloud Provider | Cannot be unilaterally revoked | Protection against physical disk theft at cloud datacenter. |
| **Customer-Managed (CMK / BYOK)** | Enterprise Security Team | Customer-defined schedule (e.g. annual) | Instant unilateral key disabling | Protects against rogue cloud admins, subpoena seizures, regulatory non-compliance. |
| **Hardware Security Module (HSM)** | Dedicated Dedicated Cloud HSM (FIPS 140-2 Level 3) | Managed via PKCS#11 API | Cryptographic zeroization | Satisfies banking, defense, and top-tier compliance mandates. |

---

## 4. Cryptographic Shredding for Immutable Lakehouses

Under GDPR Article 17 (Right to Erasure), organizations must delete a user's personal data upon request. However, rewriting historical append-only Delta Lake tables across petabytes of storage is technically unviable.

**Cryptographic Shredding Architecture**:
1. At ingestion time, encrypt each user's PII attributes using a unique per-user encryption key ($\text{Key}_{U}$) stored in a secure key store.
2. Store the encrypted PII in standard immutable Parquet files.
3. Upon receiving a certified deletion request, permanently destroy $\text{Key}_{U}$ from the key store.
4. Without $\text{Key}_{U}$, the user's records in all historical files, replicas, snapshots, and backups become mathematically unrecoverable random noise, satisfying legal erasure mandates instantly at zero rewrite cost.
