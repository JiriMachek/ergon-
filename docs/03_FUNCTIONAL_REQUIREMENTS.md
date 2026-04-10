# 03 - Funkční požadavky | ergon+ MES

## 📌 Definice funkčních požadavků

Detailní specifikace všech funkcionalit, které musí systém poskytovat.

---

## 🏭 1. Modul ŘÍZENÍ VÝROBY (Production Management)

### FR-PM-01: Vytváření a správa šarží (Batches)

**Popis:** Systém umožňuje vytváření, editaci a sledování výrobních šarží.

**Požadavky:**
- ✅ Vytvoření nové šarže z výrobní objednávky
- ✅ Přiřazení receptury (BOM) ke šarži
- ✅ Přidělení šarže na konkrétní výrobní linku
- ✅ Nastavení priorit a termínů
- ✅ Sledování stavu šarže (Neplánovaná → Plánovaná → Spuštěná → Pozastavená → Hotova → Uzavřena)
- ✅ Historii všech změn stavu
- ✅ Manuální přesun šarže mezi linkami

**Příslušné role:** PLANNER, SUPERVISOR

**API Endpoints:**
```
POST   /api/batches                      - Vytvoření nové šarže
GET    /api/batches/{id}                 - Detaily šarže
PUT    /api/batches/{id}                 - Aktualizace šarže
POST   /api/batches/{id}/assign-line    - Přidělení na linku
GET    /api/batches/search               - Hledání šarží
```

---

### FR-PM-02: Plánování výroby (Scheduling)

**Popis:** Modul pro plánování a optimalizaci výrobních kapacit.

**Požadavky:**
- ✅ Vizuální plánování (Gantt chart) pro linky
- ✅ Automatická alokace šarží na linky dle disponibility
- ✅ Detekce konfliktů a upozornění
- ✅ Manuální přeplánování drag-and-drop
- ✅ Simulace vlivů změn
- ✅ Export plánu do PDF
- ✅ Synchronizace s ERP objednávkami

**Příslušné role:** PLANNER, SUPERVISOR

---

### FR-PM-03: Sledování progresu výroby (Real-time Tracking)

**Popis:** Real-time monitoring stavu všech aktivních šarží.

**Požadavky:**
- ✅ Dashboard zobrazující stav všech šarží
- ✅ Detailní progress bar pro jednotlivé šarže
- ✅ Estim. čas do konce
- ✅ Varování při překročení času
- ✅ Zobrazení aktuálního kroku procesu
- ✅ Live notifikace změn stavu
- ✅ Historie operací pro každou šarži

**Příslušné role:** OPERATOR, SUPERVISOR

---

## 📦 2. Modul SPRÁVA MATERIÁLŮ (Material Management)

### FR-MM-01: Sledování materiálů (Traceability)

**Popis:** Úplné sledování materiálů od vstupu do výroby po výstup.

**Požadavky:**
- ✅ Skenování materiálu čárových kódem (QR/Barcode)
- ✅ Evidence vstupu materiálu do šarže
- ✅ Sledování jednotlivých komponent v recepturě
- ✅ Historie kde a kdy byl materiál použit
- ✅ Asociace materiálu s konkrétní šarží a linkou
- ✅ Detekce a varování duplicitních skenů
- ✅ Podpora více měrných jednotek (kg, m, kus atd.)

**Příslušné role:** OPERATOR

**API Endpoints:**
```
POST   /api/materials/scan               - Skenování materiálu
GET    /api/materials/{materialId}/trace - Traces materiálu
```

---

### FR-MM-02: Správa skladu a stavu materiálů

**Popis:** Vedení inventáře a dostupnosti materiálů.

**Požadavky:**
- ✅ Evidencia stavu skladu
- ✅ Automatická hlášení o nedostatku materiálu
- ✅ Rezevace materiálů pro šarže
- ✅ Historie pohybů materiálu
- ✅ Integrace s ERP skladem
- ✅ Varování o vypršení lhůty použitelnosti (shelf-life)

**Příslušné role:** PLANNER, SUPERVISOR

---

## ⚙️ 3. Modul RECEPTURY A PROCESY (Recipes & Processes)

### FR-RP-01: Správa receptur (BOM Management)

**Popis:** Vedení receptur (Bill of Materials) a procesních instrukcí.

**Požadavky:**
- ✅ Vytváření a editace receptur
- ✅ Definice jednotlivých kroků procesu
- ✅ Parametry každého kroku (teplota, čas, tlak atd.)
- ✅ Toleranční rozmezí pro parametry
- ✅ Verzionování receptur
- ✅ Schválení receptur (ADMIN approval workflow)
- ✅ Historii verzí receptur
- ✅ Vazba na ERP - stahování receptur z ERP

**Příslušné role:** TECHNICIAN, ADMIN

**Struktura receptury:**
```json
{
  "id": "RCP001",
  "name": "Výroba plastového dílu A100",
  "version": "2.1",
  "status": "APPROVED",
  "materials": [
    {
      "materialId": "MAT001",
      "quantity": 5.2,
      "unit": "kg",
      "vendorId": "VEN001"
    }
  ],
  "steps": [
    {
      "stepId": 1,
      "name": "Ohřev",
      "duration": 300,
      "parameters": {
        "temperature": { "value": 180, "min": 175, "max": 185, "unit": "°C" }
      }
    }
  ]
}
```

---

### FR-RP-02: Procesní toky (Workflow Management)

**Popis:** Definice a řízení procesních toků.

**Požadavky:**
- ✅ Definice stavů (states)
- ✅ Přechody mezi stavy (transitions)
- ✅ Podmínky pro přechody
- ✅ Automatické akce po přechodu
- ✅ Ruční schválení některých přechodů
- ✅ Paralelní procesy (kdy lze postupy seskupovat)

---

## 🔌 4. Modul INTEGRACE (Integration & Connectivity)

### FR-INT-01: OPC UA integrace (PLC Communication)

**Popis:** Propojení s PLC přes OPC UA protokol.

**Požadavky:**
- ✅ Čtení hodnot senzorů z PLC v reálném čase
- ✅ Zápis hodnot do PLC (zadání parametrů)
- ✅ Automatické logování hodnot do databáze
- ✅ Detekce ztráty spojení a reconnect
- ✅ Mapování OPC UA tagů na interní identifikátory
- ✅ Přístup k historii OPC UA (archived data)
- ✅ Sumarizace dat (průměry, min/max za interval)

**Příslušné role:** ADMIN, TECHNICIAN

---

### FR-INT-02: ERP integrace

**Popis:** Synchronizace dat s ERP systémem (objednávky, materiály, receptury).

**Požadavky:**
- ✅ Stahování výrobních objednávek z ERP
- ✅ Odesílání stavu výroby zpět do ERP
- ✅ Synchronizace receptur z ERP (pokud jsou tam vedeny)
- ✅ Synchronizace informací o materiálech
- ✅ Reporting hotových šarží do ERP
- ✅ Handling chyb a retrys
- ✅ Audit trail integrací

**Příslušné role:** ADMIN

**Vč. podpory:**
- SAP
- Navision (Dynamics 365 F&O)
- Odoo
- Vlastní XML/JSON API

---

### FR-INT-03: Desktop aplikace integrace

**Popis:** Komunikace s desktopovými aplikacemi operátorů nainstalovanými na výrobních PC.

**Požadavky:**
- ✅ Push notifikace o změnách stavu
- ✅ Offline mode - práce bez sítě
- ✅ Synchronizace dat při obnovení spojení
- ✅ Autentizace a role-based přístup
- ✅ Skenování čárových kódů z desktopového zařízení

---

## 📊 5. Modul REPORTING & ANALYTICS

### FR-RA-01: Dashboards a KPI

**Popis:** Vizualizace klíčových metrik výroby.

**Požadavky:**
- ✅ Předdefinované dashboards (Production Overview, Equipment, Quality)
- ✅ Vlastní dashboards (user-configurable)
- ✅ Real-time grafy a metriky
- ✅ Drill-down do detailů
- ✅ Porovnání plánu vs. skutečnost
- ✅ Filtrování po linence, datu, produktu
- ✅ Export do PDF a Excel

**KPI metriky:**
- OEE (Overall Equipment Effectiveness)
- Produktivita (jednotek/hodinový)
- Takt čas (plánovaný vs. skutečný)
- Počet zmetků
- Dostupnost linky (uptime)
- Kvalita výstupu

---

### FR-RA-02: Historické reporty

**Popis:** Generování sestav za konkrétní období.

**Požadavky:**
- ✅ Report "Výroba za období"
- ✅ Report "Kvalita šarží"
- ✅ Report "Zemětiny a chyby"
- ✅ Report "Spotřeba materiálů"
- ✅ Report "Efektivita linek"
- ✅ Export do PDF, Excel, CSV
- ✅ Plánované reporty (zasílání e-mailem)

---

## 🔐 6. Modul BEZPEČNOST & COMPLIANCE

### FR-SEC-01: Autentizace a autorizace

**Popis:** Řízení přístupu k systému.

**Požadavky:**
- ✅ Login s emailem a heslem
- ✅ Multi-factor authentication (MFA) pro kritické role
- ✅ SSO integrace (Azure AD, LDAP)
- ✅ Role-based access control (RBAC)
- ✅ Session management a timeout
- ✅ Automatické lockouty po neúspěšných pokusech

**Příslušné role:** ADMIN

---

### FR-SEC-02: Audit Trail

**Popis:** Zaznamenávání všech akcí v systému.

**Požadavky:**
- ✅ Každá akce je zaznamenána: KDO, KDY, CO, JAKÁ DATA
- ✅ Nelze mazat audit logy (immutable)
- ✅ Detekce anomálií a podezřelých aktivit
- ✅ Export audit logu
- ✅ Compliance report generování

**Zaznamenávány akce:**
- Přihlášení/Odhlášení
- Vytváření/Editace šarží
- Změny stavu
- Skenování materiálů
- Přistoupení k ERP datům
- Změny konfigurací

**Příslušné role:** AUDITOR, ADMIN

---

### FR-SEC-03: Bezpečnost dat

**Popis:** Ochrana dat v přenosu a v klidovém stavu.

**Požadavky:**
- ✅ HTTPS/SSL pro veškerou komunikaci
- ✅ Šifrování hesel (bcrypt)
- ✅ Šifrování citlivých polí v DB (AES-256)
- ✅ Zálohování dat a disaster recovery
- ✅ Autorizace na úrovni dat (user vidí jen svá data)
- ✅ GDPR compliance

---

## 🚨 7. Modul ALARMY & NOTIFIKACE

### FR-ALM-01: Systém alarmů

**Popis:** Hlášení problémů a deviacích v procesu.

**Požadavky:**
- ✅ Detekce exceedanc parametrů
- ✅ Kvalitativní alarmy (např. detekce zmetku)
- ✅ Operátor-aktivované alarmy
- ✅ Váhování alarmů (kritický, varování, info)
- ✅ Eskalace (pokud neřešeno do N minut → nahlásit nadřízenému)
- ✅ Notifikace (email, SMS, push, zvuk)
- ✅ Historia alarmů

**Příslušné role:** OPERATOR, SUPERVISOR

---

## � 8. Modul DESKTOP APLIKACE

### FR-DESK-01: Desktop aplikace pro operátory

**Popis:** Windows desktop aplikace nainstalovaná na výrobních počítačích pro práci v hale.

**Požadavky:**
- ✅ Dashboard s aktuálními šaržemi
- ✅ Detail šarže s instrukcemi
- ✅ Skenování čárových kódů
- ✅ Reportování problémů
- ✅ Potvrzování kroků
- ✅ Offline mode a synchronizace
- ✅ Notifikace

**Platforma:** WPF / WinUI (Windows desktop)

---

## 📋 Prioritizace funkcionalit

| Modul | Funkcionalita | Priority | Release |
|-------|---------------|----------|---------|
| PM | Batch Management | 🔴 CRITICAL | v1.0 |
| PM | Scheduling | 🔴 CRITICAL | v1.0 |
| PM | Real-time Tracking | 🔴 CRITICAL | v1.0 |
| MM | Material Traceability | 🟠 HIGH | v1.0 |
| RP | Recipe Management | 🟠 HIGH | v1.0 |
| INT | OPC UA | 🟠 HIGH | v1.0 |
| INT | ERP Integration | 🟠 HIGH | v1.1 |
| RA | Dashboards | 🟠 HIGH | v1.1 |
| SEC | RBAC + Auth | 🟠 HIGH | v1.0 |
| SEC | Audit Trail | 🟡 MEDIUM | v1.0 |
| ALM | Alarms | 🟡 MEDIUM | v1.1 |
| DESK | Desktop App | 🟡 MEDIUM | v1.2 |

---

**Datum vytvoření:** 10.4.2026  
**Status:** Iniciace projektu  
**Verze:** 0.1
