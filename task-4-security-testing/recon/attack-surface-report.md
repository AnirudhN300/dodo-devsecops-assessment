# Dodo Payments Attack Surface Report

## Scope
Domain:
dodopayments.tech

Testing Type:
Passive Reconnaissance Only

## Subdomain Discovery

Sources:
- Certificate Transparency (crt.sh)
- Subfinder
- Amass

Total discovered assets:

## Live Hosts

| Host | Status | Technology | Risk Observation |
|------|--------|------------|------------------|
| app.dodopayments.tech | 200 | React, Next.js | Public application |
| mb.dodopayments.tech | 200 | Metabase | Sensitive analytics exposure |
| sonarqube.dodopayments.tech | 200 | SonarQube | Development tool exposure |
| sentry.dodopayments.tech | 403 | Cloudflare | Error monitoring system |

## TLS Findings

Example:

Host:
app.dodopayments.tech

Observations:
- TLS version supported
- Certificate issuer
- HSTS enabled
- Weak cipher findings

## Risk Observations

High Interest Assets:
- admin panels
- dashboards
- developer tools
- APIs
- monitoring systems

No exploitation performed.
