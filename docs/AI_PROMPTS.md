# AI_PROMPTS.md - Pokyny pro AI generování kódu | ergon+ MES

Tento dokument obsahuje detailní prompty a instrukce pro AI modeling, která bude generovat kód pro ergon+ MES.

---

## 🎯 Vůdčí principy

1. **Clean Architecture** - Kód musí respektovat vrstvy (API, Application, Domain, Infrastructure)
2. **SOLID principy** - Single Responsibility, Open/Closed, Liskov Substitution, Interface Segregation, Dependency Inversion
3. **DDD Light** - Entity, Value Objects, Aggregates, Domain Events
4. **Testovatelnost** - Všechny komponenty musí být testovatelné, min. 80% coverage
5. **Dokumentace** - Kód má XML docs, komentáře pro komplexní logiku
6. **Bezpečnost** - OWASP top 10, validace inputů, SQL injection prevention (EF Core), XSS protection
7. **Performance** - Async/await, databázové indexy, caching strategy, query optimization

---

## 📝 PROMPT 1: Entity & Domain Layer

**Kontext:** Vytvořit doménové entity pro batch management v ergon+ MES

```
TASK: Generate Domain Layer for Batch Management

REQUIREMENTS:
1. Create Entity classes (Batch, Recipe, Material, Step)
2. Use Value Objects where appropriate (Quantity, Temperature, DateRange)
3. Implement domain logic in entities (not just getters/setters)
4. Create Repository interfaces (IRepository<T>)
5. Follow DDD patterns

STRUCTURE:
- ergon.Domain/
  - Entities/
    - Batch.cs
    - Recipe.cs
    - Material.cs
  - ValueObjects/
    - Quantity.cs
    - Temperature.cs
    - BatchStatus.cs
  - Interfaces/
    - IBatchRepository.cs
    - IRecipeRepository.cs
  - Enums/
    - BatchStatusEnum.cs
    - RecipeSteEnum.cs

SPECIFICATIONS:
- Batch entity must support state machine: 
  CREATED → PLANNED → READY → RUNNING → PAUSED → COMPLETED → PASSED/REJECTED → CLOSED
- Recipe contains steps with parameters (temperature, pressure, etc.)
- Material has quantity with unit (kg, liters, pcs)
- Implement fluent validation rules
- Add domain events (BatchCreated, BatchStarted, BatchCompleted)

OUTPUT:
- Well-structured, documented C# entity classes
- Proper use of access modifiers (private setters, init-only properties)
- Constructor validation
- XML documentation comments
```

**Reference:** docs/05_DATA_MODEL.md, docs/04_PROCESS_FLOWS.md

---

## 📝 PROMPT 2: API Controllers & DTOs

**Kontext:** Vytvořit REST API controllers pro batch management

```
TASK: Generate REST API Controllers for Batch Management

REQUIREMENTS:
1. Create BatchController with CRUD operations
2. Create DTOs (CreateBatchRequest, UpdateBatchRequest, BatchResponse)
3. Implement proper HTTP status codes
4. Add OpenAPI/Swagger attributes
5. Implement authorization checks

ENDPOINTS:
POST   /api/batches                      - Create batch
GET    /api/batches/{id}                 - Get batch
PUT    /api/batches/{id}                 - Update batch
DELETE /api/batches/{id}                 - Delete batch
GET    /api/batches?status=RUNNING&line_id=1  - List with filters
POST   /api/batches/{id}/start           - Start batch execution
POST   /api/batches/{id}/complete        - Mark batch complete
POST   /api/batches/{id}/assign-line     - Assign batch to line
GET    /api/batches/{id}/materials       - Get batch materials

STANDARDS:
- [Authorize(Roles = "PLANNER,SUPERVISOR")] for create/update
- [Authorize(Roles = "OPERATOR,SUPERVISOR")] for start/complete
- [Authorize] for get operations
- Return proper HTTP codes: 200, 201, 204, 400, 401, 403, 404, 409
- Include correlation IDs for tracing
- Proper error response format

OUTPUT:
- BatchController.cs
- BatchDtos.cs (CreateBatchRequest, UpdateBatchRequest, BatchResponse)
- Validation rules using FluentValidation
```

**Reference:** docs/03_FUNCTIONAL_REQUIREMENTS.md

---

## 📝 PROMPT 3: Application Services & Use Cases

**Kontext:** Vytvořit business logic layer s use cases

```
TASK: Generate Application Layer Services

REQUIREMENTS:
1. Create BatchService (Application service)
2. Implement use cases as separate classes
3. Add dependency injection
4. Implement error handling & validation
5. Add logging

USE CASES TO IMPLEMENT:
- CreateBatchUseCase
- StartBatchUseCase
- CompleteBatchUseCase
- AssignBatchToLineUseCase
- ScanMaterialUseCase
- GetBatchStatusUseCase

STRUCTURE:
- ergon.Application/
  - Services/
    - BatchService.cs
  - UseCases/
    - CreateBatchUseCase.cs
    - StartBatchUseCase.cs
    - etc.
  - Validators/
    - CreateBatchValidator.cs
  - Mappers/
    - BatchMapper.cs

PATTERNS:
- Dependency Injection (IServiceCollection)
- Try-catch with proper exception types
- FluentValidation for input validation
- ILogger for structured logging
- Result<T> pattern for success/failure handling
- Transaction management for multi-step operations

OUTPUT:
- BatchService.cs (orchestrating use cases)
- UseCases/ folder with individual use case classes
- Service registration extension (ServiceCollectionExtensions.cs)
```

**Reference:** docs/04_PROCESS_FLOWS.md

---

## 📝 PROMPT 4: Database Access Layer (EF Core)

**Kontext:** Vytvořit EF Core context a repositories

```
TASK: Generate EF Core Data Access Layer

REQUIREMENTS:
1. Create DbContext (ErgonDbContext)
2. Create Repository implementations
3. Add migrations
4. Implement Unit of Work pattern
5. Optimize queries with indexes

ENTITIES TO CONFIGURE:
- DbSet<Batch>
- DbSet<Recipe>
- DbSet<RecipeStep>
- DbSet<RecipeMaterial>
- DbSet<Material>
- DbSet<Line>
- DbSet<Equipment>
- DbSet<ParameterLog>
- DbSet<Alarm>
- DbSet<AuditLog>
- DbSet<User>

REQUIREMENTS:
- Fluent API configuration (OnModelCreating)
- Shadow properties for auditing (CreatedAt, CreatedBy, UpdatedAt)
- Value conversions for enums
- Query filters for soft deletes
- Index definitions for performance
- Cascade delete rules

REPOSITORIES:
- IBatchRepository : IRepository<Batch>
- IRecipeRepository : IRepository<Recipe>
- IMaterialRepository : IRepository<Material>
- Generic IRepository<T> interface

MIGRATIONS:
- Initial migration (schema creation)
- Seed data script (test data)

OUTPUT:
- ErgonDbContext.cs
- Repository implementations
- Migration files (Up, Down scripts)
```

**Reference:** docs/05_DATA_MODEL.md

---

## 📝 PROMPT 5: OPC UA Integration Service

**Kontext:** Vytvořit OPC UA client pro real-time data komunikaci s PLC

```
TASK: Generate OPC UA Integration Service

REQUIREMENTS:
1. Create OpcUaClientService
2. Implement subscription to tags
3. Handle data validation
4. Implement reconnect logic
5. Generate alarms on deviations
6. Log data to database

ARCHITECTURE:
- OpcUaClientService (lifecycle management)
- OpcUaSubscriptionManager (tag subscriptions)
- ParameterLoggerService (data persistence)
- AlarmService (alarm generation)

FEATURES:
- Connect to OPC UA endpoint
- Subscribe to specific tags with intervals
- Handle value changes
- Data validation (min/max, data type)
- Alarm triggers (if value exceeds limits)
- Real-time logging to ParameterLog table
- Graceful disconnection & reconnect
- Exponential backoff for retries

CONFIGURATION:
- OpcUaSettings (endpoint, security policy, tags)
- Load from appsettings.json
- Use IOptions<OpcUaSettings> pattern

ERROR HANDLING:
- Connection loss → auto-reconnect
- Invalid data → skip with warning
- Service exceptions → logged but don't crash
- Circuit breaker pattern for unavailable servers

OUTPUT:
- OpcUaClientService.cs
- OpcUaSubscriptionManager.cs
- ParameterLoggerService.cs
- OpcUaSettings configuration class
- Hosted service registration (BackgroundService)
```

**Reference:** docs/06_SYSTEM_INTEGRATION.md#1-opc-ua-integrace

---

## 📝 PROMPT 6: ERP Integration Service

**Kontext:** Vytvořit adapter pro synchronizaci s ERP systémem

```
TASK: Generate ERP Integration Service (Navision/SAP/Odoo)

REQUIREMENTS:
1. Create ErpSyncService
2. Implement polling/push mechanisms
3. Handle data transformations
4. Implement retry & rollback logic
5. Log all synchronizations

ENDPOINTS TO SYNC:
- Manufacturing Orders (pull from ERP)
- Recipes (pull from ERP)
- Materials (pull from ERP)
- Batch completion status (push to ERP)

ARCHITECTURE:
- IErpProvider (abstraction)
- NavisionErpProvider
- SapErpProvider
- OdooErpProvider
- ErpSyncService (orchestrator)
- ErpSyncRepository (track sync history)

SYNC LOGIC:
- Poll manufacturing orders every 5 minutes
- Compare with local DB (detect new/updated)
- Create batches for new MOs automatically
- On batch completion: push status back to ERP
- Retry failed syncs with exponential backoff
- Log all operations to AuditLog

CONFIGURATION:
{
  "erp": {
    "type": "navision",
    "enabled": true,
    "endpoint": "https://erp.assortis.cz/odata/",
    "username": "${ERP_USER}",
    "password": "${ERP_PASSWORD}",
    "sync_intervals": {
      "manufacturing_orders": 300,  // seconds
      "recipes": 3600,
      "materials": 3600
    }
  }
}

ERROR HANDLING:
- ERP unavailable → queue locally, retry later
- Data conflict → log and alert admin
- Duplicate orders → detect and skip
- Network errors → exponential backoff

OUTPUT:
- ErpSyncService.cs
- IErpProvider.cs & implementations
- ErpSyncRepository.cs
- Data mapping/transformation logic
- Hosted service for polling
```

**Reference:** docs/06_SYSTEM_INTEGRATION.md#2-erp-integrace

---

## 📝 PROMPT 7: Authentication & Authorization

**Kontext:** Vytvořit JWT-based auth a RBAC systém

```
TASK: Generate Authentication & Authorization Services

REQUIREMENTS:
1. Create AuthService (login, token generation)
2. Implement JWT token mechanism
3. Create RBAC (Role-Based Access Control)
4. Add password hashing (bcrypt)
5. Implement session management

COMPONENTS:
- AuthService (login/logout/refresh)
- JwtTokenService (token generation & validation)
- PasswordHashService (bcrypt)
- RoleService (role & permission management)
- AuthController (public endpoints)

FEATURES:
- Login with username/password
- JWT token generation (access + refresh tokens)
- Token expiration (access: 1h, refresh: 7d)
- Refresh token endpoint
- Logout (invalidate refresh tokens)
- Password change endpoint
- Multi-factor authentication (optional)
- Account lockout (5 failed attempts = 15 min)

AUTHORIZATION MIDDLEWARE:
- [Authorize] attribute for protected endpoints
- [Authorize(Roles = "ADMIN,PLANNER")] for role-based
- [Authorize(Policy = "CanEditRecipes")] for policy-based
- User context extracted from JWT

DATA STRUCTURES:
- JWT claims: sub, username, role, permissions, line_ids
- Refresh token stored in DB with expiration
- User roles join table

CONFIGURATION:
{
  "jwt": {
    "secret": "${JWT_SECRET}",  # min 32 chars
    "issuer": "https://ergon.assortis.cz",
    "audience": "ergon-app",
    "expiration_minutes": 60,
    "refresh_expiration_days": 7
  }
}

OUTPUT:
- AuthService.cs
- JwtTokenService.cs
- PasswordHashService.cs
- AuthController.cs
- JwtOptions configuration
- Middleware for token validation
- Extensions for DI registration
```

**Reference:** docs/02_ROLES_PERMISSIONS.md

---

## 📝 PROMPT 8: Audit Trail & Logging

**Kontext:** Vytvořit immutable audit log pro compliance

```
TASK: Generate Audit Trail & Logging Service

REQUIREMENTS:
1. Create AuditLogService
2. Implement automatic logging via middleware
3. Track all entity changes
4. Create audit trail queries
5. Ensure immutability

AUDIT EVENTS TO TRACK:
- User login/logout
- Entity creation/update/delete
- Batch state changes
- Material scans
- Parameter changes (OPC UA)
- Authorization denials
- Admin actions

LOGGED FIELDS:
- Timestamp
- User ID & username
- Action (CREATE, UPDATE, DELETE, LOGIN, etc.)
- Entity type & ID
- Old value → New value (for updates)
- IP address
- User agent
- Request ID (correlation)

ARCHITECTURE:
- AuditLogService (log creation)
- AuditInterceptor (EF Core interceptor)
- AuditMiddleware (HTTP request logging)
- AuditRepository (queries)

FEATURES:
- Automatic logging for all DbContext changes
- HTTP request/response logging
- Sensitive data masking (passwords, tokens)
- Immutable storage (append-only table)
- Efficient archivation (move old records to archive table)
- Compliance report generation

QUERIES:
GET /api/audit/log?entity_type=BATCH&entity_id=123
GET /api/audit/actions?user_id=5&action=UPDATE&date_from=...
GET /api/audit/user-actions/5

OUTPUT:
- AuditLogService.cs
- AuditInterceptor.cs
- AuditMiddleware.cs
- AuditRepository.cs
- AuditLogController.cs
- AuditLog entity
```

**Reference:** docs/02_ROLES_PERMISSIONS.md, docs/07_NON_FUNCTIONAL_REQUIREMENTS.md

---

## 📝 PROMPT 9: Unit & Integration Testing

**Kontext:** Vytvořit test suite s high coverage

```
TASK: Generate Unit & Integration Tests

REQUIREMENTS:
1. Create xUnit test projects
2. Mock external dependencies
3. Write unit tests for services
4. Write integration tests for APIs
5. Target >80% code coverage

TEST STRUCTURE:
- ergon.UnitTests/
  - Services/
    - BatchServiceTests.cs
    - AuthServiceTests.cs
    - OpcUaClientServiceTests.cs
  - Repositories/
    - BatchRepositoryTests.cs
- ergon.IntegrationTests/
  - Controllers/
    - BatchControllerTests.cs
  - Integration/
    - ErpSyncTests.cs
    - OpcUaIntegrationTests.cs

TESTING TOOLS:
- xUnit (test framework)
- Moq (mocking)
- FluentAssertions (assertions)
- TestContainers (Docker-based integration tests)
- AutoFixture (test data builders)

TEST EXAMPLES:

1. Unit Test: BatchService.CreateBatch
   - Happy path: valid input → batch created
   - Error path: invalid recipe → exception
   - Edge case: duplicate batch number → exception

2. Integration Test: POST /api/batches
   - Create request with JWT token
   - Verify 201 response
   - Check batch in database
   - Verify audit log recorded

3. OPC UA Test (mocked):
   - Mock OPC UA client
   - Send tag updates
   - Verify data logged to DB
   - Test alarm trigger

OUTPUT:
- Test classes for all services & controllers
- Test fixtures & common utilities
- TestFactory for creating test data
- CI pipeline (GitHub Actions) to run tests
- Coverage report (Codecov integration)
```

**Reference:** docs/07_NON_FUNCTIONAL_REQUIREMENTS.md#tst-01-coverage

---

## 📝 PROMPT 10: Windows Service & Deployment

**Kontext:** Vytvořit deployment a instalaci pro pozadní službu a desktop aplikace

```
TASK: Generate Windows Service and Desktop App Deployment Configuration

REQUIREMENTS:
1. Create a .NET Worker Service or Windows Service project
2. Create Windows desktop client deployment packages
3. Define installation and update flow for desktop apps
4. Setup health checks and monitoring for the background service
5. Create CI/CD pipeline for building service and desktop installers

DEPLOYMENT ARTIFACTS:
- Windows Service installer/package
- Desktop app installer/package (MSIX, MSI, or ClickOnce)
- Configuration files for service and clients
- Scripts for installing/updating Windows Service
- Health check endpoint or monitoring probes

HEALTH CHECKS:
- Service status monitor
- Database connectivity check
- OPC UA connectivity check
- Log/metrics export

CI/CD PIPELINE:
- Lint code
- Run unit tests
- Build service and desktop installers
- Publish artifacts
- Deploy to staging
- Run integration tests
- Deploy to production (manual approval)

OUTPUT:
- Service deployment scripts
- Desktop application installer definition
- .github/workflows/ or similar CI config
- Release packaging instructions
```

**Reference:** docs/07_NON_FUNCTIONAL_REQUIREMENTS.md#dep-01-versioning

---

## 🎨 CODE STYLE & STANDARDS

### Naming Conventions
```csharp
// Classes & Interfaces
public class BatchService { }
public interface IBatchRepository { }

// Methods
public async Task<Batch> GetBatchAsync(int id) { }
public void LogAuditEvent(string action, string entity) { }

// Properties
public string BatchNumber { get; set; }
public bool IsActive { get; private set; }

// Local variables
var batchId = 123;
var isValid = true;

// Constants
public const int MaxBatchSize = 1000;
private const string DefaultTimeZone = "UTC";

// Async methods always end with "Async"
public async Task<Batch> CreateBatchAsync(...) { }
```

### Project Structure
```
ergon.sln
├── ergon.Api/                    # Presentation Layer
│   ├── Controllers/
│   ├── Middleware/
│   └── appsettings.json
├── ergon.Application/            # Application Services
│   ├── Services/
│   ├── UseCases/
│   ├── Validators/
│   └── Mappers/
├── ergon.Domain/                 # Domain Layer
│   ├── Entities/
│   ├── ValueObjects/
│   ├── Interfaces/
│   └── Exceptions/
├── ergon.Infrastructure/         # Infrastructure Layer
│   ├── Data/
│   │   ├── ErgonDbContext.cs
│   │   └── Repositories/
│   ├── ExternalServices/
│   │   ├── OpcUa/
│   │   └── Erp/
│   └── Logging/
├── ergon.UnitTests/
├── ergon.IntegrationTests/
└── docs/                         # Documentation
```

### Async/Await Pattern
```csharp
// ✅ Good
public async Task<Batch> GetBatchAsync(int id)
{
    return await _repository.GetByIdAsync(id);
}

// ✅ Good - with cancellation token
public async Task<Batch> ProcessBatchAsync(int id, 
    CancellationToken cancellationToken)
{
    return await _service.ProcessAsync(id, cancellationToken);
}

// ❌ Bad - synchronous wrapper
public Batch GetBatch(int id)
{
    return _repository.GetByIdAsync(id).Result;  // DEADLOCK RISK!
}
```

### Error Handling
```csharp
// ✅ Good - specific exceptions
try
{
    var batch = await _repository.GetByIdAsync(id);
    if (batch == null)
        throw new EntityNotFoundException($"Batch {id} not found");
}
catch (EntityNotFoundException ex)
{
    _logger.LogWarning($"Batch not found: {id}");
    return NotFound(ex.Message);
}
catch (DbException ex)
{
    _logger.LogError($"Database error: {ex.Message}");
    return StatusCode(500, "Database error");
}

// ❌ Bad - generic catch
catch (Exception ex)
{
    // Too broad!
}
```

### Logging
```csharp
// ✅ Good - structured logging
_logger.LogInformation(
    "Batch {batchId} created by user {userId}",
    batch.Id, userId);

// ✅ Good - with exception
_logger.LogError(ex,
    "Failed to process batch {batchId}",
    batch.Id);

// ❌ Bad - string concatenation
_logger.LogInformation("Batch " + batch.Id + " created");
```

---

## ✅ VALIDATION CHECKLIST

Když AI generuje kód, ověř:

- [ ] Následuje Clean Architecture
- [ ] Používá async/await pattern
- [ ] Má XML documentation comments
- [ ] Korektní error handling
- [ ] Validace vstupů (FluentValidation)
- [ ] Dependency Injection registrace
- [ ] Jednotkové testy (>80%)
- [ ] OWASP bezpečnost
- [ ] SQL injection prevention (EF Core)
- [ ] Sensitive data maskování v logs
- [ ] Proper HTTP status codes
- [ ] API versioning (v1/, v2/)
- [ ] Cancellation token support

---

## 📞 Quick Reference

| Komponenta | Location | Tech |
|---|---|---|
| JWT Token | AuthService | Nuget: System.IdentityModel.Tokens.Jwt |
| Database | ErgonDbContext | EF Core, MariaDB |
| Validation | Validators/ | FluentValidation |
| Logging | Middleware | Serilog |
| OPC UA | OpcUaClientService | OpcUa.UA |
| Testing | *Tests/ | xUnit, Moq, TestContainers |
| API Docs | Swagger | Swashbuckle |

---

**Datum vytvoření:** 10.4.2026  
**Status:** Iniciace projektu  
**Verze:** 0.1  
**Target:** AI code generation prompts
