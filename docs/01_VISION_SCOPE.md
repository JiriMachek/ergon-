# 01 - Vision & Scope | ergon+ MES

## 🎯 Vize

**ergon+ MES** je moderní Manufacturing Execution System určený pro:
- Střední a větší výrobní podniky s komplexními procesy
- Datově-driven řízení výroby v reálném čase
- Úplná sledovatelnost (traceability) výrobních procesů
- Inteligentní integrace s OPC UA/PLC a ERP systémy
- Dlouhodobý vývoj s modulární architekturou

## 📋 Definice rozsahu (Scope)

### Zahrnuto (In Scope)
✅ Řízení výrobních procesů (dávková + kontinuální výroba)  
✅ Sledování šarží (Batch tracking)  
✅ Materiální toky a stavy skladů  
✅ Propojení s PLC/SCADA přes OPC UA  
✅ Integrace s ERP (Navision, SAP, Odoo apod.)  
✅ Role-based access control (RBAC)  
✅ Audit trail a compliance  
✅ Real-time dashboards & KPI reporting  
✅ Windows desktop aplikace pro výrobní PC (WPF / WinUI)  

### Nezahrnuto (Out of Scope)
❌ Vlastní SCADA systém (používáme existující)  
❌ Finanční modul (řeší ERP)  
❌ Lidské zdroje (řeší HR systém)  
❌ Vlastní IoT hardware (integrujeme existující)  

## 🎯 Primární cíle

| Cíl | Metrika | Termín |
|-----|---------|--------|
| **Řízení výroby v reálném čase** | 95% dostupnost, <2s latence | Q3 2026 |
| **Úplná sledovatelnost** | 100% šarží s históí operací | Q3 2026 |
| **Integraci s OPC UA** | 2+ PLC připojených | Q2 2026 |
| **ERP integrace** | Synchronizace receptur, objednávek | Q4 2026 |
| **Mobilní přístup** | Android app pro halové operace | Q4 2026 |
| **Compliance & audit** | Explicitní audit trail všech akcí | Q3 2026 |

## 👥 Cílové skupiny (Stakeholders)

| Role | Potřeby | Priorita |
|------|---------|----------|
| **Výrobní plánovací** | Plánování, přidělování šarží na linky | 🔴 Kritická |
| **Operátor haly** | Dashboard, hlášení problémů, sledování progresu | 🔴 Kritická |
| **Technolog** | Správa receptur, parametrů procesu | 🟠 Vysoká |
| **Vedení výroby** | KPI, zprávy, analýza výroby | 🟠 Vysoká |
| **IT/DevOps** | Nasazení, monitoring, bezpečnost | 🟡 Střední |
| **Auditor** | Compliance, historie, audit trail | 🟡 Střední |

## 🏗️ Architektonické principy

- **Modulární design** - komponenty nezávislé, lze nasadit odd. nebo všechny
- **Real-time capable** - synchronní API, event-driven architecture
- **Cloud-native ready** - Docker, Kubernetes, auto-scaling
- **Data-driven** - vše je měřitelné a analyzovatelné
- **Security first** - autentizace, autorizace, šifrování, audit
- **Interoperabilita** - standard protokoly (OPC UA, REST/JSON, MQTT)

## 📊 Očekávaný přínos

- ⏱️ Zkrácení času cyklu výroby o 15-20%
- 📉 Snížení zmetků o 10-15%
- 🔍 Zvýšená viditelnost výrobních procesů
- 📱 Rychlější rozhodování operačního týmu
- 🛡️ Splnění compliance a regulačních požadavků
- 💰 Lepší plánování zdrojů a optimalizace nákladů

---

**Datum vytvoření:** 10.4.2026  
**Status:** Iniciace projektu  
**Verze:** 0.1
