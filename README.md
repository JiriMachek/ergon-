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

## 📚 Dokumentace
Kompletní zadávací a technická dokumentace je uložena ve složce `/docs`:

1. Vision & Scope
2. Role a oprávnění
3. Funkční požadavky
4. Procesní toky
5. Datový model
6. Integrace (PLC / ERP)
7. Nefunkční požadavky
8. Roadmapa

👉 Dokumentace v `/docs` je **zdroj pravdy** pro vývoj.

---

## 🧱 Architektura (v plánu)
- Backend: ASP.NET Core
- Databáze: MariaDB / MySQL
- Architektura: Clean Architecture (DDD light)
- Integrace: OPC UA, REST API
- Autentizace: role-based + audit trail

---

## 👥 Vývoj a spolupráce
- repozitář je **private**
- vývoj probíhá přes **Issues + Pull Requests**
- hlavní větev: `main`
- feature větve: `feature/<název>`

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