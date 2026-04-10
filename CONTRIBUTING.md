# Contributing to ergon+ MES

Děkujeme za váš zájem přispět k ergon+ MES projektu! 🚀

---

## 📖 Než začnete

1. **Přečtěte si dokumentaci** v `/docs/`
   - [01_VISION_SCOPE.md](docs/01_VISION_SCOPE.md) - pochopte projekt
   - [03_FUNCTIONAL_REQUIREMENTS.md](docs/03_FUNCTIONAL_REQUIREMENTS.md) - co budujeme
   - [AI_PROMPTS.md](docs/AI_PROMPTS.md) - jak generovat kód

2. **Forkněte repozitář** a naklonujte si ho
   ```bash
   git clone https://github.com/assortis/ergon-mes.git
   cd ergon-mes
   ```

3. **Setup vývojového prostředí** (bude v DEVELOPMENT.md)

---

## 🔀 Workflow

### 1. Vytvořte feature branch
```bash
git checkout -b feature/your-feature-name
# Příklady:
# feature/batch-creation-api
# feature/opc-ua-client
# feature/user-authentication
```

### 2. Commit message format

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Příklady:**
```bash
feat(batch): add batch creation API endpoint

docs(roadmap): update v1.0 timeline to June

fix(auth): resolve JWT token expiration issue

test(integration): add OPC UA client integration tests

refactor(domain): improve Batch entity design
```

**Typy:**
- `feat`: Nová features
- `fix`: Bug fixes
- `docs`: Dokumentace
- `style`: Code formatting (bez logických změn)
- `refactor`: Code restructuring
- `test`: Test přídání
- `ci`: CI/CD changes
- `chore`: Ostatní (upgrade deps, etc.)

### 3. Pushujte na svou branch
```bash
git push origin feature/your-feature-name
```

### 4. Otevřete Pull Request (PR)

**PR Title Format:**
```
[MODULE] Brief description

Example:
[API] Add batch status update endpoint
[OPC-UA] Implement tag subscription with reconnection
[TEST] Add integration tests for authentication
```

**PR Template:**
```markdown
## Description
Detailní popis toho, co změna dělá.

## Related Issue
Closes #123 (pokud existuje GitHub issue)

## Type of Change
- [ ] Bug fix (non-breaking change that fixes an issue)
- [ ] New feature (non-breaking change that adds functionality)
- [ ] Breaking change (fix or feature that would cause existing functionality to change)
- [ ] Documentation update

## Testing
Popis jak jste testovali.

## Checklist
- [ ] Moje kód sleduje code style guidelines
- [ ] Spustil jsem unit testy (>80% coverage)
- [ ] Spustil jsem integration testy
- [ ] Aktualizoval jsem relevantní dokumentaci
- [ ] Žádné breaking changes
- [ ] PR je self-contained (jedna feature/bug)
```

---

## 💻 Code Style & Standards

### Language: C# (.NET 8)

```csharp
// ✅ Good - Clear, follows naming conventions
public async Task<Result<BatchResponse>> CreateBatchAsync(
    CreateBatchRequest request,
    CancellationToken cancellationToken)
{
    var validation = _validator.Validate(request);
    if (!validation.IsValid)
        return Result<BatchResponse>.Failure(validation.Errors);

    var batch = new Batch(request.MoId, request.LineId, request.RecipeId);
    await _repository.AddAsync(batch, cancellationToken);
    
    _logger.LogInformation(
        "Batch {batchId} created by user {userId}",
        batch.Id, _currentUser.Id);
    
    return Result<BatchResponse>.Success(_mapper.Map<BatchResponse>(batch));
}

// ❌ Bad - No error handling, bad naming
public Batch CreateBatch(CreateBatchRequest request)
{
    var b = new Batch();
    b.moId = request.mo_id;
    _repository.Add(b);
    return b;
}
```

### Key Guidelines

1. **Async/Await Pattern**
   ```csharp
   // ✅ Good
   public async Task<Batch> GetBatchAsync(int id)
   {
       return await _repository.GetByIdAsync(id);
   }
   
   // ✅ Good - with CancellationToken
   public async Task ProcessAsync(CancellationToken cancellationToken)
   {
       await _service.DoWorkAsync(cancellationToken);
   }
   
   // ❌ Bad - Blocking call
   result = asyncMethod().Result;
   ```

2. **Error Handling**
   ```csharp
   // ✅ Good - Specific exceptions
   try
   {
       var batch = await _repository.GetByIdAsync(id);
       if (batch == null)
           throw new EntityNotFoundException($"Batch {id} not found");
   }
   catch (EntityNotFoundException ex)
   {
       _logger.LogWarning($"Batch not found: {id}");
       return NotFound(new { error = ex.Message });
   }
   catch (DbException ex)
   {
       _logger.LogError(ex, "Database error occurred");
       return StatusCode(500, "Internal server error");
   }
   
   // ❌ Bad - Too generic
   catch (Exception) { }
   ```

3. **Logging**
   ```csharp
   // ✅ Good - Structured logging
   _logger.LogInformation(
       "Created batch {batchId} for order {moId}",
       batch.Id, batch.MoId);
   
   _logger.LogError(ex, "Failed to process batch {batchId}", batchId);
   
   // ❌ Bad
   _logger.LogInformation("Created batch " + batch.Id);
   ```

4. **Naming**
   ```csharp
   // ✅ Good
   public class BatchService { }
   public interface IBatchRepository { }
   public async Task<Batch> GetBatchAsync(int id) { }
   public const int MaxBatchSize = 1000;
   
   // ❌ Bad
   public class batchService { }
   public class Batch_Repository { }
   public Task<Batch> get_batch(int id) { }
   public const int max = 1000;
   ```

### File Organization

```
src/
├── ergon.Api/                   # Presentation Layer
│   ├── Controllers/
│   │   ├── BatchController.cs
│   │   └── RecipeController.cs
│   ├── Models/
│   │   ├── Requests/
│   │   │   └── CreateBatchRequest.cs
│   │   └── Responses/
│   │       └── BatchResponse.cs
│   └── Program.cs
│
├── ergon.Application/           # Application Services Layer
│   ├── Services/
│   │   └── BatchService.cs
│   ├── UseCases/
│   │   ├── CreateBatchUseCase.cs
│   │   └── StartBatchUseCase.cs
│   └── Validators/
│       └── CreateBatchValidator.cs
│
├── ergon.Domain/                # Domain Layer
│   ├── Entities/
│   │   ├── Batch.cs
│   │   └── Recipe.cs
│   ├── ValueObjects/
│   │   ├── Quantity.cs
│   │   └── Temperature.cs
│   └── Exceptions/
│       └── EntityNotFoundException.cs
│
└── ergon.Infrastructure/        # Infrastructure Layer
    ├── Data/
    │   ├── ErgonDbContext.cs
    │   └── Repositories/
    │       └── BatchRepository.cs
    └── ExternalServices/
        └── OpcUa/
            └── OpcUaClientService.cs
```

---

## 🧪 Testing

### Unit Tests

```csharp
[Fact]
public async Task CreateBatch_WithValidRequest_ShouldCreateBatch()
{
    // Arrange
    var request = new CreateBatchRequest
    {
        MoId = "MO-001",
        LineId = 1,
        RecipeId = 1,
        Quantity = 100
    };
    
    var mockRepository = new Mock<IBatchRepository>();
    var service = new BatchService(mockRepository.Object);
    
    // Act
    var result = await service.CreateBatchAsync(request);
    
    // Assert
    Assert.NotNull(result);
    mockRepository.Verify(r => r.AddAsync(It.IsAny<Batch>()), Times.Once);
}

[Fact]
public async Task CreateBatch_WithInvalidRecipe_ShouldFail()
{
    // Arrange
    var request = new CreateBatchRequest { RecipeId = 999 };
    var mockRepository = new Mock<IBatchRepository>();
    mockRepository
        .Setup(r => r.GetRecipeAsync(999))
        .ReturnsAsync((Recipe)null);
    
    var service = new BatchService(mockRepository.Object);
    
    // Act & Assert
    var exception = await Assert.ThrowsAsync<EntityNotFoundException>(
        () => service.CreateBatchAsync(request));
    
    Assert.Contains("999", exception.Message);
}
```

### Running Tests

```bash
# Run all tests
dotnet test

# Run with coverage
dotnet test /p:CollectCoverage=true /p:CoverageThreshold=80

# Run specific test file
dotnet test ergon.UnitTests/Services/BatchServiceTests.cs

# Watch mode
dotnet watch test
```

### Coverage Requirements

- **Minimum coverage:** 80% for application code
- **Unit tests:** Required for all business logic
- **Integration tests:** Required for critical workflows
- **E2E tests:** Required before production release

---

## 📚 Using AI for Code Generation

Projekt poskytuje detailní **AI prompts** v [docs/AI_PROMPTS.md](docs/AI_PROMPTS.md).

### Workflow:

1. **Přečtěte si relevantní dokumentaci:**
   - [03_FUNCTIONAL_REQUIREMENTS.md](docs/03_FUNCTIONAL_REQUIREMENTS.md) - co buduje
   - [05_DATA_MODEL.md](docs/05_DATA_MODEL.md) - dataové struktury
   - [AI_PROMPTS.md](docs/AI_PROMPTS.md) - konkrétní AI prompt

2. **Zkopírujte příslušný prompt:**
   ```
   Sekce "PROMPT 1: Entity & Domain Layer"
   nebo
   "PROMPT 2: API Controllers & DTOs"
   ```

3. **Pošlete ho AI modeling** (ChatGPT 4, Claude, etc.)

4. **Review generated code:**
   - Ověřte, že sleduje vůdčí principy
   - Spusťte unit testy (80%+ coverage)
   - Zkontrolujte XML dokumentaci

5. **Push na feature branch** podle workflow výše

---

## 🔍 Code Review Process

### Co bude rewiewer hledat:

1. ✅ **Architektura**
   - Dodržuje Clean Architecture?
   - Správné vrstvení (API → App → Domain → Infra)?

2. ✅ **Code Quality**
   - Sleduje SOLID principy?
   - Dobrá jména proměnných?
   - Nema duplicitního kódu?

3. ✅ **Testing**
   - >80% test coverage?
   - Testovány edge cases?

4. ✅ **Dokumentace**
   - XML comments na metodách?
   - README aktualizován?

5. ✅ **Bezpečnost**
   - Input validation?
   - SQL injection prevention?
   - Authentication checks?

### Jak reagovat na feedback

```bash
# 1. Proveďte změny
vim src/MyFeature.cs

# 2. Commitněte s pravděpodobností
git add .
git commit -m "refactor(batch): address review feedback"

# 3. Pushujte (force push není potřeba!)
git push origin feature/your-feature-name

# 4. Přidejte komentář v PR detailing changes
```

---

## 📄 Documentation

Každá feature musí mít dokumentaci:

### API Endpoints
```markdown
### POST /api/batches

Create a new batch.

**Request:**
```json
{
  "moId": "MO-001",
  "lineId": 1,
  "recipeId": 1,
  "quantity": 100
}
```

**Response:**
- 201 Created
```json
{
  "id": 1,
  "batchNumber": "B-2026-001",
  "status": "CREATED"
}
```

**Authorization:** Roles: PLANNER, SUPERVISOR
**Error codes:** 400, 401, 409
```

### Domain Changes
Aktualizujte [docs/05_DATA_MODEL.md](docs/05_DATA_MODEL.md) s novými entitami/poli.

### Features
Aktualizujte [docs/03_FUNCTIONAL_REQUIREMENTS.md](docs/03_FUNCTIONAL_REQUIREMENTS.md).

---

## 🚀 Deployment

### Local Development
```bash
# Setup Docker environment
docker-compose up -d

# Run migrations
dotnet ef database update

# Start application
dotnet run --project src/ergon.Api
```

### Staging
PR s hlavní branch automaticky deployuje na staging (bude CI/CD setup).

### Production
Merge do `main` vyžaduje:
- ✅ Všechny testy projdou
- ✅ Code review approval
- ✅ Alespoň 2 approvals

---

## 🆘 Když si nejste jisti

- **Discord/Slack:** Napište do #ergon-mes kanálu
- **GitHub Discussions:** Otevřete diskusi v repo
- **Issues:** Vytvořte issue s `[QUESTION]` tagem

---

## 📋 Checklist předtím, než submitnete PR

- [ ] Aktualizuji main branch (`git pull origin main`)
- [ ] Vytvoříl jsem feature branch (`git checkout -b feature/...`)
- [ ] Kód sleduje code style (SOLID, async/await, error handling)
- [ ] Xml komentáře na veřejných metodách
- [ ] Unit testy napsaný (80%+ coverage)
- [ ] Integration testy napsaný pro workflows
- [ ] Všechny testy projít lokálně
- [ ] Žádné breaking changes bez dokumentace
- [ ] Dokumentace aktualizovaná (docs/ a README)
- [ ] Závislostoměty aktualizovány (pokud je potřeba)

---

## 🙏 Děkujeme!

Vaší přispívání činí ergon+ lepší.

**Hodně štěstí s kódováním! 🚀**

---

**Poslední aktualizace:** 10.4.2026  
**Verze:** 0.1
