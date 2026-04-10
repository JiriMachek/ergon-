# ergon+ MES

**ergon+ MES** je modulární Manufacturing Execution System (MES)  
určený pro řízení výroby, sledování šarží, materiálové toky  
a integraci s PLC, SCADA a ERP systémy.

Projekt je navržen jako vícerozměrná platforma využitelná
u více zákazníků s možností konfigurace.

---

## 🎯 Cíle projektu
- řízení a sledování výrobních procesů (Batch / kontinuální výroba)
- sledovatelnost materiálů a šarží (Traceability)
- integrace s PLC (OPC UA) a ERP systémem
- audit, role a bezpečnost
- modulární architektura pro dlouhodobý rozvoj

---

## 📚 Kompletní Dokumentace ⭐

**Všechna dokumentace je v `/docs` složce:**

| Dokument | Čtěte | Obsah |
|----------|-------|-------|
| **[01_VISION_SCOPE.md](docs/01_VISION_SCOPE.md)** | Product Managers | Vize, cíle, scope, benefits |
| **[02_ROLES_PERMISSIONS.md](docs/02_ROLES_PERMISSIONS.md)** | Vedoucí IT, Security | 8 rolí, RBAC matice, autorizace |
| **[03_FUNCTIONAL_REQUIREMENTS.md](docs/03_FUNCTIONAL_REQUIREMENTS.md)** | Vývojáři, BA | 8 modulů, 30+ funkcionalit, API |
| **[04_PROCESS_FLOWS.md](docs/04_PROCESS_FLOWS.md)** | Vývojáři, QA | 6 procesů s diagramy, state machine |
| **[05_DATA_MODEL.md](docs/05_DATA_MODEL.md)** | Backend dev | ER diagram, SQL schema, indexy |
| **[06_SYSTEM_INTEGRATION.md](docs/06_SYSTEM_INTEGRATION.md)** | Backend dev | OPC UA, ERP, Desktop, Webhooks |
| **[07_NON_FUNCTIONAL_REQUIREMENTS.md](docs/07_NON_FUNCTIONAL_REQUIREMENTS.md)** | DevOps, Arch | Výkon, bezpečnost, scalability, monitoring |
| **[08_ROADMAP.md](docs/08_ROADMAP.md)** | Vedení, PM | Timeline, fáze, ressources, deployment |
| **[AI_PROMPTS.md](docs/AI_PROMPTS.md)** | Vývojáři | **10 AI promptů pro generování kódu!** |

👉 **Všechna dokumentace je zdrojem pravdy pro vývoj.**

---

## 🏗️ Architektura (v plánu)
- Core service: .NET Worker Service / Windows Service (Clean Architecture)
- Databáze: MariaDB / MySQL
- Architektura: Clean Architecture (DDD light)
- Integrace: OPC UA (Real-time PLC), service API, MQTT
- Autentizace: Windows authentication + JWT / AD integration
- Frontend: Windows desktop aplikace (WPF / WinUI)
- Desktop: Primární UI platforma; mobilní rozhraní volitelně později
- DevOps: Windows deployment, build pipeline
- Monitoring: Prometheus + Grafana, ELK stack

---

## 🚀 Quick Start - Kde začít?

### 1️⃣ **Pochopte projekt** (15 minut)
```bash
Přečtěte: docs/01_VISION_SCOPE.md
```

### 2️⃣ **Prostudujte klíčové procesy** (30 minut)
```bash
Přečtěte: docs/04_PROCESS_FLOWS.md
```

### 3️⃣ **Pro vývoj: Použijte AI prompty** ⭐
```bash
Přečtěte: docs/AI_PROMPTS.md
# Pak zkopírujte prompty a pošlete je AI modeling (ChatGPT, Claude, etc.)
```

### 4️⃣ **Setup vývoje** (TBD)
```bash
# Bude k dispozici s fází Foundation (v0.1)
```

---

## 📋 Klíčové komponenty

### Frontend
- Dashboard (Windows desktop aplikace - WPF / WinUI)
- Specializované desktop klientské aplikace pro operátory, plánovače a techniky
- Real-time monitoring

### Desktop
- Push notifications, offline sync

### Backend
- Background process service / Windows Service (.NET Worker)
- Service API for desktop applications
- OPC UA client (PLC communication)
- ERP integrations (Navision, SAP, Odoo)
- Job scheduler (batch processing)

### Databáze
- Primary: MariaDB
- Cache: Redis
- Archive: Cloud storage

### Integrace
- **OPC UA**: Real-time data z PLC
- **ERP**: Manufacturing orders, recipes, inventory
- **Desktop**: Push notifications, offline sync
- **Email/SMS**: Alarms a notifikace

---

## 🎯 Development Roadmap

```
Q2 2026           Q3 2026              Q4 2026+
├─ v0.1 - v1.0   ├─ v1.1 - v2.0       ├─ v2.1+
├─ Foundation    ├─ Extended Features ├─ Advanced
└─ Core MES      └─ Integrations      └─ AI/ML
```

**Detailní plán:** [docs/08_ROADMAP.md](docs/08_ROADMAP.md)

---

## 👥 Team & Contributing

**Aktuální status:** 🚀 Iniciace projektu (Duben 2026)

- **Backend Lead**: [TBD]
- **Frontend Lead**: [TBD]
- **DevOps**: [TBD]
- **QA Lead**: [TBD]

📝 **Contributing Guide:** [CONTRIBUTING.md](CONTRIBUTING.md)

---

## 🔐 Bezpečnost

- RBAC (Role-Based Access Control) s 8 rolemi
- Multi-factor authentication (MFA) pro admin
- JWT + OAuth2 / OpenID Connect support
- Audit trail - logujem všechny akce
- GDPR compliance
- 21 CFR Part 11 (FDA) ready

**Detaily:** [docs/02_ROLES_PERMISSIONS.md](docs/02_ROLES_PERMISSIONS.md) & [docs/07_NON_FUNCTIONAL_REQUIREMENTS.md](docs/07_NON_FUNCTIONAL_REQUIREMENTS.md#sec-01-autentizace)

---

## 📊 KPI & Metriky

Systém trackuje:
- **OEE** (Overall Equipment Effectiveness)
- **Cycle Time** (plán vs. skutečnost)
- **Defect Rate** (počet zmetků)
- **Equipment Uptime** (dostupnost linky)
- **Productivity** (jednotek/hodina)

---

## 🧪 Testing Strategy

- Unit tests: **>80% coverage** (xUnit)
- Integration tests: Všechny workflow
- E2E tests: Klíčové scénáře
- Load tests: 100+ concurrent users
- Security testing: OWASP top 10

---

## 📞 Support & Questions

- **Issues**: [GitHub Issues](https://github.com/assortis/ergon-mes/issues)
- **Documentation**: [docs/](docs/)
- **Slack**: #ergon-mes (interní)

---

## 📈 Success Metrics

| Metrika | Target | Status |
|---------|--------|--------|
| v1.0 Release | Červen 2026 | 🔵 V plánu |
| API Response Time (p95) | <500ms | ⏳ TBD |
| Uptime SLA | 99.95% | ⏳ TBD |
| Code Coverage | ≥80% | ⏳ TBD |
| Documentation Quality | 100% | ✅ V toku |

---

## 📄 License

[TBD - bude upřesněno]

---

## 🙏 Acknowledgments

- ASSORTIS Electric s.r.o.
- Control Systems team
- Všem přispěvatelům

---

**Projekt iniciován:** 10. dubna 2026  
**Status:** 🚀 Iniciace & Dokumentace  
**Verze:** 0.1

---

## 🚧 Stav projektu
🟡 Fáze: **Analýza / návrh**
- probíhá tvorba zadávací dokumentace
- vývoj kódu zatím nezahájen

---

## 📌 Poznámka
Tento repozitář je určen pro interní vývoj.
Obsah dokumentace i kódu je důvěrný.
``