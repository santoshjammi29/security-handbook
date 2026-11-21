# 🔑 Module 04: Secrets Management & Encryption Architecture

## 1. Secrets Management Hierarchy & Lifecycle

Hardcoded database connection strings, plain-text API tokens, and static passwords in pipeline scripts are catastrophic vulnerabilities. A production secrets management architecture adheres to a strict 5-stage lifecycle:

```mermaid
flowchart TD
    Gen["1. Generation (High-Entropy, Cryptographically Secure)"] --> Store["2. Storage (HSM-Backed KMS: Azure Key Vault / AWS Secrets Manager)"]
    Store --> Access["3. Access (RBAC, JIT Resolution, In-Memory Ephemeral Injection)"]
    Access --> Rotate["4. Automated Rotation (Dual-Secret Protocol, Event-Driven)"]
    Rotate --> Revoke["5. Revocation (Instant Invalidation, Zero-Downtime Rollover)"]
```

---

## 2. Envelope Encryption Architecture (DEK + KEK)

Encrypting large multi-terabyte lakehouse files directly with asymmetric KMS keys introduces prohibitive network latency and cost. Enterprise data platforms utilize **Envelope Encryption**:

```mermaid
flowchart TD
    subgraph LocalCompute["Client / Lakehouse Compute Node"]
        Plaintext["Plaintext Data (Delta Parquet / CSV)"]
        GenDEK["1. Request DEK from KMS"]
        AES["3. Fast Local AES-256-GCM Encryption"]
        Ciphertext["Encrypted Ciphertext Data"]
        EncDEK["Encrypted DEK (Stored alongside data)"]
    end

    subgraph CloudKMS["Cloud Hardware Security Module (HSM / KMS)"]
        KEK["Key Encryption Key (KEK / Root Key)<br>Never leaves the HSM boundary"]
    end

    GenDEK -->|GenerateDataKey Call| KEK
    KEK -->|Return Plaintext DEK + Encrypted DEK| LocalCompute
    Plaintext --> AES
    AES --> Ciphertext
    EncDEK -.->|Appended to Parquet Header| Ciphertext
```

### Encryption Key Classification:
1. **Data Encryption Key (DEK)**: A symmetric key (AES-256) generated locally for a specific file or batch of rows. Encrypts and decrypts data at high throughput.
2. **Key Encryption Key (KEK)**: An asymmetric or symmetric root master key stored inside a FIPS 140-2 Level 3 Hardware Security Module (HSM). The KEK is used exclusively to encrypt and decrypt the DEK. The KEK never leaves the HSM.

---

## 3. Dual-Secret Zero-Downtime Rotation Protocol

Rotating database credentials or API secrets without downtime requires supporting two valid secrets concurrently during the rollover window:

```mermaid
sequenceDiagram
    autonumber
    participant App as App / ETL Pipeline
    participant Vault as Secret Vault
    participant DB as Target Database / API
    participant Rotator as Automated Rotation Worker

    Note over Vault,DB: Normal State: Secret-v1 is active in App and DB
    Rotator->>DB: 1. Generate & Add Secret-v2 as valid credential in DB
    Rotator->>Vault: 2. Store Secret-v2 as pending/secondary version
    Rotator->>Vault: 3. Promote Secret-v2 to active, mark Secret-v1 deprecated
    App->>Vault: 4. Periodic secret refresh fetches Secret-v2
    App->>DB: 5. App connects successfully using Secret-v2
    Rotator->>DB: 6. After 24-hour verification window, purge Secret-v1 from DB
```

---

## 4. Databricks Secret Scopes: AKV-Backed vs. Databricks-Managed

- **Azure Key Vault-Backed Secret Scope**:
  - The secret scope acts as a real-time pointer to an Azure Key Vault instance.
  - Secrets are never stored or cached inside Databricks control plane storage.
  - Access is governed by Azure Key Vault access policies and Entra ID RBAC.
  - In code: `dbutils.secrets.get(scope = "akv-prod-scope", key = "postgres-connection-string")`
- **Security Rule**: Never print or serialize secret values to driver logs, console output, or table metadata. Databricks automatically redacts values retrieved via `dbutils.secrets.get` in notebook cell outputs with `[REDACTED]`.
