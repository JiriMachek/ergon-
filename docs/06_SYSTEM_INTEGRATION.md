# 06 - Systémová integrace | ergon+ MES

## 🔌 Integrační architektura

```
┌────────────────────────────────────────────────────────────────┐
│                        ergon+ MES (Process Engine)             │
│        (Windows Service / .NET Worker + MariaDB)                │
├────────────────────────────────────────────────────────────────┤
│                                                                │
│  ┌─────────────────┐  ┌──────────────┐  ┌────────────────┐   │
│  │ OPC UA Client   │  │ Service API  │  │ Notification    │   │
│  │ (Real-time)     │  │ (Integration)│  │ (Notifications)│   │
│  └────────┬────────┘  └──────┬───────┘  └────────┬───────┘   │
│           │                  │                   │            │
│           ▼                  ▼                   ▼            │
│  ┌────────────────────────────────────────────────────┐      │
│  │         Application Logic (Domain Layer)           │      │
│  │  - Batch Management                               │      │
│  │  - Recipe Management                              │      │
│  │  - Material Traceability                          │      │
│  │  - KPI Calculation                                │      │
│  └────────────────────────────────────────────────────┘      │
│           │                  │                   │            │
│           ▼                  ▼                   ▼            │
│  ┌─────────────────────────────────────────────────────────┐ │
│  │              MariaDB Database                          │ │
│  │  - Batches, Recipes, Materials, etc.                 │ │
│  │  - Parameter Log (Time-series)                       │ │
│  │  - Audit Log (Immutable)                            │ │
│  └─────────────────────────────────────────────────────────┘ │
│                                                                │
└────────────────────────────────────────────────────────────────┘
        │                  │                     │
        ▼                  ▼                     ▼
    ┌──────────┐    ┌──────────────┐    ┌──────────────┐
    │ PLC/     │    │ ERP System   │    │ Desktop App  │
    │ SCADA    │    │ (Navision/   │    │ (WPF / WinUI)
    │ (OPC UA) │    │  SAP/Odoo)   │    │              │
    └──────────┘    └──────────────┘    └──────────────┘
```

---

## 🔌 1. OPC UA INTEGRACE (Real-time PLC Communication)

### Architektura

```
PLC s OPC UA Serverem
   │
   │ OPC UA Protocol
   │ (Secure Channel)
   │
   ▼
┌──────────────────────┐
│ ergon+ OPC UA Client │
├──────────────────────┤
│ - OPC UA Library     │
│   (OPC.UA .NET)      │
│ - Data Subscription  │
│ - Tag Mapping        │
│ - Data Validation    │
└──────────┬───────────┘
           │
           ▼
    ┌──────────────┐
    │ Data Queue   │
    │ (Thread-safe)│
    └──────┬───────┘
           │
           ▼
  ┌── ─────────────────┐
  │ Parameter Logger   │
  │ Service            │
  ├────────────────────┤
  │ - Interval: 1 sec  │
  │ - Data Validation  │
  │ - Alarm Triggers   │
  │ - Real-time LOG    │
  └────────┬───────────┘
           │
           ▼
    ┌──────────────────────┐
    │ Database Write       │
    │ - parameter_log      │
    │ - Batch updates      │
    └──────────────────────┘
```

### Konfigurace OPC UA

```json
{
  "opcua": {
    "enabled": true,
    "servers": [
      {
        "name": "Line-A-PLC",
        "endpoint": "opc.tcp://192.168.1.100:4840",
        "namespace": "http://assortis.cz/plc/lineA",
        "securityPolicy": "Basic256Sha256",
        "messageMode": "SignAndEncrypt",
        "reconnect_interval_ms": 5000,
        "subscription_interval_ms": 1000,
        "buffer_size": 10000,
        "tags": [
          {
            "nodeId": "ns=2;i=5001",
            "name": "TEMP_REACTOR_A",
            "dataType": "Float",
            "unit": "°C",
            "min_value": 20,
            "max_value": 200,
            "alarm_high": 190,
            "alarm_low": 170,
            "deadband": 1.0,
            "sample_interval_ms": 1000
          },
          {
            "nodeId": "ns=2;i=5002",
            "name": "PRESSURE_REACTOR_A",
            "dataType": "Float",
            "unit": "atm",
            "min_value": 0,
            "max_value": 10
          }
        ]
      }
    ]
  }
}
```

### API pro klientské aplikace

```csharp
// GET Real-time parameter
GET /api/equipment/{equipmentId}/parameters/current?tags=TEMP_REACTOR_A,PRESSURE_REACTOR_A

// Response:
{
  "timestamp": "2026-04-10T14:25:30Z",
  "parameters": {
    "TEMP_REACTOR_A": {
      "value": 178.5,
      "unit": "°C",
      "status": "NORMAL",
      "timestamp": "2026-04-10T14:25:30Z"
    },
    "PRESSURE_REACTOR_A": {
      "value": 4.8,
      "unit": "atm",
      "status": "NORMAL"
    }
  }
}

// GET Historical data
GET /api/equipment/{equipmentId}/parameters/history\
    ?tag=TEMP_REACTOR_A\
    &start=2026-04-10T00:00:00Z\
    &end=2026-04-10T23:59:59Z\
    &interval=60000

// Response:
{
  "tag": "TEMP_REACTOR_A",
  "unit": "°C",
  "data": [
    {"timestamp": "2026-04-10T00:00:00Z", "value": 165.2, "status": "NORMAL"},
    {"timestamp": "2026-04-10T00:01:00Z", "value": 167.8, "status": "NORMAL"},
    ...
  ]
}
```

---

## 🌐 2. ERP INTEGRACE

### Podporované ERP systémy

- **SAP** (REST API / RFC)
- **Dynamics 365 / Navision** (OData API)
- **Odoo** (REST API / XML-RPC)
- **Vlastní ERP** (XML/JSON API)

### Synchronizační FLOW

```json
{
  "erp_integration": {
    "enabled": true,
    "systems": [
      {
        "name": "ERP_NAVISION",
        "type": "navision",
        "endpoint": "https://erp.assortis.cz/odata/",
        "username": "${ERP_USER}",
        "password": "${ERP_PASSWORD}",
        
        "sync_intervals": {
          "manufacturing_orders": 300000,      // 5 minut
          "recipes": 3600000,                  // 1 hodina
          "materials": 3600000                 // 1 hodina
        },
        
        "entities": {
          "manufacturing_orders": {
            "endpoint": "/odata/Company('CRONUS')/ManufacturingOrders",
            "poll_interval_sec": 300,
            "push_status_on_changes": true,
            "mappings": {
              "order_id": "No",
              "product_id": "Item_No",
              "quantity": "Quantity",
              "due_date": "Due_Date",
              "recipe_reference": "Recipe_Code"
            }
          },
          "recipes": {
            "endpoint": "/odata/Company('CRONUS')/ProductionBoms",
            "read_only": false,
            "fallback_to_local": true
          },
          "materials": {
            "endpoint": "/odata/Company('CRONUS')/Items",
            "sync_direction": "pull_only"
          }
        }
      }
    ]
  }
}
```

### API Endpoints - ERP Integration

```csharp
// Pull Manufacturing Orders z ERP
POST /api/erp/sync/manufacturing-orders

// Response:
{
  "status": "SUCCESS",
  "records_synced": 15,
  "new_batches_created": 15,
  "timestamp": "2026-04-10T14:30:00Z",
  "orders": [
    {
      "order_id": "PO-2026-001",
      "product_id": "P-A100",
      "quantity": 1000,
      "due_date": "2026-04-15T18:00:00Z",
      "status": "IMPORTED"
    }
  ]
}

// Push Batch Status zpět do ERP
PUT /api/batches/{batchId}/sync-to-erp

// Push Completion Report do ERP
POST /api/batches/{batchId}/erp/close

// Payload:
{
  "actual_start_time": "2026-04-10T08:15:00Z",
  "actual_end_time": "2026-04-10T14:25:00Z",
  "quantity_produced": 1000,
  "quality_status": "PASSED",
  "defects": 0,
  "comments": "Výroba bez problémů",
  "parameter_summary": {
    "avg_temperature": 178.2,
    "min_temperature": 176.5,
    "max_temperature": 180.1
  }
}

// Odpověď z ERP:
{
  "status": "SUCCESS",
  "erp_reference": "PROD-002226",
  "invoice_created": false,
  "inventory_updated": true
}
```

### Chybové scénáře

```csharp
// Když ERP vrátí chybu
{
  "status": "PARTIAL_SUCCESS",
  "records_synced": 12,
  "records_failed": 3,
  "errors": [
    {
      "order_id": "PO-2026-005",
      "error": "Material XYZ not found in local system",
      "action": "QUEUED_FOR_RETRY"
    }
  ]
}
```

---

## � 3. DESKTOP APLIKACE INTEGRACE

### Architektura

```
┌─────────────────────────────────┐
│  Desktop App (WPF / WinUI)      │
│  Windows PC clients              │
├─────────────────────────────────┤
│ - Local storage (SQLite)        │
│ - Notifications                 │
│ - Barcode Scanner               │
│ - Industrial peripherals        │
└────────────┬────────────────────┘
             │
             │ Service API / IPC
             │
             ▼
    ┌────────────────────┐
    │ ergon+ Service     │
    │ (Windows Service)  │
    │ - Auth Middleware  │
    │ - Rate Limiting    │
    │ - SSL/TLS          │
    └────────┬──────────┘
             │
             ▼
    ┌────────────────────┐
    │ Desktop API Layer  │
    │ (Optimized for     │
    │  desktop clients)  │
    └────────────────────┘
```

### Desktop API Endpoints

```csharp
// Authentication
POST /api/desktop/auth/login
{
  "username": "operator1",
  "password": "***",
  "device_id": "DEVICE_UUID"
}

// Response - includes refresh token
{
  "access_token": "eyJhbGci...",
  "refresh_token": "...",
  "expires_in": 3600
}

// Get Current Batches (Offline support)
GET /api/desktop/batches/assigned-to-me

// Scan Material
POST /api/desktop/batches/{batchId}/scan-material
{
  "barcode": "MAT-2026-001234",
  "quantity": 5.2,
  "unit": "kg"
}

// Report Issue
POST /api/desktop/batches/{batchId}/report-issue
{
  "issue_type": "TEMPERATURE_HIGH",
  "description": "Teplota neklesá",
  "photo_data": "base64_encoded_image",
  "severity": "HIGH"
}

// Get Real-time Notifications
WebSocket: wss://api.ergon.assortis.cz/ws/notifications
{
  "event": "BATCH_COMPLETED",
  "batch_id": 123,
  "action_required": true
}

// Offline Sync - po obnovení spojení
POST /api/desktop/sync
{
  "pending_actions": [
    {
      "action": "scan_material",
      "batch_id": 123,
      "barcode": "MAT-001",
      "timestamp": "2026-04-10T10:30:00Z"
    }
  ]
}
```

---

## 🔐 API Bezpečnost

### Authentication & Authorization

```csharp
// JWT Token Structure
{
  "header": {
    "alg": "HS256",
    "typ": "JWT"
  },
  "payload": {
    "sub": "user_123",
    "username": "operator1",
    "role": "OPERATOR",
    "permissions": ["BATCH_START", "MATERIAL_SCAN"],
    "line_ids": [1, 2, 3],
    "exp": 1681000000,
    "iat": 1680996400
  }
}

// API Key for integrations
GET /api/batches
Headers: {
  "Authorization": "Bearer eyJhbGci...",
  "X-API-Key": "sk_live_xxx"
}

// Rate Limiting
{
  "rate_limit_limit": 1000,
  "rate_limit_remaining": 999,
  "rate_limit_reset": 1681000000
}
```

---

## 📊 4. WEBHOOKS

ergon+ může odesílat notifikace externím systémům:

```csharp
// Konfigurace webhooků
POST /api/admin/webhooks
{
  "url": "https://external-system.com/webhooks/ergon",
  "events": ["batch.started", "batch.completed", "batch.failed"],
  "secret": "webhook_secret_key"
}

// Beispiel webhook payload - Batch Started
POST https://external-system.com/webhooks/ergon
Headers: {
  "X-Ergon-Signature": "sha256=...",
  "Content-Type": "application/json"
}
{
  "event": "batch.started",
  "timestamp": "2026-04-10T14:25:00Z",
  "batch_id": 123,
  "batch_number": "B-2026-000123",
  "line_id": 1,
  "recipe_code": "RCP001",
  "operator_id": 5,
  "estimated_end_time": "2026-04-10T14:55:00Z"
}

// Batch Completed
{
  "event": "batch.completed",
  "batch_id": 123,
  "status": "PASSED",
  "actual_duration_sec": 1800,
  "quality_passed": true,
  "parameter_summary": {
    "avg_temperature": 178.2,
    "avg_pressure": 4.9
  }
}
```

---

## 🔄 Synchronizační strategie

### Pull Sync (ergon+ pulls z externos)

```csharp
// Manufacturing Orders dari ERP (5 min interval)
Timer: Every 5 minutes
{
  GET /erp/api/manufacturing-orders?updated_since=2026-04-10T14:20:00Z
  Process: Merge with local DB, create batches if new
  Retry: Max 3 attempts s exponential backoff
}

// ERP inventory updates (hourly)
Timer: Every 1 hour
{
  GET /erp/api/materials?last_modified_after=...
  Sync: Local material DB
}
```

### Push Sync (ergon+ pushes do ERP)

```csharp
// Batch status updates (Real-time)
On Event: Batch status changes
{
  PUT /erp/api/manufacturing-orders/{order_id}
  Payload: { status, progress, actual_time, ... }
  Retry: Every 30 sec, max 10 times
  Queue: If ERP unavailable, queue locally
}

// Batch completion with detailed report
On Event: Batch CLOSED
{
  POST /erp/api/manufacturing-orders/{order_id}/complete
  Payload: Complete performance data, quality results
  Retry: Critical - must succeed
  Audit: Everything logged
}
```

### Konflikt Resolution

```csharp
// Pokud se data změní v obou systém (rare):
Pravila:
1. Timestamp-based: Novější záznam vyhraje
2. Source-based: ergon+ má precedent při kritických změnách
3. Manual: User je upozorněn na konflikt
```

---

## 🆘 Error Handling & Resilience

```csharp
// Circuit Breaker pattern
{
  "opc_ua_connection": {
    "failure_threshold": 5,
    "timeout_sec": 10,
    "reset_timeout_sec": 60,
    "current_state": "CLOSED"  // CLOSED, OPEN, HALF_OPEN
  }
}

// Retry with Exponential Backoff
{
  "max_retries": 5,
  "initial_delay_ms": 1000,
  "max_delay_ms": 30000,
  "exponential_base": 2.0,
  "jitter": true
}

// Fallback strategies
{
  "erp_unavailable": {
    "action": "QUEUE_LOCALLY",
    "retry_when_available": true
  },
  "opc_ua_disconnected": {
    "action": "USE_CACHED_PARAMETERS",
    "fallback_to_manual_input": true
  }
}
```

---

**Datum vytvoření:** 10.4.2026  
**Status:** Iniciace projektu  
**Verze:** 0.1
