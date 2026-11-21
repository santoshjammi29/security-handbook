# 🔒 The Security Handbook

**Enterprise Security Patterns & Practices: Cloud Perimeter Defense, DevSecOps, ETL/ELT Governance, Databricks Unity Catalog, ADF, Python Cryptography, OIDC Workload Identity & Deployments**

[![CI Validation](https://github.com/santoshjammi29/security-handbook/actions/workflows/ci.yml/badge.svg)](https://github.com/santoshjammi29/security-handbook/actions)
[![Zero Trust](https://img.shields.io/badge/Architecture-Zero_Trust_NIST_800--207-darkgreen?style=flat-square&logo=shield)](https://csrc.nist.gov/pubs/sp/800/207/final)
[![Cloud Security](https://img.shields.io/badge/Perimeter-Private_Link_&_Endpoints-blue?style=flat-square&logo=azure)](https://azure.microsoft.com/)
[![Databricks Unity Catalog](https://img.shields.io/badge/Databricks-Unity_Catalog_3--Tier-FF3621?style=flat-square&logo=databricks&logoColor=white)](https://www.databricks.com/)
[![DevSecOps](https://img.shields.io/badge/DevSecOps-OIDC_Workload_Identity-orange?style=flat-square&logo=githubactions&logoColor=white)](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/about-security-hardening-with-openid-connect)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg?style=flat-square)](LICENSE)

*The definitive enterprise handbook and production reference for security engineering in modern data, cloud, and lakehouse platforms.*  
*Covering end-to-end zero-trust architecture, federated workload identities, dynamic data masking, envelope encryption, supply chain security, and emergency incident containment.*

[Curriculum Overview](#-curriculum-overview) • [Zero-Trust Architecture](#-zero-trust-cloud-architecture) • [Whiteboard Blueprints](#-master-whiteboard-architecture-blueprints) • [Modules Directory](#-curriculum-modules-overview) • [Production Runbooks](#-battle-tested-production-runbooks) • [Interview Matrix](#-4-tier-interview-preparation-matrix)

---

## 🎯 Overview & Mission

Modern cloud data platforms are high-value targets for cyber adversaries, ransomware operators, and malicious insiders. A single leaked service principal secret in a pull request, an unmasked social security number in an analytical dashboard, an unindexed S3 bucket, or an unvetted Python package can trigger multimillion-dollar regulatory fines (GDPR, SEC, HIPAA) and catastrophic brand damage.

This repository provides **10 comprehensive production modules, 4 architecture whitepapers, 6 battle-tested incident runbooks, and a 4-tier interview guide** covering:
- **Authentication & Workload Identity:** OpenID Connect (OIDC) federation, eliminating static cloud keys, OAuth 2.0 PKCE, and JWT lifecycle security.
- **Authorization & Entitlement Governance:** Attribute-Based Access Control (ABAC), Dynamic Data Masking (DDM), Row-Level Security (RLS), and Separation of Duties (SoD).
- **Cloud Perimeter & Network Isolation:** Private Link, Private Endpoints, No Public IP (NPIP) clusters, split-horizon DNS, and egress firewall rules.
- **Secrets Management & Cryptography:** FIPS 140-2 Level 3 HSMs, Envelope Encryption (DEK + KEK), AES-256-GCM authenticated encryption, and dual-secret zero-downtime rotation.
- **DevSecOps & Supply Chain Security:** SLSA Level 3, Cosign keyless artifact signing, SBOM generation (Syft/Trivy), and distroless non-root Docker builds.
- **Python Application Security:** SQL injection prevention with parameterized bindings, safe deserialization (anti-pickle), and constant-time HMAC validation.
- **ETL, ELT & Lakehouse Governance:** The Write-Audit-Publish (WAP) pattern, PII detection, deterministic tokenization, and cryptographic shredding for GDPR Article 17.
- **Azure Data Factory & Integration Security:** Managed Identities, Key Vault linked services, Managed VNet Integration Runtime, and Secure Input/Output log masking.
- **Databricks Unity Catalog Governance:** 3-tier namespaces, downscoped temporary storage tokens, cluster access modes, and Personal Access Token (PAT) lockdown.
- **Incident Response & Postmortems:** NIST SP 800-61r2 incident response, forensic querying of Databricks system tables (`system.access.audit`), and 72-hour breach disclosure protocols.

---

## 🏛️ Zero-Trust Cloud Architecture

Under the **NIST SP 800-207 Zero-Trust Architecture**, network location provides zero implicit trust. Every single request is authenticated, authorized, encrypted, and continuously audited:

```mermaid
flowchart TD
    subgraph ZeroTrustFlow["Zero-Trust Continuous Verification Engine"]
        User["Client / Microservice / Pipeline"] --> Context["Context Evaluator (Identity, Device, Network, Time)"]
        Context --> PDP["Policy Decision Point (IdP / Entra ID / Okta / AWS IAM)"]
        PDP -->|Validate Claims & MFA| PEP["Policy Enforcement Point (API Gateway / Envoy / Lakehouse Gateway)"]
        PEP -->|Issue Ephemeral Downscoped Token 15-60 min| Resource["Protected Resource (Databricks, ADF, SQL, S3/ADLS)"]
        PEP -.-> Audit["Tamper-Evident Audit Stream (CloudTrail / Azure Monitor / system.access.audit)"]
    end
```

---

## 🖼️ Master Whiteboard Architecture Blueprints

The repository incorporates publication-grade architectural whiteboard blueprints mirrored under [`assets/whiteboards/`](assets/whiteboards) and cataloged in [`docs/whiteboard_diagrams/README.md`](docs/whiteboard_diagrams/README.md):

<table width="100%">
  <tr>
    <td width="50%" valign="top">
      <a href="docs/whiteboard_diagrams/README.md#open-table-formats-lakehouse">
        <img src="assets/whiteboards/open_table_formats_lakehouse_architecture.webp" alt="Open Table Formats Blueprint: Lakehouse Storage Engines" width="100%" />
      </a>
      <br><strong>Open Table Formats Blueprint: Lakehouse Storage Engines</strong><br>
      <em>Storage isolation, Parquet immutability, and access control boundaries across Delta Lake, Iceberg, and Hudi.</em>
    </td>
    <td width="50%" valign="top">
      <a href="docs/whiteboard_diagrams/README.md#lakehouse-acid-concurrency">
        <img src="assets/whiteboards/lakehouse_acid_concurrency.webp" alt="Lakehouse ACID Concurrency & Write Conflict Resolution Matrix" width="100%" />
      </a>
      <br><strong>Lakehouse ACID Concurrency & Write Conflict Resolution Matrix</strong><br>
      <em>Write serialization, commit conflict detection, and optimistic concurrency control mechanisms.</em>
    </td>
  </tr>
  <tr>
    <td width="50%" valign="top">
      <a href="docs/whiteboard_diagrams/README.md#scd-types-lakehouse">
        <img src="assets/whiteboards/scd_types_lakehouse.webp" alt="Slowly Changing Dimensions (SCD Types 1, 2, 3, 4, 6) in SQL" width="100%" />
      </a>
      <br><strong>Slowly Changing Dimensions (SCD Types 1, 2, 3, 4, 6) in SQL</strong><br>
      <em>Tamper-evident audit trails, historical point-in-time state tracking, and regulatory lineage preservation.</em>
    </td>
    <td width="50%" valign="top">
      <a href="docs/whiteboard_diagrams/README.md#delta-lake-under-the-hood">
        <img src="assets/whiteboards/delta_lake_mechanism.webp" alt="How Delta Lake Works Under the Hood" width="100%" />
      </a>
      <br><strong>How Delta Lake Works Under the Hood</strong><br>
      <em>Append-only transaction logs, Time Travel recovery, snapshot isolation, and ransomware recovery.</em>
    </td>
  </tr>
</table>

> 🔍 **Explore all 52 blueprints with 5-persona breakdowns, failure modes, and engine internals in [docs/whiteboard_diagrams/README.md](docs/whiteboard_diagrams/README.md)**.

---

## 📚 Curriculum Modules Overview

The curriculum is organized into **10 comprehensive production modules**:

| # | Module & Domain | Key Focus & Production Implementations | Status |
|:---:|:---|:---|:---:|
| **01** | **[Authentication, OIDC & Zero-Trust](modules/01_authn_oidc_and_zero_trust/README.md)** | OIDC workload identity federation, eliminating static cloud keys, OAuth 2.0 PKCE, and JWT lifecycle security. | ✅ Documented |
| **02** | **[Authorization, RBAC, ABAC & Governance](modules/02_authz_rbac_abac_and_governance/README.md)** | RBAC vs ABAC vs ReBAC, Separation of Duties (SoD), Dynamic Data Masking (DDM), and Row-Level Security (RLS). | ✅ Documented |
| **03** | **[Cloud Networking & Perimeter Defense](modules/03_cloud_networking_and_perimeter_defense/README.md)** | Private Link, Private Endpoints, Databricks No Public IP (NPIP), split-horizon DNS, and egress firewall lockdown. | ✅ Documented |
| **04** | **[Secrets Management & Encryption](modules/04_secrets_management_and_encryption/README.md)** | Envelope Encryption (DEK + KEK), Customer-Managed Keys (CMK / BYOK), HSM FIPS 140-2 Level 3, and dual-secret rotation. | ✅ Documented |
| **05** | **[DevSecOps, CI/CD & Supply Chain](modules/05_devsecops_cicd_and_supply_chain/README.md)** | SLSA Level 3, Cosign keyless artifact signing, SBOM generation (Syft/Trivy), and distroless non-root Docker builds. | ✅ Documented |
| **06** | **[Python Secure Coding & Cryptography](modules/06_python_secure_coding_and_cryptography/README.md)** | SQL injection defense with parameterized bindings, safe deserialization (anti-pickle), and AES-256-GCM AEAD. | ✅ Documented |
| **07** | **[ETL, ELT & Data Pipeline Security](modules/07_etl_elt_and_pipeline_security/README.md)** | Write-Audit-Publish (WAP) pattern, PII detection, deterministic tokenization, and cryptographic shredding for GDPR. | ✅ Documented |
| **08** | **[ADF & Managed Integration Security](modules/08_adf_and_azure_pipeline_security/README.md)** | Managed Identities, Key Vault linked services, Managed VNet Integration Runtime, and Secure Input/Output log masking. | ✅ Documented |
| **09** | **[Databricks & Lakehouse Security Mastery](modules/09_databricks_unity_catalog_security/README.md)** | Unity Catalog 3-tier namespaces, downscoped temporary storage tokens, cluster access modes, and PAT lockdown. | ✅ Documented |
| **10** | **[Incident Triage, Auditing & Compliance](modules/10_incident_triage_auditing_and_compliance/README.md)** | NIST SP 800-61r2 incident response, forensic querying of `system.access.audit`, and 72-hour breach disclosure protocols. | ✅ Documented |

---

## 🏛️ Architectural Deep Dives

In-depth technical architecture whitepapers:

- **[01. Zero-Trust Cloud & Data Perimeter Defense](architectures/01_zero_trust_cloud_and_data_perimeter_architecture.md)**: Network boundary isolation, Private Link, split-horizon DNS, and egress firewall filtering.
- **[02. Unity Catalog & Lakehouse Governance Internals](architectures/02_unity_catalog_and_lakehouse_governance_internals.md)**: 3-tier namespaces, decoupled storage credentials, downscoped token negotiation, and Catalyst query optimizer masking.
- **[03. Cryptography, Envelope Encryption & Key Lifecycle](architectures/03_cryptography_envelope_encryption_and_key_management.md)**: AES-256-GCM, Envelope Encryption (DEK + KEK), Customer-Managed Keys (CMK / BYOK), and cryptographic shredding.
- **[04. DevSecOps Pipelines & Workload Identity Federation](architectures/04_devsecops_pipeline_and_workload_identity_federation.md)**: OIDC token exchange, eliminating static secrets, SLSA Level 3, Cosign keyless signing, and policy-as-code.

---

## 🚨 Battle-Tested Production Runbooks

The repository includes enterprise-grade operational runbooks for emergency security response:

| Runbook | Title | Critical Failure Mode Addressed |
| :--- | :--- | :--- |
| **[Runbook 01](runbooks/01_compromised_credential_and_secret_revocation_triage.md)** | **Compromised Credential & Secret Revocation Triage** | Leaked service principal client secrets, exposed IAM keys, active session termination, and CloudTrail forensic auditing. |
| **[Runbook 02](runbooks/02_storage_account_data_exfiltration_and_public_exposure.md)** | **Storage Account Exposure & Data Exfiltration Triage** | Publicly accessible S3 buckets / ADLS Gen2 accounts, immediate network cutoff, and egress blast radius analysis. |
| **[Runbook 03](runbooks/03_databricks_unauthorized_data_access_and_pat_breach.md)** | **Databricks Unauthorized Access & PAT Breach Triage** | Compromised Personal Access Tokens, rogue cluster termination, and forensic querying of `system.access.audit`. |
| **[Runbook 04](runbooks/04_data_pipeline_ssrf_and_sql_injection_remediation.md)** | **Data Pipeline SSRF & SQL Injection Remediation** | Server-Side Request Forgery against IMDS (`169.254.169.254`), domain allowlisting, and parameterized query refactoring. |
| **[Runbook 05](runbooks/05_ransomware_and_tampered_lakehouse_disaster_recovery.md)** | **Ransomware & Tampered Lakehouse Disaster Recovery** | Malicious table deletion, Delta Lake Time Travel restoration (`RESTORE TABLE`), and immutable WORM / Object Lock recovery. |
| **[Runbook 06](runbooks/06_zero_downtime_certificate_and_secret_rotation.md)** | **Zero-Downtime Certificate & Secret Rotation** | Dual-secret rotation protocol for database credentials, mTLS certificate renewal, and zero-downtime rollover. |

---

## 🎓 4-Tier Interview Preparation Matrix

Prepare for technical security screens from Junior Data Engineer to Principal Security Architect:

1. **[Tier 1: Junior Security & Cloud Foundations](interview/01_junior_security_and_cloud_foundations.md)**: AuthN vs AuthZ, symmetric vs asymmetric encryption, SQL injection, basic IAM, and least privilege.
2. **[Tier 2: Mid-Level Data Security Engineer](interview/02_mid_level_data_security_engineer.md)**: OIDC workload identity federation, Private Endpoints, dynamic data masking, Key Vault in ADF, and anti-pickle deserialization.
3. **[Tier 3: Senior Cloud & Lakehouse Security Architect](interview/03_senior_cloud_and_lakehouse_security_architect.md)**: Unity Catalog downscoped tokens, envelope encryption mechanics, crypto-shredding, Cosign keyless signing, and WAP gating.
4. **[Tier 4: Principal Security Architect & CISO Defense](interview/04_principal_security_architect_and_ciso_defense.md)**: STRIDE threat modeling, GDPR Article 33 72-hour breach notification, cross-tenant data sharing, and FAIR risk quantification.
5. **[50-Card Rapid-Fire Technical Security Flashcard Deck](interview/05_flashcards_deck_50.md)**: Rapid-fire revision covering identity, networking, cryptography, pipelines, lakehouse governance, and compliance.

---

## 🛠️ Repository Structure

```
security-handbook/
├── README.md                                  # Master repository documentation
├── LICENSE                                    # MIT Open Source License
├── pyproject.toml                             # Python package & pytest configuration
├── requirements.txt                           # Optional development dependencies
├── TAXONOMY.md                                # Modern Data Engineering concepts taxonomy
├── .github/
│   └── workflows/
│       └── ci.yml                             # Automated GitHub Actions test runner
├── tests/
│   ├── test_runbooks_and_architectures.py     # Integrity assertion test suite
│   └── conftest.py                            # Pytest configuration
├── scripts/
│   └── run_tests.py                           # Lightweight test suite runner
├── modules/                                   # 10 Core Production Security Modules
│   ├── 01_authn_oidc_and_zero_trust/          # OIDC, Zero-Trust, OAuth 2.0 PKCE, JWTs
│   ├── 02_authz_rbac_abac_and_governance/     # RBAC, ABAC, SoD, Dynamic Data Masking
│   ├── 03_cloud_networking_and_perimeter_defense/ # Private Link, NPIP, Split-Horizon DNS
│   ├── 04_secrets_management_and_encryption/  # Envelope Encryption (DEK/KEK), CMK, HSM
│   ├── 05_devsecops_cicd_and_supply_chain/    # SLSA Level 3, Cosign, Distroless Docker
│   ├── 06_python_secure_coding_and_cryptography/ # SQL injection, Anti-Pickle, AES-256-GCM
│   ├── 07_etl_elt_and_pipeline_security/      # WAP pattern, PII detection, Crypto-shredding
│   ├── 08_adf_and_azure_pipeline_security/    # Managed Identity, Managed VNet IR, Secure Output
│   ├── 09_databricks_unity_catalog_security/  # 3-tier namespaces, Downscoped tokens, PAT lockdown
│   └── 10_incident_triage_auditing_and_compliance/ # NIST response, system.access.audit, GDPR
├── runbooks/                                  # 6 Battle-tested production incident playbooks
├── interview/                                 # 4-Tier interview question bank & flashcards
└── architectures/                             # 4 Deep-dive architectural whitepapers
```

---

## 🌐 Ecosystem Interlinking & Sister Repositories

The Modern Data Engineering Lakehouse Ecosystem is an integrated suite of 14 production-grade, specialized repositories anchored by the **[Modern Data Engineer Platform](https://github.com/santoshjammi29/moderndataengineer)**. Use this cross-reference matrix to navigate directly between complementary implementations across our repositories:

| Repository | Focus & Domain | Tightly Coupled Cross-Reference & Integration Points |
| :--- | :--- | :--- |
| **[The Databricks Handbook](https://github.com/santoshjammi29/databricks-handbook)** | 105 Databricks Production Recipes & Unity Catalog | • **Unity Catalog Governance**: Translating security policies into production Delta tables and serverless DBSQL warehouses.<br>• **Storage Credentials**: Configuring storage credentials and external locations for secure cloud data lakes. |
| **[The PySpark Handbook](https://github.com/santoshjammi29/pyspark-handbook)** | Apache Spark 3.5+, Catalyst & Arrow UDFs | • **Secure UDF Execution**: Sandboxed Python UDFs and Arrow execution without arbitrary code vulnerabilities.<br>• **Local Spill Encryption**: Enabling encrypted scratch disks and RPC communication across Spark clusters. |
| **[The SQL Handbook](https://github.com/santoshjammi29/sql-handbook)** | 105 SQL Production Exercises & Query Plans | • **Dynamic Data Masking**: ANSI SQL mask functions and row-level security predicates.<br>• **Audit Ledgers**: Append-only tamper-evident double-entry ledgers in relational databases. |
| **[The dbt Handbook](https://github.com/santoshjammi29/dbt-handbook)** | Analytics Engineering, dbt Mesh & Contracts | • **Data Contracts**: Enforcing schema constraints and column privacy contracts in CI/CD before publishing marts.<br>• **Secure Views**: Defining authorized and masked views in dbt models. |
| **[DataOps & CI/CD Handbook](https://github.com/santoshjammi29/dataops-cicd-handbook)** | GitOps, Terraform & Platform Engineering | • **OIDC Infrastructure**: Provisioning cloud federated credentials and managing secrets in Terraform/Bicep.<br>• **Automated SAST/DAST**: Running pre-commit hooks and static security scans in automated CI/CD pipelines. |
| **[Streaming Handbook](https://github.com/santoshjammi29/streaming-handbook)** | Kafka KRaft, Apache Flink & Event Architecture | • **mTLS & SASL Encryption**: Securing distributed Kafka brokers with client certificate validation.<br>• **Encrypted Payloads**: End-to-end event payload encryption in stateful stream processing. |
| **[System Design Handbook](https://github.com/santoshjammi29/data-system-design-handbook)** | Distributed Storage, Table Formats & BCDR | • **WORM Storage**: Immutable object locking against ransomware across Delta Lake and Apache Iceberg.<br>• **BCDR & Failover**: Cross-region disaster recovery and encrypted replication. |
| **[Azure Cheat Sheet](https://github.com/santoshjammi29/azure-cheat-sheet)** | Fabric OneLake, Synapse & Azure Infrastructure | • **Managed VNet Networking**: Private Endpoints and NSG configurations for ADF and Microsoft Fabric.<br>• **Azure Key Vault**: Hardware Security Module (HSM) setup and key rotation policies. |
| **[Azure Databricks Blueprints](https://github.com/santoshjammi29/azure-databricks-project-blueprints)** | 3 Enterprise Reference Architectures | • **Real-time Fraud Security**: Tokenized customer attributes and encrypted event streams in AML pipelines.<br>• **Production IAM**: Role-based access control and Unity Catalog dynamic masking in retail lakehouses. |
| **[Banking Risk Lakehouse](https://github.com/santoshjammi29/enterprise-banking-risk-alm-lakehouse)** | Regulated Financial Lakehouse (BCBS 239 & IFRS 9) | • **Regulatory Compliance**: Meeting European Central Bank (ECB) and BCBS 239 auditability standards.<br>• **Customer PII Isolation**: Segregated financial risk models with strict role-based data isolation. |
| **[Spark Experiments](https://github.com/santoshjammi29/spark-experiments)** | Empirical Skew, Salting & Spill Laboratory | • **Execution Memory Security**: Benchmarking encrypted memory spills and data serialization overhead. |
| **[Interview Handbook](https://github.com/santoshjammi29/data-engineering-interview-handbook)** | Architectural Defense Playbooks & Concepts | • **Pillar 14 Defense**: Security, identity governance, zero-trust perimeters, and regulatory compliance scenarios. |
| **[Modern Data Engineer Hub](https://github.com/santoshjammi29/moderndataengineer)** | Central Edge Presentation Platform | • **Interactive Whiteboard Viewer**: Pan, zoom, and inspect all 52 blueprints at [moderndataengineer.uk/whiteboards](https://moderndataengineer.uk/whiteboards).<br>• **Concept Encyclopedia**: 921 verified topics across security, governance, and distributed systems. |

---

## 🤝 Contributing & License

Contributions, new security runbooks, and cloud architecture patterns are welcome!  
Distributed under the **MIT License**. See [`LICENSE`](LICENSE) for details.  
Created and maintained with ❤️ by [Santosh Jammi](https://github.com/santoshjammi29).
