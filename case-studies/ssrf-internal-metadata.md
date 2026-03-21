# Case Study: Server-Side Request Forgery (SSRF) — Cloud Metadata Access

**OWASP Category:** OWASP Top 10 — A10:2021 Server-Side Request Forgery
**Severity:** Critical (in cloud environments)
**CWE:** CWE-918 (Server-Side Request Forgery)

---

## Summary

A web application with a URL-fetching feature (e.g., "import from URL",
webhook tester, PDF generator) failed to validate or restrict the URLs
it would request. An attacker could supply internal cloud metadata service
addresses, exposing AWS/GCP/Azure credentials and instance configuration.

---

## Environment (redacted lab reproduction)

Web application with a "fetch preview" feature: `POST /api/preview`
Body: `{ "url": "https://example.com" }`
Application runs on a cloud instance (AWS EC2 / GCP Compute / Azure VM).

---

## Root cause

The server-side fetch function accepted arbitrary user-supplied URLs with
no allowlist, blocklist, or DNS rebinding protections:

```python
# Vulnerable pseudo-code
@app.route('/api/preview', methods=['POST'])
def preview():
    url = request.json['url']
    response = requests.get(url)  # no validation
    return response.text
```

---

## Reproduction (lab/authorized environment only)

**AWS Metadata Service (IMDSv1):**
```
POST /api/preview
{ "url": "http://169.254.169.254/latest/meta-data/iam/security-credentials/" }
```
Response: IAM role name

```
{ "url": "http://169.254.169.254/latest/meta-data/iam/security-credentials/RoleName" }
```
Response: Temporary AWS credentials (AccessKeyId, SecretAccessKey, Token)

**GCP Metadata:**
```
{ "url": "http://metadata.google.internal/computeMetadata/v1/instance/service-accounts/default/token" }
```

---

## Impact

- Exposure of cloud instance credentials (potential full account compromise)
- Access to internal services not exposed externally (databases, admin panels)
- Port scanning of internal network via SSRF
- In severe cases: RCE via chained vulnerabilities on internal services

---

## Fix

```python
import ipaddress, socket
from urllib.parse import urlparse

ALLOWED_SCHEMES = {'https'}
BLOCKED_RANGES = [
    ipaddress.ip_network('169.254.0.0/16'),  # link-local / metadata
    ipaddress.ip_network('10.0.0.0/8'),       # RFC1918
    ipaddress.ip_network('172.16.0.0/12'),
    ipaddress.ip_network('192.168.0.0/16'),
    ipaddress.ip_network('127.0.0.0/8'),      # loopback
]

def is_safe_url(url):
    parsed = urlparse(url)
    if parsed.scheme not in ALLOWED_SCHEMES:
        return False
    ip = ipaddress.ip_address(socket.gethostbyname(parsed.hostname))
    if any(ip in net for net in BLOCKED_RANGES):
        return False
    return True
```

- Enable IMDSv2 (token-required) on AWS to limit metadata exposure
- Use egress firewall rules to block internal ranges from app servers
- Implement DNS rebinding protections

---

## Verification

1. Submit `http://169.254.169.254/` → expect `400 Bad Request` or blocked
2. Submit `http://127.0.0.1/` → expect blocked
3. Submit `http://10.0.0.1/` → expect blocked
4. Submit `https://example.com` → expect normal operation
5. Test with a DNS rebinding payload to verify runtime IP validation

---

## Key takeaway

**Validate at the network layer, not just the input layer.**
Allowlist expected destinations where possible. Treat any URL-fetching
feature as a potential SSRF vector during design and testing.
