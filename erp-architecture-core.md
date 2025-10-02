# System Architecture & Core Infrastructure Blueprint

## 1. Solution Structure الكاملة

```
ERPSaaS.sln
│
├── src/
│   ├── 1. Core/
│   │   ├── ERPSaaS.Domain/                      # Domain entities & interfaces
│   │   │   ├── Common/
│   │   │   │   ├── BaseEntity.cs
│   │   │   │   ├── IAuditableEntity.cs
│   │   │   │   ├── ISoftDeletable.cs
│   │   │   │   ├── ITenantEntity.cs
│   │   │   │   ├── IMultiCompanyEntity.cs
│   │   │   │   └── ILocalizable.cs
│   │   │   ├── ValueObjects/
│   │   │   │   ├── Money.cs
│   │   │   │   ├── Address.cs
│   │   │   │   ├── DateRange.cs
│   │   │   │   └── PhoneNumber.cs
│   │   │   ├── Enums/
│   │   │   │   ├── DocumentStatus.cs
│   │   │   │   ├── PaymentMethod.cs
│   │   │   │   └── TransactionType.cs
│   │   │   └── Exceptions/
│   │   │       ├── BusinessRuleException.cs
│   │   │       └── DomainException.cs
│   │   │
│   │   └── ERPSaaS.Application/                 # Application layer
│   │       ├── Common/
│   │       │   ├── Interfaces/
│   │       │   │   ├── IApplicationDbContext.cs
│   │       │   │   ├── IDateTime.cs
│   │       │   │   ├── ICurrentUser.cs
│   │       │   │   ├── ICurrentTenant.cs
│   │       │   │   └── IFileStorageService.cs
│   │       │   ├── Behaviours/
│   │       │   │   ├── ValidationBehaviour.cs
│   │       │   │   ├── AuthorizationBehaviour.cs
│   │       │   │   ├── PerformanceBehaviour.cs
│   │       │   │   └── UnhandledExceptionBehaviour.cs
│   │       │   ├── Models/
│   │       │   │   ├── PaginatedList.cs
│   │       │   │   ├── Result.cs
│   │       │   │   └── ApiResponse.cs
│   │       │   └── Mappings/
│   │       │       └── MappingProfile.cs
│   │       └── DependencyInjection.cs
│   │
│   ├── 2. Infrastructure/
│   │   ├── ERPSaaS.Infrastructure/              # Infrastructure implementations
│   │   │   ├── Persistence/
│   │   │   │   ├── Configurations/
│   │   │   │   │   ├── BaseEntityConfiguration.cs
│   │   │   │   │   └── AuditConfiguration.cs
│   │   │   │   ├── Interceptors/
│   │   │   │   │   ├── AuditableEntityInterceptor.cs
│   │   │   │   │   ├── SoftDeleteInterceptor.cs
│   │   │   │   │   └── TenantInterceptor.cs
│   │   │   │   ├── ApplicationDbContext.cs
│   │   │   │   └── ApplicationDbContextFactory.cs
│   │   │   ├── Identity/
│   │   │   │   ├── ApplicationUser.cs
│   │   │   │   ├── ApplicationRole.cs
│   │   │   │   └── IdentityService.cs
│   │   │   ├── Services/
│   │   │   │   ├── DateTimeService.cs
│   │   │   │   ├── CurrentUserService.cs
│   │   │   │   ├── CurrentTenantService.cs
│   │   │   │   └── FileStorageService.cs
│   │   │   ├── MultiTenancy/
│   │   │   │   ├── TenantResolver.cs
│   │   │   │   ├── TenantService.cs
│   │   │   │   └── TenantConnectionStringProvider.cs
│   │   │   └── DependencyInjection.cs
│   │   │
│   │   └── ERPSaaS.Infrastructure.Shared/       # Shared infrastructure
│   │       ├── Localization/
│   │       │   ├── LocalizationService.cs
│   │       │   ├── ResourceManager.cs
│   │       │   └── TranslationProvider.cs
│   │       ├── BackgroundJobs/
│   │       │   ├── HangfireConfiguration.cs
│   │       │   └── RecurringJobSetup.cs
│   │       ├── Caching/
│   │       │   ├── RedisCacheService.cs
│   │       │   └── DistributedCacheService.cs
│   │       └── Messaging/
│   │           ├── EventBus.cs
│   │           └── IntegrationEventHandler.cs
│   │
│   ├── 3. Modules/
│   │   ├── SaasManagement/
│   │   │   ├── ERPSaaS.Modules.SaasManagement.Domain/
│   │   │   ├── ERPSaaS.Modules.SaasManagement.Application/
│   │   │   ├── ERPSaaS.Modules.SaasManagement.Infrastructure/
│   │   │   └── ERPSaaS.Modules.SaasManagement.API/
│   │   │
│   │   ├── Finance/
│   │   │   ├── ERPSaaS.Modules.Finance.Domain/
│   │   │   │   ├── Entities/
│   │   │   │   │   ├── Account.cs
│   │   │   │   │   ├── AccountType.cs
│   │   │   │   │   ├── JournalEntry.cs
│   │   │   │   │   ├── JournalEntryLine.cs
│   │   │   │   │   ├── FiscalYear.cs
│   │   │   │   │   ├── FiscalPeriod.cs
│   │   │   │   │   ├── CostCenter.cs
│   │   │   │   │   ├── Project.cs
│   │   │   │   │   ├── Currency.cs
│   │   │   │   │   └── ExchangeRate.cs
│   │   │   │   ├── Aggregates/
│   │   │   │   │   └── JournalEntryAggregate.cs
│   │   │   │   ├── ValueObjects/
│   │   │   │   │   ├── AccountCode.cs
│   │   │   │   │   └── JournalReference.cs
│   │   │   │   ├── Events/
│   │   │   │   │   ├── JournalEntryCreatedEvent.cs
│   │   │   │   │   └── JournalEntryPostedEvent.cs
│   │   │   │   └── Specifications/
│   │   │   │       └── AccountSpecifications.cs
│   │   │   │
│   │   │   ├── ERPSaaS.Modules.Finance.Application/
│   │   │   │   ├── Features/
│   │   │   │   │   ├── Accounts/
│   │   │   │   │   │   ├── Commands/
│   │   │   │   │   │   │   ├── CreateAccount/
│   │   │   │   │   │   │   │   ├── CreateAccountCommand.cs
│   │   │   │   │   │   │   │   ├── CreateAccountCommandValidator.cs
│   │   │   │   │   │   │   │   └── CreateAccountCommandHandler.cs
│   │   │   │   │   │   │   ├── UpdateAccount/
│   │   │   │   │   │   │   └── DeleteAccount/
│   │   │   │   │   │   ├── Queries/
│   │   │   │   │   │   │   ├── GetAccounts/
│   │   │   │   │   │   │   ├── GetAccountById/
│   │   │   │   │   │   │   └── GetAccountHierarchy/
│   │   │   │   │   │   └── DTOs/
│   │   │   │   │   │       ├── AccountDto.cs
│   │   │   │   │   │       └── AccountListDto.cs
│   │   │   │   │   │
│   │   │   │   │   ├── JournalEntries/
│   │   │   │   │   │   ├── Commands/
│   │   │   │   │   │   │   ├── CreateJournalEntry/
│   │   │   │   │   │   │   ├── PostJournalEntry/
│   │   │   │   │   │   │   ├── UnpostJournalEntry/
│   │   │   │   │   │   │   └── ReverseJournalEntry/
│   │   │   │   │   │   ├── Queries/
│   │   │   │   │   │   │   ├── GetJournalEntries/
│   │   │   │   │   │   │   ├── GetJournalEntryById/
│   │   │   │   │   │   │   └── GetTrialBalance/
│   │   │   │   │   │   └── DTOs/
│   │   │   │   │   │       ├── JournalEntryDto.cs
│   │   │   │   │   │       ├── JournalEntryLineDto.cs
│   │   │   │   │   │       └── TrialBalanceDto.cs
│   │   │   │   │   │
│   │   │   │   │   ├── FinancialReports/
│   │   │   │   │   │   └── Queries/
│   │   │   │   │   │       ├── GetBalanceSheet/
│   │   │   │   │   │       ├── GetIncomeStatement/
│   │   │   │   │   │       └── GetCashFlow/
│   │   │   │   │   │
│   │   │   │   │   └── FinancialPeriods/
│   │   │   │   │       ├── Commands/
│   │   │   │   │       └── Queries/
│   │   │   │   │
│   │   │   │   ├── Services/
│   │   │   │   │   ├── IFinancialPostingService.cs
│   │   │   │   │   ├── IExchangeRateService.cs
│   │   │   │   │   ├── ICostAllocationService.cs
│   │   │   │   │   └── IFinancialReportingService.cs
│   │   │   │   │
│   │   │   │   └── IntegrationEvents/
│   │   │   │       ├── Handlers/
│   │   │   │       │   ├── SalesInvoicePostedEventHandler.cs
│   │   │   │       │   ├── PurchaseInvoicePostedEventHandler.cs
│   │   │   │       │   └── InventoryTransactionEventHandler.cs
│   │   │   │       └── Events/
│   │   │   │           ├── JournalEntryPostedIntegrationEvent.cs
│   │   │   │           └── PeriodClosedIntegrationEvent.cs
│   │   │   │
│   │   │   ├── ERPSaaS.Modules.Finance.Infrastructure/
│   │   │   │   ├── Persistence/
│   │   │   │   │   ├── Configurations/
│   │   │   │   │   │   ├── AccountConfiguration.cs
│   │   │   │   │   │   ├── JournalEntryConfiguration.cs
│   │   │   │   │   │   └── CostCenterConfiguration.cs
│   │   │   │   │   ├── FinanceDbContext.cs
│   │   │   │   │   └── Migrations/
│   │   │   │   ├── Services/
│   │   │   │   │   ├── FinancialPostingService.cs
│   │   │   │   │   ├── ExchangeRateService.cs
│   │   │   │   │   └── FinancialReportingService.cs
│   │   │   │   └── BackgroundJobs/
│   │   │   │       ├── PeriodCloseJob.cs
│   │   │   │       └── FinancialReconciliationJob.cs
│   │   │   │
│   │   │   └── ERPSaaS.Modules.Finance.API/
│   │   │       └── Controllers/
│   │   │           ├── AccountsController.cs
│   │   │           ├── JournalEntriesController.cs
│   │   │           ├── FinancialReportsController.cs
│   │   │           └── FiscalPeriodsController.cs
│   │   │
│   │   ├── Sales/
│   │   │   ├── ERPSaaS.Modules.Sales.Domain/
│   │   │   ├── ERPSaaS.Modules.Sales.Application/
│   │   │   ├── ERPSaaS.Modules.Sales.Infrastructure/
│   │   │   └── ERPSaaS.Modules.Sales.API/
│   │   │
│   │   ├── Purchasing/
│   │   ├── Inventory/
│   │   ├── HR/
│   │   ├── FixedAssets/
│   │   └── [... other 15+ modules]
│   │
│   ├── 4. Shared/
│   │   ├── ERPSaaS.Shared.Contracts/            # Shared contracts & DTOs
│   │   │   ├── Finance/
│   │   │   │   ├── IFinancialPostingRequest.cs
│   │   │   │   └── FinancialPostingResult.cs
│   │   │   ├── Common/
│   │   │   │   ├── IIntegrationEvent.cs
│   │   │   │   └── IntegrationEventBase.cs
│   │   │   └── Metadata/
│   │   │       ├── ILocalizable.cs
│   │   │       └── LocalizableDto.cs
│   │   │
│   │   └── ERPSaaS.Shared.Kernel/               # Shared kernel
│   │       ├── Metadata/
│   │       │   ├── MetadataAttribute.cs
│   │       │   ├── LocalizableAttribute.cs
│   │       │   └── DisplayMetadata.cs
│   │       ├── Extensions/
│   │       │   ├── StringExtensions.cs
│   │       │   ├── DateTimeExtensions.cs
│   │       │   └── EnumExtensions.cs
│   │       └── Helpers/
│   │           ├── CodeGenerator.cs
│   │           └── NumberToWords.cs
│   │
│   └── 5. Presentation/
│       └── ERPSaaS.API/                          # Main API Gateway
│           ├── Controllers/
│           ├── Middleware/
│           │   ├── TenantResolutionMiddleware.cs
│           │   ├── LocalizationMiddleware.cs
│           │   ├── ExceptionHandlingMiddleware.cs
│           │   └── RequestLoggingMiddleware.cs
│           ├── Filters/
│           │   ├── ValidateModelFilter.cs
│           │   └── ApiExceptionFilter.cs
│           ├── Extensions/
│           │   ├── ServiceCollectionExtensions.cs
│           │   └── ApplicationBuilderExtensions.cs
│           ├── appsettings.json
│           ├── appsettings.Development.json
│           ├── appsettings.Production.json
│           └── Program.cs
│
└── tests/
    ├── UnitTests/
    │   ├── ERPSaaS.Domain.UnitTests/
    │   ├── ERPSaaS.Application.UnitTests/
    │   └── ERPSaaS.Modules.Finance.UnitTests/
    ├── IntegrationTests/
    │   └── ERPSaaS.API.IntegrationTests/
    └── PerformanceTests/
        └── ERPSaaS.LoadTests/
```

## 2. Core Base Entities - الكيانات الأساسية المشتركة

### 2.1 BaseEntity - الكيان الأساسي

```csharp
namespace ERPSaaS.Domain.Common
{
    /// <summary>
    /// Base entity for all domain entities with common properties
    /// الكيان الأساسي لجميع الكيانات مع الخصائص المشتركة
    /// </summary>
    public abstract class BaseEntity
    {
        public long Id { get; set; }
        
        /// <summary>
        /// Row version for optimistic concurrency control
        /// نسخة الصف للتحكم في التزامن التفاؤلي
        /// </summary>
        [Timestamp]
        public byte[] RowVersion { get; set; }
        
        private readonly List<DomainEvent> _domainEvents = new();
        
        [NotMapped]
        public IReadOnlyCollection<DomainEvent> DomainEvents => _domainEvents.AsReadOnly();
        
        public void AddDomainEvent(DomainEvent domainEvent)
        {
            _domainEvents.Add(domainEvent);
        }
        
        public void RemoveDomainEvent(DomainEvent domainEvent)
        {
            _domainEvents.Remove(domainEvent);
        }
        
        public void ClearDomainEvents()
        {
            _domainEvents.Clear();
        }
    }
}
```

### 2.2 IAuditableEntity - واجهة التدقيق

```csharp
namespace ERPSaaS.Domain.Common
{
    /// <summary>
    /// Interface for entities that require audit trail
    /// واجهة للكيانات التي تتطلب مسار تدقيق
    /// </summary>
    public interface IAuditableEntity
    {
        /// <summary>
        /// User who created the record
        /// المستخدم الذي أنشأ السجل
        /// </summary>
        Guid CreatedBy { get; set; }
        
        /// <summary>
        /// Date and time when record was created (UTC)
        /// التاريخ والوقت عند إنشاء السجل (بالتوقيت العالمي)
        /// </summary>
        DateTime CreatedDateUtc { get; set; }
        
        /// <summary>
        /// User who last modified the record
        /// المستخدم الذي عدل السجل آخر مرة
        /// </summary>
        Guid? LastModifiedBy { get; set; }
        
        /// <summary>
        /// Date and time when record was last modified (UTC)
        /// التاريخ والوقت عند آخر تعديل للسجل (بالتوقيت العالمي)
        /// </summary>
        DateTime? LastModifiedDateUtc { get; set; }
        
        /// <summary>
        /// Timezone IANA identifier when record was created
        /// معرف المنطقة الزمنية IANA عند إنشاء السجل
        /// </summary>
        string CreatedTimezone { get; set; }
        
        /// <summary>
        /// Local date time when record was created
        /// التاريخ والوقت المحلي عند إنشاء السجل
        /// </summary>
        DateTime CreatedDateLocal { get; set; }
    }
}
```

### 2.3 ISoftDeletable - واجهة الحذف اللين

```csharp
namespace ERPSaaS.Domain.Common
{
    /// <summary>
    /// Interface for entities that support soft delete
    /// واجهة للكيانات التي تدعم الحذف اللين
    /// </summary>
    public interface ISoftDeletable
    {
        /// <summary>
        /// Indicates if the record is deleted
        /// يشير إلى ما إذا كان السجل محذوفًا
        /// </summary>
        bool IsDeleted { get; set; }
        
        /// <summary>
        /// User who deleted the record
        /// المستخدم الذي حذف السجل
        /// </summary>
        Guid? DeletedBy { get; set; }
        
        /// <summary>
        /// Date and time when record was deleted (UTC)
        /// التاريخ والوقت عند حذف السجل (بالتوقيت العالمي)
        /// </summary>
        DateTime? DeletedDateUtc { get; set; }
        
        /// <summary>
        /// Reason for deletion
        /// سبب الحذف
        /// </summary>
        string DeleteReason { get; set; }
    }
}
```

### 2.4 ITenantEntity - واجهة المستأجر

```csharp
namespace ERPSaaS.Domain.Common
{
    /// <summary>
    /// Interface for entities that belong to a specific tenant
    /// واجهة للكيانات التي تنتمي إلى مستأجر محدد
    /// </summary>
    public interface ITenantEntity
    {
        /// <summary>
        /// Tenant identifier
        /// معرف المستأجر
        /// </summary>
        Guid TenantId { get; set; }
    }
}
```

### 2.5 IMultiCompanyEntity - واجهة متعدد الشركات

```csharp
namespace ERPSaaS.Domain.Common
{
    /// <summary>
    /// Interface for entities that belong to a specific company within a tenant
    /// واجهة للكيانات التي تنتمي إلى شركة محددة داخل المستأجر
    /// </summary>
    public interface IMultiCompanyEntity : ITenantEntity
    {
        /// <summary>
        /// Company identifier
        /// معرف الشركة
        /// </summary>
        Guid CompanyId { get; set; }
    }
}
```

### 2.6 ILocalizable - واجهة القابلية للترجمة

```csharp
namespace ERPSaaS.Domain.Common
{
    /// <summary>
    /// Interface for entities that support multi-language content
    /// واجهة للكيانات التي تدعم المحتوى متعدد اللغات
    /// </summary>
    public interface ILocalizable
    {
        /// <summary>
        /// Navigation property to translations
        /// خاصية التنقل إلى الترجمات
        /// </summary>
        ICollection<ITranslation> Translations { get; set; }
    }
    
    /// <summary>
    /// Interface for translation entities
    /// واجهة لكيانات الترجمة
    /// </summary>
    public interface ITranslation
    {
        /// <summary>
        /// Language code (e.g., en-US, ar-SA)
        /// رمز اللغة (مثل en-US, ar-SA)
        /// </summary>
        string LanguageCode { get; set; }
        
        /// <summary>
        /// Reference to parent entity
        /// مرجع إلى الكيان الأصلي
        /// </summary>
        long EntityId { get; set; }
    }
}
```

### 2.7 FullAuditableEntity - كيان كامل قابل للتدقيق

```csharp
namespace ERPSaaS.Domain.Common
{
    /// <summary>
    /// Base entity with full auditing, soft delete, and multi-tenancy support
    /// كيان أساسي مع دعم كامل للتدقيق والحذف اللين وتعدد المستأجرين
    /// </summary>
    public abstract class FullAuditableEntity : BaseEntity, 
        IAuditableEntity, 
        ISoftDeletable, 
        ITenantEntity
    {
        // ITenantEntity
        public Guid TenantId { get; set; }
        
        // IAuditableEntity
        public Guid CreatedBy { get; set; }
        public DateTime CreatedDateUtc { get; set; }
        public Guid? LastModifiedBy { get; set; }
        public DateTime? LastModifiedDateUtc { get; set; }
        public string CreatedTimezone { get; set; }
        public DateTime CreatedDateLocal { get; set; }
        
        // ISoftDeletable
        public bool IsDeleted { get; set; }
        public Guid? DeletedBy { get; set; }
        public DateTime? DeletedDateUtc { get; set; }
        public string DeleteReason { get; set; }
    }
    
    /// <summary>
    /// Base entity with multi-company support
    /// كيان أساسي مع دعم تعدد الشركات
    /// </summary>
    public abstract class MultiCompanyEntity : FullAuditableEntity, 
        IMultiCompanyEntity
    {
        public Guid CompanyId { get; set; }
    }
}
```

## 3. Database Interceptors - معترضات قاعدة البيانات

### 3.1 AuditableEntityInterceptor

```csharp
namespace ERPSaaS.Infrastructure.Persistence.Interceptors
{
    /// <summary>
    /// Interceptor to automatically populate audit fields
    /// معترض لملء حقول التدقيق تلقائيًا
    /// </summary>
    public class AuditableEntityInterceptor : SaveChangesInterceptor
    {
        private readonly ICurrentUser _currentUser;
        private readonly IDateTime _dateTime;
        
        public AuditableEntityInterceptor(
            ICurrentUser currentUser,
            IDateTime dateTime)
        {
            _currentUser = currentUser;
            _dateTime = dateTime;
        }
        
        public override InterceptionResult<int> SavingChanges(
            DbContextEventData eventData,
            InterceptionResult<int> result)
        {
            UpdateAuditableEntities(eventData.Context);
            return base.SavingChanges(eventData, result);
        }
        
        public override ValueTask<InterceptionResult<int>> SavingChangesAsync(
            DbContextEventData eventData,
            InterceptionResult<int> result,
            CancellationToken cancellationToken = default)
        {
            UpdateAuditableEntities(eventData.Context);
            return base.SavingChangesAsync(eventData, result, cancellationToken);
        }
        
        private void UpdateAuditableEntities(DbContext context)
        {
            if (context == null) return;
            
            var userId = _currentUser.UserId ?? Guid.Empty;
            var utcNow = _dateTime.UtcNow;
            var userTimezone = _currentUser.Timezone ?? "UTC";
            var localNow = _dateTime.ConvertToTimezone(utcNow, userTimezone);
            
            foreach (var entry in context.ChangeTracker.Entries<IAuditableEntity>())
            {
                switch (entry.State)
                {
                    case EntityState.Added:
                        entry.Entity.CreatedBy = userId;
                        entry.Entity.CreatedDateUtc = utcNow;
                        entry.Entity.CreatedTimezone = userTimezone;
                        entry.Entity.CreatedDateLocal = localNow;
                        break;
                        
                    case EntityState.Modified:
                        entry.Entity.LastModifiedBy = userId;
                        entry.Entity.LastModifiedDateUtc = utcNow;
                        break;
                }
            }
        }
    }
}
```

### 3.2 SoftDeleteInterceptor

```csharp
namespace ERPSaaS.Infrastructure.Persistence.Interceptors
{
    /// <summary>
    /// Interceptor to handle soft delete
    /// معترض للتعامل مع الحذف اللين
    /// </summary>
    public class SoftDeleteInterceptor : SaveChangesInterceptor
    {
        private readonly ICurrentUser _currentUser;
        private readonly IDateTime _dateTime;
        
        public SoftDeleteInterceptor(
            ICurrentUser currentUser,
            IDateTime dateTime)
        {
            _currentUser = currentUser;
            _dateTime = dateTime;
        }
        
        public override InterceptionResult<int> SavingChanges(
            DbContextEventData eventData,
            InterceptionResult<int> result)
        {
            UpdateSoftDeletedEntities(eventData.Context);
            return base.SavingChanges(eventData, result);
        }
        
        public override ValueTask<InterceptionResult<int>> SavingChangesAsync(
            DbContextEventData eventData,
            InterceptionResult<int> result,
            CancellationToken cancellationToken = default)
        {
            UpdateSoftDeletedEntities(eventData.Context);
            return base.SavingChangesAsync(eventData, result, cancellationToken);
        }
        
        private void UpdateSoftDeletedEntities(DbContext context)
        {
            if (context == null) return;
            
            foreach (var entry in context.ChangeTracker.Entries<ISoftDeletable>())
            {
                if (entry.State == EntityState.Deleted)
                {
                    // Convert hard delete to soft delete
                    entry.State = EntityState.Modified;
                    entry.Entity.IsDeleted = true;
                    entry.Entity.DeletedBy = _currentUser.UserId;
                    entry.Entity.DeletedDateUtc = _dateTime.UtcNow;
                }
            }
        }
    }
}
```

### 3.3 TenantInterceptor

```csharp
namespace ERPSaaS.Infrastructure.Persistence.Interceptors
{
    /// <summary>
    /// Interceptor to automatically set tenant ID
    /// معترض لتعيين معرف المستأجر تلقائيًا
    /// </summary>
    public class TenantInterceptor : SaveChangesInterceptor
    {
        private readonly ICurrentTenant _currentTenant;
        
        public TenantInterceptor(ICurrentTenant currentTenant)
        {
            _currentTenant = currentTenant;
        }
        
        public override InterceptionResult<int> SavingChanges(
            DbContextEventData eventData,
            InterceptionResult<int> result)
        {
            SetTenantId(eventData.Context);
            return base.SavingChanges(eventData, result);
        }
        
        public override ValueTask<InterceptionResult<int>> SavingChangesAsync(
            DbContextEventData eventData,
            InterceptionResult<int> result,
            CancellationToken cancellationToken = default)
        {
            SetTenantId(eventData.Context);
            return base.SavingChangesAsync(eventData, result, cancellationToken);
        }
        
        private void SetTenantId(DbContext context)
        {
            if (context == null) return;
            
            var tenantId = _currentTenant.TenantId;
            if (tenantId == Guid.Empty) return;
            
            foreach (var entry in context.ChangeTracker.Entries<ITenantEntity>())
            {
                if (entry.State == EntityState.Added)
                {
                    entry.Entity.TenantId = tenantId;
                }
                
                // Prevent tenant ID changes
                if (entry.State == EntityState.Modified)
                {
                    entry.Property(nameof(ITenantEntity.TenantId)).IsModified = false;
                }
            }
        }
    }
}
```

## 4. ApplicationDbContext - سياق قاعدة البيانات الرئيسي

```csharp
namespace ERPSaaS.Infrastructure.Persistence
{
    public class ApplicationDbContext : DbContext, IApplicationDbContext
    {
        private readonly ICurrentTenant _currentTenant;
        private readonly IMediator _mediator;
        
        public ApplicationDbContext(
            DbContextOptions<ApplicationDbContext> options,
            ICurrentTenant currentTenant,
            IMediator mediator) : base(options)
        {
            _currentTenant = currentTenant;
            _mediator = mediator;
        }
        
        protected override void OnModelCreating(ModelBuilder modelBuilder)
        {
            base.OnModelCreating(modelBuilder);
            
            // Apply all configurations from assembly
            modelBuilder.ApplyConfigurationsFromAssembly(Assembly.GetExecutingAssembly());
            
            // Apply global query filters
            ApplyGlobalQueryFilters(modelBuilder);
        }
        
        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            base.OnConfiguring(optionsBuilder);
            
            // Add interceptors
            optionsBuilder.AddInterceptors(
                new AuditableEntityInterceptor(_currentUser, _dateTime),
                new SoftDeleteInterceptor(_currentUser, _dateTime),
                new TenantInterceptor(_currentTenant));
        }
        
        /// <summary>
        /// Apply global query filters for soft delete and multi-tenancy
        /// تطبيق فلاتر الاستعلام العامة للحذف اللين وتعدد المستأجرين
        /// </summary>
        private void ApplyGlobalQueryFilters(ModelBuilder modelBuilder)
        {
            foreach (var entityType in modelBuilder.Model.GetEntityTypes())
            {
                // Soft delete filter
                if (typeof(ISoftDeletable).IsAssignableFrom(entityType.ClrType))
                {
                    var parameter = Expression.Parameter(entityType.ClrType, "e");
                    var body = Expression.Equal(
                        Expression.Property(parameter, nameof(ISoftDeletable.IsDeleted)),
                        Expression.Constant(false));
                    var lambda = Expression.Lambda(body, parameter);
                    
                    modelBuilder.Entity(entityType.ClrType).HasQueryFilter(lambda);
                }
                
                // Multi-tenancy filter
                if (typeof(ITenantEntity).IsAssignableFrom(entityType.ClrType))
                {
                    var parameter = Expression.Parameter(entityType.ClrType, "e");
                    var tenantId = Expression.Constant(_currentTenant.TenantId);
                    var body = Expression.Equal(
                        Expression.Property(parameter, nameof(ITenantEntity.TenantId)),
                        tenantId);
                    var lambda = Expression.Lambda(body, parameter);
                    
                    // Combine with existing filter if any
                    var existingFilter = entityType.GetQueryFilter();
                    if (existingFilter != null)
                    {
                        var combinedBody = Expression.AndAlso(
                            existingFilter.Body,
                            Expression.Invoke(lambda, existingFilter.Parameters));
                        lambda = Expression.Lambda(combinedBody, existingFilter.Parameters);
                    }
                    
                    modelBuilder.Entity(entityType.ClrType).HasQueryFilter(lambda);
                }
            }
        }
        
        public override async Task<int> SaveChangesAsync(
            CancellationToken cancellationToken = default)
        {
            // Dispatch domain events before saving
            await DispatchDomainEvents();
            
            return await base.SaveChangesAsync(cancellationToken);
        }
        
        private async Task DispatchDomainEvents()
        {
            var domainEntities = ChangeTracker
                .Entries<BaseEntity>()
                .Where(x => x.Entity.DomainEvents.Any())
                .Select(x => x.Entity)
                .ToList();
            
            var domainEvents = domainEntities
                .SelectMany(x => x.DomainEvents)
                .ToList();
            
            domainEntities.ForEach(entity => entity.ClearDomainEvents());
            
            foreach (var domainEvent in domainEvents)
            {
                await _mediator.Publish(domainEvent);
            }
        }
    }
}
```

## 5. Module Template Structure - بنية قالب الوحدة

```
ERPSaaS.Modules.[ModuleName]/
├── Domain/
│   ├── Entities/                    # Domain entities
│   ├── ValueObjects/                # Value objects
│   ├── Aggregates/                  # Aggregate roots
│   ├── Events/                      # Domain events
│   ├── Specifications/              # Specifications pattern
│   ├── Enums/                       # Module-specific enums
│   └── Exceptions/                  # Domain exceptions
│
├── Application/
│   ├── Features/
│   │   └── [FeatureName]/
│   │       ├── Commands/
│   │       │   └── [CommandName]/
│   │       │       ├── [CommandName]Command.cs
│   │       │       ├── [CommandName]CommandValidator.cs
│   │       │       └── [CommandName]CommandHandler.cs
│   │       ├── Queries/
│   │       │   └── [QueryName]/
│   │       │       ├── [QueryName]Query.cs
│   │       │       └── [QueryName]QueryHandler.cs
│   │       └── DTOs/
│   │           ├── [FeatureName]Dto.cs
│   │           └── [FeatureName]ListDto.cs
│   ├── Services/                    # Application services interfaces
│   ├── Mappings/                    # AutoMapper profiles
│   ├── IntegrationEvents/          # Integration events
│   │   ├── Events/
│   │   └── Handlers/
│   └── DependencyInjection.cs
│
├── Infrastructure/
│   ├── Persistence/
│   │   ├── Configurations/         # EF Core configurations
│   │   ├── [ModuleName]DbContext.cs
│   │   └── Migrations/
│   ├── Services/                   # Service implementations
│   ├── BackgroundJobs/            # Hangfire jobs
│   └── DependencyInjection.cs
│
└── API/
    ├── Controllers/
    ├── ViewModels/
    └── Validators/
```

## 6. Common Database Tables - الجداول المشتركة

### 6.1 System Configuration Tables

```sql
-- ============================================
-- Tenants Management
-- إدارة المستأجرين
-- ============================================
CREATE TABLE [System].[Tenants] (
    [TenantId] UNIQUEIDENTIFIER NOT NULL DEFAULT NEWSEQUENTIALID(),
    [TenantCode] NVARCHAR(50) NOT NULL,
    [TenantName] NVARCHAR(200) NOT NULL,
    [DatabaseName] NVARCHAR(128) NOT NULL,
    [ConnectionString] NVARCHAR(MAX) NOT NULL,
    [IsActive] BIT NOT NULL DEFAULT 1,
    [SubscriptionPlan] NVARCHAR(50) NOT NULL,
    [SubscriptionStartDate] DATE NOT NULL,
    [SubscriptionEndDate] DATE NULL,
    [MaxUsers] INT NOT NULL DEFAULT 10,
    [MaxCompanies] INT NOT NULL DEFAULT 1,
    [MaxBranches] INT NOT NULL DEFAULT 5,
    [StorageQuotaGB] DECIMAL(10,2) NOT NULL DEFAULT 10.00,
    [DefaultLanguage] NVARCHAR(10) NOT NULL DEFAULT 'en',
    [DefaultTimezone] NVARCHAR(50) NOT NULL DEFAULT 'UTC',
    [DefaultCurrency] CHAR(3) NOT NULL DEFAULT 'USD',
    [CountryCode] CHAR(2) NOT NULL,
    [CreatedBy] UNIQUEIDENTIFIER NOT NULL,
    [CreatedDateUtc] DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
    [LastModifiedBy] UNIQUEIDENTIFIER NULL,
    [LastModifiedDateUtc] DATETIME2 NULL,
    CONSTRAINT [PK_Tenants] PRIMARY KEY ([TenantId]),
    CONSTRAINT [UK_Tenants_TenantCode] UNIQUE ([TenantCode])
);

-- ============================================
-- Companies
-- الشركات
-- ============================================
CREATE TABLE [System].[Companies] (
    [CompanyId] UNIQUEIDENTIFIER NOT NULL DEFAULT NEWSEQUENTIALID(),
    [TenantId] UNIQUEIDENTIFIER NOT NULL,
    [CompanyCode] NVARCHAR(20) NOT NULL,
    [CompanyName] NVARCHAR(200) NOT NULL,
    [CompanyName_AR] NVARCHAR(200) NULL,
    [LegalName] NVARCHAR(200) NOT NULL,
    [TaxRegistrationNumber] NVARCHAR(50) NULL,
    [CommercialRegistrationNumber] NVARCHAR(50) NULL,
    [IndustryType] NVARCHAR(50) NOT NULL,
    [BaseCurrency] CHAR(3) NOT NULL,
    [FiscalYearStartMonth] TINYINT NOT NULL DEFAULT 1,
    [IsHeadquarter] BIT NOT NULL DEFAULT 0,
    [Address] NVARCHAR(500) NULL,
    [City] NVARCHAR(100) NULL,
    [State] NVARCHAR(100) NULL,
    [Country] NVARCHAR(100) NOT NULL,
    [PostalCode] NVARCHAR(20) NULL,
    [Phone] NVARCHAR(20) NULL,
    [Email] NVARCHAR(100) NULL,
    [Website] NVARCHAR(200) NULL,
    [Logo] NVARCHAR(MAX) NULL,
    [IsActive] BIT NOT NULL DEFAULT 1,
    [CreatedBy] UNIQUEIDENTIFIER NOT NULL,
    [CreatedDateUtc] DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
    [CreatedTimezone] NVARCHAR(50) NOT NULL,
    [CreatedDateLocal] DATETIME2 NOT NULL,
    [LastModifiedBy] UNIQUEIDENTIFIER NULL,
    [LastModifiedDateUtc] DATETIME2 NULL,
    [IsDeleted] BIT NOT NULL DEFAULT 0,
    [DeletedBy] UNIQUEIDENTIFIER NULL,
    [DeletedDateUtc] DATETIME2 NULL,
    [DeleteReason] NVARCHAR(500) NULL,
    [RowVersion] ROWVERSION NOT NULL,
    CONSTRAINT [PK_Companies] PRIMARY KEY ([CompanyId]),
    CONSTRAINT [FK_Companies_Tenants] FOREIGN KEY ([TenantId])
        REFERENCES [System].[Tenants]([TenantId]),
    CONSTRAINT [UK_Companies_TenantCode] UNIQUE ([TenantId], [CompanyCode])
);

-- ============================================
-- Branches
-- الفروع
-- ============================================
CREATE TABLE [System].[Branches] (
    [BranchId] UNIQUEIDENTIFIER NOT NULL DEFAULT NEWSEQUENTIALID(),
    [TenantId] UNIQUEIDENTIFIER NOT NULL,
    [CompanyId] UNIQUEIDENTIFIER NOT NULL,
    [BranchCode] NVARCHAR(20) NOT NULL,
    [BranchName] NVARCHAR(200) NOT NULL,
    [BranchName_AR] NVARCHAR(200) NULL,
    [BranchType] NVARCHAR(20) NOT NULL, -- Headquarters, Regional, Local
    [ParentBranchId] UNIQUEIDENTIFIER NULL,
    [Address] NVARCHAR(500) NULL,
    [City] NVARCHAR(100) NULL,
    [State] NVARCHAR(100) NULL,
    [Country] NVARCHAR(100) NOT NULL,
    [PostalCode] NVARCHAR(20) NULL,
    [Phone] NVARCHAR(20) NULL,
    [Email] NVARCHAR(100) NULL,
    [Manager] NVARCHAR(200) NULL,
    [IsActive] BIT NOT NULL DEFAULT 1,
    [CreatedBy] UNIQUEIDENTIFIER NOT NULL,
    [CreatedDateUtc] DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
    [CreatedTimezone] NVARCHAR(50) NOT NULL,
    [CreatedDateLocal] DATETIME2 NOT NULL,
    [LastModifiedBy] UNIQUEIDENTIFIER NULL,
    [LastModifiedDateUtc] DATETIME2 NULL,
    [IsDeleted] BIT NOT NULL DEFAULT 0,
    [DeletedBy] UNIQUEIDENTIFIER NULL,
    [DeletedDateUtc] DATETIME2 NULL,
    [DeleteReason] NVARCHAR(500) NULL,
    [RowVersion] ROWVERSION NOT NULL,
    CONSTRAINT [PK_Branches] PRIMARY KEY ([BranchId]),
    CONSTRAINT [FK_Branches_Companies] FOREIGN KEY ([CompanyId])
        REFERENCES [System].[Companies]([CompanyId]),
    CONSTRAINT [FK_Branches_ParentBranch] FOREIGN KEY ([ParentBranchId])
        REFERENCES [System].[Branches]([BranchId]),
    CONSTRAINT [UK_Branches_Code] UNIQUE ([TenantId], [CompanyId], [BranchCode])
);

-- Create indexes
CREATE NONCLUSTERED INDEX [IX_Companies_Tenant_Active]
    ON [System].[Companies]([TenantId], [IsActive])
    INCLUDE ([CompanyCode], [CompanyName]);

CREATE NONCLUSTERED INDEX [IX_Branches_Company_Active]
    ON [System].[Branches]([CompanyId], [IsActive])
    INCLUDE ([BranchCode], [BranchName]);
```

تابع في الملف التالي...
