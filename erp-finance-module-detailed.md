# Finance Module - Complete Specification
## وحدة المالية - المواصفات الكاملة

## 1. Module Overview - نظرة عامة على الوحدة

### 1.1 Core Features

```
Finance Module
├── 1. Chart of Accounts Management
│   ├── Account Hierarchy
│   ├── Account Types
│   ├── Sub-accounts
│   └── Multi-dimensional Analysis
│
├── 2. Journal Entry Management
│   ├── Manual Journal Entries
│   ├── Recurring Journal Entries
│   ├── Auto-posting from other modules
│   ├── Entry Reversal
│   └── Entry Approval Workflow
│
├── 3. General Ledger
│   ├── Account Ledger
│   ├── Trial Balance
│   ├── Account Balances
│   └── Historical Data
│
├── 4. Cost Centers & Projects
│   ├── Cost Center Hierarchy
│   ├── Project Accounting
│   ├── Cost Allocation
│   └── Budget vs Actual
│
├── 5. Fiscal Periods Management
│   ├── Fiscal Year Setup
│   ├── Period Opening/Closing
│   ├── Year-end Closing
│   └── Period Locking
│
├── 6. Financial Reports
│   ├── Balance Sheet
│   ├── Income Statement (P&L)
│   ├── Cash Flow Statement
│   ├── Trial Balance
│   ├── Account Statement
│   ├── General Ledger Report
│   └── Custom Financial Reports
│
├── 7. Bank Reconciliation
│   ├── Bank Statement Import
│   ├── Matching Transactions
│   ├── Adjustment Entries
│   └── Reconciliation Reports
│
├── 8. Currency & Exchange Rates
│   ├── Multi-currency Support
│   ├── Exchange Rate Management
│   ├── Revaluation
│   └── Gain/Loss Posting
│
├── 9. Budget Management
│   ├── Budget Preparation
│   ├── Budget Allocation
│   ├── Budget Control
│   └── Budget Reports
│
└── 10. Financial Analysis
    ├── Financial Ratios
    ├── Trend Analysis
    ├── Comparative Analysis
    └── KPI Dashboard
```

## 2. Complete Database Schema - مخطط قاعدة البيانات الكامل

### 2.1 Account Types

```sql
-- ============================================
-- Account Types
-- أنواع الحسابات
-- ============================================
CREATE TABLE [Finance].[AccountTypes] (
    [AccountTypeId] SMALLINT IDENTITY(1,1) NOT NULL,
    [TypeCode] NVARCHAR(20) NOT NULL,
    [TypeName] NVARCHAR(100) NOT NULL,
    [TypeName_AR] NVARCHAR(100) NULL,
    [ParentTypeId] SMALLINT NULL,
    [NormalBalance] CHAR(1) NOT NULL, -- D=Debit, C=Credit
    [IsBalanceSheet] BIT NOT NULL,
    [StatementSection] NVARCHAR(50) NULL,
    [SortOrder] INT NOT NULL,
    [Description] NVARCHAR(500) NULL,
    CONSTRAINT [PK_AccountTypes] PRIMARY KEY ([AccountTypeId]),
    CONSTRAINT [FK_AccountTypes_Parent] FOREIGN KEY ([ParentTypeId])
        REFERENCES [Finance].[AccountTypes]([AccountTypeId]),
    CONSTRAINT [UK_AccountTypes_Code] UNIQUE ([TypeCode]),
    CONSTRAINT [CK_AccountTypes_Balance] CHECK ([NormalBalance] IN ('D', 'C'))
);

-- Insert standard account types
INSERT INTO [Finance].[AccountTypes] 
    ([TypeCode], [TypeName], [TypeName_AR], [NormalBalance], [IsBalanceSheet], [StatementSection], [SortOrder])
VALUES
    -- Assets
    ('ASSET', 'Assets', 'الأصول', 'D', 1, 'Assets', 100),
    ('CA', 'Current Assets', 'الأصول المتداولة', 'D', 1, 'Assets', 110),
    ('CASH', 'Cash & Cash Equivalents', 'النقدية وما في حكمها', 'D', 1, 'Assets', 111),
    ('AR', 'Accounts Receivable', 'العملاء', 'D', 1, 'Assets', 112),
    ('INV', 'Inventory', 'المخزون', 'D', 1, 'Assets', 113),
    ('FA', 'Fixed Assets', 'الأصول الثابتة', 'D', 1, 'Assets', 120),
    ('IA', 'Intangible Assets', 'الأصول غير الملموسة', 'D', 1, 'Assets', 130),
    
    -- Liabilities
    ('LIAB', 'Liabilities', 'الخصوم', 'C', 1, 'Liabilities', 200),
    ('CL', 'Current Liabilities', 'الخصوم المتداولة', 'C', 1, 'Liabilities', 210),
    ('AP', 'Accounts Payable', 'الموردين', 'C', 1, 'Liabilities', 211),
    ('LTL', 'Long-term Liabilities', 'الخصوم طويلة الأجل', 'C', 1, 'Liabilities', 220),
    
    -- Equity
    ('EQ', 'Equity', 'حقوق الملكية', 'C', 1, 'Equity', 300),
    ('CAP', 'Capital', 'رأس المال', 'C', 1, 'Equity', 310),
    ('RE', 'Retained Earnings', 'الأرباح المحتجزة', 'C', 1, 'Equity', 320),
    
    -- Revenue
    ('REV', 'Revenue', 'الإيرادات', 'C', 0, 'Income', 400),
    ('SALES', 'Sales Revenue', 'إيرادات المبيعات', 'C', 0, 'Income', 410),
    ('OREV', 'Other Revenue', 'إيرادات أخرى', 'C', 0, 'Income', 420),
    
    -- Expenses
    ('EXP', 'Expenses', 'المصروفات', 'D', 0, 'Expenses', 500),
    ('COGS', 'Cost of Goods Sold', 'تكلفة البضاعة المباعة', 'D', 0, 'Expenses', 510),
    ('OPEX', 'Operating Expenses', 'مصروفات التشغيل', 'D', 0, 'Expenses', 520),
    ('ADMIN', 'Administrative Expenses', 'مصروفات إدارية', 'D', 0, 'Expenses', 521),
    ('SELL', 'Selling Expenses', 'مصروفات بيع', 'D', 0, 'Expenses', 522);
```

### 2.2 Fiscal Year & Periods

```sql
-- ============================================
-- Fiscal Years
-- السنوات المالية
-- ============================================
CREATE TABLE [Finance].[FiscalYears] (
    [FiscalYearId] INT IDENTITY(1,1) NOT NULL,
    [TenantId] UNIQUEIDENTIFIER NOT NULL,
    [CompanyId] UNIQUEIDENTIFIER NOT NULL,
    [YearCode] NVARCHAR(20) NOT NULL,
    [YearName] NVARCHAR(100) NOT NULL,
    [StartDate] DATE NOT NULL,
    [EndDate] DATE NOT NULL,
    [Status] NVARCHAR(20) NOT NULL DEFAULT 'Active', -- Active, Closed, Locked
    [NumberOfPeriods] TINYINT NOT NULL DEFAULT 12,
    [IsCurrent] BIT NOT NULL DEFAULT 0,
    [ClosedBy] UNIQUEIDENTIFIER NULL,
    [ClosedDate] DATETIME2 NULL,
    [CreatedBy] UNIQUEIDENTIFIER NOT NULL,
    [CreatedDateUtc] DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
    [LastModifiedBy] UNIQUEIDENTIFIER NULL,
    [LastModifiedDateUtc] DATETIME2 NULL,
    CONSTRAINT [PK_FiscalYears] PRIMARY KEY ([FiscalYearId]),
    CONSTRAINT [UK_FiscalYears] UNIQUE ([TenantId], [CompanyId], [YearCode]),
    CONSTRAINT [CK_FiscalYears_Dates] CHECK ([EndDate] > [StartDate])
);

-- ============================================
-- Fiscal Periods
-- الفترات المالية
-- ============================================
CREATE TABLE [Finance].[FiscalPeriods] (
    [FiscalPeriodId] INT IDENTITY(1,1) NOT NULL,
    [FiscalYearId] INT NOT NULL,
    [PeriodNumber] TINYINT NOT NULL,
    [PeriodName] NVARCHAR(100) NOT NULL,
    [StartDate] DATE NOT NULL,
    [EndDate] DATE NOT NULL,
    [Status] NVARCHAR(20) NOT NULL DEFAULT 'Open', -- Open, Closed, Locked
    [IsCurrent] BIT NOT NULL DEFAULT 0,
    [ClosedBy] UNIQUEIDENTIFIER NULL,
    [ClosedDate] DATETIME2 NULL,
    CONSTRAINT [PK_FiscalPeriods] PRIMARY KEY ([FiscalPeriodId]),
    CONSTRAINT [FK_FiscalPeriods_Year] FOREIGN KEY ([FiscalYearId])
        REFERENCES [Finance].[FiscalYears]([FiscalYearId]),
    CONSTRAINT [UK_FiscalPeriods] UNIQUE ([FiscalYearId], [PeriodNumber]),
    CONSTRAINT [CK_FiscalPeriods_Dates] CHECK ([EndDate] > [StartDate])
);

CREATE NONCLUSTERED INDEX [IX_FiscalPeriods_Dates]
    ON [Finance].[FiscalPeriods]([FiscalYearId], [StartDate], [EndDate]);
```

### 2.3 Journal Types

```sql
-- ============================================
-- Journal Types
-- أنواع اليوميات
-- ============================================
CREATE TABLE [Finance].[JournalTypes] (
    [JournalTypeId] SMALLINT IDENTITY(1,1) NOT NULL,
    [TenantId] UNIQUEIDENTIFIER NULL,
    [CompanyId] UNIQUEIDENTIFIER NULL,
    [TypeCode] NVARCHAR(20) NOT NULL,
    [TypeName] NVARCHAR(100) NOT NULL,
    [TypeName_AR] NVARCHAR(100) NULL,
    [NumberSequenceId] INT NULL,
    [RequiresApproval] BIT NOT NULL DEFAULT 0,
    [AllowManualEntry] BIT NOT NULL DEFAULT 1,
    [IconClass] NVARCHAR(50) NULL,
    [ColorCode] NVARCHAR(20) NULL,
    [IsActive] BIT NOT NULL DEFAULT 1,
    [Description] NVARCHAR(500) NULL,
    CONSTRAINT [PK_JournalTypes] PRIMARY KEY ([JournalTypeId]),
    CONSTRAINT [UK_JournalTypes] UNIQUE ([TenantId], [CompanyId], [TypeCode])
);

INSERT INTO [Finance].[JournalTypes] 
    ([TenantId], [CompanyId], [TypeCode], [TypeName], [TypeName_AR], [AllowManualEntry])
VALUES
    (NULL, NULL, 'GJ', 'General Journal', 'قيد يومية عامة', 1),
    (NULL, NULL, 'SJ', 'Sales Journal', 'يومية المبيعات', 0),
    (NULL, NULL, 'PJ', 'Purchase Journal', 'يومية المشتريات', 0),
    (NULL, NULL, 'CR', 'Cash Receipts', 'قبض نقدي', 0),
    (NULL, NULL, 'CP', 'Cash Payments', 'صرف نقدي', 0),
    (NULL, NULL, 'INV', 'Inventory Journal', 'يومية المخزون', 0),
    (NULL, NULL, 'PAY', 'Payroll Journal', 'يومية الرواتب', 0),
    (NULL, NULL, 'DEP', 'Depreciation Journal', 'يومية الإهلاك', 0),
    (NULL, NULL, 'ADJ', 'Adjustment Journal', 'قيود تسوية', 1),
    (NULL, NULL, 'REV', 'Reversal Journal', 'قيود عكس', 0);
```

### 2.4 Account Balances (Summary Table)

```sql
-- ============================================
-- Account Balances Summary
-- ملخص أرصدة الحسابات
-- ============================================
CREATE TABLE [Finance].[AccountBalances] (
    [BalanceId] BIGINT IDENTITY(1,1) NOT NULL,
    [TenantId] UNIQUEIDENTIFIER NOT NULL,
    [CompanyId] UNIQUEIDENTIFIER NOT NULL,
    [AccountId] BIGINT NOT NULL,
    [FiscalYearId] INT NOT NULL,
    [FiscalPeriodId] INT NOT NULL,
    
    -- Cost Center / Project dimensions
    [CostCenterId] BIGINT NULL,
    [ProjectId] BIGINT NULL,
    
    -- Opening Balance
    [OpeningDebit] DECIMAL(19,4) NOT NULL DEFAULT 0,
    [OpeningCredit] DECIMAL(19,4) NOT NULL DEFAULT 0,
    
    -- Period Activity
    [PeriodDebit] DECIMAL(19,4) NOT NULL DEFAULT 0,
    [PeriodCredit] DECIMAL(19,4) NOT NULL DEFAULT 0,
    
    -- Closing Balance (computed)
    [ClosingDebit] AS (
        [OpeningDebit] + [PeriodDebit] - [OpeningCredit] - [PeriodCredit]
    ) PERSISTED,
    [ClosingCredit] AS (
        CASE 
            WHEN ([OpeningDebit] + [PeriodDebit]) < ([OpeningCredit] + [PeriodCredit])
            THEN ([OpeningCredit] + [PeriodCredit]) - ([OpeningDebit] + [PeriodDebit])
            ELSE 0
        END
    ) PERSISTED,
    
    -- Currency
    [CurrencyCode] CHAR(3) NOT NULL,
    
    -- Metadata
    [LastUpdatedUtc] DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
    
    CONSTRAINT [PK_AccountBalances] PRIMARY KEY ([BalanceId]),
    CONSTRAINT [FK_Balances_Account] FOREIGN KEY ([AccountId])
        REFERENCES [Finance].[ChartOfAccounts]([AccountId]),
    CONSTRAINT [FK_Balances_Period] FOREIGN KEY ([FiscalPeriodId])
        REFERENCES [Finance].[FiscalPeriods]([FiscalPeriodId]),
    CONSTRAINT [UK_AccountBalances] UNIQUE (
        [TenantId], [CompanyId], [AccountId], [FiscalPeriodId], 
        [CostCenterId], [ProjectId], [CurrencyCode]
    )
);

CREATE NONCLUSTERED INDEX [IX_AccountBalances_Lookup]
    ON [Finance].[AccountBalances]([TenantId], [CompanyId], [FiscalPeriodId], [AccountId])
    INCLUDE ([PeriodDebit], [PeriodCredit], [ClosingDebit], [ClosingCredit]);
```

### 2.5 Bank Accounts & Reconciliation

```sql
-- ============================================
-- Bank Accounts
-- الحسابات البنكية
-- ============================================
CREATE TABLE [Finance].[BankAccounts] (
    [BankAccountId] BIGINT IDENTITY(1,1) NOT NULL,
    [TenantId] UNIQUEIDENTIFIER NOT NULL,
    [CompanyId] UNIQUEIDENTIFIER NOT NULL,
    [AccountCode] NVARCHAR(20) NOT NULL,
    [AccountName] NVARCHAR(200) NOT NULL,
    [AccountName_AR] NVARCHAR(200) NULL,
    [BankName] NVARCHAR(200) NOT NULL,
    [BankBranch] NVARCHAR(200) NULL,
    [AccountNumber] NVARCHAR(50) NOT NULL,
    [IBAN] NVARCHAR(50) NULL,
    [SwiftCode] NVARCHAR(20) NULL,
    [CurrencyCode] CHAR(3) NOT NULL,
    [AccountTypeId] NVARCHAR(20) NOT NULL, -- Current, Savings, Deposit
    [GLAccountId] BIGINT NOT NULL, -- Link to GL Account
    [CurrentBalance] DECIMAL(19,4) NOT NULL DEFAULT 0,
    [IsActive] BIT NOT NULL DEFAULT 1,
    [CreatedBy] UNIQUEIDENTIFIER NOT NULL,
    [CreatedDateUtc] DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
    [LastModifiedBy] UNIQUEIDENTIFIER NULL,
    [LastModifiedDateUtc] DATETIME2 NULL,
    [IsDeleted] BIT NOT NULL DEFAULT 0,
    [RowVersion] ROWVERSION NOT NULL,
    CONSTRAINT [PK_BankAccounts] PRIMARY KEY ([BankAccountId]),
    CONSTRAINT [FK_BankAccounts_GLAccount] FOREIGN KEY ([GLAccountId])
        REFERENCES [Finance].[ChartOfAccounts]([AccountId]),
    CONSTRAINT [UK_BankAccounts] UNIQUE ([TenantId], [CompanyId], [AccountCode])
);

-- ============================================
-- Bank Reconciliation
-- تسوية البنك
-- ============================================
CREATE TABLE [Finance].[BankReconciliations] (
    [ReconciliationId] BIGINT IDENTITY(1,1) NOT NULL,
    [TenantId] UNIQUEIDENTIFIER NOT NULL,
    [CompanyId] UNIQUEIDENTIFIER NOT NULL,
    [BankAccountId] BIGINT NOT NULL,
    [ReconciliationDate] DATE NOT NULL,
    [StatementDate] DATE NOT NULL,
    [StatementReference] NVARCHAR(50) NULL,
    [OpeningBalance] DECIMAL(19,4) NOT NULL,
    [ClosingBalance] DECIMAL(19,4) NOT NULL,
    [BookBalance] DECIMAL(19,4) NOT NULL,
    [ReconciledBalance] DECIMAL(19,4) NOT NULL,
    [Difference] AS ([ClosingBalance] - [ReconciledBalance]) PERSISTED,
    [Status] NVARCHAR(20) NOT NULL DEFAULT 'InProgress', -- InProgress, Completed, Posted
    [IsBalanced] AS (CASE WHEN ABS([ClosingBalance] - [ReconciledBalance]) < 0.01 THEN 1 ELSE 0 END) PERSISTED,
    [CompletedBy] UNIQUEIDENTIFIER NULL,
    [CompletedDate] DATETIME2 NULL,
    [Notes] NVARCHAR(MAX) NULL,
    [CreatedBy] UNIQUEIDENTIFIER NOT NULL,
    [CreatedDateUtc] DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
    CONSTRAINT [PK_BankReconciliations] PRIMARY KEY ([ReconciliationId]),
    CONSTRAINT [FK_Reconciliations_BankAccount] FOREIGN KEY ([BankAccountId])
        REFERENCES [Finance].[BankAccounts]([BankAccountId])
);

-- ============================================
-- Bank Statement Lines
-- سطور كشف الحساب البنكي
-- ============================================
CREATE TABLE [Finance].[BankStatementLines] (
    [StatementLineId] BIGINT IDENTITY(1,1) NOT NULL,
    [ReconciliationId] BIGINT NOT NULL,
    [LineNumber] INT NOT NULL,
    [TransactionDate] DATE NOT NULL,
    [ValueDate] DATE NOT NULL,
    [Description] NVARCHAR(500) NOT NULL,
    [Reference] NVARCHAR(100) NULL,
    [DebitAmount] DECIMAL(19,4) NOT NULL DEFAULT 0,
    [CreditAmount] DECIMAL(19,4) NOT NULL DEFAULT 0,
    [Balance] DECIMAL(19,4) NOT NULL,
    [IsReconciled] BIT NOT NULL DEFAULT 0,
    [ReconciledJournalEntryId] BIGINT NULL,
    [ReconciledDate] DATETIME2 NULL,
    [ReconciledBy] UNIQUEIDENTIFIER NULL,
    CONSTRAINT [PK_BankStatementLines] PRIMARY KEY ([StatementLineId]),
    CONSTRAINT [FK_StatementLines_Reconciliation] FOREIGN KEY ([ReconciliationId])
        REFERENCES [Finance].[BankReconciliations]([ReconciliationId]) ON DELETE CASCADE
);
```

### 2.6 Budgets

```sql
-- ============================================
-- Budget Headers
-- رؤوس الموازنات
-- ============================================
CREATE TABLE [Finance].[Budgets] (
    [BudgetId] BIGINT IDENTITY(1,1) NOT NULL,
    [TenantId] UNIQUEIDENTIFIER NOT NULL,
    [CompanyId] UNIQUEIDENTIFIER NOT NULL,
    [BudgetCode] NVARCHAR(20) NOT NULL,
    [BudgetName] NVARCHAR(200) NOT NULL,
    [BudgetName_AR] NVARCHAR(200) NULL,
    [FiscalYearId] INT NOT NULL,
    [BudgetType] NVARCHAR(20) NOT NULL, -- Annual, Quarterly, Monthly
    [Status] NVARCHAR(20) NOT NULL DEFAULT 'Draft', -- Draft, Approved, Active, Closed
    [IsActive] BIT NOT NULL DEFAULT 0,
    [ApprovedBy] UNIQUEIDENTIFIER NULL,
    [ApprovedDate] DATETIME2 NULL,
    [Notes] NVARCHAR(MAX) NULL,
    [CreatedBy] UNIQUEIDENTIFIER NOT NULL,
    [CreatedDateUtc] DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
    [LastModifiedBy] UNIQUEIDENTIFIER NULL,
    [LastModifiedDateUtc] DATETIME2 NULL,
    CONSTRAINT [PK_Budgets] PRIMARY KEY ([BudgetId]),
    CONSTRAINT [FK_Budgets_FiscalYear] FOREIGN KEY ([FiscalYearId])
        REFERENCES [Finance].[FiscalYears]([FiscalYearId]),
    CONSTRAINT [UK_Budgets] UNIQUE ([TenantId], [CompanyId], [BudgetCode])
);

-- ============================================
-- Budget Lines
-- سطور الموازنة
-- ============================================
CREATE TABLE [Finance].[BudgetLines] (
    [BudgetLineId] BIGINT IDENTITY(1,1) NOT NULL,
    [BudgetId] BIGINT NOT NULL,
    [AccountId] BIGINT NOT NULL,
    [FiscalPeriodId] INT NOT NULL,
    [CostCenterId] BIGINT NULL,
    [ProjectId] BIGINT NULL,
    [BudgetAmount] DECIMAL(19,4) NOT NULL,
    [RevisedAmount] DECIMAL(19,4) NULL,
    [ActualAmount] DECIMAL(19,4) NOT NULL DEFAULT 0,
    [CommittedAmount] DECIMAL(19,4) NOT NULL DEFAULT 0,
    [VarianceAmount] AS ([BudgetAmount] - [ActualAmount] - [CommittedAmount]) PERSISTED,
    [VariancePercent] AS (
        CASE WHEN [BudgetAmount] <> 0 
        THEN (([BudgetAmount] - [ActualAmount] - [CommittedAmount]) / [BudgetAmount] * 100)
        ELSE 0 END
    ) PERSISTED,
    [Notes] NVARCHAR(500) NULL,
    CONSTRAINT [PK_BudgetLines] PRIMARY KEY ([BudgetLineId]),
    CONSTRAINT [FK_BudgetLines_Budget] FOREIGN KEY ([BudgetId])
        REFERENCES [Finance].[Budgets]([BudgetId]) ON DELETE CASCADE,
    CONSTRAINT [FK_BudgetLines_Account] FOREIGN KEY ([AccountId])
        REFERENCES [Finance].[ChartOfAccounts]([AccountId]),
    CONSTRAINT [FK_BudgetLines_Period] FOREIGN KEY ([FiscalPeriodId])
        REFERENCES [Finance].[FiscalPeriods]([FiscalPeriodId]),
    CONSTRAINT [UK_BudgetLines] UNIQUE ([BudgetId], [AccountId], [FiscalPeriodId], [CostCenterId], [ProjectId])
);
```

## 3. Business Logic Implementation - تطبيق منطق الأعمال

### 3.1 Chart of Accounts Service

```csharp
namespace ERPSaaS.Modules.Finance.Application.Services
{
    public interface IChartOfAccountsService
    {
        Task<Result<Account>> CreateAccountAsync(CreateAccountCommand command);
        Task<Result<Account>> UpdateAccountAsync(UpdateAccountCommand command);
        Task<Result> DeleteAccountAsync(long accountId);
        Task<Result<Account>> GetAccountByIdAsync(long accountId);
        Task<Result<List<AccountDto>>> GetAccountHierarchyAsync(Guid companyId);
        Task<Result> ValidateAccountCodeAsync(string accountCode, Guid companyId);
        Task<Result<AccountBalance>> GetAccountBalanceAsync(long accountId, int fiscalPeriodId);
    }
    
    public class ChartOfAccountsService : IChartOfAccountsService
    {
        private readonly ApplicationDbContext _context;
        private readonly ICurrentUser _currentUser;
        private readonly ICurrentTenant _currentTenant;
        
        public async Task<Result<Account>> CreateAccountAsync(CreateAccountCommand command)
        {
            // 1. Validate account code uniqueness
            var exists = await _context.Set<Account>()
                .AnyAsync(a => 
                    a.TenantId == _currentTenant.TenantId &&
                    a.CompanyId == command.CompanyId &&
                    a.AccountCode == command.AccountCode);
            
            if (exists)
            {
                return Result<Account>.Failure("Account code already exists");
            }
            
            // 2. Validate parent account
            Account parentAccount = null;
            if (command.ParentAccountId.HasValue)
            {
                parentAccount = await _context.Set<Account>()
                    .FirstOrDefaultAsync(a => a.AccountId == command.ParentAccountId.Value);
                
                if (parentAccount == null)
                {
                    return Result<Account>.Failure("Parent account not found");
                }
                
                if (!parentAccount.AllowDirectPosting)
                {
                    return Result<Account>.Failure("Cannot add sub-accounts to posting accounts");
                }
            }
            
            // 3. Calculate hierarchy path
            var hierarchyPath = parentAccount != null 
                ? parentAccount.HierarchyPath.GetDescendant(null, null)
                : HierarchyId.Parse("/");
            
            // 4. Determine account level
            var accountLevel = (byte)(parentAccount?.AccountLevel + 1 ?? 0);
            
            // 5. Build full account path
            var fullPath = parentAccount != null
                ? $"{parentAccount.FullAccountPath}-{command.AccountCode}"
                : command.AccountCode;
            
            // 6. Create account
            var account = new Account
            {
                TenantId = _currentTenant.TenantId,
                CompanyId = command.CompanyId,
                AccountCode = command.AccountCode,
                AccountName = command.AccountName,
                AccountName_AR = command.AccountName_AR,
                ParentAccountId = command.ParentAccountId,
                AccountTypeId = command.AccountTypeId,
                AccountLevel = accountLevel,
                HierarchyPath = hierarchyPath,
                FullAccountPath = fullPath,
                NormalBalance = command.NormalBalance,
                IsActive = true,
                AllowDirectPosting = command.AllowDirectPosting,
                AllowManualEntry = command.AllowManualEntry,
                RequiresCostCenter = command.RequiresCostCenter,
                RequiresProject = command.RequiresProject,
                CurrencyCode = command.CurrencyCode,
                Description = command.Description,
                CreatedBy = _currentUser.UserId.Value
            };
            
            _context.Set<Account>().Add(account);
            await _context.SaveChangesAsync();
            
            return Result<Account>.Success(account);
        }
        
        public async Task<Result<AccountBalance>> GetAccountBalanceAsync(
            long accountId, 
            int fiscalPeriodId)
        {
            var balance = await _context.Set<AccountBalance>()
                .FirstOrDefaultAsync(b => 
                    b.TenantId == _currentTenant.TenantId &&
                    b.AccountId == accountId &&
                    b.FiscalPeriodId == fiscalPeriodId);
            
            if (balance == null)
            {
                // Calculate from journal entries
                balance = await CalculateAccountBalanceAsync(accountId, fiscalPeriodId);
            }
            
            return Result<AccountBalance>.Success(balance);
        }
        
        private async Task<AccountBalance> CalculateAccountBalanceAsync(
            long accountId,
            int fiscalPeriodId)
        {
            var period = await _context.Set<FiscalPeriod>()
                .Include(p => p.FiscalYear)
                .FirstOrDefaultAsync(p => p.FiscalPeriodId == fiscalPeriodId);
            
            // Get all journal entry lines for this account in the period
            var lines = await _context.Set<JournalEntryLine>()
                .Where(l => l.AccountId == accountId)
                .Where(l => l.PostingDate >= period.StartDate && l.PostingDate <= period.EndDate)
                .Where(l => l.JournalEntry.Status == "Posted")
                .ToListAsync();
            
            var periodDebit = lines.Sum(l => l.DebitAmount);
            var periodCredit = lines.Sum(l => l.CreditAmount);
            
            // Get opening balance (previous period closing or zero)
            var openingBalance = await GetOpeningBalanceAsync(accountId, period);
            
            var balance = new AccountBalance
            {
                TenantId = _currentTenant.TenantId,
                CompanyId = period.FiscalYear.CompanyId,
                AccountId = accountId,
                FiscalYearId = period.FiscalYearId,
                FiscalPeriodId = fiscalPeriodId,
                OpeningDebit = openingBalance.Debit,
                OpeningCredit = openingBalance.Credit,
                PeriodDebit = periodDebit,
                PeriodCredit = periodCredit,
                CurrencyCode = "SAR" // TODO: Get from account
            };
            
            return balance;
        }
    }
}
```

### 3.2 Trial Balance Generation

```csharp
public interface IFinancialReportService
{
    Task<Result<TrialBalanceReport>> GenerateTrialBalanceAsync(TrialBalanceRequest request);
    Task<Result<BalanceSheetReport>> GenerateBalanceSheetAsync(BalanceSheetRequest request);
    Task<Result<IncomeStatementReport>> GenerateIncomeStatementAsync(IncomeStatementRequest request);
}

public class FinancialReportService : IFinancialReportService
{
    public async Task<Result<TrialBalanceReport>> GenerateTrialBalanceAsync(
        TrialBalanceRequest request)
    {
        var report = new TrialBalanceReport
        {
            CompanyId = request.CompanyId,
            AsOfDate = request.AsOfDate,
            IncludeZeroBalances = request.IncludeZeroBalances,
            CostCenterId = request.CostCenterId,
            ProjectId = request.ProjectId
        };
        
        // Get fiscal period for the date
        var fiscalPeriod = await GetFiscalPeriodForDateAsync(
            request.CompanyId,
            request.AsOfDate);
        
        if (fiscalPeriod == null)
        {
            return Result<TrialBalanceReport>.Failure("No fiscal period found for date");
        }
        
        // Get all accounts with balances
        var query = _context.Set<Account>()
            .Where(a => a.TenantId == _currentTenant.TenantId)
            .Where(a => a.CompanyId == request.CompanyId)
            .Where(a => a.IsActive)
            .Where(a => a.AllowDirectPosting); // Only posting accounts
        
        var accounts = await query.ToListAsync();
        
        foreach (var account in accounts)
        {
            var balance = await GetAccountBalanceAsync(
                account.AccountId,
                fiscalPeriod.FiscalPeriodId,
                request.CostCenterId,
                request.ProjectId);
            
            if (!request.IncludeZeroBalances && 
                balance.ClosingDebit == 0 && 
                balance.ClosingCredit == 0)
            {
                continue;
            }
            
            report.Lines.Add(new TrialBalanceLineDto
            {
                AccountCode = account.AccountCode,
                AccountName = account.AccountName,
                AccountType = account.AccountType.TypeName,
                OpeningDebit = balance.OpeningDebit,
                OpeningCredit = balance.OpeningCredit,
                PeriodDebit = balance.PeriodDebit,
                PeriodCredit = balance.PeriodCredit,
                ClosingDebit = balance.ClosingDebit,
                ClosingCredit = balance.ClosingCredit
            });
        }
        
        // Calculate totals
        report.TotalOpeningDebit = report.Lines.Sum(l => l.OpeningDebit);
        report.TotalOpeningCredit = report.Lines.Sum(l => l.OpeningCredit);
        report.TotalPeriodDebit = report.Lines.Sum(l => l.PeriodDebit);
        report.TotalPeriodCredit = report.Lines.Sum(l => l.PeriodCredit);
        report.TotalClosingDebit = report.Lines.Sum(l => l.ClosingDebit);
        report.TotalClosingCredit = report.Lines.Sum(l => l.ClosingCredit);
        
        // Verify balance
        report.IsBalanced = Math.Abs(report.TotalClosingDebit - report.TotalClosingCredit) < 0.01m;
        
        return Result<TrialBalanceReport>.Success(report);
    }
}
```

تابع في ملف Sales Module التفصيلي...
