# ⚡ Production Runbook 01: Compromised Credential & Secret Revocation Triage

> **Incident Severity**: SEV-1 (Critical Security Breach)  
> **Target Systems**: Microsoft Entra ID, AWS IAM, Azure Key Vault, AWS Secrets Manager, GitHub Actions  
> **Objective**: Immediately contain, revoke, and remediate leaked API keys, service principal client secrets, or cloud IAM credentials.

---

## 1. Emergency Decision Tree

```mermaid
flowchart TD
    Alert["SEV-1 Alert: Leaked Credential Detected (Gitleaks / CloudWatch / SecOps)"] --> Isolate{"Credential Type?"}
    
    Isolate -->|Cloud Service Principal / IAM User| RevokeSP["Phase 1: Invalidate Key / Secret in Entra ID or AWS IAM"]
    Isolate -->|Databricks Personal Access Token (PAT)| RevokePAT["Phase 1: Revoke PAT via Databricks Token API"]
    Isolate -->|Database Connection String| RotateDB["Phase 1: Trigger Dual-Secret Rollover in Vault & Target DB"]
    
    RevokeSP --> TerminateSessions["Phase 2: Terminate Active Sessions & Invalidate Ephemeral Tokens"]
    RevokePAT --> TerminateSessions
    RotateDB --> TerminateSessions
    
    TerminateSessions --> ScopeAudit["Phase 3: Forensic Audit Log Analysis (CloudTrail / Azure Activity)"]
    ScopeAudit --> ExfilCheck{"Was Data Exfiltrated?"}
    
    ExfilCheck -->|Yes| LegalNotify["Trigger Legal & Regulatory Breach Protocol (< 72h GDPR/SEC)"]
    ExfilCheck -->|No| LandFix["Re-issue Ephemeral OIDC Federated Identity & Close Incident"]
```

---

## 2. Emergency Remediation & Containment Protocol

### Step 1: Immediate Credential Invalidation ($< 5\text{ minutes}$)

#### Azure Entra ID Service Principal Client Secret:
```bash
# Delete the compromised client secret immediately
az ad app credential delete \
  --id <APPLICATION_CLIENT_ID> \
  --key-id <COMPROMISED_KEY_ID>

# Revoke all active refresh tokens and user/service sessions
az rest --method POST \
  --uri "https://graph.microsoft.com/v1.0/servicePrincipals/<OBJECT_ID>/revokeSignInSessions"
```

#### AWS IAM Access Key:
```bash
# Deactivate the compromised access key immediately
aws iam update-access-key \
  --user-name <IAM_USER_NAME> \
  --access-key-id <COMPROMISED_ACCESS_KEY_ID> \
  --status Inactive

# Delete the access key after verifying no critical dependencies
aws iam delete-access-key \
  --user-name <IAM_USER_NAME> \
  --access-key-id <COMPROMISED_ACCESS_KEY_ID>
```

---

## 3. Forensic Scope & Impact Recovery

### Step 2: Query CloudTrail / Azure Activity Log for Malicious Actions
Isolate all API operations executed by the compromised identity over the prior 30 days:

```bash
# AWS CloudTrail: Find all S3 GetObject and IAM calls made by the compromised key
aws cloudtrail lookup-events \
  --lookup-attributes AttributeKey=AccessKeyId,AttributeValue=<COMPROMISED_ACCESS_KEY_ID> \
  --start-time $(date -v-14d +%s) \
  --query 'Events[*].{Time:EventTime,Name:EventName,IP:SourceIPAddress,UserAgent:UserAgent}' \
  --output table
```

### Step 3: Forensic Checklist
- [ ] Were any new IAM users, access keys, or role trust relationships created? (Check for persistence backdoors).
- [ ] Were any S3 buckets, ADLS containers, or databases made public or downloaded from anomalous IP addresses?
- [ ] Were any compute clusters or virtual machines provisioned? (Check for cryptojacking).

---

## 4. Root Cause Eradication & Post-Mortem Action Items
1. Migrate the offending workflow to **OIDC Workload Identity Federation** (zero static secrets).
2. Install pre-commit hooks (`gitleaks protect --staged`) across developer workstations.
3. Enable GitHub secret scanning and push protection on all organizational repositories.
