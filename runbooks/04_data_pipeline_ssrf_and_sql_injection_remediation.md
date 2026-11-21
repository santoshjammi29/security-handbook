# ⚡ Production Runbook 04: Data Pipeline SSRF & SQL Injection Remediation

> **Incident Severity**: SEV-2  
> **Target Systems**: Python Data Pipelines, Azure Data Factory, Apache Airflow, PostgreSQL, Snowflake  
> **Objective**: Identify, contain, and remediate Server-Side Request Forgery (SSRF) and SQL injection vulnerabilities within data pipelines and webhook ingestion activities.

---

## 1. Emergency Decision Tree

```mermaid
flowchart TD
    Alert["SEV-2 Alert: SQL Injection or SSRF Attempt Detected in Data Pipeline"] --> Classify{"Vulnerability Type?"}
    
    Classify -->|Server-Side Request Forgery (SSRF)| CheckDest{"Destination IP / URL?"}
    CheckDest -->|Instance Metadata (169.254.169.254)| BlockIMDS["Phase 1: Block IMDS Metadata IP on Host Firewall / iptables"]
    CheckDest -->|Internal Private Subnet Scan| BlockInternal["Phase 1: Enforce Strict Egress Allowlist on Webhook Activity"]
    
    Classify -->|SQL Injection Vulnerability| FindPipeline["Phase 1: Locate Offending Pipeline / SQL Query String"]
    FindPipeline --> TerminateRun["Kill Active Ingestion Runs & Isolate Database Session"]
    TerminateRun --> ParamRewrite["Phase 2: Rewrite Query with Strict Parameterized Bindings"]
    
    BlockIMDS --> Recovery["Phase 3: Verify No Cloud IAM Metadata Credentials Leaked"]
    BlockInternal --> Recovery
    ParamRewrite --> Recovery
    
    Recovery --> PostMortem["Phase 4: Implement SAST Linting (Bandit / Semgrep) in CI/CD"]
```

---

## 2. Emergency Remediation: Server-Side Request Forgery (SSRF)

### Threat Scenario:
An ingestion pipeline accepts a user-provided webhook or callback URL (e.g. `callback_url = "http://169.254.169.254/latest/meta-data/"`). The worker node calls this endpoint, leaking the node's IAM instance profile credentials or internal cluster tokens to the external caller.

### Step 1: Block Instance Metadata Service (IMDS) via Host Firewall
```bash
# Block all non-root access to the AWS / Azure / GCP Metadata IP
sudo iptables -A OUTPUT -m owner ! --uid-owner root -d 169.254.169.254 -j DROP
```

### Step 2: Enforce Strict Domain Allowlists in Application Code
```python
import ipaddress
import socket
from urllib.parse import urlparse

PERMITTED_DOMAINS = {"api.partner-ingest.com", "hooks.slack.com"}

def validate_outbound_url(url: str) -> bool:
    parsed = urlparse(url)
    if parsed.scheme != "https":
        raise ValueError("Only HTTPS protocol is permitted.")
        
    if parsed.hostname not in PERMITTED_DOMAINS:
        raise ValueError(f"Target domain {parsed.hostname} is not whitelisted.")
        
    # Resolve IP and block private / loopback / link-local addresses
    resolved_ip = socket.gethostbyname(parsed.hostname)
    ip_obj = ipaddress.ip_address(resolved_ip)
    
    if ip_obj.is_private or ip_obj.is_loopback or ip_obj.is_link_local:
        raise ValueError("Destination IP resolves to a private network address.")
        
    return True
```

---

## 3. Emergency Remediation: SQL Injection

### Step 1: Identify and Terminate Active Malicious Sessions
```sql
-- PostgreSQL: Terminate active query running injected statement
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE query LIKE '%OR 1=1%' OR query LIKE '%; DROP TABLE%';
```

### Step 2: Refactor Dynamic Query to Parameterized Bindings
```python
# ❌ VULNERABLE: String formatting susceptible to SQL injection
cursor.execute(f"UPDATE fact_orders SET status = 'CANCELLED' WHERE order_id = '{raw_order_id}'")

# ✅ REMEDIATED: Bound parameters safely handled by the database driver
cursor.execute("UPDATE fact_orders SET status = 'CANCELLED' WHERE order_id = %s", (raw_order_id,))
```

---

## 4. Recovery & Verification
- [ ] Verify that no tables, schemas, or users were dropped, created, or modified during the exploit attempt.
- [ ] Check CloudTrail / Azure Activity Logs for unauthorized role assumptions using stolen metadata credentials.
- [ ] Add Semgrep and Bandit rules to the CI/CD pipeline to block unparameterized SQL queries in pull requests.
