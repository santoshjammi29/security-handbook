# 🎓 Interview Guide 01: Junior Security & Cloud Foundations

## Core Competency Checklist:
- [ ] Authentication (AuthN) vs Authorization (AuthZ) distinctions
- [ ] Symmetric vs Asymmetric encryption fundamentals
- [ ] SQL injection mechanisms and mitigation via parameterization
- [ ] Basic Cloud IAM (Users, Roles, Policies, Service Accounts)
- [ ] Environment variable vs hardcoded credential anti-patterns

---

### Question 1: What is the fundamental difference between Authentication (AuthN) and Authorization (AuthZ)?
**Answer**:
- **Authentication (AuthN)** verifies **who you are** (identity verification). Examples: logging in with a username/password + MFA challenge, presenting a valid TLS client certificate, or verifying a signed JWT token.
- **Authorization (AuthZ)** determines **what you are allowed to do** (permissions and access rights). Examples: verifying whether an authenticated user has `SELECT` permissions on a specific Delta table or write permissions to an S3 bucket.
- *Analogy*: Authentication is showing your passport at border control; Authorization is the visa stamp determining which countries you are allowed to enter and for how long.

---

### Question 2: Why is string concatenation or f-string interpolation dangerous when constructing SQL queries in Python, and how does parameterization solve it?
**Answer**:
- Constructing queries via `f"SELECT * FROM users WHERE email = '{email}'"` allows untrusted user input to break out of string literals and alter the SQL abstract syntax tree (AST). For example, inputting `' OR '1'='1` transforms the logic, leaking unauthorized data or allowing chained commands (`'; DROP TABLE users; --`).
- **Parameterization** (or prepared statements) sends the SQL query structure and the user parameter values in **separate protocol packets** to the database engine. The database compiles the execution plan first and treats the parameter values strictly as literal scalar data, rendering SQL syntax injection mathematically impossible.

---

### Question 3: What is the difference between Symmetric and Asymmetric encryption, and when is each used in data engineering?
**Answer**:
- **Symmetric Encryption** uses the **same secret key** for both encryption and decryption (e.g. AES-256-GCM). It is computationally fast, hardware-accelerated by CPU instruction sets (AES-NI), and ideal for bulk data encryption (Parquet files, disk storage, database tables).
- **Asymmetric Encryption** uses a **mathematically linked key pair**: a Public Key (used to encrypt or verify signatures) and a Private Key (used to decrypt or generate signatures) (e.g. RSA-4096, ECDSA). It is computationally expensive and primarily used for identity assertions (JWT signing), key exchange protocols (TLS handshakes), and digital signatures.

---

### Question 4: Why should developers never commit API keys or connection strings to Git, even in private repositories?
**Answer**:
1. **Immutable Git History**: Committing a secret persists it in the `.git` packfiles forever. Even if deleted in a subsequent commit, it remains accessible via `git checkout` or commit SHA inspection.
2. **Broad Developer Access**: Private repositories are accessible to every developer, contractor, and CI/CD runner in the organization, violating the principle of least privilege.
3. **Third-Party Integrations**: Repositories are cloned to developer laptops, mirrored across fork repositories, and indexed by third-party developer tools.
- *Best Practice*: Use environment variables, secret managers (Azure Key Vault, AWS Secrets Manager), and enforce pre-commit linters (`gitleaks protect`).

---

### Question 5: What is the Principle of Least Privilege (PoLP)?
**Answer**:
- The Principle of Least Privilege dictates that any identity (user, service principal, background job) should be granted **only the minimum set of permissions necessary** to complete its intended task, and for the minimum duration required.
- In data engineering, this means an ingestion pipeline should only be granted `Write` permissions to its specific landing directory, rather than `Contributor` or `Storage Account Owner` permissions across the entire cloud subscription.
