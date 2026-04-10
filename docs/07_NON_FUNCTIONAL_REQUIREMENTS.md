# 07 - Nefunkční požadavky | ergon+ MES

## 🏗️ Architektonické požadavky

### AR-01: Clean Architecture (DDD Light)

**Požadavek:** Aplikace se řídí principy Clean Architecture s vrstvami:
- **Presentation Layer** (Controllers, ViewModels, DTOs)
- **Application Layer** (Use Cases, Services)
- **Domain Layer** (Entities, Value Objects, Business Logic)
- **Infrastructure Layer** (Repositories, External Integrations)

**Benefit:** Testovatelnost, maintainability, flexibilita

```
ergon.Api (Presentation)
  ├── Controllers (HTTP endpoints)
  ├── ViewModels (Request/Response DTOs)
  └── Middlewares (Auth, Logging, Error Handling)

ergon.Application (Use Cases)
  ├── Services (Business Logic)
  ├── Mappers
  ├── Validators
  └── Interfaces

ergon.Domain (Business Rules)
  ├── Entities (Batch, Recipe, Material)
  ├── Value Objects (Quantity, Temperature Range)
  ├── Aggregates
  └── Domain Events

ergon.Infrastructure (Technical Implementations)
  ├── Repositories
  ├── OpcUaClient
  ├── ErpIntegration
  ├── EmailService
  └── Logging
```

---

## ⚡ Performance požadavky

### PER-01: Response Time

| Endpoint / Action | Target | Priority |
|---|---|---|
| GET /batches (list) | < 500ms | HIGH |
| GET /batches/{id} (detail) | < 200ms | HIGH |
| POST /batches (create) | < 300ms | HIGH |
| POST /batches/{id}/start (action) | < 200ms | CRITICAL |
| GET /dashboard/kpi | < 1000ms | MEDIUM |
| Parameter logging (OPC UA → DB) | < 100ms | CRITICAL |

### PER-02: Throughput

- **Batch operations:** 100+ concurrent users
- **OPC UA subscription:** 1000+ tags @ 1Hz
- **Parameter logging:** 10,000+ records/second
- **API calls:** 500+ requests/second
- **WebSocket connections:** 500+ simultaneous

### PER-03: Database Performance

```sql
-- Query targets (with proper indexing)
SELECT * FROM batches WHERE status = 'RUNNING'
  -- < 50ms pro 100,000 batches

SELECT * FROM parameter_log WHERE batch_id = ? AND timestamp > ?
  -- < 200ms para 1M records

SELECT * FROM audit_log WHERE entity_id = ?
  -- < 300ms pro 10M records
```

### PER-04: Caching Strategy

```csharp
{
  "cache": {
    "redis": {
      "enabled": true,
      "endpoints": ["redis-1:6379", "redis-2:6379"],
      "ttl_config": {
        "users": 3600,              // 1 hour
        "roles": 7200,              // 2 hours
        "recipes": 86400,           // 24 hours
        "current_batch_status": 30, // 30 sec
        "equipment_parameters": 10  // 10 sec
      }
    }
  }
}
```

---

## 🔐 Bezpečnostní požadavky

### SEC-01: Autentizace

- **Multi-factor Authentication (MFA)** pro kritické role (ADMIN, AUDITOR)
- **SSO** integrace (Azure AD / LDAP)
- **Session timeout:** 30 minut nečinnosti
- **Lockout:** 5 neúspěšných pokusů = 15 minut blokace
- **Password policy:**
  - Minimálně 12 znaků
  - Kombinace: UPPER + lower + numbers + symbols
  - Expiraci: 90 dní
  - Historika: Nelze opakovat posledních 5 hesel

### SEC-02: Autorizace

- **RBAC** na všech endpointech
- **Attribute-based access control** pro citlivá data
- **Row-level security:** Uživatel vidí jen data své linie/skupiny
- **Principle of Least Privilege** - výchozí je DENY

### SEC-03: Šifrování

```csharp
{
  "encryption": {
    "transport": {
      "protocol": "TLS",
      "version": "1.3",
      "cipher_suites": ["TLS_AES_256_GCM_SHA384", "TLS_AES_128_GCM_SHA256"],
      "hsts": { "enabled": true, "max_age": 31536000 }
    },
    "at_rest": {
      "algorithm": "AES-256-GCM",
      "key_derivation": "PBKDF2",
      "fields": [
        "user.password_hash",
        "user.mfa_secret",
        "material.vendor_code"  // sensitive data
      ]
    },
    "key_management": {
      "provider": "Azure Key Vault",  // nebo HashiCorp Vault
      "rotation_days": 90,
      "backup": "encrypted_offline"
    }
  }
}
```

### SEC-04: Audit & Compliance

- **All actions logged** se USER, TIMESTAMP, ACTION, DATA (old → new)
- **Immutable audit log** - nelze mazat
- **Audit retention:** minimum 7 let (dle regulací)
- **GDPR compliance:**
  - Right to be forgotten (archivace)
  - Data portability (export funkcionalita)
  - Consent tracking
- **21 CFR Part 11** (FDA validace pro farmaceutiku)

---

## 📊 Scalability požadavky

### SCL-01: Horizontální škálování

```yaml
kubernetes:
  deployment:
    replicas: 3
    pod_disruption_budget:
      min_available: 1
    
    resources:
      requests:
        memory: "512Mi"
        cpu: "250m"
      limits:
        memory: "2Gi"
        cpu: "1000m"
    
    autoscaling:
      enabled: true
      min_replicas: 3
      max_replicas: 20
      target_cpu: 70%
      target_memory: 80%
```

### SCL-02: Database Scaling

- **Read replicas** pro reporting (slave databases)
- **Connection pooling:** 100+ concurrent connections
- **Sharding:** Pokud je potřeba, partition podle line_id nebo batch_id
- **Archivace:** parameter_log se archivuje měsíčně do cold storage

### SCL-03: Cost Optimization

- **Cloudové dedupliky:** Spot instances pro non-critical, Reserved instances pro Critical
- **Auto-shutdown** dev/test envs mimo pracovní dobu
- **Resource limits** na namespace level
- **Monitoring:** CloudWatch / DataDog pro cost tracking

---

## 🔄 Dostupnost (Availability)

### AVL-01: Uptime SLA

```
Kritické služby:    99.95% uptime (Rolling monthly)
- Production control live
- OPC UA connectivity
- Batch state management

High importance:     99.5% uptime
- Dashboard
- Reporting
- Desktop API

Other services:      99.0% uptime
- Admin panels
- Configuration UIs
```

### AVL-02: Disaster Recovery

```csharp
{
  "disaster_recovery": {
    "backup": {
      "frequency": "hourly",
      "full_backup": "daily",
      "retention_days": 30,
      "verification": "weekly_restore_test",
      "location": "geo_redundant"
    },
    
    "rto_rpo": {
      "kritical_services": { "rto_min": 15, "rpo_min": 1 },
      "high_priority": { "rto_min": 60, "rpo_min": 5 },
      "normal": { "rto_min": 480, "rpo_min": 60 }
    },
    
    "failover": {
      "db_primary_to_secondary": "automatic",
      "health_check_interval_sec": 10,
      "failover_trigger_threshold": 2
    }
  }
}
```

### AVL-03: System Health Monitoring

```csharp
// Endpoint pro health check
GET /health

{
  "status": "healthy",
  "timestamp": "2026-04-10T14:30:00Z",
  "checks": {
    "database_connectivity": "ok",
    "opc_ua_connection": "ok",
    "cache_redis": "ok",
    "disk_space": "ok",
    "memory": "ok",
    "cpu": "ok"
  }
}

// Critical alerts if:
- Database offline > 1 min
- OPC UA disconnected > 2 min
- Memory usage > 90%
- API errors > 1% (5 min average)
```

---

## 🧪 Testovací požadavky

### TST-01: Coverage

- **Unit tests:** ≥ 80% code coverage (Controllers a Services)
- **Integration tests:** Všechny kritické workflow
- **E2E tests:** Klíčové uživatelské cesty
- **Load tests:** Simulace 100+ concurrent users

### TST-02: Test Automation

```bash
# Continuous Integration pipeline
1. Unit tests (Maven/MSBuild)
2. Code quality analysis (SonarQube)
3. Security scanning (OWASP, Snyk)
4. Integration tests (Docker containers)
5. Deploy to staging
6. E2E tests (Selenium/Cypress)
7. Performance baseline
8. Deploy to production (if all pass)
```

### TST-03: Manual Testing

- **Smoke tests:** Před každou deploy
- **UAT:** Se stakeholders před release
- **Exploratory testing:** Régulière QA audit

---

## 📱 Kompatibilita & Podporované Verze

### COM-01: Browser Support

| Browser | Minimální verze |
|---------|-----------------|
| Chrome | 90+ |
| Firefox | 88+ |
| Safari | 14+ |
| Edge | 90+ |

### COM-02: Desktop App Support

```json
{
  "desktop": {
    "windows": {
      "min_version": "10.0.19041",
      "recommended_version": "10.0.22000"
    }
  }
}
```

---

## 📝 Dokumentace & Operability

### DOC-01: API Documentation

- **OpenAPI/Swagger** specifikace (auto-generated)
- **Interactive API docs** (Swagger UI)
- **Code examples** pro každý endpoint
- **Error codes** reference guide

### DOC-02: Operations Manual

- **Deployment guide** (dev, staging, prod)
- **Troubleshooting guide**
- **Monitoring & alerting** setup
- **Maintenance procedures** (backup, restore, upgrades)

### DOC-03: Architecture Documentation

- **Architecture Decision Records (ADR)** pro klíčové rozhodnutí
- **System design** diagramy
- **Integration guides** pro ERP/PLC
- **Configuration reference**

---

## 🎯 Monitoring & Observability

### MON-01: Metriky (Metrics)

```csharp
{
  "prometheus_metrics": {
    "application": {
      "api_request_duration_seconds": "histogram",
      "api_request_total": "counter",
      "api_request_errors_total": "counter",
      "database_query_duration_seconds": "histogram",
      "opc_ua_value_updates_total": "counter",
      "batch_processing_duration_seconds": "histogram"
    },
    "business": {
      "active_batches": "gauge",
      "completed_batches_total": "counter",
      "failed_batches_total": "counter",
      "kpi_oee": "gauge",
      "kpi_productivity": "gauge"
    },
    "system": {
      "memory_usage_bytes": "gauge",
      "cpu_usage_percent": "gauge",
      "disk_usage_bytes": "gauge"
    }
  },
  
  "export": "Prometheus (every 15 sec)"
}
```

### MON-02: Logging (Logs)

```csharp
{
  "logging": {
    "format": "JSON",
    "providers": ["Console", "File", "CloudWatch"],
    "levels": {
      "default": "Information",
      "Microsoft": "Warning",
      "ergon": "Debug"  // Dev only
    },
    "structured_fields": {
      "timestamp": true,
      "level": true,
      "logger": true,
      "user_id": true,
      "request_id": true,
      "duration_ms": true,
      "exception": true
    },
    "retention": {
      "days": 30,
      "archive_to_s3": true,
      "searchable_days": 7
    }
  }
}
```

### MON-03: Tracing (Traces)

```csharp
// Distributed tracing s OpenTelemetry
{
  "tracing": {
    "enabled": true,
    "sampler": "AlwaysSampler",  // For critical paths
    "exporters": ["Jaeger", "Azure Application Insights"],
    "instrumentations": [
      "AspNetCore",
      "HttpClient",
      "SqlClient",
      "OpcUa"
    ]
  }
}

// Example: Trace batch creation
[
  {
    "span": "POST /api/batches",
    "duration": 250,
    "children": [
      "validate_input (10ms)",
      "db_create_batch (50ms)",
      "db_assign_materials (30ms)",
      "send_notification (150ms)"
    ]
  }
]
```

---

## 🚀 Deployment & Release Management

### DEP-01: Versioning

- **Semantic versioning:** MAJOR.MINOR.PATCH
- **API versioning:** /api/v1/, /api/v2/ (podpoří 2 verze v prod)
- **Database migrations:** Versionované, automátické s deploym

### DEP-02: Release Frequency

- **Patch releases:** Weekly (bug fixes, security patches)
- **Minor releases:** Bi-weekly (features)
- **Major releases:** Quarterly (breaking changes)

### DEP-03: Deployment Strategy

```
Blue-Green Deployment:
┌─────────────┐         ┌─────────────┐         ┌─────────────┐
│  Prod v1.2  │         │  Staging v2.0│         │  Test v3.0  │
│  (BLUE)     │─────────│ (deployment) │─────────│ (features)  │
└─────────────┘         └─────────────┘         └─────────────┘
      ▲                         │                      │
      │                         ▼                      │
      └─────────────────────────┘──────────────────────┘
                                │
                    Switch traffic (if OK)
```

---

## 💰 Operační náklady

- **Cloud**: AWS / Azure ~5,000-10,000 Kč/měsíc (dle load)
- **Database**: MariaDB managed ~2,000 Kč/měsíc
- **Monitoring**: Datadog / New Relic ~3,000 Kč/měsíc
- **Storage**: Archivní storage ~1,000 Kč/měsíc
- **Support/SLA**: 5,000 Kč/měsíc (opcí)

---

**Datum vytvoření:** 10.4.2026  
**Status:** Iniciace projektu  
**Verze:** 0.1
