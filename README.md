# ledger-api

Payments microservice for tokenising PANs and serving transaction metadata.
Deployed on Kubernetes in the `payments` namespace.

## Endpoints

| Method | Path            | Description                          |
|--------|-----------------|--------------------------------------|
| GET    | `/health`       | Liveness check                       |
| POST   | `/tokenize`     | `{"pan": "..."}` → opaque token      |
| GET    | `/transactions` | Recent transaction records           |
| POST   | `/import`       | Import a YAML configuration blob     |
| GET    | `/fetch?url=`   | Fetch a remote resource by URL       |


---

# Security Gate Policy

The CI pipeline enforces the following security gates to ensure secure software delivery.

## Semgrep
- Performs Static Application Security Testing (SAST).
- The pipeline should fail on High or Critical security findings.
- Medium findings should be reviewed before merging.

## Gitleaks
- Detects hardcoded secrets and credentials.
- The pipeline fails if plaintext secrets are detected.
- Kubernetes Sealed Secrets are allowlisted.

## Trivy Filesystem Scan
- Scans the repository for vulnerabilities and misconfigurations.
- Critical vulnerabilities should block deployment.
- High vulnerabilities require review before release.

## Trivy Image Scan
- Scans the Docker image before publishing.
- Container images must not contain Critical vulnerabilities.

## Cosign Image Signing
- Every Docker image published to GHCR is digitally signed using Cosign.
- Only signed images are considered trusted for deployment.

## SLSA Provenance
- Build provenance is generated for every published container image.
- Provenance is stored with the image in the container registry.

## ArgoCD GitOps
- Deployments are managed from Git as the single source of truth.
- Automatic synchronization keeps the cluster aligned with Git.
- Self-heal restores resources if manual changes or drift occur.
