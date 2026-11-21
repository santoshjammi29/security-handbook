# ⚡ 50-Card Rapid-Fire Technical Security Flashcard Deck

---

### 1. What does ZTA stand for, and what are its three core principles?
**Answer**: Zero-Trust Architecture (NIST SP 800-207). 1) Verify explicitly; 2) Use least-privileged access; 3) Assume breach.

---

### 2. How does OIDC eliminate static secrets in CI/CD pipelines?
**Answer**: Ephemeral OIDC ID Tokens signed by GitHub/GitLab are exchanged with cloud STS for short-lived (15–60 min) access tokens, eliminating permanent static client secrets.

---

### 3. What is the difference between Authentication (AuthN) and Authorization (AuthZ)?
**Answer**: AuthN verifies *who you are* (identity); AuthZ determines *what you can do* (permissions/entitlements).

---

### 4. What is RBAC, and what is its main limitation in large enterprises?
**Answer**: Role-Based Access Control. Its main drawback is **Role Explosion**, where hundreds of permutations are created to satisfy slight permission variations.

---

### 5. What is ABAC, and how does it prevent role explosion?
**Answer**: Attribute-Based Access Control evaluates policies dynamically using attributes of the subject, resource, action, and environment context without creating static roles.

---

### 6. What is a Toxic Combination in enterprise access governance?
**Answer**: A Separation of Duties (SoD) violation where a single user holds conflicting entitlements (e.g. creating a payment batch and approving it).

---

### 7. What is Private Link?
**Answer**: A cloud networking service providing private, dedicated connectivity to cloud resources over the cloud backbone without exposing traffic to the public internet.

---

### 8. What is a Private Endpoint?
**Answer**: A virtual network interface (NIC) with a private IP address in a customer subnet that acts as the entry point for Private Link traffic.

---

### 9. What is Split-Horizon DNS in cloud networking?
**Answer**: A DNS configuration where internal VNet queries for a service resolve to a private IP, while external internet queries resolve to a rejected or public IP.

---

### 10. What does NPIP mean in Databricks cluster networking?
**Answer**: No Public IP (Secure Cluster Connectivity). Cluster worker nodes have zero public IPs and communicate with the control plane via an outbound encrypted reverse relay.

---

### 11. What is Envelope Encryption?
**Answer**: A cryptographic pattern where data is encrypted locally using a fast symmetric Data Encryption Key (DEK), and the DEK is encrypted using a master Key Encryption Key (KEK) inside a KMS/HSM.

---

### 12. What is a DEK, and what algorithm is typically used for it?
**Answer**: Data Encryption Key. Typically a 256-bit symmetric key using AES-256-GCM authenticated encryption.

---

### 13. What is a KEK, and where does it reside?
**Answer**: Key Encryption Key (Root Master Key). It resides inside a FIPS 140-2 Level 3 Hardware Security Module (HSM) and never leaves the HSM boundary.

---

### 14. What is the difference between CMK (BYOK) and Service-Managed Keys?
**Answer**: Customer-Managed Keys (CMK) give the enterprise complete ownership, rotation control, and unilateral revocation rights; Service-Managed Keys are controlled entirely by the cloud provider.

---

### 15. What is Cryptographic Shredding?
**Answer**: Deleting the unique per-user encryption key used to encrypt that user's PII, rendering their historical records across immutable lakehouse Parquet files permanently unrecoverable.

---

### 16. What is the Dual-Secret Rotation Protocol?
**Answer**: Maintaining two active credentials concurrently during a rollover window so applications can migrate without experiencing connection dropouts.

---

### 17. What is the difference between SAST and DAST?
**Answer**: SAST (Static Application Security Testing) analyzes source code without executing it; DAST (Dynamic Application Security Testing) tests running applications externally for vulnerabilities.

---

### 18. What is an SBOM, and why is it important?
**Answer**: Software Bill of Materials. A formal machine-readable inventory of all third-party libraries, modules, and dependencies in a software artifact used to track CVEs.

---

### 19. What is Cosign used for in container security?
**Answer**: Signing and verifying container images and artifacts keylessly using Sigstore, Fulcio certificates, and the Rekor transparency ledger.

---

### 20. Why should production Docker containers run as non-root users?
**Answer**: Running as non-root mitigates container escape attacks, preventing an attacker who achieves remote code execution from obtaining root privileges on the host node.

---

### 21. What is a Distroless container image?
**Answer**: An image containing only the application and its runtime dependencies, stripping package managers (`apt`, `apk`), shells (`bash`), and OS utilities to minimize attack surface.

---

### 22. What causes SQL injection in Python applications?
**Answer**: Concatenating or interpolating untrusted input directly into raw SQL query strings instead of using parameterized bindings or prepared statements.

---

### 23. Why is `pickle.loads()` unsafe on untrusted data?
**Answer**: Python's `pickle` module executes arbitrary bytecode during deserialization, allowing attackers to trigger Remote Code Execution (RCE).

---

### 24. What is an AEAD cipher, and give an example?
**Answer**: Authenticated Encryption with Associated Data (e.g. AES-256-GCM). It simultaneously guarantees confidentiality and authenticity (tamper detection).

---

### 25. Why must string comparisons of cryptographic hashes use `hmac.compare_digest()`?
**Answer**: To prevent timing side-channel attacks by evaluating all bytes in constant time rather than terminating on the first mismatched byte.

---

### 26. What is the Write-Audit-Publish (WAP) pattern?
**Answer**: Writing data to an isolated staging snapshot, running automated quality and security assertions (PII leaks, schema validity), and publishing to production only if audits pass.

---

### 27. What is Deterministic Tokenization?
**Answer**: Replacing sensitive data with surrogate tokens generated via keyed HMAC or AES-SIV so identical plaintext produces identical tokens, preserving relational join capabilities.

---

### 28. What is the difference between Random Tokenization and Deterministic Tokenization?
**Answer**: Random tokenization assigns arbitrary UUIDs and breaks joins; deterministic tokenization preserves joinability across tables while masking the underlying value.

---

### 29. What is Azure Data Factory's Managed Virtual Network IR?
**Answer**: A dedicated, isolated cloud VNet provisioned automatically for ADF that connects to data sources via Managed Private Endpoints, eliminating public internet routing.

---

### 30. What happens if you do not enable "Secure Output" on an ADF activity that fetches a password?
**Answer**: The plaintext password is written into ADF run execution history, Azure Monitor logs, and alert diagnostics, readable by anyone with `Reader` role.

---

### 31. What is the difference between System-Assigned and User-Assigned Managed Identity?
**Answer**: System-Assigned is tied to a single resource lifecycle; User-Assigned is a standalone Azure resource that can be shared across multiple services.

---

### 32. What is Databricks Unity Catalog?
**Answer**: A centralized 3-tier governance and access control layer (`catalog.schema.table`) providing fine-grained access control, auditing, and lineage across workspaces.

---

### 33. Why did Unity Catalog eliminate DBFS mount points?
**Answer**: DBFS mounts stored raw storage master keys on cluster filesystems, allowing users with notebook access to extract master credentials and bypass table permissions.

---

### 34. What are Downscoped Tokens in Unity Catalog?
**Answer**: Ephemeral cloud storage credentials (SAS or STS) negotiated by Unity Catalog on behalf of the cluster, restricted to the exact table storage path prefix for 1 hour.

---

### 35. What is the difference between Unity Catalog Shared and Single-User access modes?
**Answer**: Shared mode enforces multi-tenant user isolation and SQL/Python masking; Single-User mode dedicates the cluster to one user for ML/native libraries.

---

### 36. How do Dynamic Row Filters work in Unity Catalog?
**Answer**: A SQL function is bound to the table; Catalyst query optimizer inlines the function as a `WHERE` predicate during query planning based on user group membership.

---

### 37. What is the maximum recommended lifetime for Databricks Personal Access Tokens (PATs)?
**Answer**: 30 days maximum, with user PAT generation disabled globally in favor of Service Principal OAuth M2M tokens for production jobs.

---

### 38. Where are Unity Catalog audit logs stored for forensic analysis?
**Answer**: In the Databricks System Tables under `system.access.audit`.

---

### 39. What is Server-Side Request Forgery (SSRF)?
**Answer**: An attack where an application is coerced into making unintended HTTP requests to internal resources or cloud instance metadata endpoints (`169.254.169.254`).

---

### 40. How do you protect cloud VMs and containers against IMDS SSRF attacks?
**Answer**: Enforce IMDSv2 (requiring token-backed session headers) and configure host firewall/iptables rules to block non-root processes from accessing `169.254.169.254`.

---

### 41. How does Delta Lake Time Travel recover from malicious `DELETE` statements?
**Answer**: Delta writes are append-only; running `RESTORE TABLE t TO VERSION AS OF n` points the transaction log metadata back to the pre-attack state in seconds.

---

### 42. What is S3 Object Lock (Compliance Mode)?
**Answer**: A WORM (Write-Once-Read-Many) storage policy preventing objects from being deleted or overwritten by anyone, including the AWS root account, for a retention period.

---

### 43. What is GDPR Article 33?
**Answer**: The European regulation requiring organizations to report confirmed personal data breaches to supervisory authorities within 72 hours of becoming aware.

---

### 44. What does SLSA stand for in software supply chain security?
**Answer**: Supply-chain Levels for Software Artifacts. A security framework guaranteeing artifact integrity from source to build and packaging.

---

### 45. What is a Replay Attack against authentication tokens, and how is it prevented?
**Answer**: An attacker intercepts and reuses a valid token. Prevented using short token expirations (15 min), one-time nonces, and refresh token family rotation.

---

### 46. What is FIPS 140-2 Level 3?
**Answer**: A U.S. government computer security standard specifying physical tamper-resistance and zeroization mechanisms for cryptographic hardware modules (HSMs).

---

### 47. What is the role of an Identity Provider (IdP)?
**Answer**: A centralized service (Microsoft Entra ID, Okta, Ping) that authenticates users, manages credentials, and issues security tokens (SAML, OIDC).

---

### 48. What is mTLS (Mutual TLS)?
**Answer**: A security protocol where both client and server authenticate each other using X.509 digital certificates before establishing an encrypted TLS connection.

---

### 49. What is Dynamic Data Masking (DDM)?
**Answer**: Obfuscating sensitive column values (e.g. credit cards to `XXXX-XXXX-XXXX-1234`) on-the-fly during query projection without altering the underlying data on disk.

---

### 50. What is Policy-as-Code (PaC), and give an example tool?
**Answer**: Defining and enforcing infrastructure and security compliance rules through declarative code files. Examples: Open Policy Agent (Rego), Checkov, and tfsec.
