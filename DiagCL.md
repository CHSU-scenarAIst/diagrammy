# UML Диаграмма Классов — DiplomDb System

## Условные обозначения

| Символ | Значение |
|--------|----------|
| `+` | public |
| `-` | private |
| `#` | protected |
| `~` | internal |
| `<<abstract>>` | абстрактный класс |
| `<<interface>>` | интерфейс |
| `<<static>>` | статический класс |
| `*` | множество (0..* или 1..*) |
| `0..1` | ноль или один |
| `1` | ровно один |

---

## UML Class Diagram

```mermaid
classDiagram
    %% ============================================
    %% DOMAIN LAYER — Entities
    %% ============================================

    class BaseEntity {
        <<abstract>>
        +Guid Id*
        +DateTime CreatedAt*
        +DateTime UpdatedAt*
        +bool IsDeleted*
    }

    class ActionEntity {
        +string Name*
        +ICollection~StepEntity~ Steps*
        +ActionEntity Create(string name)*
        -ActionEntity()
    }

    class ScenarioEntity {
        +Guid? ParentScenarioId*
        +string? UserRequest*
        +JsonDocument? LlmContext*
        +int ChildCount*
        +ScenarioEntity? ParentScenario*
        +ICollection~ScenarioEntity~ ChildScenarios*
        +ICollection~ScenarioStepEntity~ ScenarioSteps*
        +ICollection~SessionEntity~ Sessions*
        +ScenarioEntity Create(Guid? parentScenarioId, string? userRequest, JsonDocument? llmContext)*
        -ScenarioEntity()
    }

    class StepEntity {
        +Guid ActionId*
        +Guid ObjectId*
        +ActionEntity Action*
        +ObjectEntity Object*
        +ICollection~ScenarioStepEntity~ ScenarioSteps*
        +StepEntity Create(Guid actionId, Guid objectId)*
        -StepEntity()
    }

    class ScenarioStepEntity {
        +Guid ScenarioId*
        +Guid StepId*
        +int Order*
        +ScenarioEntity Scenario*
        +StepEntity Step*
        +ScenarioStepEntity Create(Guid scenarioId, Guid stepId, int order)*
        -ScenarioStepEntity()
    }

    class ObjectEntity {
        +string Name*
        +ICollection~StepEntity~ Steps*
        +ObjectEntity Create(string name)*
        -ObjectEntity()
    }

    class SessionEntity {
        +Guid ScenarioId*
        +string CourseName*
        +ScenarioEntity Scenario*
        +SessionEntity Create(Guid scenarioId, string courseName)*
        -SessionEntity()
    }

    %% ============================================
    %% DOMAIN LAYER — Repository Interfaces
    %% ============================================

    class IActionRepository {
        <<interface>>
    }

    class IScenarioRepository {
        <<interface>>
    }

    class IStepRepository {
        <<interface>>
    }

    class IScenarioStepRepository {
        <<interface>>
    }

    class IObjectRepository {
        <<interface>>
    }

    class ISessionRepository {
        <<interface>>
    }

    %% ============================================
    %% DOMAIN LAYER — Specifications
    %% ============================================

    class ActiveEntitiesSpec~T~ {
        +ActiveEntitiesSpec()
    }

    class ActionsByIdsSpec {
        +ActionsByIdsSpec(IEnumerable~Guid~ ids)
    }

    class ScenariosWithActionsSpec {
        +ScenariosWithActionsSpec()
    }

    class ScenariosByParentIdSpec {
        +ScenariosByParentIdSpec(Guid? parentId)
    }

    class StepsByActionIdSpec {
        +StepsByActionIdSpec(Guid actionId)
    }

    class ObjectsByNameSpec {
        +ObjectsByNameSpec(string name)
    }

    class SessionsByScenarioIdSpec {
        +SessionsByScenarioIdSpec(Guid scenarioId)
    }

    class ScenarioStepsByScenarioIdSpec {
        +ScenarioStepsByScenarioIdSpec(Guid scenarioId)
    }

    class StepsWithActionsAndObjectsSpec {
        +StepsWithActionsAndObjectsSpec()
        +StepsWithActionsAndObjectsSpec(Guid id)
    }

    %% ============================================
    %% DATA ACCESS LAYER
    %% ============================================

    class ApplicationDbContext {
        +DbSet~ActionEntity~ Actions*
        +DbSet~ScenarioEntity~ Scenarios*
        +DbSet~ObjectEntity~ Objects*
        +DbSet~StepEntity~ Steps*
        +DbSet~SessionEntity~ Sessions*
        +DbSet~ScenarioStepEntity~ ScenarioSteps*
        #void OnModelCreating(ModelBuilder modelBuilder)
        +Task~int~ SaveChangesAsync(CancellationToken cancellationToken)
        +int SaveChanges()
        +void Save()
    }

    class BeautifulDataAccess {
        <<static>>
        +IServiceCollection AddBeautifulDataAccess(IServiceCollection services, IConfiguration configuration)
        +IServiceCollection MakeDbContextInMemory(IServiceCollection services)
        -void WithSeedData(ApplicationDbContext context)
        -IServiceCollection WithDependencies(IServiceCollection services)
    }

    class BaseEntityConfiguration~T~ {
        <<abstract>>
        #void Configure(EntityTypeBuilder~T~ builder)
    }

    class ActionConfiguration {
        ~void Configure(EntityTypeBuilder~ActionEntity~ builder)
    }

    class ObjectConfiguration {
        ~void Configure(EntityTypeBuilder~ObjectEntity~ builder)
    }

    class ScenarioConfiguration {
        ~void Configure(EntityTypeBuilder~ScenarioEntity~ builder)
    }

    class StepConfiguration {
        ~void Configure(EntityTypeBuilder~StepEntity~ builder)
    }

    class SessionConfiguration {
        ~void Configure(EntityTypeBuilder~SessionEntity~ builder)
    }

    class ScenarioStepConfiguration {
        ~void Configure(EntityTypeBuilder~ScenarioStepEntity~ builder)
    }

    class ActionRepository {
        ~ActionRepository(ApplicationDbContext context)
    }

    class ObjectRepository {
        ~ObjectRepository(ApplicationDbContext context)
    }

    class ScenarioRepository {
        ~ScenarioRepository(ApplicationDbContext context)
    }

    class StepRepository {
        ~StepRepository(ApplicationDbContext context)
    }

    class ScenarioStepRepository {
        ~ScenarioStepRepository(ApplicationDbContext context)
    }

    class SessionRepository {
        ~SessionRepository(ApplicationDbContext context)
    }

    %% ============================================
    %% DTO LAYER — Requests
    %% ============================================

    class CreateScenarioRequest {
        <<record>>
        +string Name*
        +List~Guid~ ActionIds*
    }

    class CreateActionRequest {
        <<record>>
        +string Name*
    }

    class CreateObjectRequest {
        <<record>>
        +string Name*
    }

    class CreateStepRequest {
        <<record>>
        +Guid ActionId*
        +Guid ObjectId*
    }

    class CreateScenarioStepRequest {
        <<record>>
        +Guid ScenarioId*
        +Guid StepId*
        +int Order*
    }

    class CreateSessionRequest {
        <<record>>
        +Guid ScenarioId*
        +string CourseName*
    }

    %% ============================================
    %% DTO LAYER — Responses
    %% ============================================

    class ActionResponse {
        <<record>>
        +Guid Id*
        +string Name*
    }

    class ScenarioResponse {
        <<record>>
        +Guid Id*
        +string Name*
        +List~ActionResponse~ Actions*
    }

    class ObjectResponse {
        <<record>>
        +Guid Id*
        +string Name*
    }

    class StepResponse {
        <<record>>
        +Guid Id*
        +Guid ActionId*
        +Guid ObjectId*
        +ActionResponse? Action*
        +ObjectResponse? Object*
    }

    class ScenarioStepResponse {
        <<record>>
        +Guid Id*
        +Guid ScenarioId*
        +Guid StepId*
        +int Order*
        +ScenarioResponse? Scenario*
        +StepResponse? Step*
    }

    class SessionResponse {
        <<record>>
        +Guid Id*
        +Guid ScenarioId*
        +string CourseName*
        +ScenarioResponse? Scenario*
    }

    %% ============================================
    %% DTO LAYER — Validators
    %% ============================================

    class IValidationMarker {
        <<interface>>
    }

    class CreateScenarioValidator {
        ~CreateScenarioValidator()
    }

    %% ============================================
    %% API LAYER — Services (Interfaces)
    %% ============================================

    class IBaseService~TEntity, TRequest, TResponse~ {
        <<interface>>
        +Task~IEnumerable~TResponse~~ GetAllAsync(CancellationToken cancellationToken)
        +Task~TResponse?~ GetByIdAsync(Guid id, CancellationToken cancellationToken)
        +Task~TResponse~ CreateAsync(TRequest request, CancellationToken cancellationToken)
        +Task~TResponse~ UpdateAsync(Guid id, TRequest request, CancellationToken cancellationToken)
        +Task DeleteAsync(Guid id, CancellationToken cancellationToken)
    }

    class IActionService {
        <<interface>>
        +Task~IEnumerable~ActionResponse~~ GetByIdsAsync(IEnumerable~Guid~ ids, CancellationToken cancellationToken)
    }

    class IScenarioService {
        <<interface>>
        +Task~IEnumerable~ScenarioResponse~~ GetByParentIdAsync(Guid? parentId, CancellationToken cancellationToken)
        +Task~IEnumerable~ScenarioResponse~~ GetWithActionsAsync(CancellationToken cancellationToken)
    }

    class IObjectService {
        <<interface>>
        +Task~IEnumerable~ObjectResponse~~ SearchByNameAsync(string name, CancellationToken cancellationToken)
    }

    class IStepService {
        <<interface>>
        +Task~IEnumerable~StepResponse~~ GetByActionIdAsync(Guid actionId, CancellationToken cancellationToken)
    }

    class IScenarioStepService {
        <<interface>>
        +Task~IEnumerable~ScenarioStepResponse~~ GetByScenarioIdAsync(Guid scenarioId, CancellationToken cancellationToken)
        +Task~IEnumerable~ScenarioStepResponse~~ GetOrderedByScenarioIdAsync(Guid scenarioId, CancellationToken cancellationToken)
    }

    class ISessionService {
        <<interface>>
        +Task~IEnumerable~SessionResponse~~ GetByScenarioIdAsync(Guid scenarioId, CancellationToken cancellationToken)
    }

    %% ============================================
    %% API LAYER — Services (Implementations)
    %% ============================================

    class BaseService~TEntity, TRequest, TResponse~ {
        <<abstract>>
        #IRepositoryBase~TEntity~ Repository*
        #IMapper Mapper*
        #ILogger~BaseService~ Logger*
        #BaseService(IRepositoryBase~TEntity~ repository, IMapper mapper, ILogger~BaseService~ logger)
        +Task~IEnumerable~TResponse~~ GetAllAsync(CancellationToken cancellationToken)
        +Task~TResponse?~ GetByIdAsync(Guid id, CancellationToken cancellationToken)
        +Task~TResponse~ CreateAsync(TRequest request, CancellationToken cancellationToken)*
        +Task~TResponse~ UpdateAsync(Guid id, TRequest request, CancellationToken cancellationToken)*
        +Task DeleteAsync(Guid id, CancellationToken cancellationToken)
        #Task ValidateDependenciesExistAsync(TRequest request, CancellationToken cancellationToken)
    }

    class ActionService {
        +ActionService(IActionRepository repository, IMapper mapper, ILogger~ActionService~ logger)
        +Task~ActionResponse~ CreateAsync(CreateActionRequest request, CancellationToken cancellationToken)
        +Task~ActionResponse~ UpdateAsync(Guid id, CreateActionRequest request, CancellationToken cancellationToken)
        +Task~IEnumerable~ActionResponse~~ GetByIdsAsync(IEnumerable~Guid~ ids, CancellationToken cancellationToken)
    }

    class ScenarioService {
        -IActionRepository _actionRepository*
        -IScenarioStepRepository _scenarioStepRepository*
        +ScenarioService(IScenarioRepository repository, IActionRepository actionRepository, IScenarioStepRepository scenarioStepRepository, IMapper mapper, ILogger~ScenarioService~ logger)
        +Task~ScenarioResponse~ CreateAsync(CreateScenarioRequest request, CancellationToken cancellationToken)
        +Task~ScenarioResponse~ UpdateAsync(Guid id, CreateScenarioRequest request, CancellationToken cancellationToken)
        #Task ValidateDependenciesExistAsync(CreateScenarioRequest request, CancellationToken cancellationToken)
        +Task~IEnumerable~ScenarioResponse~~ GetByParentIdAsync(Guid? parentId, CancellationToken cancellationToken)
        +Task~IEnumerable~ScenarioResponse~~ GetWithActionsAsync(CancellationToken cancellationToken)
    }

    class ObjectService {
        +ObjectService(IObjectRepository repository, IMapper mapper, ILogger~ObjectService~ logger)
        +Task~ObjectResponse~ CreateAsync(CreateObjectRequest request, CancellationToken cancellationToken)
        +Task~ObjectResponse~ UpdateAsync(Guid id, CreateObjectRequest request, CancellationToken cancellationToken)
        +Task~IEnumerable~ObjectResponse~~ SearchByNameAsync(string name, CancellationToken cancellationToken)
    }

    class StepService {
        -IActionRepository _actionRepository*
        -IObjectRepository _objectRepository*
        +StepService(IStepRepository repository, IActionRepository actionRepository, IObjectRepository objectRepository, IMapper mapper, ILogger~StepService~ logger)
        +Task~StepResponse~ CreateAsync(CreateStepRequest request, CancellationToken cancellationToken)
        +Task~StepResponse~ UpdateAsync(Guid id, CreateStepRequest request, CancellationToken cancellationToken)
        #Task ValidateDependenciesExistAsync(CreateStepRequest request, CancellationToken cancellationToken)
        +Task~IEnumerable~StepResponse~~ GetAllAsync(CancellationToken cancellationToken)
        +Task~StepResponse?~ GetByIdAsync(Guid id, CancellationToken cancellationToken)
        +Task~IEnumerable~StepResponse~~ GetByActionIdAsync(Guid actionId, CancellationToken cancellationToken)
    }

    class ScenarioStepService {
        -IScenarioRepository _scenarioRepository*
        -IStepRepository _stepRepository*
        +ScenarioStepService(IScenarioStepRepository repository, IScenarioRepository scenarioRepository, IStepRepository stepRepository, IMapper mapper, ILogger~ScenarioStepService~ logger)
        +Task~ScenarioStepResponse~ CreateAsync(CreateScenarioStepRequest request, CancellationToken cancellationToken)
        +Task~ScenarioStepResponse~ UpdateAsync(Guid id, CreateScenarioStepRequest request, CancellationToken cancellationToken)
        #Task ValidateDependenciesExistAsync(CreateScenarioStepRequest request, CancellationToken cancellationToken)
        +Task~IEnumerable~ScenarioStepResponse~~ GetByScenarioIdAsync(Guid scenarioId, CancellationToken cancellationToken)
        +Task~IEnumerable~ScenarioStepResponse~~ GetOrderedByScenarioIdAsync(Guid scenarioId, CancellationToken cancellationToken)
    }

    class SessionService {
        -IScenarioRepository _scenarioRepository*
        +SessionService(ISessionRepository repository, IScenarioRepository scenarioRepository, IMapper mapper, ILogger~SessionService~ logger)
        +Task~SessionResponse~ CreateAsync(CreateSessionRequest request, CancellationToken cancellationToken)
        +Task~SessionResponse~ UpdateAsync(Guid id, CreateSessionRequest request, CancellationToken cancellationToken)
        #Task ValidateDependenciesExistAsync(CreateSessionRequest request, CancellationToken cancellationToken)
        +Task~IEnumerable~SessionResponse~~ GetByScenarioIdAsync(Guid scenarioId, CancellationToken cancellationToken)
    }

    %% ============================================
    %% API LAYER — Controllers
    %% ============================================

    class ActionController {
        -IActionService actionService*
        +ActionController(IActionService actionService)
        +Task~ActionResult~IEnumerable~ActionResponse~~~ Get(CancellationToken token)
        +Task~ActionResult~ActionResponse~~ GetById(Guid id, CancellationToken token)
        +Task~ActionResult~IEnumerable~ActionResponse~~~ GetByIds(List~Guid~ ids, CancellationToken token)
        +Task~ActionResult~ActionResponse~~ Create(CreateActionRequest request, CancellationToken token)
        +Task~ActionResult~ActionResponse~~ Update(Guid id, CreateActionRequest request, CancellationToken token)
        +Task~IActionResult~ Delete(Guid id, CancellationToken token)
    }

    class ScenarioController {
        -IScenarioService scenarioService*
        +ScenarioController(IScenarioService scenarioService)
        +Task~ActionResult~IEnumerable~ScenarioResponse~~~ Get(CancellationToken token)
        +Task~ActionResult~IEnumerable~ScenarioResponse~~~ GetByParentId(Guid? parentId, CancellationToken token)
        +Task~ActionResult~ScenarioResponse~~ GetById(Guid id, CancellationToken token)
        +Task~ActionResult~ScenarioResponse~~ Create(CreateScenarioRequest request, CancellationToken ct)
        +Task~ActionResult~ScenarioResponse~~ Update(Guid id, CreateScenarioRequest request, CancellationToken ct)
        +Task~IActionResult~ Delete(Guid id, CancellationToken ct)
    }

    class ObjectController {
        -IObjectService objectService*
        +ObjectController(IObjectService objectService)
        +Task~ActionResult~IEnumerable~ObjectResponse~~~ Get(CancellationToken token)
        +Task~ActionResult~ObjectResponse~~ GetById(Guid id, CancellationToken token)
        +Task~ActionResult~IEnumerable~ObjectResponse~~~ SearchByName(string name, CancellationToken token)
        +Task~ActionResult~ObjectResponse~~ Create(CreateObjectRequest request, CancellationToken token)
        +Task~ActionResult~ObjectResponse~~ Update(Guid id, CreateObjectRequest request, CancellationToken token)
        +Task~IActionResult~ Delete(Guid id, CancellationToken token)
    }

    class StepController {
        -IStepService stepService*
        +StepController(IStepService stepService)
        +Task~ActionResult~IEnumerable~StepResponse~~~ Get(CancellationToken token)
        +Task~ActionResult~StepResponse~~ GetById(Guid id, CancellationToken token)
        +Task~ActionResult~IEnumerable~StepResponse~~~ GetByActionId(Guid actionId, CancellationToken token)
        +Task~ActionResult~StepResponse~~ Create(CreateStepRequest request, CancellationToken token)
        +Task~ActionResult~StepResponse~~ Update(Guid id, CreateStepRequest request, CancellationToken token)
        +Task~IActionResult~ Delete(Guid id, CancellationToken token)
    }

    class ScenarioStepController {
        -IScenarioStepService scenarioStepService*
        +ScenarioStepController(IScenarioStepService scenarioStepService)
        +Task~ActionResult~IEnumerable~ScenarioStepResponse~~~ Get(CancellationToken token)
        +Task~ActionResult~ScenarioStepResponse~~ GetById(Guid id, CancellationToken token)
        +Task~ActionResult~IEnumerable~ScenarioStepResponse~~~ GetByScenarioId(Guid scenarioId, CancellationToken token)
        +Task~ActionResult~IEnumerable~ScenarioStepResponse~~~ GetOrderedByScenarioId(Guid scenarioId, CancellationToken token)
        +Task~ActionResult~ScenarioStepResponse~~ Create(CreateScenarioStepRequest request, CancellationToken token)
        +Task~ActionResult~ScenarioStepResponse~~ Update(Guid id, CreateScenarioStepRequest request, CancellationToken token)
        +Task~IActionResult~ Delete(Guid id, CancellationToken token)
    }

    class SessionController {
        -ISessionService sessionService*
        +SessionController(ISessionService sessionService)
        +Task~ActionResult~IEnumerable~SessionResponse~~~ Get(CancellationToken token)
        +Task~ActionResult~SessionResponse~~ GetById(Guid id, CancellationToken token)
        +Task~ActionResult~IEnumerable~SessionResponse~~~ GetByScenarioId(Guid scenarioId, CancellationToken token)
        +Task~ActionResult~SessionResponse~~ Create(CreateSessionRequest request, CancellationToken token)
        +Task~ActionResult~SessionResponse~~ Update(Guid id, CreateSessionRequest request, CancellationToken token)
        +Task~IActionResult~ Delete(Guid id, CancellationToken token)
    }

    %% ============================================
    %% API LAYER — Other
    %% ============================================

    class BeautifulBuilderService {
        <<static>>
        +IServiceCollection MakeBeautifulCors(IServiceCollection services)
        +IServiceCollection MakeBeautifulSwagger(IServiceCollection services)
        +IServiceCollection MakeBeautifulValidation(IServiceCollection services)
        +IServiceCollection MakeBeautifulServices(IServiceCollection services)
    }

    class MappingProfile {
        +MappingProfile()
    }

    %% ============================================
    %% ENTITY INHERITANCE
    %% ============================================

    BaseEntity <|-- ActionEntity : наследование
    BaseEntity <|-- ScenarioEntity : наследование
    BaseEntity <|-- StepEntity : наследование
    BaseEntity <|-- ScenarioStepEntity : наследование
    BaseEntity <|-- ObjectEntity : наследование
    BaseEntity <|-- SessionEntity : наследование

    %% ============================================
    %% ENTITY RELATIONSHIPS
    %% ============================================

    ActionEntity "1" --> "*" StepEntity : содержит
    ObjectEntity "1" --> "*" StepEntity : содержит
    StepEntity "*" --> "1" ActionEntity : ссылается
    StepEntity "*" --> "1" ObjectEntity : ссылается
    ScenarioEntity "0..1" --> "*" ScenarioEntity : родитель-потомок
    ScenarioEntity "1" --> "*" ScenarioStepEntity : содержит
    ScenarioEntity "1" --> "*" SessionEntity : содержит
    ScenarioStepEntity "*" --> "1" ScenarioEntity : ссылается
    ScenarioStepEntity "*" --> "1" StepEntity : ссылается
    SessionEntity "*" --> "1" ScenarioEntity : ссылается

    %% ============================================
    %% REPOSITORY INHERITANCE
    %% ============================================

    IRepositoryBase~ActionEntity~ <|.. IActionRepository : реализация
    IRepositoryBase~ScenarioEntity~ <|.. IScenarioRepository : реализация
    IRepositoryBase~StepEntity~ <|.. IStepRepository : реализация
    IRepositoryBase~ScenarioStepEntity~ <|.. IScenarioStepRepository : реализация
    IRepositoryBase~ObjectEntity~ <|.. IObjectRepository : реализация
    IRepositoryBase~SessionEntity~ <|.. ISessionRepository : реализация

    %% ============================================
    %% SPECIFICATION INHERITANCE
    %% ============================================

    Specification~ActionEntity~ <|-- ActionsByIdsSpec : наследование
    Specification~ScenarioEntity~ <|-- ScenariosWithActionsSpec : наследование
    Specification~ScenarioEntity~ <|-- ScenariosByParentIdSpec : наследование
    Specification~StepEntity~ <|-- StepsByActionIdSpec : наследование
    Specification~ObjectEntity~ <|-- ObjectsByNameSpec : наследование
    Specification~SessionEntity~ <|-- SessionsByScenarioIdSpec : наследование
    Specification~ScenarioStepEntity~ <|-- ScenarioStepsByScenarioIdSpec : наследование
    Specification~StepEntity~ <|-- StepsWithActionsAndObjectsSpec : наследование
    Specification~T~ <|-- ActiveEntitiesSpec~T~ : наследование

    %% ============================================
    %% DATA ACCESS INHERITANCE & RELATIONSHIPS
    %% ============================================

    DbContext <|-- ApplicationDbContext : наследование
    IEntityTypeConfiguration~T~ <|.. BaseEntityConfiguration~T~ : реализация
    BaseEntityConfiguration~T~ <|-- ActionConfiguration : наследование
    BaseEntityConfiguration~T~ <|-- ObjectConfiguration : наследование
    BaseEntityConfiguration~T~ <|-- ScenarioConfiguration : наследование
    BaseEntityConfiguration~T~ <|-- StepConfiguration : наследование
    BaseEntityConfiguration~T~ <|-- SessionConfiguration : наследование
    BaseEntityConfiguration~T~ <|-- ScenarioStepConfiguration : наследование

    RepositoryBase~ActionEntity~ <|-- ActionRepository : наследование
    RepositoryBase~ObjectEntity~ <|-- ObjectRepository : наследование
    RepositoryBase~ScenarioEntity~ <|-- ScenarioRepository : наследование
    RepositoryBase~StepEntity~ <|-- StepRepository : наследование
    RepositoryBase~ScenarioStepEntity~ <|-- ScenarioStepRepository : наследование
    RepositoryBase~SessionEntity~ <|-- SessionRepository : наследование

    IActionRepository <|.. ActionRepository : реализация
    IObjectRepository <|.. ObjectRepository : реализация
    IScenarioRepository <|.. ScenarioRepository : реализация
    IStepRepository <|.. StepRepository : реализация
    IScenarioStepRepository <|.. ScenarioStepRepository : реализация
    ISessionRepository <|.. SessionRepository : реализация

    ApplicationDbContext "1" --> "*" ActionConfiguration : конфигурирует
    ApplicationDbContext "1" --> "*" ObjectConfiguration : конфигурирует
    ApplicationDbContext "1" --> "*" ScenarioConfiguration : конфигурирует
    ApplicationDbContext "1" --> "*" StepConfiguration : конфигурирует
    ApplicationDbContext "1" --> "*" SessionConfiguration : конфигурирует
    ApplicationDbContext "1" --> "*" ScenarioStepConfiguration : конфигурирует

    ApplicationDbContext "1" --> "1" ActionRepository : использует
    ApplicationDbContext "1" --> "1" ObjectRepository : использует
    ApplicationDbContext "1" --> "1" ScenarioRepository : использует
    ApplicationDbContext "1" --> "1" StepRepository : использует
    ApplicationDbContext "1" --> "1" ScenarioStepRepository : использует
    ApplicationDbContext "1" --> "1" SessionRepository : использует

    BeautifulDataAccess ..> ApplicationDbContext : регистрирует
    BeautifulDataAccess ..> ActionRepository : регистрирует
    BeautifulDataAccess ..> ObjectRepository : регистрирует
    BeautifulDataAccess ..> ScenarioRepository : регистрирует
    BeautifulDataAccess ..> StepRepository : регистрирует
    BeautifulDataAccess ..> ScenarioStepRepository : регистрирует
    BeautifulDataAccess ..> SessionRepository : регистрирует

    %% ============================================
    %% DTO RELATIONSHIPS
    %% ============================================

    CreateScenarioRequest ..> ActionResponse : содержит ActionIds
    ScenarioResponse "1" --> "*" ActionResponse : содержит
    StepResponse "0..1" --> "0..1" ActionResponse : ссылается
    StepResponse "0..1" --> "0..1" ObjectResponse : ссылается
    ScenarioStepResponse "0..1" --> "0..1" ScenarioResponse : ссылается
    ScenarioStepResponse "0..1" --> "0..1" StepResponse : ссылается
    SessionResponse "0..1" --> "0..1" ScenarioResponse : ссылается

    AbstractValidator~CreateScenarioRequest~ <|-- CreateScenarioValidator : наследование
    CreateScenarioValidator ..> CreateScenarioRequest : валидирует

    %% ============================================
    %% SERVICE INHERITANCE & RELATIONSHIPS
    %% ============================================

    IBaseService~TEntity, TRequest, TResponse~ <|.. IActionService : расширение
    IBaseService~TEntity, TRequest, TResponse~ <|.. IScenarioService : расширение
    IBaseService~TEntity, TRequest, TResponse~ <|.. IObjectService : расширение
    IBaseService~TEntity, TRequest, TResponse~ <|.. IStepService : расширение
    IBaseService~TEntity, TRequest, TResponse~ <|.. IScenarioStepService : расширение
    IBaseService~TEntity, TRequest, TResponse~ <|.. ISessionService : расширение

    IBaseService~TEntity, TRequest, TResponse~ <|.. BaseService~TEntity, TRequest, TResponse~ : реализация
    BaseService~ActionEntity, CreateActionRequest, ActionResponse~ <|-- ActionService : наследование
    BaseService~ScenarioEntity, CreateScenarioRequest, ScenarioResponse~ <|-- ScenarioService : наследование
    BaseService~ObjectEntity, CreateObjectRequest, ObjectResponse~ <|-- ObjectService : наследование
    BaseService~StepEntity, CreateStepRequest, StepResponse~ <|-- StepService : наследование
    BaseService~ScenarioStepEntity, CreateScenarioStepRequest, ScenarioStepResponse~ <|-- ScenarioStepService : наследование
    BaseService~SessionEntity, CreateSessionRequest, SessionResponse~ <|-- SessionService : наследование

    IActionService <|.. ActionService : реализация
    IScenarioService <|.. ScenarioService : реализация
    IObjectService <|.. ObjectService : реализация
    IStepService <|.. StepService : реализация
    IScenarioStepService <|.. ScenarioStepService : реализация
    ISessionService <|.. SessionService : реализация

    ActionService "1" --> "1" IActionRepository : использует
    ScenarioService "1" --> "1" IScenarioRepository : использует
    ScenarioService "1" --> "1" IActionRepository : использует
    ScenarioService "1" --> "1" IScenarioStepRepository : использует
    ObjectService "1" --> "1" IObjectRepository : использует
    StepService "1" --> "1" IStepRepository : использует
    StepService "1" --> "1" IActionRepository : использует
    StepService "1" --> "1" IObjectRepository : использует
    ScenarioStepService "1" --> "1" IScenarioStepRepository : использует
    ScenarioStepService "1" --> "1" IScenarioRepository : использует
    ScenarioStepService "1" --> "1" IStepRepository : использует
    SessionService "1" --> "1" ISessionRepository : использует
    SessionService "1" --> "1" IScenarioRepository : использует

    %% ============================================
    %% CONTROLLER RELATIONSHIPS
    %% ============================================

    ControllerBase <|-- ActionController : наследование
    ControllerBase <|-- ScenarioController : наследование
    ControllerBase <|-- ObjectController : наследование
    ControllerBase <|-- StepController : наследование
    ControllerBase <|-- ScenarioStepController : наследование
    ControllerBase <|-- SessionController : наследование

    ActionController "1" --> "1" IActionService : использует
    ScenarioController "1" --> "1" IScenarioService : использует
    ObjectController "1" --> "1" IObjectService : использует
    StepController "1" --> "1" IStepService : использует
    ScenarioStepController "1" --> "1" IScenarioStepService : использует
    SessionController "1" --> "1" ISessionService : использует

    %% ============================================
    %% API LAYER OTHER
    %% ============================================

    Profile <|-- MappingProfile : наследование
    MappingProfile ..> ActionEntity : маппит
    MappingProfile ..> ScenarioEntity : маппит
    MappingProfile ..> ObjectEntity : маппит
    MappingProfile ..> StepEntity : маппит
    MappingProfile ..> ScenarioStepEntity : маппит
    MappingProfile ..> SessionEntity : маппит
    MappingProfile ..> ActionResponse : маппит
    MappingProfile ..> ScenarioResponse : маппит
    MappingProfile ..> ObjectResponse : маппит
    MappingProfile ..> StepResponse : маппит
    MappingProfile ..> ScenarioStepResponse : маппит
    MappingProfile ..> SessionResponse : маппит

    BeautifulBuilderService ..> IScenarioService : регистрирует
    BeautifulBuilderService ..> IActionService : регистрирует
    BeautifulBuilderService ..> IObjectService : регистрирует
    BeautifulBuilderService ..> IStepService : регистрирует
    BeautifulBuilderService ..> IScenarioStepService : регистрирует
    BeautifulBuilderService ..> ISessionService : регистрирует
    BeautifulBuilderService ..> MappingProfile : регистрирует
    BeautifulBuilderService ..> IValidationMarker : использует
```

---

## Сводная таблица классов по слоям

| Слой | Классы | Кол-во |
|------|--------|--------|
| **Domain — Entities** | BaseEntity, ActionEntity, ScenarioEntity, StepEntity, ScenarioStepEntity, ObjectEntity, SessionEntity | 7 |
| **Domain — Repository Interfaces** | IActionRepository, IScenarioRepository, IStepRepository, IScenarioStepRepository, IObjectRepository, ISessionRepository | 6 |
| **Domain — Specifications** | ActiveEntitiesSpec~T~, ActionsByIdsSpec, ScenariosWithActionsSpec, ScenariosByParentIdSpec, StepsByActionIdSpec, ObjectsByNameSpec, SessionsByScenarioIdSpec, ScenarioStepsByScenarioIdSpec, StepsWithActionsAndObjectsSpec | 9 |
| **DataAccess — Context** | ApplicationDbContext | 1 |
| **DataAccess — Configurations** | BaseEntityConfiguration~T~, ActionConfiguration, ObjectConfiguration, ScenarioConfiguration, StepConfiguration, SessionConfiguration, ScenarioStepConfiguration | 7 |
| **DataAccess — Repositories** | ActionRepository, ObjectRepository, ScenarioRepository, StepRepository, ScenarioStepRepository, SessionRepository | 6 |
| **DataAccess — Utilities** | BeautifulDataAccess | 1 |
| **DTO — Requests** | CreateScenarioRequest, CreateActionRequest, CreateObjectRequest, CreateStepRequest, CreateScenarioStepRequest, CreateSessionRequest | 6 |
| **DTO — Responses** | ActionResponse, ScenarioResponse, ObjectResponse, StepResponse, ScenarioStepResponse, SessionResponse | 6 |
| **DTO — Validators** | CreateScenarioValidator, IValidationMarker | 2 |
| **API — Service Interfaces** | IBaseService~TEntity,TRequest,TResponse~, IActionService, IScenarioService, IObjectService, IStepService, IScenarioStepService, ISessionService | 7 |
| **API — Service Implementations** | BaseService~TEntity,TRequest,TResponse~, ActionService, ScenarioService, ObjectService, StepService, ScenarioStepService, SessionService | 7 |
| **API — Controllers** | ActionController, ScenarioController, ObjectController, StepController, ScenarioStepController, SessionController | 6 |
| **API — Other** | BeautifulBuilderService, MappingProfile | 2 |
| **ИТОГО** | | **79** |
