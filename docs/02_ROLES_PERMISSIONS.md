# 02 - Role a oprávnění | ergon+ MES

## 🔐 Systém autorizace

Systém používá **Role-Based Access Control (RBAC)** s granulárními oprávněními.

## 👥 Role a schopnosti

### 1. 🔧 **Administrator (ADMIN)**
**Přístup:** Veškerá konfigurační a správní oprávnění

| Oprávnění | Popis |
|-----------|--------|
| `SYSTEM_CONFIG` | Konfigurace systému, linek, strojů |
| `USER_MANAGEMENT` | Správa uživatelů a rolí |
| `RECIPE_ADMIN` | Správa a schválení receptur |
| `AUDIT_FULL` | Přístup k plnému audit logu |
| `DATA_EXPORT` | Export dat v jakýmkoliv formátu |
| `INTEGRATION_MANAGE` | Správa integrací (PLC, ERP) |
| `BACKUP_RESTORE` | Zálohování a obnova dat |

---

### 2. 📋 **Planner (SCHEDULER)**
**Přístup:** Plánování výroby, přidělování šarží

| Oprávnění | Popis |
|-----------|--------|
| `BATCH_CREATE` | Vytváření nových šarží |
| `BATCH_ASSIGN_LINE` | Přidělování šarží na linky |
| `BATCH_RESCHEDULE` | Plánování a přeplánování |
| `MO_VIEW` | Zobrazení výrobních objednávek |
| `RECIPE_VIEW` | Přístup k recepturám |
| `REPORT_SCHEDULE` | Přístup k plánům a reportům |

---

### 3. 🎯 **Operator (OPERATOR)**
**Přístup:** Řízení výroby, zadávání dat z haly

| Oprávnění | Popis |
|-----------|--------|
| `BATCH_START` | Spuštění šarže na lince |
| `BATCH_UPDATE_STATUS` | Aktualizace stavu šarže |
| `MATERIAL_SCAN` | Skenování materiálů/čárových kódů |
| `PARAMETER_LOG` | Logování procesních parametrů |
| `ISSUE_REPORT` | Hlášení problémů nebo odchylek |
| `BATCH_VIEW` | Zobrazení aktuálního stavu šarže |
| `ACKNOWLEDGE_ALARM` | Potvrzení alarmů |

---

### 4. 👀 **Supervisor (SUPERVISOR)**
**Přístup:** Dohled nad výrobou, sledování KPI

| Oprávnění | Popis |
|-----------|--------|
| `BATCH_VIEW_ALL` | Zobrazení všech šarží |
| `KPI_DASHBOARD` | Přístup k dashboardu KPI |
| `REPORT_VIEW` | Zobrazení výrobních zpráv |
| `ISSUE_RESOLVE` | Řešení hlášených problémů |
| `BATCH_PAUSE_RESUME` | Pozastavení/pokračování šarže |
| `OPERATOR_NOTIFY` | Odesílání zpráv operátorům |

---

### 5. 🧪 **Technician (TECHNICIAN)**
**Přístup:** Technická konfiguracija, údržba

| Oprávnění | Popis |
|-----------|--------|
| `RECIPE_CREATE` | Vytváření a editace receptur |
| `PARAMETER_CONFIG` | Konfigurace parametrů procesu |
| `EQUIPMENT_MAINTAIN` | Údržba informací o strojích |
| `SENSOR_CONFIG` | Konfigurace senzorů a OPC UA |
| `QUALITY_PARAM` | Nastavení kvalitativních parametrů |

---

### 6. 📊 **Analyst (ANALYST)**
**Přístup:** Analýza dat, reporting

| Oprávnění | Popis |
|-----------|--------|
| `BATCH_HISTORY_VIEW` | Historická data šarží |
| `ANALYTICS_EXPORT` | Export dat pro analýzu |
| `REPORT_CUSTOM` | Vytváření vlastních zpráv |
| `KPI_VIEW` | Zobrazení KPI a statistik |
| `TREND_ANALYSIS` | Analýza trendů výroby |

---

### 7. 🛡️ **Auditor (AUDITOR)**
**Přístup:** Compliance, audit trail

| Oprávnění | Popis |
|-----------|--------|
| `AUDIT_LOG_VIEW` | Zobrazení audit logu |
| `COMPLIANCE_CHECK` | Kontrola compliance |
| `DATA_INTEGRITY` | Ověření integrity dat |
| `REPORT_AUDIT` | Audit zprávy |

---

### 8. 🔍 **Viewer (VIEWER)**
**Přístup:** Jen pro čtení, základní přehled

| Oprávnění | Popis |
|-----------|--------|
| `READ_ONLY` | Pouze zobrazení dat |
| `DASHBOARD_VIEW` | Přístup k veřejným dashboardům |

---

## 🔀 Matice rolí a oprávnění

```
┌─────────────────┬───────┬────────┬──────────┬────────────┬──────────┬─────────┬────────┬────────┐
│ Oprávnění       │ ADMIN │ PLANNER│ OPERATOR │ SUPERVISOR │ TECHNICIAN│ ANALYST │ AUDITOR│ VIEWER │
├─────────────────┼───────┼────────┼──────────┼────────────┼──────────┼─────────┼────────┼────────┤
│ BATCH_VIEW      │  ✅   │   ✅   │    ✅    │     ✅     │    ✅    │   ✅    │   ✅   │   ✅   │
│ BATCH_CREATE    │  ✅   │   ✅   │    ✅    │     ✅     │    ❌    │   ❌    │   ❌   │   ❌   │
│ BATCH_START     │  ✅   │   ❌   │    ✅    │     ✅     │    ❌    │   ❌    │   ❌   │   ❌   │
│ RECIPE_MANAGE   │  ✅   │   ❌   │    ❌    │     ❌     │    ✅    │   ❌    │   ❌   │   ❌   │
│ KPI_VIEW        │  ✅   │   ✅   │    ❌    │     ✅     │    ❌    │   ✅    │   ✅   │   ❌   │
│ AUDIT_VIEW      │  ✅   │   ❌   │    ❌    │     ❌     │    ❌    │   ❌    │   ✅   │   ❌   │
│ SYSTEM_CONFIG   │  ✅   │   ❌   │    ❌    │     ❌     │    ❌    │   ❌    │   ❌   │   ❌   │
│ READ_ONLY       │  ❌   │   ❌   │    ❌    │     ❌     │    ❌    │   ❌    │   ❌   │   ✅   │
└─────────────────┴───────┴────────┴──────────┴────────────┴──────────┴─────────┴────────┴────────┘
```

## 🎯 Funkční role ve výrobě

Pozn: Jedna osoba může mít více rolí!

| Funkce | Doporučené role |
|--------|-----------------|
| **Vedoucí výroby** | SUPERVISOR + SCHEDULER |
| **Vedoucí technologu** | TECHNICIAN + ANALYST |
| **Vedoucí IT** | ADMIN |
| **Řadový operátor** | OPERATOR |
| **Interní auditor** | AUDITOR |
| **Data scientist** | ANALYST |

## 🔐 Bezpečnost

- Všechny akce jsou **zaznamenány do audit logu** s uživatelem a časem
- Citlivé operace vyžadují **schválení nadřízením** (SUPERVISOR/ADMIN)
- Hesla jsou **hashována** (bcrypt, SHA-256)
- Přístup je **logován** - podezřelé aktivity se hlášou
- Každá role je **audit-able** - kontrola kdo co dělal

## 📱 API Endpoints - Kontrola přístupu

```csharp
// Příklad: Kontrola oprávnění v API
[Authorize(Roles = "OPERATOR,SUPERVISOR")]
[HttpPost("/batch/{id}/start")]
public async Task<IActionResult> StartBatch(int id) { ... }

[Authorize(Policy = "CanReadAuditLog")]
[HttpGet("/audit/log")]
public async Task<IActionResult> GetAuditLog() { ... }
```

---

**Datum vytvoření:** 10.4.2026  
**Status:** Iniciace projektu  
**Verze:** 0.1
