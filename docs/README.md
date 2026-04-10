# 📚 Dokumentace | ergon+ MES

Kompletní dokumentace pro vývoj moderního Manufacturing Execution System (MES) pro řízení výroby.

## 📋 Obsah dokumentace

### 1. **[01_VISION_SCOPE.md](01_VISION_SCOPE.md)** - Vize a scope projektu
   - Vize a cíle systému
   - Primární cíle a metriky
   - Cílové skupiny (stakeholders)
   - Architektonické principy
   - Očekávaný přínos

### 2. **[02_ROLES_PERMISSIONS.md](02_ROLES_PERMISSIONS.md)** - Role a oprávnění
   - Definice 8 hlavních rolí (ADMIN, PLANNER, OPERATOR, apod.)
   - Matice oprávnění
   - RBAC implementace
   - API endpoint autorizace

### 3. **[03_FUNCTIONAL_REQUIREMENTS.md](03_FUNCTIONAL_REQUIREMENTS.md)** - Funkční požadavky
   - 8 modulů: Production Mgmt, Material Mgmt, Recipes, Integration, Reporting, Security, Alarms, Desktop App
   - Detailní specifikace všech funkcionalit
   - API endpointy
   - Prioritizace funkcionalit (v1.0 vs. v1.1+)

### 4. **[04_PROCESS_FLOWS.md](04_PROCESS_FLOWS.md)** - Procesní toky
   - 6 klíčových procesů s ASCII diagramy:
     1. Vytváření a spuštění šarže
     2. Monitoring během výroby
     3. Ukončení a zavření šarže
     4. Materiálový tok a traceability
     5. Integrace s ERP
     6. OPC UA data flow
   - Státní stroj (Batch States)

### 5. **[05_DATA_MODEL.md](05_DATA_MODEL.md)** - Datový model
   - Entity-Relationship diagram
   - Detailní schéma všech tabulek (SQL)
   - Vztahy a constraints
   - Indexování a optimalizace
   - Redis caching strategie

### 6. **[06_SYSTEM_INTEGRATION.md](06_SYSTEM_INTEGRATION.md)** - Systémová integrace
   - OPC UA integrace (PLC communication) - Real-time monitoring
   - ERP integrace (Navision, SAP, Odoo) - Synchronizace dat
   - Desktop aplikace integrace - Service API
   - Webhooks - Informování externích systémů
   - Error handling & resilience

### 7. **[07_NON_FUNCTIONAL_REQUIREMENTS.md](07_NON_FUNCTIONAL_REQUIREMENTS.md)** - Nefunkční požadavky
   - Clean Architecture
   - Performance požadavky (response time, throughput)
   - Bezpečnostní požadavky (autentizace, šifrování, audit)
   - Scalability & Availability (99.95% SLA)
   - Monitoring & Observability
   - Testovací strategie

### 8. **[08_ROADMAP.md](08_ROADMAP.md)** - Technická roadmapa
   - Fáze vývoje (v0.1 → v1.0 → v2.0 → v3.0)
   - Timeline: Q2 2026 (4 měsíce)/Phase 0, Q3 2026 (2 měsíce)/Phase 1, Q4 2026+/Phase 2
   - Detailní plán jednotlivých verzí
   - Deployment strategie (Blue-Green)
   - Resource plánování

### 9. **[AI_PROMPTS.md](AI_PROMPTS.md)** - **AI prompty pro generování kódu** ⭐
   - 10 detailních promptů pro AI:
     1. Entity & Domain Layer
     2. API Controllers & DTOs
     3. Application Services & Use Cases
     4. Database Access Layer (EF Core)
     5. OPC UA Integration Service
     6. ERP Integration Service
     7. Authentication & Authorization
     8. Audit Trail & Logging
     9. Unit & Integration Testing
     10. Windows Service & Deployment
   - Code style & standards
   - Validation checklist
   - Quick reference

---

## 🎯 Jak používat tuto dokumentaci

### 👨‍💼 **Pro Product Manager / Vedoucí projektu**
Čtěte: **Vision_Scope** → **Functional_Requirements** → **Roadmap**

### 👨‍💻 **Pro vývojáře (Backend)**
Čtěte: **Functional_Requirements** → **Data_Model** → **System_Integration** → **AI_PROMPTS**

### 🏗️ **Pro DevOps / Infrastrukturu**
Čtěte: **Non_Functional_Requirements** → **Roadmap** → **AI_PROMPTS** (Docker part)

### 🔐 **Pro Security / Compliance**
Čtěte: **Roles_Permissions** → **Non_Functional_Requirements** (Security & Audit)

### � **Pro Desktop vývojáře**
Čtěte: **System_Integration** (Desktop API) → **Functional_Requirements** (Module 8)

### 🧪 **Pro QA / Testers**
Čtěte: **Process_Flows** → **Functional_Requirements** → **AI_PROMPTS** (Testing)

---

## 🚀 Quick Start

1. **Projděte Vision_Scope** - pochopte, co budujete
2. **Prostudujte Process_Flows** - pochopte, jak systém funguje
3. **Zaměřte se na Functional_Requirements** pro vaši roli
4. **Použijte AI_PROMPTS** pro generování konkrétního kódu
5. **Ověřte Data_Model** pro datové struktury

---

## 🔗 Klíčové odkazy

- **Datový model**: [05_DATA_MODEL.md](05_DATA_MODEL.md)
- **API specifikace**: [03_FUNCTIONAL_REQUIREMENTS.md](03_FUNCTIONAL_REQUIREMENTS.md) + [06_SYSTEM_INTEGRATION.md](06_SYSTEM_INTEGRATION.md)
- **Procesní toky**: [04_PROCESS_FLOWS.md](04_PROCESS_FLOWS.md)
- **Bezpečnost & Audit**: [02_ROLES_PERMISSIONS.md](02_ROLES_PERMISSIONS.md) + [07_NON_FUNCTIONAL_REQUIREMENTS.md](07_NON_FUNCTIONAL_REQUIREMENTS.md)
- **Vývojářské prompty**: [AI_PROMPTS.md](AI_PROMPTS.md)

---

## 📊 Status dokumentace

| Dokument | Status | Verze | Datum |
|----------|--------|-------|-------|
| Vision & Scope | ✅ Complete | 0.1 | 10.4.2026 |
| Roles & Permissions | ✅ Complete | 0.1 | 10.4.2026 |
| Functional Requirements | ✅ Complete | 0.1 | 10.4.2026 |
| Process Flows | ✅ Complete | 0.1 | 10.4.2026 |
| Data Model | ✅ Complete | 0.1 | 10.4.2026 |
| System Integration | ✅ Complete | 0.1 | 10.4.2026 |
| Non-Functional Req. | ✅ Complete | 0.1 | 10.4.2026 |
| Roadmap | ✅ Complete | 0.1 | 10.4.2026 |
| AI Prompts | ✅ Complete | 0.1 | 10.4.2026 |

---

## 📝 Revizní poznámky

**v0.1 (10.4.2026)** - Iniciální vytvoření dokumentace pro start vývoje

---

**Poslední aktualizace:** 10.4.2026  
**Verze:** 0.1  
**Udržovatel:** Projekt ergo+ MES
