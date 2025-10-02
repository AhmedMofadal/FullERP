# Financial Integration Framework - إطار الربط المالي

## 1. Financial Posting Rules Schema - مخطط قواعد الترحيل المالي

### 1.1 Core Financial Integration Tables

```sql
-- ============================================
-- Posting Rules Configuration
-- تكوين قواعد الترحيل
-- ============================================
CREATE TABLE [Finance].[PostingRuleGroups] (
    [RuleGroupId] INT IDENTITY(1,1) NOT NULL,
    [TenantId] UNIQUEIDENTIFIER NOT NULL,
    [CompanyId] UNIQUEIDENTIFIER NOT NULL,
    [RuleGroupCode] NVARCHAR(50) NOT NULL,
    [RuleGroupName] NVARCHAR(200) NOT NULL,
    [RuleGroupName_AR] NVARCHAR(200) NULL,
    [SourceModule] NVARCHAR(50) NOT NULL, -- Sales, Purchasing, Inventory, HR, etc.
    [DocumentType] NVARCHAR(50) NOT NULL, -- Invoice, Receipt, Payment, etc.
    [Description] NVARCHAR(500) NULL,
    [IsActive] BIT NOT NULL DEFAULT 1,
    [EffectiveFromDate] DATE NOT NULL,
    [EffectiveToDate] DATE NULL,
    [Priority] INT NOT NULL DEFAULT 0,
    [CreatedBy] UNIQUEIDENTIFIER NOT NULL,
    [CreatedDateUtc] DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
    [LastModifiedBy] UNIQUEIDENTIFIER NULL,
    [LastModifiedDateUtc] DATETIME2 NULL,
    CONSTRAINT [PK_PostingRuleGroups] PRIMARY KEY ([RuleGroupId]),
    CONSTRAINT [UK_PostingRuleGroups] UNIQUE ([TenantId], [CompanyId], [RuleGroupCode])
);

-- ============================================
-- Posting Rule Details
-- تفاصيل قواعد الترحيل
-- ============================================
CREATE TABLE [Finance].[PostingRules] (
    [PostingRuleId] INT IDENTITY(1,1) NOT NULL,
    [RuleGroupId] INT NOT NULL,
    [RuleSequence] INT NOT NULL,
    [RuleName] NVARCHAR(200) NOT NULL,
    [AccountDeterminationType] NVARCHAR(50) NOT NULL, -- Fixed, Dynamic, Expression
    
    -- Debit Side
    [DebitAccountId] BIGINT NULL, -- For Fixed type
    [DebitAccountExpression] NVARCHAR(500) NULL, -- For Dynamic/Expression type
    [DebitAmountExpression] NVARCHAR(500) NOT NULL,
    
    -- Credit Side
    [CreditAccountId] BIGINT NULL,
    [CreditAccountExpression] NVARCHAR(500) NULL,
    [CreditAmountExpression] NVARCHAR(500) NOT NULL,
    
    -- Additional Dimensions
    [RequiresCostCenter] BIT NOT NULL DEFAULT 0,
    [CostCenterExpression] NVARCHAR(500) NULL,
    [RequiresProject] BIT NOT NULL DEFAULT 0,
    [ProjectExpression] NVARCHAR(500) NULL,
    [RequiresDepartment] BIT NOT NULL DEFAULT 0,
    [DepartmentExpression] NVARCHAR(500) NULL,
    
    -- Description & Reference
    [DescriptionTemplate] NVARCHAR(500) NULL,
    [ReferenceTemplate] NVARCHAR(200) NULL,
    
    -- Conditions
    [ConditionExpression] NVARCHAR(1000) NULL, -- When to apply this rule
    
    [IsActive] BIT NOT NULL DEFAULT 1,
    [CreatedBy] UNIQUEIDENTIFIER NOT NULL,
    [CreatedDateUtc] DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
    CONSTRAINT [PK_PostingRules] PRIMARY KEY ([PostingRuleId]),
    CONSTRAINT [FK_PostingRules_RuleGroups] FOREIGN KEY ([RuleGroupId])
        REFERENCES [Finance].[PostingRuleGroups]([RuleGroupId])
);

-- ============================================
-- Account Mappings (for module-specific accounts)
-- تعيينات الحسابات (للحسابات الخاصة بالوحدات)
-- ============================================
CREATE TABLE [Finance].[AccountMappings] (
    [MappingId] INT IDENTITY(1,1) NOT NULL,
    [TenantId] UNIQUEIDENTIFIER NOT NULL,
    [CompanyId] UNIQUEIDENTIFIER NOT NULL,
    [MappingKey] NVARCHAR(100) NOT NULL, -- e.g., 'SalesRevenue', 'COGS', 'AR', 'AP'
    [AccountId] BIGINT NOT NULL,
    [Description] NVARCHAR(500) NULL,
    [IsActive] BIT NOT NULL DEFAULT 1,
    CONSTRAINT [PK_AccountMappings] PRIMARY KEY ([MappingId]),
    CONSTRAINT [FK_AccountMappings_Accounts] FOREIGN KEY ([AccountId])
        REFERENCES [Finance].[ChartOfAccounts]([AccountId]),
    CONSTRAINT [UK_AccountMappings] UNIQUE ([TenantId], [CompanyId], [MappingKey])
);

-- ============================================
-- Source Document Tracking
-- تتبع المستندات المصدر
-- ============================================
CREATE TABLE [Finance].[SourceDocuments] (
    [SourceDocumentId] BIGINT IDENTITY(1,1) NOT NULL,
    [TenantId] UNIQUEIDENTIFIER NOT NULL,
    [CompanyId] UNIQUEIDENTIFIER NOT NULL,
    [SourceModule] NVARCHAR(50) NOT NULL,
    [DocumentType] NVARCHAR(50) NOT NULL,
    [DocumentId] BIGINT NOT NULL,
    [DocumentNumber] NVARCHAR(50) NOT NULL,
    [DocumentDate] DATE NOT NULL,
    [DocumentStatus] NVARCHAR(20) NOT NULL,
    [TotalAmount] DECIMAL(19,4) NOT NULL,
    [CurrencyCode] CHAR(3) NOT NULL,
    [ExchangeRate] DECIMAL(10,6) NOT NULL DEFAULT 1,
    [IsPosted] BIT NOT NULL DEFAULT 0,
    [PostedDate] DATETIME2 NULL,
    [JournalEntryId] BIGINT NULL,
    [PostingError] NVARCHAR(MAX) NULL,
    [RetryCount] INT NOT NULL DEFAULT 0,
    [CreatedDateUtc] DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
    CONSTRAINT [PK_SourceDocuments] PRIMARY KEY ([SourceDocumentId]),
    CONSTRAINT [UK_SourceDocuments] UNIQUE ([TenantId], [SourceModule], [DocumentType], [DocumentId])
);

CREATE NONCLUSTERED INDEX [IX_SourceDocuments_NotPosted]
    ON [Finance].[SourceDocuments]([TenantId], [CompanyId], [IsPosted])
    WHERE [IsPosted] = 0;

-- ============================================
-- Journal Entry Reversal Tracking
-- تتبع عكس القيود
-- ============================================
CREATE TABLE [Finance].[JournalEntryReversals] (
    [ReversalId] BIGINT IDENTITY(1,1) NOT NULL,
    [OriginalJournalEntryId] BIGINT NOT NULL,
    [ReversalJournalEntryId] BIGINT NOT NULL,
    [ReversalReason] NVARCHAR(500) NOT NULL,
    [ReversalType] NVARCHAR(20) NOT NULL, -- Full, Partial
    [ReversedBy] UNIQUEIDENTIFIER NOT NULL,
    [ReversedDateUtc] DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
    CONSTRAINT [PK_JournalEntryReversals] PRIMARY KEY ([ReversalId]),
    CONSTRAINT [FK_Reversals_Original] FOREIGN KEY ([OriginalJournalEntryId])
        REFERENCES [Finance].[JournalEntries]([JournalEntryId]),
    CONSTRAINT [FK_Reversals_Reversal] FOREIGN KEY ([ReversalJournalEntryId])
        REFERENCES [Finance].[JournalEntries]([JournalEntryId])
);

-- ============================================
-- Financial Period Lock
-- قفل الفترات المالية
-- ============================================
CREATE TABLE [Finance].[PeriodLocks] (
    [PeriodLockId] INT IDENTITY(1,1) NOT NULL,
    [TenantId] UNIQUEIDENTIFIER NOT NULL,
    [CompanyId] UNIQUEIDENTIFIER NOT NULL,
    [FiscalYearId] INT NOT NULL,
    [PeriodNumber] TINYINT NOT NULL,
    [LockStatus] NVARCHAR(20) NOT NULL, -- Open, SoftLock, HardLock
    [LockDate] DATETIME2 NULL,
    [LockedBy] UNIQUEIDENTIFIER NULL,
    [LockReason] NVARCHAR(500) NULL,
    [CanOverride] BIT NOT NULL DEFAULT 0,
    CONSTRAINT [PK_PeriodLocks] PRIMARY KEY ([PeriodLockId]),
    CONSTRAINT [UK_PeriodLocks] UNIQUE ([TenantId], [CompanyId], [FiscalYearId], [PeriodNumber])
);
```

### 1.2 Chart of Accounts Extended Schema

```sql
-- ============================================
-- Chart of Accounts with Extended Attributes
-- دليل الحسابات مع السمات الموسعة
-- ============================================
CREATE TABLE [Finance].[ChartOfAccounts] (
    [AccountId] BIGINT IDENTITY(1,1) NOT NULL,
    [TenantId] UNIQUEIDENTIFIER NOT NULL,
    [CompanyId] UNIQUEIDENTIFIER NOT NULL,
    [AccountCode] NVARCHAR(20) NOT NULL,
    [AccountName] NVARCHAR(200) NOT NULL,
    [AccountName_AR] NVARCHAR(200) NULL,
    [ParentAccountId] BIGINT NULL,
    [AccountTypeId] SMALLINT NOT NULL,
    [AccountLevel] TINYINT NOT NULL,
    
    -- Hierarchy
    [HierarchyPath] HIERARCHYID NOT NULL,
    [FullAccountPath] NVARCHAR(500) NOT NULL, -- e.g., '1-10-100-1000'
    
    -- Account Properties
    [NormalBalance] CHAR(1) NOT NULL, -- D=Debit, C=Credit
    [IsActive] BIT NOT NULL DEFAULT 1,
    [IsSystemAccount] BIT NOT NULL DEFAULT 0,
    [AllowDirectPosting] BIT NOT NULL DEFAULT 1,
    [AllowManualEntry] BIT NOT NULL DEFAULT 1,
    
    -- Posting Requirements
    [RequiresCostCenter] BIT NOT NULL DEFAULT 0,
    [RequiresProject] BIT NOT NULL DEFAULT 0,
    [RequiresDepartment] BIT NOT NULL DEFAULT 0,
    [RequiresEmployee] BIT NOT NULL DEFAULT 0,
    [RequiresCustomer] BIT NOT NULL DEFAULT 0,
    [RequiresSupplier] BIT NOT NULL DEFAULT 0,
    
    -- Currency
    [CurrencyCode] CHAR(3) NULL,
    [AllowMultiCurrency] BIT NOT NULL DEFAULT 0,
    
    -- Reconciliation
    [RequiresReconciliation] BIT NOT NULL DEFAULT 0,
    [ReconciliationFrequency] NVARCHAR(20) NULL, -- Daily, Weekly, Monthly
    
    -- Budget & Control
    [HasBudget] BIT NOT NULL DEFAULT 0,
    [BudgetControlLevel] NVARCHAR(20) NULL, -- None, Warning, Block
    
    -- Cash Flow
    [CashFlowCategory] NVARCHAR(50) NULL,
    [CashFlowType] NVARCHAR(20) NULL, -- Operating, Investing, Financing
    
    -- Description & Notes
    [Description] NVARCHAR(500) NULL,
    [Notes] NVARCHAR(MAX) NULL,
    
    -- Audit
    [CreatedBy] UNIQUEIDENTIFIER NOT NULL,
    [CreatedDateUtc] DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
    [CreatedTimezone] NVARCHAR(50) NOT NULL,
    [CreatedDateLocal] DATETIME2 NOT NULL,
    [LastModifiedBy] UNIQUEIDENTIFIER NULL,
    [LastModifiedDateUtc] DATETIME2 NULL,
    [IsDeleted] BIT NOT NULL DEFAULT 0,
    [DeletedBy] UNIQUEIDENTIFIER NULL,
    [DeletedDateUtc] DATETIME2 NULL,
    [RowVersion] ROWVERSION NOT NULL,
    
    CONSTRAINT [PK_ChartOfAccounts] PRIMARY KEY ([AccountId]),
    CONSTRAINT [FK_Accounts_Parent] FOREIGN KEY ([ParentAccountId])
        REFERENCES [Finance].[ChartOfAccounts]([AccountId]),
    CONSTRAINT [FK_Accounts_AccountType] FOREIGN KEY ([AccountTypeId])
        REFERENCES [Finance].[AccountTypes]([AccountTypeId]),
    CONSTRAINT [UK_Accounts_Code] UNIQUE ([TenantId], [CompanyId], [AccountCode])
);

-- Create computed column for level
ALTER TABLE [Finance].[ChartOfAccounts]
ADD [Level] AS [HierarchyPath].GetLevel() PERSISTED;

-- Create indexes
CREATE NONCLUSTERED INDEX [IX_Accounts_Hierarchy]
    ON [Finance].[ChartOfAccounts]([TenantId], [CompanyId], [HierarchyPath])
    INCLUDE ([AccountCode], [AccountName]);

CREATE NONCLUSTERED INDEX [IX_Accounts_Parent]
    ON [Finance].[ChartOfAccounts]([ParentAccountId])
    INCLUDE ([AccountCode], [AccountName], [IsActive]);

-- ============================================
-- Cost Centers
-- مراكز التكلفة
-- ============================================
CREATE TABLE [Finance].[CostCenters] (
    [CostCenterId] BIGINT IDENTITY(1,1) NOT NULL,
    [TenantId] UNIQUEIDENTIFIER NOT NULL,
    [CompanyId] UNIQUEIDENTIFIER NOT NULL,
    [CostCenterCode] NVARCHAR(20) NOT NULL,
    [CostCenterName] NVARCHAR(200) NOT NULL,
    [CostCenterName_AR] NVARCHAR(200) NULL,
    [ParentCostCenterId] BIGINT NULL,
    [CostCenterType] NVARCHAR(20) NOT NULL, -- Department, Project, Location, Product
    [HierarchyPath] HIERARCHYID NOT NULL,
    [ManagerEmployeeId] BIGINT NULL,
    [IsActive] BIT NOT NULL DEFAULT 1,
    [Description] NVARCHAR(500) NULL,
    [CreatedBy] UNIQUEIDENTIFIER NOT NULL,
    [CreatedDateUtc] DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
    [LastModifiedBy] UNIQUEIDENTIFIER NULL,
    [LastModifiedDateUtc] DATETIME2 NULL,
    [IsDeleted] BIT NOT NULL DEFAULT 0,
    [RowVersion] ROWVERSION NOT NULL,
    CONSTRAINT [PK_CostCenters] PRIMARY KEY ([CostCenterId]),
    CONSTRAINT [FK_CostCenters_Parent] FOREIGN KEY ([ParentCostCenterId])
        REFERENCES [Finance].[CostCenters]([CostCenterId]),
    CONSTRAINT [UK_CostCenters_Code] UNIQUE ([TenantId], [CompanyId], [CostCenterCode])
);

-- ============================================
-- Projects (for project accounting)
-- المشاريع (للمحاسبة على المشاريع)
-- ============================================
CREATE TABLE [Finance].[Projects] (
    [ProjectId] BIGINT IDENTITY(1,1) NOT NULL,
    [TenantId] UNIQUEIDENTIFIER NOT NULL,
    [CompanyId] UNIQUEIDENTIFIER NOT NULL,
    [ProjectCode] NVARCHAR(20) NOT NULL,
    [ProjectName] NVARCHAR(200) NOT NULL,
    [ProjectName_AR] NVARCHAR(200) NULL,
    [ParentProjectId] BIGINT NULL,
    [ProjectType] NVARCHAR(20) NOT NULL, -- Internal, External, Capital
    [ProjectStatus] NVARCHAR(20) NOT NULL, -- Planning, Active, OnHold, Completed, Cancelled
    [StartDate] DATE NOT NULL,
    [EstimatedEndDate] DATE NULL,
    [ActualEndDate] DATE NULL,
    [ProjectManagerId] BIGINT NULL,
    [CustomerId] BIGINT NULL,
    [BudgetAmount] DECIMAL(19,4) NULL,
    [ContractValue] DECIMAL(19,4) NULL,
    [CurrencyCode] CHAR(3) NOT NULL,
    [IsActive] BIT NOT NULL DEFAULT 1,
    [Description] NVARCHAR(500) NULL,
    [CreatedBy] UNIQUEIDENTIFIER NOT NULL,
    [CreatedDateUtc] DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
    [LastModifiedBy] UNIQUEIDENTIFIER NULL,
    [LastModifiedDateUtc] DATETIME2 NULL,
    [IsDeleted] BIT NOT NULL DEFAULT 0,
    [RowVersion] ROWVERSION NOT NULL,
    CONSTRAINT [PK_Projects] PRIMARY KEY ([ProjectId]),
    CONSTRAINT [FK_Projects_Parent] FOREIGN KEY ([ParentProjectId])
        REFERENCES [Finance].[Projects]([ProjectId]),
    CONSTRAINT [UK_Projects_Code] UNIQUE ([TenantId], [CompanyId], [ProjectCode])
);
```

### 1.3 Journal Entries Extended Schema

```sql
-- ============================================
-- Journal Entry Header
-- رأس القيد اليومي
-- ============================================
CREATE TABLE [Finance].[JournalEntries] (
    [JournalEntryId] BIGINT IDENTITY(1,1) NOT NULL,
    [TenantId] UNIQUEIDENTIFIER NOT NULL,
    [CompanyId] UNIQUEIDENTIFIER NOT NULL,
    [BranchId] UNIQUEIDENTIFIER NULL,
    
    -- Document Information
    [JournalCode] NVARCHAR(20) NOT NULL,
    [JournalTypeId] SMALLINT NOT NULL,
    [PostingDate] DATE NOT NULL,
    [DocumentDate] DATE NOT NULL,
    [FiscalYearId] INT NOT NULL,
    [FiscalPeriodId] INT NOT NULL,
    
    -- Amounts
    [TotalDebit] DECIMAL(19,4) NOT NULL,
    [TotalCredit] DECIMAL(19,4) NOT NULL,
    [CurrencyCode] CHAR(3) NOT NULL,
    [ExchangeRate] DECIMAL(10,6) NOT NULL DEFAULT 1,
    
    -- Status & Workflow
    [Status] NVARCHAR(20) NOT NULL DEFAULT 'Draft', -- Draft, Posted, Approved, Reversed, Cancelled
    [IsReversed] BIT NOT NULL DEFAULT 0,
    [IsRecurring] BIT NOT NULL DEFAULT 0,
    [RecurrencePattern] NVARCHAR(100) NULL,
    
    -- Source Tracking
    [SourceModule] NVARCHAR(50) NULL,
    [SourceDocumentType] NVARCHAR(50) NULL,
    [SourceDocumentId] BIGINT NULL,
    [SourceDocumentNumber] NVARCHAR(50) NULL,
    [IsAutoGenerated] BIT NOT NULL DEFAULT 0,
    
    -- References
    [Reference] NVARCHAR(50) NULL,
    [Description] NVARCHAR(500) NULL,
    [Notes] NVARCHAR(MAX) NULL,
    [Attachments] NVARCHAR(MAX) NULL, -- JSON array of attachment URLs
    
    -- Posting Information
    [PostedBy] UNIQUEIDENTIFIER NULL,
    [PostedDateUtc] DATETIME2 NULL,
    [ApprovedBy] UNIQUEIDENTIFIER NULL,
    [ApprovedDateUtc] DATETIME2 NULL,
    
    -- Audit Trail
    [CreatedBy] UNIQUEIDENTIFIER NOT NULL,
    [CreatedDateUtc] DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
    [CreatedTimezone] NVARCHAR(50) NOT NULL,
    [CreatedDateLocal] DATETIME2 NOT NULL,
    [LastModifiedBy] UNIQUEIDENTIFIER NULL,
    [LastModifiedDateUtc] DATETIME2 NULL,
    [IsDeleted] BIT NOT NULL DEFAULT 0,
    [DeletedBy] UNIQUEIDENTIFIER NULL,
    [DeletedDateUtc] DATETIME2 NULL,
    [RowVersion] ROWVERSION NOT NULL,
    
    CONSTRAINT [PK_JournalEntries] PRIMARY KEY ([JournalEntryId]),
    CONSTRAINT [FK_JournalEntries_Companies] FOREIGN KEY ([CompanyId])
        REFERENCES [System].[Companies]([CompanyId]),
    CONSTRAINT [FK_JournalEntries_JournalTypes] FOREIGN KEY ([JournalTypeId])
        REFERENCES [Finance].[JournalTypes]([JournalTypeId]),
    CONSTRAINT [CK_JournalEntries_Balance] CHECK ([TotalDebit] = [TotalCredit])
);

-- Partitioning by PostingDate
CREATE PARTITION SCHEME ps_JournalEntries
AS PARTITION pf_MonthlyFinancial
ALL TO ([PRIMARY]);

-- ============================================
-- Journal Entry Lines
-- سطور القيد اليومي
-- ============================================
CREATE TABLE [Finance].[JournalEntryLines] (
    [LineId] BIGINT IDENTITY(1,1) NOT NULL,
    [JournalEntryId] BIGINT NOT NULL,
    [LineNumber] INT NOT NULL,
    [PostingDate] DATE NOT NULL, -- For partitioning
    
    -- Account Information
    [AccountId] BIGINT NOT NULL,
    [SubAccountId] BIGINT NULL,
    
    -- Dimensions
    [CostCenterId] BIGINT NULL,
    [ProjectId] BIGINT NULL,
    [DepartmentId] BIGINT NULL,
    [EmployeeId] BIGINT NULL,
    [CustomerId] BIGINT NULL,
    [SupplierId] BIGINT NULL,
    
    -- Amounts
    [DebitAmount] DECIMAL(19,4) NOT NULL DEFAULT 0,
    [CreditAmount] DECIMAL(19,4) NOT NULL DEFAULT 0,
    [CurrencyCode] CHAR(3) NOT NULL,
    [ExchangeRate] DECIMAL(10,6) NOT NULL DEFAULT 1,
    [DebitAmountBase] AS ([DebitAmount] * [ExchangeRate]) PERSISTED,
    [CreditAmountBase] AS ([CreditAmount] * [ExchangeRate]) PERSISTED,
    
    -- Description & References
    [Description] NVARCHAR(500) NULL,
    [Reference] NVARCHAR(50) NULL,
    [AnalysisCode] NVARCHAR(50) NULL,
    
    -- Source Tracking
    [SourceLineId] BIGINT NULL,
    
    CONSTRAINT [PK_JournalEntryLines] PRIMARY KEY ([LineId]),
    CONSTRAINT [FK_JournalEntryLines_Header] FOREIGN KEY ([JournalEntryId])
        REFERENCES [Finance].[JournalEntries]([JournalEntryId]) ON DELETE CASCADE,
    CONSTRAINT [FK_JournalEntryLines_Account] FOREIGN KEY ([AccountId])
        REFERENCES [Finance].[ChartOfAccounts]([AccountId]),
    CONSTRAINT [CK_JournalEntryLines_Amount] CHECK (
        ([DebitAmount] > 0 AND [CreditAmount] = 0) OR 
        ([CreditAmount] > 0 AND [DebitAmount] = 0)
    )
) ON ps_JournalEntries([PostingDate]);

-- Create indexes
CREATE NONCLUSTERED INDEX [IX_JournalEntries_Posting]
    ON [Finance].[JournalEntries]([TenantId], [CompanyId], [PostingDate], [Status])
    INCLUDE ([JournalCode], [TotalDebit], [TotalCredit]);

CREATE NONCLUSTERED INDEX [IX_JournalEntries_Source]
    ON [Finance].[JournalEntries]([TenantId], [SourceModule], [SourceDocumentType], [SourceDocumentId])
    WHERE [SourceModule] IS NOT NULL;

CREATE NONCLUSTERED INDEX [IX_JournalEntryLines_Account]
    ON [Finance].[JournalEntryLines]([AccountId], [PostingDate])
    INCLUDE ([DebitAmount], [CreditAmount], [CostCenterId], [ProjectId]);
```

## 2. Financial Posting Service - خدمة الترحيل المالي

### 2.1 IFinancialPostingService Interface

```csharp
namespace ERPSaaS.Modules.Finance.Application.Services
{
    /// <summary>
    /// Service for posting financial transactions from various modules
    /// خدمة لترحيل المعاملات المالية من الوحدات المختلفة
    /// </summary>
    public interface IFinancialPostingService
    {
        /// <summary>
        /// Post a source document to general ledger
        /// ترحيل مستند مصدر إلى دفتر الأستاذ العام
        /// </summary>
        Task<Result<JournalEntry>> PostDocumentAsync(
            FinancialPostingRequest request,
            CancellationToken cancellationToken = default);
        
        /// <summary>
        /// Reverse a posted journal entry
        /// عكس قيد يومي مرحل
        /// </summary>
        Task<Result<JournalEntry>> ReverseJournalEntryAsync(
            long journalEntryId,
            string reason,
            DateTime? reversalDate = null,
            CancellationToken cancellationToken = default);
        
        /// <summary>
        /// Recreate journal entries for a period (retroactive posting)
        /// إعادة إنشاء القيود اليومية لفترة (ترحيل بأثر رجعي)
        /// </summary>
        Task<Result<RecreateEntriesResult>> RecreateJournalEntriesAsync(
            RecreateEntriesRequest request,
            CancellationToken cancellationToken = default);
        
        /// <summary>
        /// Get posting rules for a document type
        /// الحصول على قواعد الترحيل لنوع مستند
        /// </summary>
        Task<List<PostingRule>> GetPostingRulesAsync(
            string sourceModule,
            string documentType,
            Guid companyId);
        
        /// <summary>
        /// Validate posting before actual posting
        /// التحقق من الترحيل قبل الترحيل الفعلي
        /// </summary>
        Task<ValidationResult> ValidatePostingAsync(
            FinancialPostingRequest request);
        
        /// <summary>
        /// Check if period is locked
        /// التحقق مما إذا كانت الفترة مقفلة
        /// </summary>
        Task<bool> IsPeriodLockedAsync(
            Guid companyId,
            DateTime postingDate);
        
        /// <summary>
        /// Get journal entry by source document
        /// الحصول على القيد اليومي حسب المستند المصدر
        /// </summary>
        Task<JournalEntry> GetJournalEntryBySourceAsync(
            string sourceModule,
            string documentType,
            long documentId);
    }
}
```

### 2.2 FinancialPostingRequest Model

```csharp
namespace ERPSaaS.Modules.Finance.Application.Models
{
    /// <summary>
    /// Request model for financial posting
    /// نموذج طلب للترحيل المالي
    /// </summary>
    public class FinancialPostingRequest
    {
        public Guid TenantId { get; set; }
        public Guid CompanyId { get; set; }
        public Guid? BranchId { get; set; }
        
        // Source Document Information
        public string SourceModule { get; set; }
        public string DocumentType { get; set; }
        public long DocumentId { get; set; }
        public string DocumentNumber { get; set; }
        
        // Dates
        public DateTime DocumentDate { get; set; }
        public DateTime PostingDate { get; set; }
        
        // Currency
        public string CurrencyCode { get; set; }
        public decimal ExchangeRate { get; set; } = 1m;
        
        // References
        public string Reference { get; set; }
        public string Description { get; set; }
        public string Notes { get; set; }
        
        // Posting Lines
        public List<PostingLineRequest> Lines { get; set; } = new();
        
        // Options
        public bool AutoPost { get; set; } = true;
        public bool ValidateOnly { get; set; } = false;
        public bool AllowPeriodOverride { get; set; } = false;
    }
    
    public class PostingLineRequest
    {
        public int LineNumber { get; set; }
        
        // Account
        public long? AccountId { get; set; }
        public string AccountCode { get; set; }
        
        // Amount
        public decimal DebitAmount { get; set; }
        public decimal CreditAmount { get; set; }
        
        // Dimensions
        public long? CostCenterId { get; set; }
        public long? ProjectId { get; set; }
        public long? DepartmentId { get; set; }
        public long? EmployeeId { get; set; }
        public long? CustomerId { get; set; }
        public long? SupplierId { get; set; }
        
        // Description
        public string Description { get; set; }
        public string Reference { get; set; }
        public string AnalysisCode { get; set; }
        
        // Source tracking
        public long? SourceLineId { get; set; }
    }
}
```

### 2.3 FinancialPostingService Implementation

```csharp
namespace ERPSaaS.Modules.Finance.Infrastructure.Services
{
    public class FinancialPostingService : IFinancialPostingService
    {
        private readonly ApplicationDbContext _context;
        private readonly ICurrentUser _currentUser;
        private readonly IDateTime _dateTime;
        private readonly ILogger<FinancialPostingService> _logger;
        private readonly IEventBus _eventBus;
        
        public FinancialPostingService(
            ApplicationDbContext context,
            ICurrentUser currentUser,
            IDateTime dateTime,
            ILogger<FinancialPostingService> logger,
            IEventBus eventBus)
        {
            _context = context;
            _currentUser = currentUser;
            _dateTime = dateTime;
            _logger = logger;
            _eventBus = eventBus;
        }
        
        public async Task<Result<JournalEntry>> PostDocumentAsync(
            FinancialPostingRequest request,
            CancellationToken cancellationToken = default)
        {
            using var transaction = await _context.Database.BeginTransactionAsync(cancellationToken);
            
            try
            {
                // 1. Validate request
                var validationResult = await ValidatePostingAsync(request);
                if (!validationResult.IsValid)
                {
                    return Result<JournalEntry>.Failure(validationResult.Errors);
                }
                
                if (request.ValidateOnly)
                {
                    return Result<JournalEntry>.Success(null);
                }
                
                // 2. Check if period is locked
                var isPeriodLocked = await IsPeriodLockedAsync(
                    request.CompanyId,
                    request.PostingDate);
                
                if (isPeriodLocked && !request.AllowPeriodOverride)
                {
                    return Result<JournalEntry>.Failure("Period is locked for posting");
                }
                
                // 3. Check if document already posted
                var existingEntry = await GetJournalEntryBySourceAsync(
                    request.SourceModule,
                    request.DocumentType,
                    request.DocumentId);
                
                if (existingEntry != null)
                {
                    return Result<JournalEntry>.Failure("Document already posted");
                }
                
                // 4. Get fiscal period
                var fiscalPeriod = await GetFiscalPeriodAsync(
                    request.CompanyId,
                    request.PostingDate);
                
                if (fiscalPeriod == null)
                {
                    return Result<JournalEntry>.Failure("No fiscal period found for posting date");
                }
                
                // 5. Apply posting rules if no lines provided
                if (request.Lines == null || !request.Lines.Any())
                {
                    var postingRules = await GetPostingRulesAsync(
                        request.SourceModule,
                        request.DocumentType,
                        request.CompanyId);
                    
                    if (!postingRules.Any())
                    {
                        return Result<JournalEntry>.Failure("No posting rules configured");
                    }
                    
                    request.Lines = await ApplyPostingRulesAsync(
                        postingRules,
                        request);
                }
                
                // 6. Resolve account IDs
                await ResolveAccountIdsAsync(request.Lines);
                
                // 7. Create journal entry
                var journalCode = await GenerateJournalCodeAsync(request.CompanyId);
                
                var journalEntry = new JournalEntry
                {
                    TenantId = request.TenantId,
                    CompanyId = request.CompanyId,
                    BranchId = request.BranchId,
                    JournalCode = journalCode,
                    JournalTypeId = await GetJournalTypeIdAsync(request.SourceModule),
                    PostingDate = request.PostingDate,
                    DocumentDate = request.DocumentDate,
                    FiscalYearId = fiscalPeriod.FiscalYearId,
                    FiscalPeriodId = fiscalPeriod.FiscalPeriodId,
                    CurrencyCode = request.CurrencyCode,
                    ExchangeRate = request.ExchangeRate,
                    SourceModule = request.SourceModule,
                    SourceDocumentType = request.DocumentType,
                    SourceDocumentId = request.DocumentId,
                    SourceDocumentNumber = request.DocumentNumber,
                    IsAutoGenerated = true,
                    Reference = request.Reference,
                    Description = request.Description,
                    Notes = request.Notes,
                    Status = request.AutoPost ? "Posted" : "Draft",
                    CreatedBy = _currentUser.UserId.Value,
                    CreatedDateUtc = _dateTime.UtcNow,
                    CreatedTimezone = _currentUser.Timezone,
                    CreatedDateLocal = _dateTime.Now
                };
                
                // 8. Add lines
                foreach (var lineRequest in request.Lines)
                {
                    var line = new JournalEntryLine
                    {
                        LineNumber = lineRequest.LineNumber,
                        PostingDate = request.PostingDate,
                        AccountId = lineRequest.AccountId.Value,
                        CostCenterId = lineRequest.CostCenterId,
                        ProjectId = lineRequest.ProjectId,
                        DepartmentId = lineRequest.DepartmentId,
                        EmployeeId = lineRequest.EmployeeId,
                        CustomerId = lineRequest.CustomerId,
                        SupplierId = lineRequest.SupplierId,
                        DebitAmount = lineRequest.DebitAmount,
                        CreditAmount = lineRequest.CreditAmount,
                        CurrencyCode = request.CurrencyCode,
                        ExchangeRate = request.ExchangeRate,
                        Description = lineRequest.Description,
                        Reference = lineRequest.Reference,
                        AnalysisCode = lineRequest.AnalysisCode,
                        SourceLineId = lineRequest.SourceLineId
                    };
                    
                    journalEntry.Lines.Add(line);
                }
                
                // 9. Calculate totals
                journalEntry.TotalDebit = journalEntry.Lines.Sum(l => l.DebitAmount);
                journalEntry.TotalCredit = journalEntry.Lines.Sum(l => l.CreditAmount);
                
                // 10. Verify balance
                if (journalEntry.TotalDebit != journalEntry.TotalCredit)
                {
                    return Result<JournalEntry>.Failure("Journal entry is not balanced");
                }
                
                // 11. Set posting information
                if (request.AutoPost)
                {
                    journalEntry.PostedBy = _currentUser.UserId;
                    journalEntry.PostedDateUtc = _dateTime.UtcNow;
                }
                
                // 12. Save to database
                _context.Set<JournalEntry>().Add(journalEntry);
                await _context.SaveChangesAsync(cancellationToken);
                
                // 13. Update source document tracking
                var sourceDocument = new SourceDocument
                {
                    TenantId = request.TenantId,
                    CompanyId = request.CompanyId,
                    SourceModule = request.SourceModule,
                    DocumentType = request.DocumentType,
                    DocumentId = request.DocumentId,
                    DocumentNumber = request.DocumentNumber,
                    DocumentDate = request.DocumentDate,
                    DocumentStatus = "Posted",
                    TotalAmount = journalEntry.TotalDebit,
                    CurrencyCode = request.CurrencyCode,
                    ExchangeRate = request.ExchangeRate,
                    IsPosted = true,
                    PostedDate = _dateTime.UtcNow,
                    JournalEntryId = journalEntry.JournalEntryId
                };
                
                _context.Set<SourceDocument>().Add(sourceDocument);
                await _context.SaveChangesAsync(cancellationToken);
                
                // 14. Commit transaction
                await transaction.CommitAsync(cancellationToken);
                
                // 15. Publish integration event
                await _eventBus.PublishAsync(new JournalEntryPostedEvent
                {
                    JournalEntryId = journalEntry.JournalEntryId,
                    SourceModule = request.SourceModule,
                    DocumentId = request.DocumentId,
                    PostingDate = request.PostingDate,
                    TotalAmount = journalEntry.TotalDebit
                });
                
                _logger.LogInformation(
                    $"Posted journal entry {journalEntry.JournalCode} for {request.SourceModule} document {request.DocumentNumber}");
                
                return Result<JournalEntry>.Success(journalEntry);
            }
            catch (Exception ex)
            {
                await transaction.RollbackAsync(cancellationToken);
                _logger.LogError(ex, "Error posting financial transaction");
                return Result<JournalEntry>.Failure($"Error posting transaction: {ex.Message}");
            }
        }
        
        public async Task<Result<JournalEntry>> ReverseJournalEntryAsync(
            long journalEntryId,
            string reason,
            DateTime? reversalDate = null,
            CancellationToken cancellationToken = default)
        {
            using var transaction = await _context.Database.BeginTransactionAsync(cancellationToken);
            
            try
            {
                // 1. Get original entry
                var originalEntry = await _context.Set<JournalEntry>()
                    .Include(je => je.Lines)
                    .FirstOrDefaultAsync(je => je.JournalEntryId == journalEntryId, cancellationToken);
                
                if (originalEntry == null)
                {
                    return Result<JournalEntry>.Failure("Journal entry not found");
                }
                
                if (originalEntry.Status != "Posted")
                {
                    return Result<JournalEntry>.Failure("Only posted entries can be reversed");
                }
                
                if (originalEntry.IsReversed)
                {
                    return Result<JournalEntry>.Failure("Entry is already reversed");
                }
                
                // 2. Check period lock
                var postingDate = reversalDate ?? _dateTime.Now.Date;
                var isPeriodLocked = await IsPeriodLockedAsync(originalEntry.CompanyId, postingDate);
                
                if (isPeriodLocked)
                {
                    return Result<JournalEntry>.Failure("Period is locked");
                }
                
                // 3. Get fiscal period
                var fiscalPeriod = await GetFiscalPeriodAsync(originalEntry.CompanyId, postingDate);
                
                // 4. Create reversal entry
                var journalCode = await GenerateJournalCodeAsync(originalEntry.CompanyId);
                
                var reversalEntry = new JournalEntry
                {
                    TenantId = originalEntry.TenantId,
                    CompanyId = originalEntry.CompanyId,
                    BranchId = originalEntry.BranchId,
                    JournalCode = journalCode,
                    JournalTypeId = originalEntry.JournalTypeId,
                    PostingDate = postingDate,
                    DocumentDate = postingDate,
                    FiscalYearId = fiscalPeriod.FiscalYearId,
                    FiscalPeriodId = fiscalPeriod.FiscalPeriodId,
                    TotalDebit = originalEntry.TotalDebit,
                    TotalCredit = originalEntry.TotalCredit,
                    CurrencyCode = originalEntry.CurrencyCode,
                    ExchangeRate = originalEntry.ExchangeRate,
                    SourceModule = originalEntry.SourceModule,
                    SourceDocumentType = originalEntry.SourceDocumentType,
                    SourceDocumentId = originalEntry.SourceDocumentId,
                    SourceDocumentNumber = originalEntry.SourceDocumentNumber,
                    IsAutoGenerated = true,
                    Reference = $"REV-{originalEntry.JournalCode}",
                    Description = $"Reversal of {originalEntry.JournalCode} - {reason}",
                    Status = "Posted",
                    PostedBy = _currentUser.UserId,
                    PostedDateUtc = _dateTime.UtcNow,
                    CreatedBy = _currentUser.UserId.Value,
                    CreatedDateUtc = _dateTime.UtcNow,
                    CreatedTimezone = _currentUser.Timezone,
                    CreatedDateLocal = _dateTime.Now
                };
                
                // 5. Add reversed lines (swap debit/credit)
                foreach (var originalLine in originalEntry.Lines)
                {
                    var reversalLine = new JournalEntryLine
                    {
                        LineNumber = originalLine.LineNumber,
                        PostingDate = postingDate,
                        AccountId = originalLine.AccountId,
                        CostCenterId = originalLine.CostCenterId,
                        ProjectId = originalLine.ProjectId,
                        DepartmentId = originalLine.DepartmentId,
                        EmployeeId = originalLine.EmployeeId,
                        CustomerId = originalLine.CustomerId,
                        SupplierId = originalLine.SupplierId,
                        // Swap debit and credit
                        DebitAmount = originalLine.CreditAmount,
                        CreditAmount = originalLine.DebitAmount,
                        CurrencyCode = originalLine.CurrencyCode,
                        ExchangeRate = originalLine.ExchangeRate,
                        Description = $"Reversal: {originalLine.Description}",
                        Reference = originalLine.Reference
                    };
                    
                    reversalEntry.Lines.Add(reversalLine);
                }
                
                // 6. Save reversal entry
                _context.Set<JournalEntry>().Add(reversalEntry);
                await _context.SaveChangesAsync(cancellationToken);
                
                // 7. Mark original as reversed
                originalEntry.IsReversed = true;
                originalEntry.LastModifiedBy = _currentUser.UserId;
                originalEntry.LastModifiedDateUtc = _dateTime.UtcNow;
                
                // 8. Create reversal tracking
                var reversalTracking = new JournalEntryReversal
                {
                    OriginalJournalEntryId = originalEntry.JournalEntryId,
                    ReversalJournalEntryId = reversalEntry.JournalEntryId,
                    ReversalReason = reason,
                    ReversalType = "Full",
                    ReversedBy = _currentUser.UserId.Value,
                    ReversedDateUtc = _dateTime.UtcNow
                };
                
                _context.Set<JournalEntryReversal>().Add(reversalTracking);
                await _context.SaveChangesAsync(cancellationToken);
                
                await transaction.CommitAsync(cancellationToken);
                
                _logger.LogInformation(
                    $"Reversed journal entry {originalEntry.JournalCode} with {reversalEntry.JournalCode}");
                
                return Result<JournalEntry>.Success(reversalEntry);
            }
            catch (Exception ex)
            {
                await transaction.RollbackAsync(cancellationToken);
                _logger.LogError(ex, "Error reversing journal entry");
                return Result<JournalEntry>.Failure($"Error reversing entry: {ex.Message}");
            }
        }
        
        // Helper methods...
        private async Task<List<PostingLineRequest>> ApplyPostingRulesAsync(
            List<PostingRule> rules,
            FinancialPostingRequest request)
        {
            var lines = new List<PostingLineRequest>();
            int lineNumber = 1;
            
            foreach (var rule in rules.OrderBy(r => r.RuleSequence))
            {
                // Evaluate condition
                if (!string.IsNullOrEmpty(rule.ConditionExpression))
                {
                    // TODO: Implement expression evaluation
                    // Skip rule if condition not met
                }
                
                // Create debit line
                var debitLine = new PostingLineRequest
                {
                    LineNumber = lineNumber++,
                    AccountId = rule.DebitAccountId,
                    DebitAmount = EvaluateAmountExpression(rule.DebitAmountExpression, request),
                    Description = EvaluateTemplate(rule.DescriptionTemplate, request),
                    Reference = EvaluateTemplate(rule.ReferenceTemplate, request)
                };
                
                lines.Add(debitLine);
                
                // Create credit line
                var creditLine = new PostingLineRequest
                {
                    LineNumber = lineNumber++,
                    AccountId = rule.CreditAccountId,
                    CreditAmount = EvaluateAmountExpression(rule.CreditAmountExpression, request),
                    Description = EvaluateTemplate(rule.DescriptionTemplate, request),
                    Reference = EvaluateTemplate(rule.ReferenceTemplate, request)
                };
                
                lines.Add(creditLine);
            }
            
            return lines;
        }
    }
}
```

تابع في الملف الرابع...
