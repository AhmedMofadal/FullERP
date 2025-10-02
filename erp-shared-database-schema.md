# Core Shared Database Schema & Master Data

## 1. Master Data Tables - جداول البيانات الرئيسية المشتركة

### 1.1 Currencies & Exchange Rates

```sql
-- ============================================
-- Currencies
-- العملات
-- ============================================
CREATE TABLE [Master].[Currencies] (
    [CurrencyId] SMALLINT IDENTITY(1,1) NOT NULL,
    [CurrencyCode] CHAR(3) NOT NULL, -- ISO 4217
    [CurrencyName] NVARCHAR(100) NOT NULL,
    [CurrencyName_AR] NVARCHAR(100) NULL,
    [CurrencySymbol] NVARCHAR(10) NOT NULL,
    [MinorUnit] TINYINT NOT NULL DEFAULT 2, -- Decimal places
    [IsActive] BIT NOT NULL DEFAULT 1,
    [SortOrder] INT NOT NULL DEFAULT 0,
    CONSTRAINT [PK_Currencies] PRIMARY KEY ([CurrencyId]),
    CONSTRAINT [UK_Currencies_Code] UNIQUE ([CurrencyCode])
);

-- Insert common currencies
INSERT INTO [Master].[Currencies] ([CurrencyCode], [CurrencyName], [CurrencyName_AR], [CurrencySymbol], [MinorUnit])
VALUES 
    ('USD', 'US Dollar', 'دولار أمريكي', '$', 2),
    ('EUR', 'Euro', 'يورو', '€', 2),
    ('GBP', 'British Pound', 'جنيه إسترليني', '£', 2),
    ('SAR', 'Saudi Riyal', 'ريال سعودي', 'ر.س', 2),
    ('AED', 'UAE Dirham', 'درهم إماراتي', 'د.إ', 2),
    ('EGP', 'Egyptian Pound', 'جنيه مصري', 'ج.م', 2),
    ('JPY', 'Japanese Yen', 'ين ياباني', '¥', 0),
    ('CNY', 'Chinese Yuan', 'يوان صيني', '¥', 2),
    ('INR', 'Indian Rupee', 'روبية هندية', '₹', 2),
    ('RUB', 'Russian Ruble', 'روبل روسي', '₽', 2);

-- ============================================
-- Exchange Rates
-- أسعار الصرف
-- ============================================
CREATE TABLE [Master].[ExchangeRates] (
    [ExchangeRateId] BIGINT IDENTITY(1,1) NOT NULL,
    [TenantId] UNIQUEIDENTIFIER NOT NULL,
    [FromCurrencyCode] CHAR(3) NOT NULL,
    [ToCurrencyCode] CHAR(3) NOT NULL,
    [RateDate] DATE NOT NULL,
    [Rate] DECIMAL(18,10) NOT NULL,
    [RateType] NVARCHAR(20) NOT NULL DEFAULT 'Official', -- Official, Market, Custom
    [RateSource] NVARCHAR(100) NULL,
    [IsActive] BIT NOT NULL DEFAULT 1,
    [CreatedBy] UNIQUEIDENTIFIER NOT NULL,
    [CreatedDateUtc] DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
    [LastModifiedBy] UNIQUEIDENTIFIER NULL,
    [LastModifiedDateUtc] DATETIME2 NULL,
    CONSTRAINT [PK_ExchangeRates] PRIMARY KEY ([ExchangeRateId]),
    CONSTRAINT [UK_ExchangeRates] UNIQUE ([TenantId], [FromCurrencyCode], [ToCurrencyCode], [RateDate], [RateType])
);

CREATE NONCLUSTERED INDEX [IX_ExchangeRates_Lookup]
    ON [Master].[ExchangeRates]([TenantId], [FromCurrencyCode], [ToCurrencyCode], [RateDate] DESC);
```

### 1.2 Countries, States & Cities

```sql
-- ============================================
-- Countries
-- الدول
-- ============================================
CREATE TABLE [Master].[Countries] (
    [CountryId] SMALLINT IDENTITY(1,1) NOT NULL,
    [CountryCode] CHAR(2) NOT NULL, -- ISO 3166-1 alpha-2
    [CountryCode3] CHAR(3) NOT NULL, -- ISO 3166-1 alpha-3
    [CountryName] NVARCHAR(100) NOT NULL,
    [CountryName_AR] NVARCHAR(100) NULL,
    [PhoneCode] NVARCHAR(10) NULL,
    [CurrencyCode] CHAR(3) NULL,
    [TimeZones] NVARCHAR(500) NULL, -- JSON array
    [IsActive] BIT NOT NULL DEFAULT 1,
    [SortOrder] INT NOT NULL DEFAULT 0,
    CONSTRAINT [PK_Countries] PRIMARY KEY ([CountryId]),
    CONSTRAINT [UK_Countries_Code] UNIQUE ([CountryCode])
);

-- ============================================
-- States/Provinces
-- الولايات/المحافظات
-- ============================================
CREATE TABLE [Master].[States] (
    [StateId] INT IDENTITY(1,1) NOT NULL,
    [CountryId] SMALLINT NOT NULL,
    [StateCode] NVARCHAR(10) NOT NULL,
    [StateName] NVARCHAR(100) NOT NULL,
    [StateName_AR] NVARCHAR(100) NULL,
    [IsActive] BIT NOT NULL DEFAULT 1,
    CONSTRAINT [PK_States] PRIMARY KEY ([StateId]),
    CONSTRAINT [FK_States_Countries] FOREIGN KEY ([CountryId])
        REFERENCES [Master].[Countries]([CountryId]),
    CONSTRAINT [UK_States] UNIQUE ([CountryId], [StateCode])
);

-- ============================================
-- Cities
-- المدن
-- ============================================
CREATE TABLE [Master].[Cities] (
    [CityId] INT IDENTITY(1,1) NOT NULL,
    [StateId] INT NOT NULL,
    [CityName] NVARCHAR(100) NOT NULL,
    [CityName_AR] NVARCHAR(100) NULL,
    [PostalCode] NVARCHAR(20) NULL,
    [Latitude] DECIMAL(10,8) NULL,
    [Longitude] DECIMAL(11,8) NULL,
    [IsActive] BIT NOT NULL DEFAULT 1,
    CONSTRAINT [PK_Cities] PRIMARY KEY ([CityId]),
    CONSTRAINT [FK_Cities_States] FOREIGN KEY ([StateId])
        REFERENCES [Master].[States]([StateId])
);
```

### 1.3 Units of Measure

```sql
-- ============================================
-- Unit Categories
-- فئات الوحدات
-- ============================================
CREATE TABLE [Master].[UnitCategories] (
    [CategoryId] SMALLINT IDENTITY(1,1) NOT NULL,
    [CategoryCode] NVARCHAR(20) NOT NULL,
    [CategoryName] NVARCHAR(100) NOT NULL,
    [CategoryName_AR] NVARCHAR(100) NULL,
    [Description] NVARCHAR(500) NULL,
    CONSTRAINT [PK_UnitCategories] PRIMARY KEY ([CategoryId]),
    CONSTRAINT [UK_UnitCategories_Code] UNIQUE ([CategoryCode])
);

INSERT INTO [Master].[UnitCategories] ([CategoryCode], [CategoryName], [CategoryName_AR])
VALUES 
    ('LENGTH', 'Length', 'الطول'),
    ('WEIGHT', 'Weight', 'الوزن'),
    ('VOLUME', 'Volume', 'الحجم'),
    ('AREA', 'Area', 'المساحة'),
    ('TIME', 'Time', 'الوقت'),
    ('QUANTITY', 'Quantity', 'الكمية'),
    ('TEMPERATURE', 'Temperature', 'درجة الحرارة');

-- ============================================
-- Units of Measure
-- وحدات القياس
-- ============================================
CREATE TABLE [Master].[UnitsOfMeasure] (
    [UnitId] INT IDENTITY(1,1) NOT NULL,
    [CategoryId] SMALLINT NOT NULL,
    [UnitCode] NVARCHAR(20) NOT NULL,
    [UnitName] NVARCHAR(100) NOT NULL,
    [UnitName_AR] NVARCHAR(100) NULL,
    [UnitSymbol] NVARCHAR(10) NOT NULL,
    [IsBaseUnit] BIT NOT NULL DEFAULT 0,
    [ConversionFactor] DECIMAL(18,8) NULL, -- To base unit
    [IsActive] BIT NOT NULL DEFAULT 1,
    CONSTRAINT [PK_UnitsOfMeasure] PRIMARY KEY ([UnitId]),
    CONSTRAINT [FK_Units_Categories] FOREIGN KEY ([CategoryId])
        REFERENCES [Master].[UnitCategories]([CategoryId]),
    CONSTRAINT [UK_Units_Code] UNIQUE ([UnitCode])
);

-- Insert common units
INSERT INTO [Master].[UnitsOfMeasure] ([CategoryId], [UnitCode], [UnitName], [UnitName_AR], [UnitSymbol], [IsBaseUnit], [ConversionFactor])
VALUES 
    -- Length
    (1, 'M', 'Meter', 'متر', 'm', 1, 1),
    (1, 'CM', 'Centimeter', 'سنتيمتر', 'cm', 0, 0.01),
    (1, 'KM', 'Kilometer', 'كيلومتر', 'km', 0, 1000),
    (1, 'IN', 'Inch', 'بوصة', 'in', 0, 0.0254),
    (1, 'FT', 'Foot', 'قدم', 'ft', 0, 0.3048),
    -- Weight
    (2, 'KG', 'Kilogram', 'كيلوجرام', 'kg', 1, 1),
    (2, 'G', 'Gram', 'جرام', 'g', 0, 0.001),
    (2, 'TON', 'Ton', 'طن', 't', 0, 1000),
    (2, 'LB', 'Pound', 'رطل', 'lb', 0, 0.453592),
    -- Volume
    (3, 'L', 'Liter', 'لتر', 'L', 1, 1),
    (3, 'ML', 'Milliliter', 'ميليلتر', 'mL', 0, 0.001),
    (3, 'M3', 'Cubic Meter', 'متر مكعب', 'm³', 0, 1000),
    -- Quantity
    (6, 'PCS', 'Pieces', 'قطعة', 'pcs', 1, 1),
    (6, 'BOX', 'Box', 'صندوق', 'box', 0, NULL),
    (6, 'CTN', 'Carton', 'كرتون', 'ctn', 0, NULL),
    (6, 'PAL', 'Pallet', 'طبلية', 'pal', 0, NULL);

-- ============================================
-- Unit Conversions
-- تحويلات الوحدات
-- ============================================
CREATE TABLE [Master].[UnitConversions] (
    [ConversionId] INT IDENTITY(1,1) NOT NULL,
    [FromUnitId] INT NOT NULL,
    [ToUnitId] INT NOT NULL,
    [ConversionFactor] DECIMAL(18,8) NOT NULL,
    [IsReversible] BIT NOT NULL DEFAULT 1,
    CONSTRAINT [PK_UnitConversions] PRIMARY KEY ([ConversionId]),
    CONSTRAINT [FK_Conversions_FromUnit] FOREIGN KEY ([FromUnitId])
        REFERENCES [Master].[UnitsOfMeasure]([UnitId]),
    CONSTRAINT [FK_Conversions_ToUnit] FOREIGN KEY ([ToUnitId])
        REFERENCES [Master].[UnitsOfMeasure]([UnitId]),
    CONSTRAINT [UK_UnitConversions] UNIQUE ([FromUnitId], [ToUnitId])
);
```

### 1.4 Tax Types & Tax Groups

```sql
-- ============================================
-- Tax Types
-- أنواع الضرائب
-- ============================================
CREATE TABLE [Master].[TaxTypes] (
    [TaxTypeId] INT IDENTITY(1,1) NOT NULL,
    [TenantId] UNIQUEIDENTIFIER NULL, -- NULL for system-wide
    [CompanyId] UNIQUEIDENTIFIER NULL,
    [TaxCode] NVARCHAR(20) NOT NULL,
    [TaxName] NVARCHAR(100) NOT NULL,
    [TaxName_AR] NVARCHAR(100) NULL,
    [TaxCategory] NVARCHAR(50) NOT NULL, -- VAT, Sales Tax, Withholding, Excise
    [DefaultRate] DECIMAL(5,2) NOT NULL,
    [CalculationMethod] NVARCHAR(20) NOT NULL DEFAULT 'Percentage', -- Percentage, Fixed, Compound
    [IsInclusive] BIT NOT NULL DEFAULT 0,
    [IsRecoverable] BIT NOT NULL DEFAULT 1,
    [TaxAccountId] BIGINT NULL,
    [IsActive] BIT NOT NULL DEFAULT 1,
    [EffectiveFromDate] DATE NOT NULL,
    [EffectiveToDate] DATE NULL,
    [CreatedBy] UNIQUEIDENTIFIER NOT NULL,
    [CreatedDateUtc] DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
    CONSTRAINT [PK_TaxTypes] PRIMARY KEY ([TaxTypeId]),
    CONSTRAINT [UK_TaxTypes] UNIQUE ([TenantId], [CompanyId], [TaxCode])
);

-- ============================================
-- Tax Groups
-- مجموعات الضرائب
-- ============================================
CREATE TABLE [Master].[TaxGroups] (
    [TaxGroupId] INT IDENTITY(1,1) NOT NULL,
    [TenantId] UNIQUEIDENTIFIER NOT NULL,
    [CompanyId] UNIQUEIDENTIFIER NOT NULL,
    [TaxGroupCode] NVARCHAR(20) NOT NULL,
    [TaxGroupName] NVARCHAR(100) NOT NULL,
    [TaxGroupName_AR] NVARCHAR(100) NULL,
    [Description] NVARCHAR(500) NULL,
    [IsActive] BIT NOT NULL DEFAULT 1,
    CONSTRAINT [PK_TaxGroups] PRIMARY KEY ([TaxGroupId]),
    CONSTRAINT [UK_TaxGroups] UNIQUE ([TenantId], [CompanyId], [TaxGroupCode])
);

-- ============================================
-- Tax Group Details
-- تفاصيل مجموعات الضرائب
-- ============================================
CREATE TABLE [Master].[TaxGroupDetails] (
    [DetailId] INT IDENTITY(1,1) NOT NULL,
    [TaxGroupId] INT NOT NULL,
    [TaxTypeId] INT NOT NULL,
    [Sequence] INT NOT NULL,
    [TaxRate] DECIMAL(5,2) NOT NULL,
    [IsCompound] BIT NOT NULL DEFAULT 0,
    CONSTRAINT [PK_TaxGroupDetails] PRIMARY KEY ([DetailId]),
    CONSTRAINT [FK_TaxGroupDetails_Group] FOREIGN KEY ([TaxGroupId])
        REFERENCES [Master].[TaxGroups]([TaxGroupId]) ON DELETE CASCADE,
    CONSTRAINT [FK_TaxGroupDetails_TaxType] FOREIGN KEY ([TaxTypeId])
        REFERENCES [Master].[TaxTypes]([TaxTypeId]),
    CONSTRAINT [UK_TaxGroupDetails] UNIQUE ([TaxGroupId], [TaxTypeId])
);
```

### 1.5 Payment Terms & Methods

```sql
-- ============================================
-- Payment Terms
-- شروط الدفع
-- ============================================
CREATE TABLE [Master].[PaymentTerms] (
    [PaymentTermId] INT IDENTITY(1,1) NOT NULL,
    [TenantId] UNIQUEIDENTIFIER NULL,
    [CompanyId] UNIQUEIDENTIFIER NULL,
    [TermCode] NVARCHAR(20) NOT NULL,
    [TermName] NVARCHAR(100) NOT NULL,
    [TermName_AR] NVARCHAR(100) NULL,
    [TermType] NVARCHAR(20) NOT NULL, -- Net, Installments, COD
    [DueDays] INT NOT NULL DEFAULT 0,
    [DiscountDays] INT NULL,
    [DiscountPercent] DECIMAL(5,2) NULL,
    [NumberOfInstallments] INT NULL,
    [Description] NVARCHAR(500) NULL,
    [IsActive] BIT NOT NULL DEFAULT 1,
    CONSTRAINT [PK_PaymentTerms] PRIMARY KEY ([PaymentTermId]),
    CONSTRAINT [UK_PaymentTerms] UNIQUE ([TenantId], [CompanyId], [TermCode])
);

INSERT INTO [Master].[PaymentTerms] ([TenantId], [CompanyId], [TermCode], [TermName], [TermName_AR], [TermType], [DueDays])
VALUES 
    (NULL, NULL, 'COD', 'Cash on Delivery', 'الدفع عند التسليم', 'COD', 0),
    (NULL, NULL, 'NET30', 'Net 30 Days', 'خلال 30 يوم', 'Net', 30),
    (NULL, NULL, 'NET60', 'Net 60 Days', 'خلال 60 يوم', 'Net', 60),
    (NULL, NULL, 'NET90', 'Net 90 Days', 'خلال 90 يوم', 'Net', 90),
    (NULL, NULL, '2/10NET30', '2% 10 Days, Net 30', 'خصم 2% خلال 10 أيام، الباقي خلال 30', 'Net', 30);

-- ============================================
-- Payment Methods
-- طرق الدفع
-- ============================================
CREATE TABLE [Master].[PaymentMethods] (
    [PaymentMethodId] INT IDENTITY(1,1) NOT NULL,
    [TenantId] UNIQUEIDENTIFIER NULL,
    [CompanyId] UNIQUEIDENTIFIER NULL,
    [MethodCode] NVARCHAR(20) NOT NULL,
    [MethodName] NVARCHAR(100) NOT NULL,
    [MethodName_AR] NVARCHAR(100) NULL,
    [MethodType] NVARCHAR(20) NOT NULL, -- Cash, Card, Bank, Cheque, Wire
    [RequireReference] BIT NOT NULL DEFAULT 0,
    [RequireApproval] BIT NOT NULL DEFAULT 0,
    [DefaultCashAccountId] BIGINT NULL,
    [DefaultBankAccountId] BIGINT NULL,
    [IconClass] NVARCHAR(50) NULL,
    [IsActive] BIT NOT NULL DEFAULT 1,
    CONSTRAINT [PK_PaymentMethods] PRIMARY KEY ([PaymentMethodId]),
    CONSTRAINT [UK_PaymentMethods] UNIQUE ([TenantId], [CompanyId], [MethodCode])
);

INSERT INTO [Master].[PaymentMethods] ([TenantId], [CompanyId], [MethodCode], [MethodName], [MethodName_AR], [MethodType])
VALUES 
    (NULL, NULL, 'CASH', 'Cash', 'نقدي', 'Cash'),
    (NULL, NULL, 'CARD', 'Credit/Debit Card', 'بطاقة ائتمان', 'Card'),
    (NULL, NULL, 'BANK', 'Bank Transfer', 'تحويل بنكي', 'Bank'),
    (NULL, NULL, 'CHEQUE', 'Cheque', 'شيك', 'Cheque'),
    (NULL, NULL, 'WIRE', 'Wire Transfer', 'تحويل سلكي', 'Wire');
```

### 1.6 Number Sequences

```sql
-- ============================================
-- Number Sequence Definitions
-- تعريفات تسلسل الأرقام
-- ============================================
CREATE TABLE [System].[NumberSequences] (
    [SequenceId] INT IDENTITY(1,1) NOT NULL,
    [TenantId] UNIQUEIDENTIFIER NOT NULL,
    [CompanyId] UNIQUEIDENTIFIER NULL,
    [SequenceCode] NVARCHAR(50) NOT NULL,
    [SequenceName] NVARCHAR(200) NOT NULL,
    [Module] NVARCHAR(50) NOT NULL,
    [DocumentType] NVARCHAR(50) NOT NULL,
    [Prefix] NVARCHAR(20) NULL,
    [Suffix] NVARCHAR(20) NULL,
    [NextNumber] BIGINT NOT NULL DEFAULT 1,
    [MinimumDigits] TINYINT NOT NULL DEFAULT 6,
    [IncrementBy] INT NOT NULL DEFAULT 1,
    [ResetFrequency] NVARCHAR(20) NOT NULL DEFAULT 'Never', -- Never, Yearly, Monthly, Daily
    [LastResetDate] DATE NULL,
    [AllowGaps] BIT NOT NULL DEFAULT 1,
    [IsActive] BIT NOT NULL DEFAULT 1,
    [CreatedBy] UNIQUEIDENTIFIER NOT NULL,
    [CreatedDateUtc] DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
    [LastModifiedBy] UNIQUEIDENTIFIER NULL,
    [LastModifiedDateUtc] DATETIME2 NULL,
    [RowVersion] ROWVERSION NOT NULL,
    CONSTRAINT [PK_NumberSequences] PRIMARY KEY ([SequenceId]),
    CONSTRAINT [UK_NumberSequences] UNIQUE ([TenantId], [CompanyId], [SequenceCode])
);

-- ============================================
-- Number Sequence Allocations (for tracking used numbers)
-- تخصيصات تسلسل الأرقام (لتتبع الأرقام المستخدمة)
-- ============================================
CREATE TABLE [System].[NumberSequenceAllocations] (
    [AllocationId] BIGINT IDENTITY(1,1) NOT NULL,
    [SequenceId] INT NOT NULL,
    [AllocatedNumber] BIGINT NOT NULL,
    [AllocatedCode] NVARCHAR(50) NOT NULL,
    [EntityType] NVARCHAR(100) NOT NULL,
    [EntityId] BIGINT NOT NULL,
    [AllocatedDate] DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
    [AllocatedBy] UNIQUEIDENTIFIER NOT NULL,
    CONSTRAINT [PK_NumberSequenceAllocations] PRIMARY KEY ([AllocationId]),
    CONSTRAINT [FK_Allocations_Sequences] FOREIGN KEY ([SequenceId])
        REFERENCES [System].[NumberSequences]([SequenceId]),
    CONSTRAINT [UK_Allocations] UNIQUE ([SequenceId], [AllocatedNumber])
);

CREATE NONCLUSTERED INDEX [IX_Allocations_Entity]
    ON [System].[NumberSequenceAllocations]([EntityType], [EntityId]);

-- ============================================
-- Stored Procedure to Get Next Number
-- إجراء مخزن للحصول على الرقم التالي
-- ============================================
CREATE PROCEDURE [System].[sp_GetNextNumber]
    @TenantId UNIQUEIDENTIFIER,
    @CompanyId UNIQUEIDENTIFIER,
    @SequenceCode NVARCHAR(50),
    @NextCode NVARCHAR(50) OUTPUT
AS
BEGIN
    SET NOCOUNT ON;
    
    DECLARE @SequenceId INT;
    DECLARE @Prefix NVARCHAR(20);
    DECLARE @Suffix NVARCHAR(20);
    DECLARE @NextNumber BIGINT;
    DECLARE @MinimumDigits TINYINT;
    DECLARE @IncrementBy INT;
    DECLARE @ResetFrequency NVARCHAR(20);
    DECLARE @LastResetDate DATE;
    DECLARE @CurrentDate DATE = CAST(GETDATE() AS DATE);
    DECLARE @ShouldReset BIT = 0;
    
    -- Lock the sequence row
    SELECT 
        @SequenceId = SequenceId,
        @Prefix = Prefix,
        @Suffix = Suffix,
        @NextNumber = NextNumber,
        @MinimumDigits = MinimumDigits,
        @IncrementBy = IncrementBy,
        @ResetFrequency = ResetFrequency,
        @LastResetDate = LastResetDate
    FROM [System].[NumberSequences] WITH (UPDLOCK, HOLDLOCK)
    WHERE TenantId = @TenantId
      AND (CompanyId = @CompanyId OR CompanyId IS NULL)
      AND SequenceCode = @SequenceCode
      AND IsActive = 1;
    
    IF @SequenceId IS NULL
    BEGIN
        RAISERROR('Number sequence not found', 16, 1);
        RETURN;
    END
    
    -- Check if reset is needed
    IF @ResetFrequency = 'Yearly' AND YEAR(@LastResetDate) < YEAR(@CurrentDate)
        SET @ShouldReset = 1;
    ELSE IF @ResetFrequency = 'Monthly' AND 
            (YEAR(@LastResetDate) < YEAR(@CurrentDate) OR 
             (YEAR(@LastResetDate) = YEAR(@CurrentDate) AND MONTH(@LastResetDate) < MONTH(@CurrentDate)))
        SET @ShouldReset = 1;
    ELSE IF @ResetFrequency = 'Daily' AND @LastResetDate < @CurrentDate
        SET @ShouldReset = 1;
    
    -- Reset if needed
    IF @ShouldReset = 1
    BEGIN
        SET @NextNumber = 1;
        UPDATE [System].[NumberSequences]
        SET LastResetDate = @CurrentDate
        WHERE SequenceId = @SequenceId;
    END
    
    -- Format the number with padding
    DECLARE @NumberPart NVARCHAR(20) = RIGHT('000000000000' + CAST(@NextNumber AS NVARCHAR), @MinimumDigits);
    
    -- Build the full code
    SET @NextCode = ISNULL(@Prefix, '') + @NumberPart + ISNULL(@Suffix, '');
    
    -- Update next number
    UPDATE [System].[NumberSequences]
    SET NextNumber = @NextNumber + @IncrementBy,
        LastModifiedDateUtc = SYSUTCDATETIME()
    WHERE SequenceId = @SequenceId;
    
    RETURN 0;
END;
GO
```

### 1.7 Attachments & Documents

```sql
-- ============================================
-- Document Attachments
-- مرفقات المستندات
-- ============================================
CREATE TABLE [System].[Attachments] (
    [AttachmentId] BIGINT IDENTITY(1,1) NOT NULL,
    [TenantId] UNIQUEIDENTIFIER NOT NULL,
    [EntityType] NVARCHAR(100) NOT NULL,
    [EntityId] BIGINT NOT NULL,
    [FileName] NVARCHAR(255) NOT NULL,
    [FileExtension] NVARCHAR(10) NOT NULL,
    [FileSize] BIGINT NOT NULL,
    [MimeType] NVARCHAR(100) NOT NULL,
    [StoragePath] NVARCHAR(500) NOT NULL,
    [StorageProvider] NVARCHAR(50) NOT NULL DEFAULT 'Local', -- Local, Azure, AWS
    [FileHash] NVARCHAR(128) NULL,
    [Description] NVARCHAR(500) NULL,
    [Category] NVARCHAR(50) NULL,
    [IsPublic] BIT NOT NULL DEFAULT 0,
    [UploadedBy] UNIQUEIDENTIFIER NOT NULL,
    [UploadedDateUtc] DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
    [IsDeleted] BIT NOT NULL DEFAULT 0,
    [DeletedBy] UNIQUEIDENTIFIER NULL,
    [DeletedDateUtc] DATETIME2 NULL,
    CONSTRAINT [PK_Attachments] PRIMARY KEY ([AttachmentId])
);

CREATE NONCLUSTERED INDEX [IX_Attachments_Entity]
    ON [System].[Attachments]([TenantId], [EntityType], [EntityId])
    INCLUDE ([FileName], [FileSize], [UploadedDateUtc])
    WHERE [IsDeleted] = 0;
```

### 1.8 Approval Workflows

```sql
-- ============================================
-- Approval Workflow Definitions
-- تعريفات سير عمل الموافقات
-- ============================================
CREATE TABLE [System].[ApprovalWorkflows] (
    [WorkflowId] INT IDENTITY(1,1) NOT NULL,
    [TenantId] UNIQUEIDENTIFIER NOT NULL,
    [CompanyId] UNIQUEIDENTIFIER NULL,
    [WorkflowCode] NVARCHAR(50) NOT NULL,
    [WorkflowName] NVARCHAR(200) NOT NULL,
    [Module] NVARCHAR(50) NOT NULL,
    [DocumentType] NVARCHAR(50) NOT NULL,
    [IsActive] BIT NOT NULL DEFAULT 1,
    [CreatedBy] UNIQUEIDENTIFIER NOT NULL,
    [CreatedDateUtc] DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
    CONSTRAINT [PK_ApprovalWorkflows] PRIMARY KEY ([WorkflowId]),
    CONSTRAINT [UK_ApprovalWorkflows] UNIQUE ([TenantId], [CompanyId], [WorkflowCode])
);

-- ============================================
-- Approval Workflow Steps
-- خطوات سير عمل الموافقات
-- ============================================
CREATE TABLE [System].[ApprovalWorkflowSteps] (
    [StepId] INT IDENTITY(1,1) NOT NULL,
    [WorkflowId] INT NOT NULL,
    [StepNumber] INT NOT NULL,
    [StepName] NVARCHAR(200) NOT NULL,
    [ApproverType] NVARCHAR(20) NOT NULL, -- User, Role, Manager, Custom
    [ApproverUserId] UNIQUEIDENTIFIER NULL,
    [ApproverRoleId] UNIQUEIDENTIFIER NULL,
    [ApprovalCondition] NVARCHAR(1000) NULL, -- Expression
    [IsRequired] BIT NOT NULL DEFAULT 1,
    [AllowDelegate] BIT NOT NULL DEFAULT 1,
    [NotifyOnPending] BIT NOT NULL DEFAULT 1,
    [ReminderAfterHours] INT NULL,
    CONSTRAINT [PK_ApprovalWorkflowSteps] PRIMARY KEY ([StepId]),
    CONSTRAINT [FK_WorkflowSteps_Workflow] FOREIGN KEY ([WorkflowId])
        REFERENCES [System].[ApprovalWorkflows]([WorkflowId]) ON DELETE CASCADE
);

-- ============================================
-- Approval Instances
-- حالات الموافقات
-- ============================================
CREATE TABLE [System].[ApprovalInstances] (
    [InstanceId] BIGINT IDENTITY(1,1) NOT NULL,
    [WorkflowId] INT NOT NULL,
    [EntityType] NVARCHAR(100) NOT NULL,
    [EntityId] BIGINT NOT NULL,
    [CurrentStepNumber] INT NOT NULL,
    [Status] NVARCHAR(20) NOT NULL, -- Pending, Approved, Rejected, Cancelled
    [InitiatedBy] UNIQUEIDENTIFIER NOT NULL,
    [InitiatedDateUtc] DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
    [CompletedDateUtc] DATETIME2 NULL,
    CONSTRAINT [PK_ApprovalInstances] PRIMARY KEY ([InstanceId]),
    CONSTRAINT [FK_Instances_Workflow] FOREIGN KEY ([WorkflowId])
        REFERENCES [System].[ApprovalWorkflows]([WorkflowId])
);

-- ============================================
-- Approval Actions
-- إجراءات الموافقات
-- ============================================
CREATE TABLE [System].[ApprovalActions] (
    [ActionId] BIGINT IDENTITY(1,1) NOT NULL,
    [InstanceId] BIGINT NOT NULL,
    [StepNumber] INT NOT NULL,
    [Action] NVARCHAR(20) NOT NULL, -- Approve, Reject, Delegate, Cancel
    [Comments] NVARCHAR(MAX) NULL,
    [ActedBy] UNIQUEIDENTIFIER NOT NULL,
    [ActedDateUtc] DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
    [DelegatedTo] UNIQUEIDENTIFIER NULL,
    CONSTRAINT [PK_ApprovalActions] PRIMARY KEY ([ActionId]),
    CONSTRAINT [FK_Actions_Instance] FOREIGN KEY ([InstanceId])
        REFERENCES [System].[ApprovalInstances]([InstanceId])
);
```

تابع في الملف الخامس للوحدة المالية التفصيلية...
