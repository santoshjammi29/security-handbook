# 🧠 Enterprise Cloud Perimeter, Lakehouse Security & Governance — Curriculum Topics Directory

> **Curriculum Scope**: **37 Production Topics** verified against real-world lakehouse internals, query engines, and distributed architectures.
> **Live Web Hub**: Explore interactive search and full visual graph at [moderndataengineer.uk/concepts](https://moderndataengineer.uk/concepts).

---

## 📖 Overview & Learning Path

The security curriculum covers zero-trust network boundaries, keyless OIDC workload identity federation, Unity Catalog attribute-based access control, envelope encryption, and regulatory auditing.

### 🎯 Difficulty Progression
- `🟢 Easy`: Foundational primitives, syntax, and operational core concepts.
- `🟡 Intermediate`: Production configuration, optimization patterns, and pipeline resiliency.
- `🔴 Advanced`: Engine internals, memory management, shuffle mechanics, and fault recovery.
- `🟣 Architect`: Cross-region distributed systems, enterprise governance, high-availability, and FinOps.

---

## 📑 Topics Directory at a Glance

| Module / Pillar | Topics | Key Primitives Covered | Reference Links |
|:---|:---:|:---|:---|
| 🔐 **[Zero-Trust Identity, Authentication & Secrets Management](#zero-trust-identity-authentication--secrets-management)** | **7** | Secrets Management, Airflow Connection Secrets Backend, Databricks Secret Scopes (Azure Key Vault), and 4 more | [Jump to Section](#zero-trust-identity-authentication--secrets-management) |
| 🌐 **[Cloud Network Isolation & Perimeter Defense](#cloud-network-isolation--perimeter-defense)** | **3** | Managed Private Endpoints, Managed Virtual Network (Managed VNet), ADF Managed Private Endpoint Provisioning & Timeout Bypass | [Jump to Section](#cloud-network-isolation--perimeter-defense) |
| 🏛️ **[Unity Catalog, Fine-Grained Access Control & Dynamic Data Masking](#unity-catalog-fine-grained-access-control--dynamic-data-masking)** | **11** | Unity Catalog, Attribute-Based Dynamic Column Masking (ABAC), Dynamic Views for Security, and 8 more | [Jump to Section](#unity-catalog-fine-grained-access-control--dynamic-data-masking) |
| 📜 **[Data Lineage, Auditing & Regulatory Governance](#data-lineage-auditing--regulatory-governance)** | **16** | Data Quality Evidence with Lineage, Data Lineage, OpenLineage Operational Telemetry Standard, and 13 more | [Jump to Section](#data-lineage-auditing--regulatory-governance) |

---

## 📚 Comprehensive Topic Specifications

### 🔐 Zero-Trust Identity, Authentication & Secrets Management
*OIDC federation, eliminating static credentials, secret vault backends, and rate limiting.*

**Total Topics**: 7

| # | Topic / Concept | Difficulty | Architectural Definition & Core Principle |
|:---:|---|:---:|---|
| 1 | **Secrets Management** | `🟢 Easy` | The secure credential management framework in Databricks (Secret Scopes backed by Azure Key Vault or encrypted storage) used to securely inject tokens, API keys, and connection strings at runtime. |
| 2 | **Airflow Connection Secrets Backend** | `🟡 Intermediate` | Storing Airflow database and API connections in cloud secret managers (HashiCorp Vault, AWS Secrets Manager). |
| 3 | **Databricks Secret Scopes (Azure Key Vault)** | `🟡 Intermediate` | Secure credential storage backed by cloud key vaults, accessed in notebooks via `dbutils.secrets.get()`. |
| 4 | **Databricks Secrets** | `🟡 Intermediate` | A Databricks-managed key-value secret store that securely injects sensitive values (API keys, connection strings, passwords) into notebooks and job runs without exposing them as plaintext. |
| 5 | **Token Bucket Algorithm** | `🟡 Intermediate` | A rate limiting algorithm that models a bucket holding a maximum of N tokens, where tokens are added at a fixed rate and each request consumes one token, allowing controlled burst traffic. |
| 6 | **Token Bucket Rate Limiting Algorithm** | `🟡 Intermediate` | An algorithm that meters API requests by accumulating tokens at a constant rate up to a bucket capacity. |
| 7 | **Custom Secret Backends** | `🟣 Architect` | Custom Secret Backends enable Airflow to retrieve Connections and Variables directly from enterprise secret management systems at runtime without storing secrets in the metadata database. |

### 🌐 Cloud Network Isolation & Perimeter Defense
*Private Link, Managed VNets, Private Endpoints, and egress filtering.*

**Total Topics**: 3

| # | Topic / Concept | Difficulty | Architectural Definition & Core Principle |
|:---:|---|:---:|---|
| 1 | **Managed Private Endpoints** | `🟡 Intermediate` | Private IP addresses provisioned inside the ADF Managed Virtual Network to secure connections to Azure resources. |
| 2 | **Managed Virtual Network (Managed VNet)** | `🟡 Intermediate` | A secure networking feature that runs ADF Integration Runtimes inside a dedicated private network boundary, connecting to services via Private Endpoints. |
| 3 | **ADF Managed Private Endpoint Provisioning & Timeout Bypass** | `🟣 Architect` | Enterprise network security architecture managing lifecycle, approval protocols, and timeout bypass for private endpoints inside ADF Managed Virtual Networks. |

### 🏛️ Unity Catalog, Fine-Grained Access Control & Dynamic Data Masking
*3-tier namespaces, ABAC dynamic row filters, column masks, and volume isolation.*

**Total Topics**: 11

| # | Topic / Concept | Difficulty | Architectural Definition & Core Principle |
|:---:|---|:---:|---|
| 1 | **Unity Catalog** | `🟢 Easy` | Unity Catalog is Databricks' centralized, multi-cloud governance solution providing unified access control, data auditing, lineage tracking, and discovery across all workspace assets. |
| 2 | **Attribute-Based Dynamic Column Masking (ABAC)** | `🟡 Intermediate` | A runtime security policy engine that evaluates user attributes (role, department, clearance) to conditionally mask, hash, or redact sensitive column values without duplicating tables. |
| 3 | **Dynamic Views for Security** | `🟡 Intermediate` | Security-hardened SQL views that evaluate user context functions (`CURRENT_USER()`, `IS_MEMBER()`) at runtime to provide row-level filtering and column-level masking without duplicating data. |
| 4 | **Row-Level Security (RLS)** | `🟡 Intermediate` | A security feature that restricts database row access for specific users based on active role filters. |
| 5 | **Unity Catalog Volumes (Managed vs. External)** | `🟡 Intermediate` | A Unity Catalog securable object that extends centralized catalog governance, audit trails, and SQL permissions to unstructured and semi-structured files. |
| 6 | **Unity Catalog 3-Level Namespace & Attribute-Based Access Control (ABAC)** | `🔴 Advanced` | Unified lakehouse governance with catalog.schema.table namespace and tag-based dynamic masking policies. |
| 7 | **Unity Catalog Fine-Grained Access Control** | `🔴 Advanced` | Fine-Grained Access Control in Unity Catalog enforces cell-level security via Row Filters and Column Masks defined with standard SQL functions. |
| 8 | **Object-Level Security (OLS) & Dynamic RLS at Scale** | `🟣 Architect` | Fine-grained security architecture enforcing dynamic row filters and complete column/table metadata obfuscation across enterprise user bases. |
| 9 | **Unity Catalog Dynamic Column Masking & Row Filters** | `🟣 Architect` | Declarative SQL-based fine-grained access control policies enforced transparently at the Unity Catalog metadata layer. |
| 10 | **Unity Catalog Dynamic Row Filters & Column Masks** | `🟣 Architect` | Fine-grained SQL access control mechanisms in Unity Catalog that dynamically redact columns or filter table rows based on the querying user's group membership. |
| 11 | **Unity Catalog Enterprise Governance** | `🟣 Architect` | The enterprise governance operating model for Unity Catalog encompassing account-level identity management, catalog segregation, classification tagging, and regulatory audit compliance. |

### 📜 Data Lineage, Auditing & Regulatory Governance
*System tables audit logs, OpenLineage, Purview metadata enrichment, and bi-temporal auditing.*

**Total Topics**: 16

| # | Topic / Concept | Difficulty | Architectural Definition & Core Principle |
|:---:|---|:---:|---|
| 1 | **Data Quality Evidence with Lineage** | `🟢 Easy` | A record connecting a quality check result to the data version, pipeline run, rules, and upstream inputs that produced it. |
| 2 | **Data Lineage** | `🟡 Intermediate` | The automated, real-time metadata tracking engine in Unity Catalog that captures column-level and table-level data flow lineage across queries, notebooks, workflows, and dashboards. |
| 3 | **OpenLineage Operational Telemetry Standard** | `🟡 Intermediate` | An open standard for automated collection of end-to-end dataset lineage, pipeline job execution metadata, and runtime assertions across data platforms. |
| 4 | **OpenLineage Standard Operational Run Facets** | `🟡 Intermediate` | An open standard specification that captures job execution events, input/output dataset schemas, row counts, and error states emitted during pipeline runs. |
| 5 | **Unity Catalog Lineage Tracking** | `🟡 Intermediate` | Automated capture of column-level and table-level data flow across queries, jobs, and dashboards. |
| 6 | **Audit Logging** | `🔴 Advanced` | Audit Logging captures, records, and exports detailed security audit trails of all user and system activities occurring across Databricks workspaces and Unity Catalog. |
| 7 | **Delta Lake VACUUM & Retention Governance** | `🔴 Advanced` | The maintenance command in Delta Lake that removes uncommitted and historical Parquet data files that are older than the specified retention threshold. |
| 8 | **Purview Metadata Enrichment & Non-Destructive Cataloging** | `🔴 Advanced` | The architectural principle ensuring that metadata enrichment, classifications, and business definitions applied in Microsoft Purview do not modify source Fabric storage objects. |
| 9 | **Unity Catalog System Tables** | `🔴 Advanced` | Unity Catalog System Tables are analytical Delta tables hosted in the system catalog that expose operational metadata, cost telemetry, query history, and data lineage. |
| 10 | **Column-Level Encryption & Tokenization (FPE)** | `🟣 Architect` | Cryptographic data protection mechanisms that encrypt or tokenize sensitive PII attributes while preserving original data formats and enabling analytical queries. |
| 11 | **Data Mesh Federated Governance** | `🟣 Architect` | A decentralized data governance framework balancing domain autonomy with standardized enterprise interoperability policies. |
| 12 | **Data Mesh Federated Governance & Data Product Contracts (ODCS)** | `🟣 Architect` | Decentralized domain ownership with centralized automated compliance and declarative schema contracts. |
| 13 | **FinOps Cost Attribution Matrix for Shared Catalogs** | `🟣 Architect` | The allocation framework that attributes compute, storage, and egress spend back to specific business units and data products across shared enterprise lakehouse infrastructure. |
| 14 | **Governance with Unity Catalog + Purview** | `🟣 Architect` | The enterprise governance integration between Databricks Unity Catalog and Microsoft Purview, unifying lakehouse metadata with company-wide data governance. |
| 15 | **Temporal Tables & Point-In-Time Bi-Temporal Auditing** | `🟣 Architect` | System-versioned relational table architecture automatically recording the full history of data modifications with UTC period start/end timestamps. |
| 16 | **Unity Catalog System Tables for Lakehouse FinOps** | `🟣 Architect` | Centrally governed, read-only analytical schemas providing real-time telemetry on compute usage, billing, audit access, and lineage across an entire account. |

---

## 📐 Related Whiteboard Blueprints
For deep visual architecture walkthroughs of these topics, consult:
- 🖼️ [Master Whiteboard Blueprints](../docs/whiteboard_diagrams/handbook_whiteboard_blueprints.md)
- 📊 [Spark Catalyst & Architecture Cheatsheet](../assets/whiteboards/spark_architecture_catalyst_cheatsheet.jpg)
- 🔀 [Join Strategies Decision Matrix](../assets/whiteboards/spark_join_strategies_decision_matrix.jpg)
- ⚡ [Skew & Spill Memory Triage](../assets/whiteboards/spark_performance_skew_spill_triage.jpg)

---

## 🌐 Ecosystem Interlinking
All curriculum topics are synchronized across the 14-repository Modern Data Engineering ecosystem:
- [Modern Data Engineer Hub](https://github.com/santoshjammi29/moderndataengineer) — Central edge web platform and 921-topic encyclopedia
- [PySpark Handbook](https://github.com/santoshjammi29/pyspark-handbook) — Distributed compute & PySpark internals
- [Databricks Handbook](https://github.com/santoshjammi29/databricks-handbook) — Unity Catalog, Delta Lake 3.x, and Lakeflow
- [SQL Handbook](https://github.com/santoshjammi29/sql-handbook) — Query engine optimization & relational internals
- [Streaming Handbook](https://github.com/santoshjammi29/streaming-handbook) — Event streaming, Kafka, and Flink
- [Security Handbook](https://github.com/santoshjammi29/security-handbook) — Zero-trust, IAM, and lakehouse governance
- [Data System Design Handbook](https://github.com/santoshjammi29/data-system-design-handbook) — Distributed architecture and scalability
- [DataOps & CI/CD Handbook](https://github.com/santoshjammi29/dataops-cicd-handbook) — Airflow, orchestration, and infrastructure
- [dbt Handbook](https://github.com/santoshjammi29/dbt-handbook) — Analytics engineering, contracts, and modeling
- [Azure Cheat Sheet](https://github.com/santoshjammi29/azure-cheat-sheet) — Fabric, OneLake, and ADF enterprise blueprints
- [Data Engineering Interview Handbook](https://github.com/santoshjammi29/data-engineering-interview-handbook) — 20 Knowledge Pillars & architectural defense
