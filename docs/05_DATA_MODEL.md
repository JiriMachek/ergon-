# 05 - Datový model | ergon+ MES

## 📊 Entity-Relationship Diagram

```
┌─────────────┐         ┌──────────────┐
│   USERS     ├────────┤   ROLES      │
├─────────────┤         ├──────────────┤
│ id (PK)     │         │ id (PK)      │
│ username    │         │ name         │
│ email       │         │ description  │
│ password    │         │ permissions  │
│ role_id(FK) │         │              │
│ active      │         └──────────────┘
│ created_at  │
│ updated_at  │
│ last_login  │
└─────────────┘
       │
       │
       └─────────┬──────────────────────────────────┐
                 │                                  │
                 ▼                                  ▼
         ┌──────────────┐              ┌──────────────────┐
         │  LINES       │              │  EQUIPMENT       │
         ├──────────────┤              ├──────────────────┤
         │ id (PK)      │              │ id (PK)          │
         │ name         │              │ line_id (FK)     │
         │ description  │              │ name             │
         │ status       │              │ type             │
         │ created_at   │              │ status           │
         │ updated_at   │              │ parameters (JSON)│
         └──────────────┘              └──────────────────┘
              │
              │
              ├──────────┬──────────────────────────────┐
              │          │                              │
              ▼          ▼                              ▼
        ┌──────────┐ ┌──────────────┐      ┌──────────────┐
        │ BATCHES  │ │  RECIPES     │      │  MATERIALS   │
        ├──────────┤ ├──────────────┤      ├──────────────┤
        │ id (PK)  │ │ id (PK)      │      │ id (PK)      │
        │ mo_id(FK)│ │ name         │      │ name         │
        │ line_id()│ │ version      │      │ sku          │
        │ recipe() │ │ status       │      │ unit         │
        │ status   │ │ steps (JSON) │      │ vendor_id    │
        │ priority │ │ created_by   │      │ shelf_life   │
        │ due_date │ │ approved_by  │      │ created_at   │
        │ created_ │ │ created_at   │      │              │
        │ at       │ │ updated_at   │      └──────────────┘
        │ start_at │ └──────────────┘              │
        │ end_at   │         │                     │
        │ comments │         │                     │
        └──┬───────┘         │                     │
           │        ┌────────┴─────────┐           │
           │        │                  ▼           │
           │        │          ┌──────────────────┬┘
           │        │          │ RECIPE_MATERIALS │
           │        │          ├──────────────────┤
           │        │          │ id (PK)          │
           │        │          │ recipe_id (FK)   │
           │        │          │ material_id (FK) │
           │        │          │ quantity         │
           │        │          │ unit             │
           │        │          │ position         │
           │        │          └──────────────────┘
           │        │
           │        └───────────────────────┬──────┐
           │                                │      │
           ▼                                ▼      ▼
    ┌──────────────┐              ┌──────────────────┐
    │ MES_BATCHES  │              │ RECIPE_STEPS     │
    │ (batch matl) │              ├──────────────────┤
    ├──────────────┤              │ id (PK)          │
    │ id (PK)      │              │ recipe_id (FK)   │
    │ batch_id(FK) │              │ step_number      │
    │ material_id()│              │ name             │
    │ material_num │              │ duration_sec     │
    │ scanned_at   │              │ parameters (JSON)│
    │ quantity     │              │ conditions       │
    │ operator_id()│              │ position         │
    │ status       │              └──────────────────┘
    └──────────────┘
           │
           ▼
    ┌─────────────────┐
    │ PARAMETER_LOG   │    (Real-time data from OPC UA)
    ├─────────────────┤
    │ id (PK)         │
    │ batch_id (FK)   │
    │ equipment_id()  │
    │ opc_tag         │
    │ parameter_name  │
    │ value           │
    │ unit            │
    │ timestamp       │
    │ status          │  (NORMAL, WARNING, ALARM)
    │ min_limit       │
    │ max_limit       │
    └─────────────────┘
           │
           ▼
    ┌──────────────────┐
    │ ALARMS           │
    ├──────────────────┤
    │ id (PK)          │
    │ batch_id (FK)    │
    │ type             │  (PROCESS, QUALITY, EQUIPMENT)
    │ severity         │  (CRITICAL, WARNING, INFO)
    │ message          │
    │ created_at       │
    │ acknowledged_at  │
    │ acknowledged_by()│
    │ resolved_at      │
    │ resolution_notes │
    └──────────────────┘
           │
           ▼
    ┌──────────────────────┐
    │ AUDIT_LOG            │
    ├──────────────────────┤
    │ id (PK)              │
    │ user_id (FK)         │
    │ action               │
    │ entity_type          │
    │ entity_id            │
    │ old_value            │
    │ new_value            │
    │ timestamp            │
    │ ip_address           │
    │ user_agent           │
    └──────────────────────┘
```

---

## 📋 Detailní schéma tabulek

### 1. USERS
```sql
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(100) NOT NULL UNIQUE,
    email VARCHAR(100) NOT NULL UNIQUE,
    password_hash VARCHAR(255) NOT NULL,
    role_id INT NOT NULL FOREIGN KEY REFERENCES roles(id),
    is_active BOOLEAN DEFAULT TRUE,
    is_mfa_enabled BOOLEAN DEFAULT FALSE,
    failed_login_attempts INT DEFAULT 0,
    locked_until DATETIME NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    last_login DATETIME NULL,
    INDEX idx_username (username),
    INDEX idx_email (email),
    INDEX idx_role_id (role_id)
);
```

### 2. ROLES
```sql
CREATE TABLE roles (
    id INT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(50) NOT NULL UNIQUE,
    description TEXT,
    permissions JSON NOT NULL,
    is_system_role BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);
```

### 3. BATCHES
```sql
CREATE TABLE batches (
    id INT PRIMARY KEY AUTO_INCREMENT,
    batch_number VARCHAR(50) NOT NULL UNIQUE,
    mo_id VARCHAR(50),                    -- Manufacturing Order ID z ERP
    line_id INT NOT NULL FOREIGN KEY REFERENCES lines(id),
    recipe_id INT NOT NULL FOREIGN KEY REFERENCES recipes(id),
    status ENUM('CREATED','PLANNED','READY','RUNNING','PAUSED',
                'COMPLETED','PASSED','REJECTED','CLOSED','FAILED') 
           DEFAULT 'CREATED',
    priority INT DEFAULT 5,                -- 1=highest, 10=lowest
    priority_reason VARCHAR(255),
    planned_start_time DATETIME,
    planned_end_time DATETIME,
    actual_start_time DATETIME,
    actual_end_time DATETIME,
    quantity DECIMAL(10,2) NOT NULL,
    quantity_unit VARCHAR(10),
    target_qty_min DECIMAL(10,2),
    target_qty_max DECIMAL(10,2),
    defects_count INT DEFAULT 0,
    comments TEXT,
    created_by INT FOREIGN KEY REFERENCES users(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    closed_by INT FOREIGN KEY REFERENCES users(id),
    closed_at DATETIME,
    
    INDEX idx_status (status),
    INDEX idx_line_id (line_id),
    INDEX idx_recipe_id (recipe_id),
    INDEX idx_created_at (created_at),
    INDEX idx_batch_number (batch_number)
);
```

### 4. RECIPES
```sql
CREATE TABLE recipes (
    id INT PRIMARY KEY AUTO_INCREMENT,
    code VARCHAR(50) NOT NULL UNIQUE,
    name VARCHAR(200) NOT NULL,
    description TEXT,
    version VARCHAR(10) NOT NULL,
    status ENUM('DRAFT','APPROVED','DEPRECATED','ARCHIVED') DEFAULT 'DRAFT',
    steps JSON NOT NULL,                  -- Array of steps with parameters
    total_duration_sec INT,
    target_yield_percent DECIMAL(5,2),
    created_by INT NOT NULL FOREIGN KEY REFERENCES users(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    approved_by INT FOREIGN KEY REFERENCES users(id),
    approved_at DATETIME,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    change_note TEXT,
    
    INDEX idx_code (code),
    INDEX idx_status (status),
    INDEX idx_name (name)
);
```

### 5. RECIPES_MATERIALS
```sql
CREATE TABLE recipe_materials (
    id INT PRIMARY KEY AUTO_INCREMENT,
    recipe_id INT NOT NULL FOREIGN KEY REFERENCES recipes(id),
    material_id INT NOT NULL FOREIGN KEY REFERENCES materials(id),
    quantity DECIMAL(10,2) NOT NULL,
    unit VARCHAR(10) NOT NULL,           -- kg, m, pcs, liters
    position INT NOT NULL,                -- Order in recipe
    tolerance_min DECIMAL(10,2),
    tolerance_max DECIMAL(10,2),
    is_optional BOOLEAN DEFAULT FALSE,
    notes TEXT,
    
    UNIQUE KEY unique_recipe_material (recipe_id, material_id),
    INDEX idx_recipe_id (recipe_id),
    INDEX idx_material_id (material_id)
);
```

### 6. RECIPE_STEPS
```sql
CREATE TABLE recipe_steps (
    id INT PRIMARY KEY AUTO_INCREMENT,
    recipe_id INT NOT NULL FOREIGN KEY REFERENCES recipes(id),
    step_number INT NOT NULL,
    name VARCHAR(200) NOT NULL,
    description TEXT,
    type ENUM('HEATING','COOLING','MIXING','WAITING','TRANSFER','QUALITY_CHECK'),
    duration_seconds INT NOT NULL,
    equipment_required INT FOREIGN KEY REFERENCES equipment(id),
    
    -- Parameters as JSON:
    -- {"temperature": {"value": 180, "min": 175, "max": 185, "unit": "°C"},
    --  "pressure": {"value": 5, "min": 4, "max": 6, "unit": "atm"}}
    parameters JSON,
    
    -- Transition rules
    transition_type ENUM('AUTO','MANUAL','CONDITIONAL') DEFAULT 'AUTO',
    transition_condition VARCHAR(500),    -- e.g., "temp > 175 AND pressure < 6"
    
    failure_action ENUM('STOP','ALARM','RETRY','SKIP') DEFAULT 'ALARM',
    
    UNIQUE KEY unique_recipe_step (recipe_id, step_number),
    INDEX idx_recipe_id (recipe_id)
);
```

### 7. LINES
```sql
CREATE TABLE lines (
    id INT PRIMARY KEY AUTO_INCREMENT,
    code VARCHAR(50) NOT NULL UNIQUE,
    name VARCHAR(200) NOT NULL,
    description TEXT,
    status ENUM('ACTIVE','MAINTENANCE','INACTIVE') DEFAULT 'ACTIVE',
    opc_ua_endpoint VARCHAR(255),
    opc_ua_namespace VARCHAR(255),
    location_info VARCHAR(255),
    max_concurrent_batches INT DEFAULT 1,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    INDEX idx_code (code),
    INDEX idx_status (status)
);
```

### 8. EQUIPMENT
```sql
CREATE TABLE equipment (
    id INT PRIMARY KEY AUTO_INCREMENT,
    line_id INT NOT NULL FOREIGN KEY REFERENCES lines(id),
    code VARCHAR(50) NOT NULL UNIQUE,
    name VARCHAR(200) NOT NULL,
    type ENUM('REACTOR','MIXER','PUMP','COMPRESSOR','SENSOR','PLC'),
    manufacturer VARCHAR(100),
    model VARCHAR(100),
    serial_number VARCHAR(100),
    opc_ua_node_id VARCHAR(255),
    parameters_schema JSON,  -- Supported parameters
    status ENUM('OPERATIONAL','MAINTENANCE','FAILED','OFFLINE') DEFAULT 'OPERATIONAL',
    last_maintenance DATETIME,
    next_maintenance DATETIME,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    INDEX idx_line_id (line_id),
    INDEX idx_code (code),
    INDEX idx_status (status)
);
```

### 9. MATERIALS
```sql
CREATE TABLE materials (
    id INT PRIMARY KEY AUTO_INCREMENT,
    code VARCHAR(50) NOT NULL UNIQUE,
    name VARCHAR(200) NOT NULL,
    description TEXT,
    type ENUM('RAW_MATERIAL','COMPONENT','PACKAGING','ADDITIVE'),
    unit VARCHAR(10) NOT NULL,            -- kg, m, pcs
    vendor_id VARCHAR(50),                -- External vendor ID
    shelf_life_days INT,
    storage_conditions VARCHAR(500),
    is_hazardous BOOLEAN DEFAULT FALSE,
    cost_per_unit DECIMAL(10,2),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    
    INDEX idx_code (code),
    INDEX idx_name (name)
);
```

### 10. BATCH_MATERIALS (Material scans)
```sql
CREATE TABLE batch_materials (
    id INT PRIMARY KEY AUTO_INCREMENT,
    batch_id INT NOT NULL FOREIGN KEY REFERENCES batches(id),
    material_id INT NOT NULL FOREIGN KEY REFERENCES materials(id),
    material_lot_number VARCHAR(100),
    recipe_material_id INT FOREIGN KEY REFERENCES recipe_materials(id),
    quantity_scanned DECIMAL(10,2) NOT NULL,
    quantity_used DECIMAL(10,2) DEFAULT NULL,
    scanned_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    scanned_by INT NOT NULL FOREIGN KEY REFERENCES users(id),
    used_at DATETIME,
    used_by INT FOREIGN KEY REFERENCES users(id),
    notes TEXT,
    
    INDEX idx_batch_id (batch_id),
    INDEX idx_material_id (material_id),
    INDEX idx_scanned_at (scanned_at)
);
```

### 11. PARAMETER_LOG
```sql
CREATE TABLE parameter_log (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    batch_id INT NOT NULL FOREIGN KEY REFERENCES batches(id),
    equipment_id INT FOREIGN KEY REFERENCES equipment(id),
    opc_tag VARCHAR(255) NOT NULL,
    parameter_name VARCHAR(100),
    value DECIMAL(15,4),
    unit VARCHAR(20),
    status ENUM('NORMAL','WARNING','ALARM') DEFAULT 'NORMAL',
    min_limit DECIMAL(15,4),
    max_limit DECIMAL(15,4),
    timestamp DATETIME NOT NULL,
    recorded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    INDEX idx_batch_id_timestamp (batch_id, timestamp),
    INDEX idx_equipment_id (equipment_id),
    INDEX idx_timestamp (timestamp),
    INDEX idx_opc_tag (opc_tag)
);
```

### 12. ALARMS
```sql
CREATE TABLE alarms (
    id INT PRIMARY KEY AUTO_INCREMENT,
    batch_id INT NOT NULL FOREIGN KEY REFERENCES batches(id),
    equipment_id INT FOREIGN KEY REFERENCES equipment(id),
    type ENUM('PROCESS','QUALITY','EQUIPMENT','MATERIAL','COMPLIANCE'),
    severity ENUM('CRITICAL','HIGH','MEDIUM','LOW','INFO') DEFAULT 'MEDIUM',
    code VARCHAR(50),
    title VARCHAR(200) NOT NULL,
    description TEXT,
    message TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    acknowledged BOOLEAN DEFAULT FALSE,
    acknowledged_by INT FOREIGN KEY REFERENCES users(id),
    acknowledged_at DATETIME,
    resolved BOOLEAN DEFAULT FALSE,
    resolved_at DATETIME,
    resolution_notes TEXT,
    escalated_at DATETIME,
    escalated_to INT FOREIGN KEY REFERENCES users(id),
    
    INDEX idx_batch_id (batch_id),
    INDEX idx_equipment_id (equipment_id),
    INDEX idx_severity (severity),
    INDEX idx_created_at (created_at),
    INDEX idx_acknowledged (acknowledged),
    INDEX idx_resolved (resolved)
);
```

### 13. AUDIT_LOG
```sql
CREATE TABLE audit_log (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    user_id INT FOREIGN KEY REFERENCES users(id),
    action VARCHAR(100),                  -- CREATE, UPDATE, DELETE, etc.
    entity_type VARCHAR(100),             -- BATCH, RECIPE, MATERIAL, etc.
    entity_id INT,
    old_value JSON,
    new_value JSON,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    ip_address VARCHAR(45),
    user_agent TEXT,
    
    INDEX idx_user_id (user_id),
    INDEX idx_entity_type_id (entity_type, entity_id),
    INDEX idx_timestamp (timestamp),
    INDEX idx_action (action)
);
```

---

## 🔑 Klíčové relace

```
USERS ──┬── ROLES ── (permissions)
        │
        └── AUDIT_LOG

BATCHES ──┬── LINES ── EQUIPMENT ── (OPC UA)
          ├── RECIPES ── RECIPE_STEPS
          │                  │
          │            RECIPE_MATERIALS
          │                  │
          ├── BATCH_MATERIALS ── MATERIALS
          │
          ├── ALARMS
          └── PARAMETER_LOG ── (Real-time sensor data)

PARAMETER_LOG ── (archív z OPC UA)
```

---

## 📊 Indexování a výkon

- **Primary Keys**: Všechny tabulky
- **Foreign Keys**: Bezpečnost referenční integrity
- **Indexy na vysokopoužívané sloupce**: status, batch_id, timestamp
- **Partitioning**: PARAMETER_LOG partitionovaná měsíčně (pro archivaci)
- **Caching**: Redis cache pro:
  - User sessions
  - Role definitions
  - Current batch statuses
  - OPC UA tag mappings

---

**Datum vytvoření:** 10.4.2026  
**Status:** Iniciace projektu  
**Verze:** 0.1
