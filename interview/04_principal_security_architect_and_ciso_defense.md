# 🎓 Interview Guide 04: Principal Security Architect & CISO Defense

## Core Competency Checklist:
- [ ] Enterprise Threat Modeling using STRIDE & attack trees
- [ ] Multi-Cloud Identity Governance & Cross-Tenant Trust Boundaries
- [ ] Compliance Architecture: SOC 2 Type II, ISO 27001, HIPAA, GDPR Articles 32/33
- [ ] Incident Response Governance & 72-Hour Breach Disclosure Protocols
- [ ] Defense-in-Depth Economics & Risk Quantification (FAIR model)

---

### Question 1: How do you apply the STRIDE threat modeling framework to an enterprise lakehouse ingestion pipeline?
**Answer**:
When threat-modeling an end-to-end ingestion pipeline (e.g. Kafka $\rightarrow$ Spark Structured Streaming $\rightarrow$ Delta Lake $\rightarrow$ Power BI):
1. **Spoofing Identity**: An unauthorized client impersonates an IoT gateway to inject forged telemetry. *Mitigation*: Mutual TLS (mTLS) with client certificate verification and SASL/SCRAM-SHA-512 authentication.
2. **Tampering with Data**: An attacker alters in-transit records to falsify financial metrics. *Mitigation*: Enforce TLS 1.3 in transit and cryptographic HMAC checksum verification across microbatches.
3. **Repudiation**: A rogue employee exports confidential user records and denies doing so. *Mitigation*: Append-only, tamper-evident audit streaming to `system.access.audit` and CloudTrail sent to an immutable, write-locked S3 bucket.
4. **Information Disclosure**: Plaintext PII is exposed in lakehouse storage files. *Mitigation*: Envelope encryption with Customer-Managed Keys (CMK), Unity Catalog dynamic data masking, and Private Link network perimeter isolation.
5. **Denial of Service (DoS)**: Malicious actors flood the ingestion API with malformed packets, exhausting cluster worker memory. *Mitigation*: Token bucket rate-limiting at the API Gateway, autoscaling admission controllers, and message payload size limits.
6. **Elevation of Privilege**: A data analyst elevates permissions to `Account Admin` via legacy cluster credential passthrough. *Mitigation*: Unity Catalog Shared Cluster access mode, deprecation of DBFS mounts, and Azure PIM / AWS IAM JIT approval workflows.

---

### Question 2: In the event of a confirmed lakehouse PII exfiltration, walk through the executive incident response protocol for GDPR Article 33 compliance.
**Answer**:
1. **Hour 0–4 (Containment & Forensic Freezing)**:
   - Sever compromised credentials immediately (revoke OAuth tokens, rotate service principal secrets).
   - Snapshot active cluster memory buffers and isolate cloud storage logs (`system.access.audit`, CloudTrail, VPC Flow Logs).
   - Convene the Incident Response Team (Lead Architect, SecOps, Legal Counsel, Chief Information Security Officer).
2. **Hour 4–24 (Forensic Blast Radius Determination)**:
   - Query system access logs to determine the exact timestamp range and identity of exfiltrated data.
   - Categorize whether exfiltrated records contained special categories of personal data (GDPR Article 9: biometric, health, financial).
   - Evaluate whether encryption was active: if records were encrypted under AES-256-GCM and the master KEK was **not** compromised, the risk to data subjects is categorized as low.
3. **Hour 24–72 (Regulatory Notification)**:
   - If unencrypted PII was exfiltrated, GDPR Article 33 mandates notifying the competent Supervisory Authority (e.g. DPC, ICO, CNIL) **within 72 hours** of becoming aware of the breach.
   - The notification must detail: nature of the breach, approximate number of data subjects affected, contact details of the Data Protection Officer (DPO), likely consequences, and remediation measures implemented.

---

### Question 3: How do you design an automated, zero-trust cross-account and cross-tenant data-sharing architecture between partner enterprises?
**Answer**:
- **The Anti-Pattern**: Sharing raw storage account SAS tokens or creating permanent IAM access keys for external partner organizations.
- **The Modern Zero-Trust Solution (Delta Sharing / Snowflake Secure Data Sharing)**:
  1. **Open Governance Protocol**: Utilize open protocols like **Delta Sharing** where data is shared directly from existing Delta tables without copying physical data.
  2. **Decoupled Verification**:
     - The recipient application authenticates using an ephemeral bearer token issued by the recipient's identity provider.
     - The sharing server verifies the recipient's identity and returns pre-signed, downscoped URLs valid for 15 minutes for the exact Parquet files requested.
  3. **No Direct Storage Access**: The recipient never receives credentials to the provider's underlying cloud storage account, and provider data is never exposed over public endpoints.

---

### Question 4: How do you mathematically quantify and justify the ROI of moving from service-managed keys to an HSM-backed Customer-Managed Key (CMK) architecture?
**Answer**:
- Using the **FAIR (Factor Analysis of Information Risk)** framework:
  $$\text{Annual Loss Expectancy (ALE)} = \text{Single Loss Expectancy (SLE)} \times \text{Annual Rate of Occurrence (ARO)}$$
- **Risk Scenario**: A cloud provider subpoena, insider threat, or co-tenant virtualization breakout exposing sensitive intellectual property or regulated banking data.
- **Single Loss Expectancy (SLE)**: Fines (up to 4% of global turnover under GDPR), regulatory penalties (SEC/OCC enforcement), class-action litigation, and customer churn ($>\$20\text{M}$ for an enterprise).
- **Control Benefit**: With HSM-backed Customer-Managed Keys (FIPS 140-2 Level 3), data is encrypted under an asymmetric root key that the cloud provider cannot decrypt without explicit HSM authorization. If a rogue entity accesses physical storage media, the data is mathematically unreadable.
- *Justification*: The annualized cost of dedicated Cloud HSM / KMS infrastructure ($\approx \$50,000/\text{year}$) is an order of magnitude lower than the reduction in probabilistic exposure ($ALE \Delta > \$500,000/\text{year}$).
