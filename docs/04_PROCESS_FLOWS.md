# 04 - Procesní toky | ergon+ MES

## 🎯 Základní výrobnice procesy

Detailní specifikace klíčových procesů v MES systému.

---

## 📦 PROCES 1: Vytváření a spuštění výrobní šarže

```
┌─────────────────────────────────────────────────────────────────────┐
│                    BATCH CREATION & START FLOW                      │
└─────────────────────────────────────────────────────────────────────┘

┌──────────────────────────┐
│  Planner vytvoří šarži   │
│  z výr. objednávky (MO)  │
└──────────────┬───────────┘
               │
               ▼
        ┌──────────────┐
        │ Volba lékařaky  │
        │ a receptury   │
        └──────┬───────┘
               │
               ▼
     ┌─────────────────────┐
     │ Priřazení prioritu  │
     │ a termínu           │
     └──────┬──────────────┘
            │
            ▼
    ┌───────────────┐
    │ Přidělení na  │
    │ konkrétní     │
    │ výrobní linku │◄─────── Kontrola dostupnosti
    └───────┬───────┘
            │
            ▼
  ┌──────────────────────┐
  │ Příprava materiálů:  │
  │ - Skenování kódů     │
  │ - Ověření dostupnosti│
  │ - Rezevace materiálu │
  └──────┬───────────────┘
         │ Materially OK?
         │
    ┌────┴─────┐
    │ ANO       │ NE
    ▼           ▼
 ┌──────┐   ┌──────────────┐
 │ Batch│   │ ALERT!       │
 │ READY│   │ Chybí mat.   │
 │ stav │   │ → Očekávat   │
 └──┬───┘   └──────────────┘
    │
    ▼
 ┌───────────────────┐
 │ Operátor:         │
 │ Potvrd. start     │
 │ (skenuje QR)      │
 └──┬────────────────┘
    │
    ▼
 ┌──────────────────────┐
 │ BATCH: STARTED       │
 │ - Záznam v DB        │
 │ - OPC UA → linku     │
 │ - Notifikace op.     │
 └──────────────────────┘
    │
    ▼
 ┌──────────────────────┐
 │ Monitoring paralů:   │
 │ - Teplota, tlak...   │
 │ - Real-time logging  │
 │ - Alarmy/Deviace     │
 └──────────────────────┘
```

**Betrokované role:** PLANNER → OPERATOR → SUPERVISOR  
**Systémy:** ERP, OPC UA, mobilní app, databáze  
**Čas:** 15-30 minut od vytvoření po start

---

## 🔄 PROCES 2: Monitoring a kontrola během výroby

```
┌──────────────────────────────────────────────────────────┐
│         PRODUCTION MONITORING & CONTROL FLOW             │
└──────────────────────────────────────────────────────────┘

     ┌─────────────────────┐
     │ Batch STARTED       │
     │ Linka vyrábí...     │
     └──────────┬──────────┘
                │
        ┌───────┴───────┐
        │               │
        ▼               ▼
    ┌─────────┐    ┌──────────────┐
    │ Všechno │   │ DEVIACE!     │
    │ v pořád.│   │ Параметр XX: │
    │         │   │ 185°C (limit │
    │         │   │ je 180-185)  │
    └────┬────┘   └──┬───────────┘
         │           │
         │           ▼
         │       ┌─────────────────────┐
         │       │ ALARM vygenerován   │
         │       │ - Sítě notifikace   │
         │       │ - Email operátorovi │
         │       │ - Zvuk v hale       │
         │       └──┬──────────────────┘
         │          │
         │          ▼
         │      ┌──────────────────┐
         │      │ Operátor řeší:   │
         │      │ 1. Zreguluje     │
         │      │ 2. Přepne mód    │
         │      │ 3. Zavolá tech   │
         │      │ 4. Označí alarm  │
         │      │    ACK           │
         │      └──┬───────────────┘
         │         │
         │         ▼
         │   ┌────────────────┐
         │   │ Normalizace    │
         │   │ Parametr OK    │
         │   └─┬──────────────┘
         │     │
         └─────┴──────────┐
                          │
                          ▼
                   ┌──────────────┐
                   │ Batch pokr.  │
                   │ v časovém    │
                   │ plánu         │
                   └────┬─────────┘
                        │
                        ▼
                ┌──────────────────┐
                │ Logování všech   │
                │ parametrů        │
                │ (každých 10 sec) │
                │ → Databáze       │
                └───────┬──────────┘
                        │
                        ▼
                  Byl poslední krok?
                        │
                   ┌────┴─────┐
                   │ ANO      │ NE
                   ▼          ▼
              ┌─────────┐  (pokr. next step)
              │ Další   │
              │ krok?   │
              └────┬────┘
```

**Betrokované role:** OPERATOR, SUPERVISOR  
**Systémy:** OPC UA (real-time), Databáze, Notifikace  
**Frekvence:** Každých 10 sekund logování

---

## ✅ PROCES 3: Ukončení a zavření šarže

```
┌──────────────────────────────────────────────────┐
│         BATCH COMPLETION & CLOSURE FLOW          │
└──────────────────────────────────────────────────┘

    ┌──────────────────┐
    │ Poslední krok    │
    │ dokončen         │
    └────────┬─────────┘
             │
             ▼
    ┌──────────────────┐
    │ Operátor:        │
    │ Potvrd. konec    │
    │ (vizuální check) │
    └────────┬─────────┘
             │
             ▼
    ┌──────────────────┐
    │ Batch: COMPLETED │
    │ Status = HOTOVA  │
    └────────┬─────────┘
             │
    ┌────────┴───────────┐
    │                    │
    ▼                    ▼
 ┌─────────────┐   ┌──────────────────┐
 │ Výstupní    │   │ Zaznamenání      │
 │ kontrola    │   │ všech parametrů  │
 │ (pokud je   │   │ během výroby     │
 │ definována) │   └──┬───────────────┘
 └────┬────────┘      │
      │               │
      ▼               │
   ┌──────┐           │
   │ Výstup│           │
   │OK?    │           │
   └─┬──┬──┘           │
     │  │              │
  ANO│  │NE            │
     │  └──────────┐   │
     │             ▼   │
     │        ┌────────────┐
     │        │ REJECT!    │
     │        │ Šarž neusp.│
     │        │ Status:    │
     │        │ FAILED     │
     │        └────┬───────┘
     │             │
     │             ▼
     │        ┌──────────────────┐
     │        │ Notifikace:      │
     │        │ - SUPERVISOR     │
     │        │ - TECHNICIAN     │
     │        │ ROOT CAUSE anal. │
     │        └──────────────────┘
     │
     ▼
 ┌──────────────────────┐
 │ Batch: CLOSED        │
 │ - Vygenerován report │
 │ - Odeslán do ERP     │
 │ - Audit trail update │
 │ - Archivováno        │
 └──┬───────────────────┘
    │
    ▼
 ┌──────────────────────┐
 │ KPI aktualizace:     │
 │ - Čas cyklu          │
 │ - Hmotnostní výtěž   │
 │ - DefectsRate        │
 │ - OEE                │
 └──────────────────────┘
```

**Betrokované role:** OPERATOR, SUPERVISOR, TECHNICIAN  
**Systémy:** Databáze, ERP, OPC UA, Reporting  
**Čas:** 5-15 minut od startu closing do archivování

---

## 📊 PROCES 4: Materiálový tok a traceability

```
┌────────────────────────────────────────────────────────────┐
│        MATERIAL FLOW & TRACEABILITY PROCESS                │
└────────────────────────────────────────────────────────────┘

┌──────────────────┐
│ Příprava batch   │
│ definován BOM:   │
│ - Suroviny       │
│ - Komponenty     │
│ - Pomocné lát.   │
└────────┬─────────┘
         │
         ▼
    ┌──────────────────┐
    │ Sklad:           │
    │ Rezervace materi │
    │ podle BOM        │
    └────────┬─────────┘
             │
             ▼
    ┌──────────────────┐
    │ Fyzický výběr    │
    │ z SKLADu:        │
    │ - Pracovník      │
    │   vydbere        │
    │ - Má instrukce   │
    │   zadání         │
    └────────┬─────────┘
             │
             ▼
   ┌─────────────────────┐
   │ Operátor SKENUJE    │
   │ QR kódy materiálů   │
   │ - Čárák → BOM item  │
   │ - Registr. v batch  │
   │ - Dávka: MAT001     │
   │   č. šarže: X123    │
   └────────┬────────────┘
            │
      ┌─────┴─────┐
      │ MATCH OK? │
      └─────┬─────┘
            │
       ┌────┼────┐
   MATCH  │ MISMATCH
       │  │
       ▼  ▼
    ┌─┐ ┌─────────────┐
    │ │ │ ALARM!      │
    │✓│ │ Nesprávný   │
    │ │ │ materiál    │
    └─┘ │ nebo dávka  │
        │ → БЛОКÁДА   │
        │   šarže     │
        └─────────────┘
             │
             ▼
    ┌─────────────────┐
    │ Záznam v DB:    │
    │ - Material ID   │
    │ - Batch ID      │
    │ - Čas skenování │
    │ - Operátor ID   │
    │ Traceability    │
    │ připravena!     │
    └────────┬────────┘
             │
    ┌────────┴──────────┐
    │                   │
    ▼                   ▼
 ┌─────────┐      ┌──────────────┐
 │ Výroba  │      │ Pokud vrácení │
 │ pokr.   │      │ mat., vrátí   │
 │         │      │ se do skladu  │
 │         │      │ se záznamem   │
 │         │      └──────────────┘
 └────┬────┘
      │
      ▼
   ┌──────────────┐
   │ Po ukončení  │
   │ batch:       │
   │ Výstupní     │
   │ materiál     │
   │ také         │
   │ zaznamenán   │
   │ → Traceability│
   │ hotova!      │
   └──────────────┘
```

**Betrokované role:** OPERATOR, PLANNER  
**Systémy:** SKlad (ERP), Desktop App, Databáze  
**kritické:** Detekování duplicate a mismatch

---

## 🔗 PROCES 5: Integrace s ERP - Výrobní objednávka

```
┌───────────────────────────────────────────────────────────┐
│      ERP INTEGRATION - MANUFACTURING ORDER FLOW           │
└───────────────────────────────────────────────────────────┘

┌──────────────────────┐
│ ERP vytvoří MO       │
│ (Manufacturing Order)│
│ - Produkt            │
│ - Množství           │
│ - Termín             │
│ - Receptura ref.     │
└──────────┬───────────┘
           │
           ▼
  ┌───────────────────────┐
  │ ergon+ PULL z ERP     │
  │ (Sync API)            │
  │ - Pravidelně (1x5min) │
  │ - Nebo webhook        │
  │ - REST API GET        │
  └──────────┬────────────┘
             │
             ▼
    ┌────────────────────┐
    │ Parse MO data      │
    │ - Validace         │
    │ - Duplicity check   │
    │ - Status: NEW      │
    └──────────┬─────────┘
               │
               ▼
    ┌──────────────────────┐
    │ Stažení receptury z  │
    │ ERP (pokud vedena    │
    │ v ERP):              │
    │ - BOM                │
    │ - Parametry procesu  │
    │ - Toleranční rozmezí │
    └──────────┬───────────┘
               │
               ▼
    ┌──────────────────────┐
    │ Uložení do ergon+    │
    │ databáze             │
    │ Status: IMPORTED     │
    └──────────┬───────────┘
               │
               ▼
    ┌──────────────────────┐
    │ Notifikace planeru   │
    │ "Nová MO k plánování"│
    │ Dashboard upd.       │
    └──────────┬───────────┘
               │
               ▼
    ┌──────────────────────┐
    │ Planner:             │
    │ - Vytvoří batches    │
    │ - Naplánuje linky    │
    │ - Potvrdí časový     │
    │   plán               │
    └──────────┬───────────┘
               │
               ▼
    ┌──────────────────────┐
    │ PUSH statusu zpět do │
    │ ERP (API PUT):       │
    │ - Status: PLANNED    │
    │ - Plánovaný termín   │
    │ - Přidělená linka    │
    └──────────┬───────────┘
               │
        ┌──────┴──────────┐
        │                 │
        ▼                 ▼
    (Výroba)         ┌──────────────┐
    Batch START      │ ERP updaten  │
        │            └──────────────┘
        │
        ▼
   (výroba běží)
        │
        ▼
    COMPLETED
        │
        ▼
    ┌────────────────────────┐
    │ PUSH final status do   │
    │ ERP:                   │
    │ - Status: COMPLETED    │
    │ - Skutečný čas         │
    │ - Kvalitativní data    │
    │ - Logistické info      │
    │ - Výroba je v ERP      │
    │   zaúčtovaná           │
    └────────────────────────┘
```

**Betrokované role:** ADMIN (integrace), PLANNER (plánování)  
**Systémy:** ergon+ ↔ ERP APIs  
**Frekvence:** 5 minut pull, real-time push

---

## 🔌 PROCES 6: OPC UA data flow

```
┌──────────────────────────────────────────────────────┐
│         OPC UA REAL-TIME DATA ACQUISITION            │
└──────────────────────────────────────────────────────┘

┌─────────────────────┐
│ PLC s OPC UA        │
│ serverem:           │
│ - Teplota (T1)      │
│ - Tlak (P1)         │
│ - Průtok (F1)       │
│ - Polohy (M1-M4)    │
│ - Stavy relé (R1-R8)│
└──────────┬──────────┘
           │
           ▼
    ┌──────────────────┐
    │ ergon+ OPC Client│
    │ - Subscribes na  │
    │   tagy           │
    │ - Interval: 1 sec│
    └────────┬─────────┘
             │
             ▼
   ┌──────────────────────┐
   │ Real-time stream     │
   │ T1=180.2°C, P1=5atm │
   │ F1=25.5 l/min, ...  │
   └────────┬─────────────┘
            │
     ┌──────┴───────┐
     │              │
     ▼              ▼
┌──────────┐   ┌─────────────────┐
│ Validace │   │ Kontrola limitů │
│ Datového │   │ - Teplota: <180 │
│ typu     │   │ - Tlak: <6      │
│ (float,  │   │ - Výchylka: <X% │
│ bool)    │   └────────┬────────┘
└───┬──────┘            │
    │          ┌────────┴────────┐
    │          │                 │
    │          ▼                 ▼
    │       ┌────────────┐  ┌──────────┐
    │       │ OK - Log  │  │ VÝCHYLKA!│
    │       │ do DB     │  │ ALARM    │
    │       └─────┬──────┘  └──┬───────┘
    │             │            │
    │             │            ▼
    │             │      ┌────────────┐
    │             │      │ Notifikace │
    │             │      │ - Operátor│
    │             │      │ - Supervisor
    │             │      │ - Email    │
    │             │      │ - SMS      │
    │             │      └──────┬─────┘
    │             │             │
    └─────────┬───┴─────────────┘
              │
              ▼
    ┌──────────────────────┐
    │ INSERT do tabulky    │
    │ parameter_log:       │
    │ - batch_id           │
    │ - line_id            │
    │ - opc_tag            │
    │ - value              │
    │ - timestamp          │
    │ - status             │
    └──────────┬───────────┘
               │
               ▼
    ┌──────────────────────┐
    │ Query za posledních  │
    │ 1 hodiny → Stats:    │
    │ - AVG, MIN, MAX      │
    │ - StdDev             │
    │ → Dashboard update   │
    └──────────────────────┘
```

**Betrokované role:** ADMIN (konfig), TECHNICIAN (mapování)  
**Systémy:** OPC UA server, Databáze, Real-time monitoring  
**Latence:** <1 sec od měření po záznam

---

## 📌 Souhrn stavů šarže (Batch States)

```
CREATED
  ↓
PLANNED (přiděleno na linku)
  ↓
READY (všechny materiály k dispozici)
  ↓
RUNNING (spuštěna výroba)
  ├→ PAUSED (operátor pozastavil)
  │   └→ RUNNING (pokračuje)
  └→ COMPLETED (všechny kroky hotovy)
      │
      ├→ PASSED (prošla výstupní kontrolou)
      │   └→ CLOSED (zaúčtováno v ERP)
      │
      └→ REJECTED (selhala kontrola)
          └→ FAILED (definitivně nezdařená)
```

---

**Datum vytvoření:** 10.4.2026  
**Status:** Iniciace projektu  
**Verze:** 0.1
