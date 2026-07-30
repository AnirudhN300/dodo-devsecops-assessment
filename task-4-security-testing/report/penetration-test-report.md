# Dodo Payments Security Assessment
## Task 4 – Penetration Testing Report

**Author:** Anirudh N  
**Assessment Date:** 30 July 2026

---

# Assessment Information

| Item | Value |
|------|-------|
| Assessment Type | Authorized Web Application Penetration Test |
| Target | Local Ledger API (http://127.0.0.1:8080) |
| Assessment Date | 30 July 2026 |
| Tester | Anirudh N |
| Methodology | OWASP Web Security Testing Guide (WSTG) |

---

# Executive Summary

An authorized penetration test was conducted against the provided vulnerable Ledger API application running locally as part of the Dodo Payments DevSecOps Assessment.

The objective was to identify common web application vulnerabilities based on the OWASP Top 10 while following the provided Rules of Engagement. The assessment included reconnaissance, endpoint enumeration, manual penetration testing, source code review, and validation of identified security issues.

Three confirmed security vulnerabilities were identified:

1. Sensitive Information Exposure
2. Server-Side Request Forgery (SSRF)
3. Unsafe YAML Deserialization

Additional testing was performed for SQL Injection, Cross-Site Scripting (XSS), authentication weaknesses, endpoint discovery, and tokenization functionality. These tests did not reveal exploitable vulnerabilities.

---

# Scope

## Authorized Target

Local Ledger API

```
http://127.0.0.1:8080
```

## In Scope

- Ledger API running locally
- Source code supplied with the assessment
- HTTP endpoints

## Out of Scope

- Production Dodo Payments infrastructure
- dodopayments.tech production services
- Third-party infrastructure
- Denial-of-Service attacks
- Social Engineering

---

# Methodology

The assessment followed the OWASP Web Security Testing Guide (WSTG).

The engagement consisted of two phases.

## Phase 1 – Reconnaissance

Performed passive reconnaissance against the authorized assessment environment.

Activities included:

- Subdomain Enumeration
- DNS Enumeration
- Live Host Discovery
- HTTP Enumeration
- Technology Fingerprinting
- TLS Analysis
- Attack Surface Mapping

### Reconnaissance Tools

- Amass
- Subfinder
- crt.sh
- httpx
- WhatWeb
- testssl.sh

---

## Phase 2 – Penetration Testing

The application was tested using manual HTTP requests, endpoint enumeration, source code inspection, and vulnerability validation.

### Penetration Testing Tools

| Tool | Purpose |
|------|---------|
| curl | Manual HTTP request testing |
| ffuf | Endpoint discovery |
| Source Code Review | Vulnerability validation |
| Burp Suite Community | Available for HTTP interception (manual testing completed primarily using curl and source code review) |

---

# Evidence Collected

Evidence was collected through:

- HTTP responses
- Manual request testing
- Source code inspection
- Terminal output
- Screenshots
- Endpoint enumeration

---

# Findings

---

# Finding 1

## Sensitive Information Exposure

### Severity

**High**

### CVSS v3.1

**7.5 (High)**

**Vector**

```
CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:N
```

### Affected Endpoint

```
GET /transactions
```

### Description

The `/transactions` endpoint is accessible without authentication and exposes sensitive payment information, including complete payment card numbers (PANs).

### Proof of Concept

Request

```http
GET /transactions
```

Response

```json
{
  "transactions": [
    {
      "id": "txn_1001",
      "pan": "4242424242424242",
      "amount": 4200,
      "currency": "USD",
      "status": "captured"
    }
  ]
}
```

### Evidence

Screenshot

```
screenshots/transactions.png
```

### Impact

An unauthenticated attacker can retrieve sensitive transaction information and payment card numbers.

### Remediation

- Require authentication.
- Implement Role-Based Access Control (RBAC).
- Mask PAN values.
- Return only authorized user data.

---

# Finding 2

## Server-Side Request Forgery (SSRF)

### Severity

**High**

### CVSS v3.1

**8.1 (High)**

**Vector**

```
CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:N/A:L
```

### Affected Endpoint

```
GET /fetch
```

### Description

The application accepts a user-supplied URL and performs an outbound HTTP request without validation.

### Proof of Concept

Request

```http
GET /fetch?url=http://127.0.0.1:8080/health
```

Response

```json
{
  "status_code":200,
  "body":"{\"status\":\"ok\"}"
}
```

### Evidence

Screenshot

```
screenshots/fetch-ssrf.png
```

### Impact

An attacker may use the application to access internal services that would normally not be externally accessible.

### Remediation

- Validate destination URLs.
- Allowlist approved domains.
- Block localhost and internal IP ranges.
- Disable unnecessary outbound requests.

---

# Finding 3

## Unsafe YAML Deserialization

### Severity

**Medium**

### CVSS v3.1

**6.5 (Medium)**

**Vector**

```
CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:L/I:L/A:L
```

### Affected Endpoint

```
POST /import
```

### Description

The application uses the unsafe `yaml.load()` function when processing user supplied YAML data.

### Source Code

```python
config = yaml.load(request.data)
```

### Proof of Concept

Request

```yaml
name: test
role: admin
```

Response

```json
{
    "loaded":"{'name':'test','role':'admin'}"
}
```

### Evidence

Screenshot

```
screenshots/import-yaml.png
```

### Impact

Unsafe deserialization may allow malicious YAML payloads to instantiate arbitrary Python objects.

### Remediation

Replace

```python
yaml.load()
```

with

```python
yaml.safe_load()
```

---

# Additional Security Testing

## Endpoint Enumeration

### Tool

ffuf

### Results

```
/health
/transactions
/tokenize
/import
/fetch
```

### Evidence

```
screenshots/ffuf.png
```

---

## Tokenization Endpoint

### Request

```http
POST /tokenize
```

### Result

The endpoint successfully generated opaque payment tokens.

### Status

No security vulnerability identified.

### Evidence

```
screenshots/tokenize.png
```

---

## Health Endpoint

### Request

```http
GET /health
```

### Result

```
{
    "status":"ok"
}
```

### Status

Informational endpoint.

---

## SQL Injection Testing

The following payloads were tested manually.

```
?id='
```

```
?id=1
```

```
?id=1 OR 1=1
```

No SQL Injection vulnerability was identified.

---

## Cross-Site Scripting (XSS)

Payload tested

```html
<script>alert(1)</script>
```

No reflected or stored Cross-Site Scripting vulnerability was observed.

---

## Authentication Testing

The `/transactions` endpoint was accessible without authentication.

This directly resulted in the Sensitive Information Exposure finding documented above.

---

# Source Code Review

The supplied application source code was reviewed to validate identified vulnerabilities.

| Line | Observation |
|------|-------------|
| 33 | Transactions endpoint lacks authentication |
| 39 | Uses unsafe `yaml.load()` |
| 46 | Performs unrestricted `requests.get()` |

Evidence

```
screenshots/app-source.png
```

---

# Attack Chain

The identified vulnerabilities can be chained together to increase impact.

```
Unauthenticated User

        │

        ▼

Access /transactions

        │

        ▼

Obtain Payment Card Data

        │

        ▼

Exploit SSRF (/fetch)

        │

        ▼

Access Internal Services
```

---

# Mapping Findings to Tasks 1–3

| Finding | Preventive Control |
|----------|-------------------|
| Sensitive Information Exposure | Authentication, RBAC, Secure Code Review |
| SSRF | Semgrep SAST Rules |
| Unsafe YAML Deserialization | Semgrep Static Analysis |
| Hardcoded Secrets | Gitleaks |
| Container Vulnerabilities | Trivy |
| Image Integrity | Cosign |
| Supply Chain Integrity | SLSA Provenance |

---

# Retest

A remediation retest was not performed because source code modifications were outside the scope of this assessment.

---

# Conclusion

The assessment successfully identified multiple security weaknesses within the vulnerable Ledger API application.

Three confirmed vulnerabilities were discovered:

- Sensitive Information Exposure
- Server-Side Request Forgery (SSRF)
- Unsafe YAML Deserialization

Additional testing was conducted for SQL Injection, Cross-Site Scripting (XSS), endpoint enumeration, authentication, and tokenization functionality. No exploitable vulnerabilities were identified in these areas.

The application would benefit from stronger authentication controls, secure input validation, safer deserialization practices, and restrictions on outbound HTTP requests.

Overall, the assessment demonstrates the importance of secure coding practices, proper access control, and automated security testing within the software development lifecycle.

---

# Appendix

## Screenshots

```
screenshots/httpx-results.png
screenshots/whatweb-results.png
screenshots/dns-records.png
screenshots/tls-summary.png
screenshots/attack-surface-report.png
screenshots/transactions.png
screenshots/tokenize.png
screenshots/fetch-ssrf.png
screenshots/import-yaml.png
screenshots/ffuf.png
screenshots/app-source.png
```

## Source Code Reviewed

```
app/app.py
```

## Reconnaissance Artifacts

```
recon/httpx-results.txt
recon/whatweb-results.txt
recon/dns-records.txt
recon/tls-summary.txt
recon/attack-surface-report.md
```
