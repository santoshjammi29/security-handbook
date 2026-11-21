# 🎓 Interview Guide 03: Senior Cloud & Lakehouse Security Architect

## Core Competency Checklist:
- [ ] Unity Catalog 3-tier governance and downscoped token delegation
- [ ] Envelope Encryption mechanics (DEK + KEK) and KMS throughput limits
- [ ] Cryptographic Shredding for GDPR compliance on immutable Parquet files
- [ ] Supply chain security with SLSA Level 3 and Cosign container signing
- [ ] Zero-Trust egress filtering and Cloud Firewall SNI inspection

---

### Question 1: How does Databricks Unity Catalog protect cloud storage credentials from being compromised by cluster users?
**Answer**:
- In legacy DBFS mount architectures, the cluster driver and worker nodes stored long-lived master cloud credentials (IAM instance profiles or storage keys) in memory or environment variables, allowing any user with notebook access to extract them via Python or bash commands.
- Under **Unity Catalog**:
  1. The compute cluster possesses **zero cloud storage credentials**.
  2. When a user runs a query (`SELECT * FROM catalog.schema.table`), the cluster query planner submits the query plan to the Unity Catalog control plane.
  3. Unity Catalog evaluates RBAC policies, row filters, and column masks.
  4. Unity Catalog communicates with the cloud provider (AWS STS or Azure Entra ID) to negotiate an **ephemeral downscoped token** (SAS or STS token).
  5. The token is valid for only 1 hour and strictly scoped to the exact path prefix of the table's underlying Parquet directory.
  6. The cluster uses this downscoped token to read data and never gains access to other storage accounts or parent directories.

---

### Question 2: Walk through the mechanics of Envelope Encryption. Why is it used over direct KMS encryption?
**Answer**:
1. **Direct KMS Limitations**:
   - Centralized KMS services (AWS KMS, Azure Key Vault) have strict request rate limits (e.g. 10,000 requests/sec). Encrypting millions of individual records or multi-terabyte files directly via KMS API calls exhausts quotas, incurs astronomical costs, and introduces intolerable network latency.
2. **Envelope Encryption Pipeline**:
   - The application calls `KMS:GenerateDataKey(KEK)`, which generates a 256-bit symmetric Data Encryption Key (DEK) inside a FIPS 140-2 Level 3 Hardware Security Module (HSM).
   - The KMS returns two items: the **Plaintext DEK** and the **Encrypted DEK** (encrypted under the master KEK).
   - The application encrypts the local data using the Plaintext DEK via hardware-accelerated **AES-256-GCM** ($10+\text{ GB/sec}$ throughput).
   - The application securely wipes the Plaintext DEK from RAM and appends the Encrypted DEK to the file header.
- *Outcome*: High-throughput local encryption with centralized, auditable root key management.

---

### Question 3: How does Cryptographic Shredding solve the conflict between immutable Delta Lake storage and the GDPR Right to Be Forgotten?
**Answer**:
- **The Conflict**: Delta Lake and Apache Iceberg store data in immutable, compressed Parquet files with append-only transaction logs. Fulfilling a GDPR Article 17 deletion request requires scanning petabytes of historical files, deleting specific rows, re-writing new Parquet files, and re-writing historical checkpoints. This is computationally expensive, risks write conflicts, and leaves historical versions accessible via Time Travel.
- **The Crypto-Shredding Solution**:
  1. At ingestion, each customer's PII is encrypted using a unique, customer-specific encryption key ($\text{Key}_{U}$) managed in a secure key store.
  2. The encrypted data is written into standard immutable Parquet files.
  3. When an erasure request is certified, the system permanently destroys $\text{Key}_{U}$ from the key store.
  4. Without $\text{Key}_{U}$, the customer's PII across all historical Parquet files, snapshots, and backups becomes mathematically unrecoverable ciphertext, satisfying legal deletion mandates instantly without touching physical storage files.

---

### Question 4: How does Cosign keyless signing work, and how does it prevent supply chain attacks in containerized data workloads?
**Answer**:
- Traditional code signing required creating and storing long-lived private GPG keys in CI/CD secrets—introducing another critical secret that could be leaked or stolen.
- **Cosign Keyless Signing (Sigstore)**:
  1. When a container image is built in GitHub Actions, the runner requests an OIDC token proving its identity (repo name, commit SHA).
  2. The runner presents this token to **Fulcio** (an ephemeral Certificate Authority), which issues a short-lived X.509 certificate (valid for $\approx 10\text{ minutes}$) binding the runner's identity to an ephemeral public key.
  3. The runner signs the container image hash and writes the cryptographic signature to **Rekor** (a public, tamper-evident transparency log).
  4. The private key is discarded immediately.
- *At Deployment*: Kubernetes admission controllers or deployment gates query Rekor and verify that the image signature was generated by the approved GitHub workflow, preventing attackers from deploying unauthorized or poisoned container images to production clusters.

---

### Question 5: What is the Write-Audit-Publish (WAP) pattern, and how does it function as a security gate?
**Answer**:
- In standard ELT, transformations write directly to production tables. If malicious data, unmasked PII, or SQL injection artifacts slip through, production data is contaminated immediately.
- Under **Write-Audit-Publish**:
  1. **Write**: The pipeline writes transformed data to an isolated, non-production staging branch or temporary snapshot (using Delta Lake table clones or Git-like catalog branching).
  2. **Audit**: An automated test suite executes security assertions: scanning for unmasked credit cards/SSNs, checking schema constraints, verifying row-count variance, and checking cryptographic integrity.
  3. **Publish**: If all audit assertions pass with 100% success, a metadata fast-forward operation atomically merges the staging snapshot into the production table. If any assertion fails, the staging branch is discarded, protecting production data with zero blast radius.
