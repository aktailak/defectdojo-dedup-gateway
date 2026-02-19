# DefectDojo Deduplication Gateway

Enterprise-grade ingestion gateway for DefectDojo.

The Deduplication Gateway acts as a pre-import control layer between security scanners and DefectDojo. It prevents duplicate findings, enforces ingestion policy, and optionally imports only new findings.

Designed for CI/CD environments and Kubernetes-native deployments.

---

## 🎯 Purpose

Modern CI pipelines generate large volumes of security findings across:

- Secrets scanning (Gitleaks)
- SAST (Semgrep)
- Container scanning (Trivy)
- Infrastructure scanning
- Custom scanners

Directly importing all findings into DefectDojo can lead to:

- Duplicate findings
- Increased storage usage
- Performance overhead
- Reduced signal-to-noise ratio

This gateway introduces a controlled ingestion layer that:

- Deduplicates findings before import
- Normalizes scanner data (future-ready)
- Enforces import policies
- Imports only new findings
- Returns structured results to CI

---

## 🏗 Architecture

```text
Security Scanner (CI)
│
│ JSON Report
▼
Deduplication Gateway
│
├── Normalize (scanner adapter)
├── Fetch existing findings
├── Deduplicate
├── Import new findings
└── Store result (Redis)
▼
CI retrieves result → controls pipeline
```
---

## 🔌 Supported Scanners

| Scanner | Status |
|----------|--------|
| Gitleaks | Supported |
| Trivy | Planned |
| Semgrep | Planned |
| Custom JSON | Planned |

---

## 📡 API Design (Scalable)

The API is scanner-agnostic.

### POST `/deduplicate/{scanner}`

Start a deduplication task.

Example:
POST /deduplicate/gitleaks

#### Request Body

```json
{
  "product_id": 1,
  "engagement_id": 10,
  "auto_import": true,
  "report": [...]
}
```
Response
{
  "task_id": "uuid",
  "status": "started"
}

GET /tasks/{task_id}

Retrieve task result.
{
  "status": "completed",
  "scanner": "gitleaks",
  "new_count": 1,
  "duplicates_count": 30,
  "new_findings": [...],
  "import_result": {...}
}

---

## 🔐 Authentication

Bearer token authentication:

Authorization: Bearer <DEDUP_API_TOKEN>

---

⚙️ Configuration

DEFECTDOJO_API_URL (Required): Base URL

DEFECTDOJO_API_TOKEN (Required): API token

DEDUP_API_TOKEN (Required): Gateway access token

REDIS_URL (Optional): Redis connection string

REDIS_TTL (Optional): Task result retention

---

## ☸ Kubernetes Deployment

Designed for deployment under the defectdojo namespace as an independent microservice.

Recommended Deployment Pattern

Separate Deployment & Service

Dedicated Redis instance

Optional Ingress under /dedup

Independent scaling

NetworkPolicy isolation

---

## 🔁 Deduplication Model

Each scanner adapter defines:

dedup_key = <stable unique fingerprint>


For Gitleaks:

secret + file_path


Future scanners can implement custom strategies.

---

## 🛡 Security Recommendations

HTTPS only

Store tokens in Kubernetes Secrets

Restrict ingress access

Enable network policies

Rotate API tokens periodically

Avoid disabling SSL verification in production

---

## 📈 Roadmap

Plugin-based scanner adapters

Structured logging

Prometheus metrics

Rate limiting

HMAC authentication

Async task queue (Celery/RQ)

Bulk pagination support

---

## 🔍 Design Principles

Stateless API layer

Scanner-agnostic

No modifications to DefectDojo core

Uses official DefectDojo APIs only

Fully removable without impact

---

## Disclaimer

This project is not affiliated with or endorsed by OWASP DefectDojo.

It is an external ingestion control layer for enterprise CI/CD environments.
