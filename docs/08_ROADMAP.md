# 08 - Technická roadmapa | ergon+ MES

## 🗓️ Fáze vývoje

```
Q2 2026 (Duben-Červen)              Q3 2026 (Červenec-Září)          Q4 2026 + (Říjen+)
│                                   │                               │
├─ PHASE 0: Foundation              ├─ PHASE 2: Extended Features   ├─ PHASE 3: Advanced
│  v0.1 - v0.5                      │  v1.1 - v1.3                 │  v2.0+
│                                   │                               │
│ ✅ Setup & Architecture           │ ✅ ERP Integration             │ ✅ AI/ML Features
│ ✅ Basic CRUD APIs                │ ✅ Dashboards & KPI            │ ✅ Predictive Analytics
│ ✅ Auth & RBAC                    │ ✅ Desktop App v1              │ ✅ Advanced Reporting
│ ✅ OPC UA Integration             │ ✅ Advanced Alarms             │ ✅ Multi-site Support
│ ✅ Real-time Monitoring          │ ✅ Compliance (21 CFR 11)      │ ✅ Custom Workflows
│ ✅ Quality Assurance              │ ✅ Performance Optimization     │ ✅ Blockchain Audit
│                                   │ ✅ Cloud deployment             │ ✅ AR Visualization
│ RELEASE: v1.0 (Červen 2026)      │ RELEASE: v2.0 (Září 2026)    │ RELEASE: v3.0 (2027)
│                                   │                               │
└───────────────────────────────────┴───────────────────────────────┴──────────────────
```

---

## 📋 PHASE 0: Foundation (v0.1 - v1.0) | Duben-Červen 2026

### Cíl
Vybudovat stabilní základ MES systému s core funcionalitou pro řízení výroby a monitoring.

### v0.1 - Setup & Infrastructure (1 týden)

**Aktivita:**
- ✅ Git repozitář setup (GitHub/GitLab)
- ✅ CI/CD pipeline (GitHub Actions / GitLab CI)
- ✅ Windows Service deployment setup
- ✅ Development environment for backend service a desktop aplikace
- ✅ Logging & Monitoring (ELK stack / CloudWatch)

**Deliverables:**
- Windows Service installer / deployment package
- GitHub Actions workflow
- Local development setup guide

---

### v0.2 - Core API & Database (2 týdny)

**Aktivita:**
- ✅ Windows Service / .NET Worker project setup (Clean Architecture)
- ✅ Database schema (MariaDB)
  - Users, Roles, Lines, Equipment
  - Batches, Recipes, Materials
  - Parameter_Log, Alarms, Audit_Log
- ✅ Basic service interfaces pro desktop aplikace
  - Batches management
  - Recipes management
  - Materials management
- ✅ Entity Framework Core migrations

**Deliverables:**
```
POST   /api/batches                      - Create batch
GET    /api/batches/{id}                 - Get batch details
PUT    /api/batches/{id}                 - Update batch
DELETE /api/batches/{id}                 - Delete batch
GET    /api/batches?status=RUNNING       - List with filters

POST   /api/recipes                      - Create recipe
PUT    /api/recipes/{id}                 - Update recipe

POST   /api/materials                    - Create material
GET    /api/materials                    - List materials
```

**Tech Stack:**
- .NET 8 Worker Service / Windows Service
- Entity Framework Core
- MariaDB 11
- WPF / WinUI for desktop clients

---

### v0.3 - Authentication & Authorization (1.5 týdne)

**Aktivita:**
- ✅ User management service
- ✅ Password hashing (bcrypt)
- ✅ JWT token generation
- ✅ Role-Based Access Control (RBAC)
- ✅ Login API endpoint
- ✅ Token refresh mechanism
- ✅ Session management

**Deliverables:**
```
POST   /api/auth/login                   - Login
POST   /api/auth/refresh                 - Refresh token
POST   /api/auth/logout                  - Logout
POST   /api/auth/change-password         - Change password

POST   /api/admin/users                  - Create user
GET    /api/admin/users                  - List users
PUT    /api/admin/users/{id}             - Update user
DELETE /api/admin/users/{id}             - Delete user

Middleware:
- Authorization middleware
- Token validation
- Role checking
```

**Test Coverage:**
- Unit tests pro auth logic (>80%)
- Integration tests pro login flow

---

### v0.4 - OPC UA Integration (2 týdny)

**Aktivita:**
- ✅ OPC UA Client library (OPC.UA .NET)
- ✅ Connection management (reconnect logic)
- ✅ Tag subscription
- ✅ Real-time data logging
- ✅ Parameter validation & alarms
- ✅ Data aggregation (avg, min, max)

**Architecture:**
```
OpcUaClientService
├── Connect/Disconnect
├── Subscribe to tags
├── Handle value changes
└── Log to DB

ParameterLoggerService
├── Batching writes
├── Alarm triggers
└── Real-time cache update
```

**Deliverables:**
```
GET    /api/equipment/{id}/parameters/current
GET    /api/equipment/{id}/parameters/history?tag=TEMP_A&start=...&end=...
GET    /api/equipment/{id}/tags
```

**Configuration:**
```json
{
  "opcua": {
    "servers": [
      {
        "endpoint": "opc.tcp://192.168.1.100:4840",
        "tags": [{...}]
      }
    ]
  }
}
```

---

### v0.5 - Batch Lifecycle Management (1.5 týdne)

**Aktivita:**
- ✅ Batch state machine
- ✅ Transitions workflow
- ✅ Material assignment
- ✅ Line allocation
- ✅ Status updates
- ✅ WebSocket real-time updates

**State Flow:**
```
CREATED → PLANNED → READY → RUNNING → PAUSED → COMPLETED → PASSED/REJECTED → CLOSED
```

**Deliverables:**
```
POST   /api/batches/{id}/start           - Start batch
POST   /api/batches/{id}/pause           - Pause batch
POST   /api/batches/{id}/resume          - Resume batch
POST   /api/batches/{id}/complete        - Mark complete
POST   /api/batches/{id}/assign-line     - Assign to line

GET    /ws/batches/{id}                  - WebSocket for real-time updates
```

---

### v0.6 - Material Traceability (1 týden)

**Aktivita:**
- ✅ Material scanning endpoint
- ✅ Barcode/QR code processing
- ✅ Material ↔ Batch association
- ✅ Validation against BOM
- ✅ Alert on mismatch
- ✅ Traceability log

**Deliverables:**
```
POST   /api/batches/{id}/materials/scan
{
  "barcode": "MAT-2026-001234",
  "quantity": 5.2,
  "unit": "kg"
}

GET    /api/materials/{id}/trace          - Full traceability
```

---

### v0.7 - Alarms & Notifications (1 týden)

**Aktivita:**
- ✅ Alarm generation rules
- ✅ Severity levels (CRITICAL, HIGH, MEDIUM, LOW)
- ✅ Acknowledgment workflow
- ✅ Email notifications
- ✅ WebSocket push
- ✅ Alarm history

**Deliverables:**
```
POST   /api/alarms/{id}/acknowledge
GET    /api/alarms?severity=CRITICAL
GET    /ws/alarms                         - Live alarm stream

Email/SMS integration ready
```

---

### v0.8 - Audit Trail & Logging (1 týden)

**Aktivita:**
- ✅ Audit middleware (log every action)
- ✅ Immutable log storage
- ✅ User action tracking
- ✅ Data change history
- ✅ Compliance reporting

**Deliverables:**
```
GET    /api/audit/log?entity_type=BATCH&entity_id=123
GET    /api/audit/actions?user_id=5&date_from=...&date_to=...
```

---

### v0.9 - QA & Testing (2 týdny)

**Aktivita:**
- ✅ Unit test suite (>80% coverage)
- ✅ Integration tests
- ✅ Load testing (100+ concurrent users)
- ✅ Security testing
- ✅ UAT preparation

**Testování:**
- API contract testing
- Database integrity
- Performance benchmarks
- User acceptance scenarios

---

### **RELEASE v1.0** (Konec června 2026)

**Status: PRODUCTION READY**

**Obsahuje:**
✅ Core MES functionality
✅ Real-time monitoring
✅ Material traceability
✅ OPC UA integration
✅ RBAC & audit trail
✅ API v1 (stable)
✅ Documentation

**Not included:**
- ERP integration (v1.1+)
- Desktop app (v1.2+)
- Advanced dashboards (v1.1+)

---

## 📊 PHASE 1: Extended Features (v1.1 - v1.3) | Červenec-Září 2026

### v1.1 - ERP Integration (2 týdny)

**Aktivita:**
- ✅ ERP adapter framework
- ✅ Navision/SAP integration
- ✅ Manufacturing order sync
- ✅ Recipe sync
- ✅ Completion reporting
- ✅ Error handling & retries

**Deliverables:**
```
POST   /api/erp/sync/manufacturing-orders
PUT    /api/batches/{id}/sync-to-erp
```

---

### v1.2 - Dashboards & KPI (1.5 týdne)

**Aktivita:**
- ✅ Manufacturing dashboard
- ✅ KPI calculations (OEE, productivity)
- ✅ Real-time charts
- ✅ Historic reports
- ✅ Export to PDF/Excel

**KPI:**
- Overall Equipment Effectiveness (OEE)
- Cycle time
- Defect rate
- Equipment uptime

---

### v1.3 - Desktop App v1.0 (2 týdny)

**Aktivita:**
- ✅ WPF / WinUI project setup
- ✅ Batch listing & details
- ✅ Barcode scanner integration
- ✅ Offline mode
- ✅ Notifications
- ✅ Windows desktop deployment

**Features:**
- View assigned batches
- Scan materials
- Report issues
- Real-time updates

---

### **RELEASE v2.0** (Konec září 2026)

**Production deployment readiness**

---

## 🔮 PHASE 2: Advanced Features (v2.0+) | Říjen 2026+

### v2.1 - Advanced Analytics & Reporting

**Aktivita:**
- Predictive maintenance
- Trend analysis
- Customizable reports
- Data exporty pro AI/ML

---

### v2.2 - Multi-site Support

**Aktivita:**
- Centralized dashboard
- Cross-site reporting
- Federated authentication
- Data federation

---

### v2.3 - Compliance & Validation

**Aktivita:**
- 21 CFR Part 11 (FDA)
- IEC 61511 (Safety)
- ISO certifications

---

## 📐 Architecture Timeline

```
Q2 2026                    Q3 2026                 Q4 2026+
│                          │                       │
├─ Basic API Layer         ├─ Advanced APIs        ├─ GraphQL API
├─ Monolithic (single svc) ├─ Async processing     ├─ Microservices
├─ MariaDB only            ├─ Redis cache (core)   ├─ Data warehouse
│                          ├─ Elasticsearch        ├─ Advanced caching
│                          │                       │
└─ Kubernetes              └─ Auto-scaling         └─ Multi-region
   (single cluster)           (better tuning)         (DR setup)
```

---

## 🎯 Success Metrics

### Release v1.0
- ✅ Zero critical bugs
- ✅ 99.5% uptime (pilot 2 týdny)
- ✅ <500ms API response time (p95)
- ✅ 100 concurrent users without degradation
- ✅ Customer (ASSORTIS) sign-off

### Release v2.0
- ✅ ERP integration working
- ✅ Desktop app deployed to production PCs
- ✅ 99.95% uptime SLA met
- ✅ Customer-specific features implemented
- ✅ Prepared for multi-site deployment

---

## 🚀 Deployment Timeline

```
Timeline            Environment        Batch Size    Rollback Plan
─────────────────────────────────────────────────────────────
Week 1-4           DEV                Manual        Instant (git revert)
Week 5-8           STAGING            Manual        Instant
Week 9-10          PRODUCTION (pilot)  1 line       Instant (blue-green)
Week 11-12         PRODUCTION (expand) 2-3 lines    Instant
Q3 2026+           PROD (all)          All lines    Rolling (5% segments)
```

---

## 💼 Resource Planning

### Team Structure (v1.0)

```
Core Service (.NET)   - 2 developers (1 senior, 1 junior)
Desktop apps          - 1 Windows UI developer
DevOps/Infrastructure  - 1 DevOps engineer
QA/Testing            - 1 tester
Product Manager       - 1 (shared)
Architect/Tech Lead   - 1 (part-time consultant)
```

### Estimation

| Phase | Duration | Effort | Team Size |
|-------|----------|--------|-----------|
| v0.1-v0.5 | 3 weeks | 120 man-days | 5 |
| v0.6-v1.0 | 3 weeks | 120 man-days | 5 |
| v1.1-v2.0 | 2 weeks | 80 man-days | 4 |
| **Total** | **8 weeks** | **320 man-days** | **5 avg** |

---

**Datum vytvoření:** 10.4.2026  
**Status:** Iniciace projektu  
**Verze:** 0.1
