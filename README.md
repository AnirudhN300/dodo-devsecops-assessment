# Ledger API - DevSecOps Security Assessment

## Overview

Ledger API is a Flask-based payments microservice designed to tokenize PAN data and provide transaction metadata through REST APIs.

This project demonstrates a complete DevSecOps lifecycle including:

- Application setup and API validation
- Secure coding practices
- Containerization
- Security testing
- Reconnaissance and attack surface discovery
- CI/CD security gates
- Kubernetes GitOps deployment approach

---

# Architecture

## Application Flow

```
Developer
    |
    |
Git Repository
    |
    |
CI/CD Pipeline
    |
    |
Security Gates
    |
    |
Docker Image
    |
    |
Kubernetes Cluster
    |
    |
Ledger API Service
```

Architecture diagram:

```
docs/architecture-diagram.png
```

---

# Technology Stack

| Component | Technology |
|-----------|------------|
| Backend | Python Flask |
| API | REST API |
| Containerization | Docker |
| Orchestration | Kubernetes |
| Security Proxy | Burp Suite |
| Reconnaissance | Amass, crt.sh, httpx |
| Technology Discovery | WhatWeb |
| SAST | Semgrep |
| Secret Detection | Gitleaks |
| Vulnerability Scanner | Trivy |
| Image Security | Cosign |
| GitOps | ArgoCD |

---

# API Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/health` | Application health check |
| POST | `/tokenize` | Converts PAN into secure token |
| GET | `/transactions` | Returns transaction records |
| POST | `/import` | Imports YAML configuration |
| GET | `/fetch?url=` | Fetches remote resource |

---

# Task 1: Application Setup

## Approach

The Flask Ledger API was configured and tested locally before performing security validation.

A Python virtual environment was used to isolate application dependencies.

## Running Application

```bash
cd app
source venv/bin/activate
python app.py
```

Application URL:

```
http://localhost:8080
```

## Validation

Health endpoint:

```
GET /health
```

Transactions endpoint:

```
GET /transactions
```

Both endpoints returned successful HTTP responses.

## Evidence

Screenshots:

```
screenshots/application-running.png
screenshots/api-health.png
screenshots/api-transactions.png
```

---

# Task 2: API Security Testing Using Burp Suite

## Approach

Burp Suite Community Edition was used as an intercepting proxy to capture and analyze API requests.

Testing process:

1. Started Flask application.
2. Configured Burp proxy.
3. Accessed API through Burp browser.
4. Captured HTTP requests.
5. Verified API responses.

## Tested APIs

```
GET /health

GET /transactions
```

## Results

| Endpoint | Status |
|----------|--------|
| `/health` | 200 OK |
| `/transactions` | 200 OK |

## Evidence

```
screenshots/burp-http-history.png

screenshots/burp-health-request.png

screenshots/burp-transactions-request.png
```

---

# Task 3: Reconnaissance and Attack Surface Discovery

## Approach

Reconnaissance was performed to identify exposed services, domains, and application technologies.

---

## Amass

Purpose:

- Subdomain discovery

Output:

```
task-4-security-testing/recon/amass-subdomains.txt
```

---

## crt.sh

Purpose:

- Certificate transparency lookup

Output:

```
task-4-security-testing/recon/crtsh-subdomains.txt
```

---

## httpx

Purpose:

- HTTP service discovery
- Status code validation

Output:

```
task-4-security-testing/recon/httpx-results.txt
```

---

## WhatWeb

Purpose:

- Technology fingerprinting

Output:

```
task-4-security-testing/recon/whatweb-results.txt
```

---

## Attack Surface Report

Detailed findings:

```
task-4-security-testing/recon/attack-surface-report.md
```

---

# Task 4: Security Gate Policy

The CI pipeline integrates multiple security checks to ensure secure software delivery.

---

## Semgrep

Purpose:

Static Application Security Testing (SAST).

Security policy:

- Critical and High findings block the pipeline.
- Medium findings require review.

---

## Gitleaks

Purpose:

Detect exposed secrets and credentials.

Security policy:

- Builds fail when plaintext secrets are detected.
- Sensitive files are excluded using `.gitignore`.

---

## Trivy Filesystem Scan

Purpose:

- Scan dependencies.
- Detect vulnerabilities and misconfigurations.

Security policy:

- Critical vulnerabilities block deployment.

---

## Trivy Image Scan

Purpose:

Scan Docker images before publishing.

Security policy:

- Container images must not contain critical vulnerabilities.

---

## Cosign Image Signing

Purpose:

Secure container image verification.

Security policy:

- Only signed images are trusted for deployment.

---

## SLSA Provenance

Purpose:

Generate build provenance for container images.

Benefits:

- Improves software supply chain security.
- Provides build traceability.

---

## ArgoCD GitOps

Purpose:

Manage Kubernetes deployments using Git as the source of truth.

Features:

- Automated synchronization.
- Drift detection.
- Self-healing deployment.

---

# Security Design Decisions

## Source Code Security

- Environment files are excluded.
- Private keys and certificates are ignored.
- Secrets are not stored in source control.

## Container Security

- Container images are scanned before release.
- Vulnerabilities are identified before deployment.
- Signed images improve trust.

## Deployment Security

- Kubernetes deployments follow GitOps principles.
- Security checks are integrated into CI/CD.

---

# Project Structure

```
ledger-api-assignment/

├── app/
│   └── Flask application

├── task-4-security-testing/
│   └── recon/
│       ├── amass-subdomains.txt
│       ├── crtsh-subdomains.txt
│       ├── httpx-results.txt
│       ├── whatweb-results.txt
│       └── attack-surface-report.md

├── screenshots/
│   ├── burp-http-history.png
│   ├── burp-health-request.png
│   ├── burp-transactions-request.png
│   └── recon-results.png

├── docs/
│   └── architecture-diagram.png

└── README.md
```

---

# Verification Evidence

Completed verification:

✅ Flask application running  
✅ API health validation  
✅ Transaction API testing  
✅ Burp Suite request interception  
✅ Reconnaissance scans  
✅ Technology fingerprinting  
✅ Security gate documentation  
✅ Architecture documentation  

---

# Conclusion

This project demonstrates a DevSecOps approach where security is integrated throughout the software development lifecycle.

The implementation combines secure development practices, automated security validation, API testing, reconnaissance, and deployment security controls to improve application reliability and security.