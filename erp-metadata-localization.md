# Metadata Framework & Localization Engine

## 1. Metadata Database Schema - مخطط قاعدة بيانات الميتاداتا

### 1.1 Languages & Localization Tables

```sql
-- ============================================
-- Languages Master Table
-- جدول اللغات الرئيسي
-- ============================================
CREATE TABLE [Localization].[Languages] (
    [LanguageId] SMALLINT IDENTITY(1,1) NOT NULL,
    [LanguageCode] NVARCHAR(10) NOT NULL, -- en-US, ar-SA, fr-FR
    [LanguageName] NVARCHAR(100) NOT NULL,
    [LanguageName_Native] NVARCHAR(100) NOT NULL, -- English, العربية, Français
    [IsRTL] BIT NOT NULL DEFAULT 0,
    [DateFormat] NVARCHAR(20) NOT NULL DEFAULT 'yyyy-MM-dd',
    [DateTimeFormat] NVARCHAR(30) NOT NULL DEFAULT 'yyyy-MM-dd HH:mm:ss',
    [TimeFormat] NVARCHAR(20) NOT NULL DEFAULT 'HH:mm:ss',
    [NumberFormat] NVARCHAR(20) NOT NULL DEFAULT '#,##0.00',
    [CurrencyFormat] NVARCHAR(20) NOT NULL DEFAULT '#,##0.00',
    [DecimalSeparator] CHAR(1) NOT NULL DEFAULT '.',
    [ThousandsSeparator] CHAR(1) NOT NULL DEFAULT ',',
    [FirstDayOfWeek] TINYINT NOT NULL DEFAULT 0, -- 0=Sunday, 1=Monday
    [IsActive] BIT NOT NULL DEFAULT 1,
    [SortOrder] INT NOT NULL DEFAULT 0,
    CONSTRAINT [PK_Languages] PRIMARY KEY ([LanguageId]),
    CONSTRAINT [UK_Languages_Code] UNIQUE ([LanguageCode])
);

-- Insert default languages
INSERT INTO [Localization].[Languages] 
    ([LanguageCode], [LanguageName], [LanguageName_Native], [IsRTL], 
     [DateFormat], [NumberFormat], [FirstDayOfWeek])
VALUES 
    ('en-US', 'English (United States)', 'English', 0, 'MM/dd/yyyy', '#,##0.00', 0),
    ('ar-SA', 'Arabic (Saudi Arabia)', 'العربية (السعودية)', 1, 'dd/MM/yyyy', '#,##0.00', 6),
    ('ar-EG', 'Arabic (Egypt)', 'العربية (مصر)', 1, 'dd/MM/yyyy', '#,##0.00', 6),
    ('ar-AE', 'Arabic (UAE)', 'العربية (الإمارات)', 1, 'dd/MM/yyyy', '#,##0.00', 6),
    ('es-ES', 'Spanish (Spain)', 'Español', 0, 'dd/MM/yyyy', '#.##0,00', 1),
    ('fr-FR', 'French (France)', 'Français', 0, 'dd/MM/yyyy', '# ##0,00', 1),
    ('de-DE', 'German (Germany)', 'Deutsch', 0, 'dd.MM.yyyy', '#.##0,00', 1),
    ('zh-CN', 'Chinese (Simplified)', '简体中文', 0, 'yyyy-MM-dd', '#,##0.00', 1),
    ('ja-JP', 'Japanese', '日本語', 0, 'yyyy/MM/dd', '#,##0.00', 0),
    ('ru-RU', 'Russian', 'Русский', 0, 'dd.MM.yyyy', '# ##0,00', 1),
    ('pt-BR', 'Portuguese (Brazil)', 'Português', 0, 'dd/MM/yyyy', '#.##0,00', 0),
    ('hi-IN', 'Hindi', 'हिन्दी', 0, 'dd/MM/yyyy', '#,##0.00', 0);

-- ============================================
-- Resource Categories
-- فئات الموارد القابلة للترجمة
-- ============================================
CREATE TABLE [Localization].[ResourceCategories] (
    [CategoryId] SMALLINT IDENTITY(1,1) NOT NULL,
    [CategoryCode] NVARCHAR(50) NOT NULL,
    [CategoryName] NVARCHAR(100) NOT NULL,
    [Description] NVARCHAR(500) NULL,
    [SortOrder] INT NOT NULL DEFAULT 0,
    CONSTRAINT [PK_ResourceCategories] PRIMARY KEY ([CategoryId]),
    CONSTRAINT [UK_ResourceCategories_Code] UNIQUE ([CategoryCode])
);

-- Insert resource categories
INSERT INTO [Localization].[ResourceCategories] 
    ([CategoryCode], [CategoryName], [Description])
VALUES 
    ('UI', 'User Interface', 'UI labels, buttons, menus'),
    ('FieldLabel', 'Field Labels', 'Form field labels'),
    ('ValidationMessage', 'Validation Messages', 'Validation error messages'),
    ('ErrorMessage', 'Error Messages', 'System error messages'),
    ('SuccessMessage', 'Success Messages', 'Success notifications'),
    ('Email', 'Email Templates', 'Email subject and body templates'),
    ('Report', 'Reports', 'Report titles and descriptions'),
    ('Help', 'Help Content', 'Help and documentation content'),
    ('StaticData', 'Static Data', 'Master data translations');

-- ============================================
-- Translation Resources
-- موارد الترجمة
-- ============================================
CREATE TABLE [Localization].[TranslationResources] (
    [ResourceId] BIGINT IDENTITY(1,1) NOT NULL,
    [TenantId] UNIQUEIDENTIFIER NULL, -- NULL for system-wide translations
    [CategoryId] SMALLINT NOT NULL,
    [ResourceKey] NVARCHAR(200) NOT NULL,
    [Module] NVARCHAR(50) NOT NULL,
    [Context] NVARCHAR(100) NULL, -- Additional context information
    [DefaultText] NVARCHAR(MAX) NOT NULL, -- Default English text
    [Description] NVARCHAR(500) NULL,
    [IsSystemResource] BIT NOT NULL DEFAULT 0,
    [CreatedBy] UNIQUEIDENTIFIER NOT NULL,
    [CreatedDateUtc] DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
    [LastModifiedBy] UNIQUEIDENTIFIER NULL,
    [LastModifiedDateUtc] DATETIME2 NULL,
    CONSTRAINT [PK_TranslationResources] PRIMARY KEY ([ResourceId]),
    CONSTRAINT [FK_Resources_Categories] FOREIGN KEY ([CategoryId])
        REFERENCES [Localization].[ResourceCategories]([CategoryId]),
    CONSTRAINT [UK_Resources] UNIQUE ([TenantId], [ResourceKey])
);

-- ============================================
-- Translations
-- الترجمات
-- ============================================
CREATE TABLE [Localization].[Translations] (
    [TranslationId] BIGINT IDENTITY(1,1) NOT NULL,
    [ResourceId] BIGINT NOT NULL,
    [LanguageId] SMALLINT NOT NULL,
    [TranslatedText] NVARCHAR(MAX) NOT NULL,
    [IsApproved] BIT NOT NULL DEFAULT 0,
    [ApprovedBy] UNIQUEIDENTIFIER NULL,
    [ApprovedDateUtc] DATETIME2 NULL,
    [TranslationQuality] TINYINT NULL, -- 1-5 rating
    [TranslationType] NVARCHAR(20) NOT NULL DEFAULT 'Manual', -- Manual, AI, Professional
    [CreatedBy] UNIQUEIDENTIFIER NOT NULL,
    [CreatedDateUtc] DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
    [LastModifiedBy] UNIQUEIDENTIFIER NULL,
    [LastModifiedDateUtc] DATETIME2 NULL,
    CONSTRAINT [PK_Translations] PRIMARY KEY ([TranslationId]),
    CONSTRAINT [FK_Translations_Resources] FOREIGN KEY ([ResourceId])
        REFERENCES [Localization].[TranslationResources]([ResourceId]) ON DELETE CASCADE,
    CONSTRAINT [FK_Translations_Languages] FOREIGN KEY ([LanguageId])
        REFERENCES [Localization].[Languages]([LanguageId]),
    CONSTRAINT [UK_Translations] UNIQUE ([ResourceId], [LanguageId])
);

-- Create indexes for performance
CREATE NONCLUSTERED INDEX [IX_Resources_Tenant_Module]
    ON [Localization].[TranslationResources]([TenantId], [Module])
    INCLUDE ([ResourceKey], [DefaultText]);

CREATE NONCLUSTERED INDEX [IX_Translations_Resource_Language]
    ON [Localization].[Translations]([ResourceId], [LanguageId])
    INCLUDE ([TranslatedText], [IsApproved]);

-- ============================================
-- Entity Translations (for master data)
-- ترجمات الكيانات (للبيانات الرئيسية)
-- ============================================
CREATE TABLE [Localization].[EntityTranslations] (
    [EntityTranslationId] BIGINT IDENTITY(1,1) NOT NULL,
    [TenantId] UNIQUEIDENTIFIER NOT NULL,
    [EntityType] NVARCHAR(100) NOT NULL, -- e.g., 'Product', 'Category', 'Account'
    [EntityId] BIGINT NOT NULL,
    [FieldName] NVARCHAR(50) NOT NULL, -- e.g., 'Name', 'Description'
    [LanguageId] SMALLINT NOT NULL,
    [TranslatedValue] NVARCHAR(MAX) NOT NULL,
    [CreatedBy] UNIQUEIDENTIFIER NOT NULL,
    [CreatedDateUtc] DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
    [LastModifiedBy] UNIQUEIDENTIFIER NULL,
    [LastModifiedDateUtc] DATETIME2 NULL,
    CONSTRAINT [PK_EntityTranslations] PRIMARY KEY ([EntityTranslationId]),
    CONSTRAINT [FK_EntityTranslations_Languages] FOREIGN KEY ([LanguageId])
        REFERENCES [Localization].[Languages]([LanguageId]),
    CONSTRAINT [UK_EntityTranslations] UNIQUE ([TenantId], [EntityType], [EntityId], [FieldName], [LanguageId])
);

CREATE NONCLUSTERED INDEX [IX_EntityTranslations_Lookup]
    ON [Localization].[EntityTranslations]([TenantId], [EntityType], [EntityId], [LanguageId])
    INCLUDE ([FieldName], [TranslatedValue]);
```

### 1.2 Metadata Tables for Dynamic Forms & Fields

```sql
-- ============================================
-- Module Definitions
-- تعريفات الوحدات
-- ============================================
CREATE TABLE [Metadata].[ModuleDefinitions] (
    [ModuleId] SMALLINT IDENTITY(1,1) NOT NULL,
    [ModuleCode] NVARCHAR(50) NOT NULL,
    [ModuleName] NVARCHAR(100) NOT NULL,
    [ModuleName_AR] NVARCHAR(100) NULL,
    [ModuleIcon] NVARCHAR(50) NULL,
    [ModuleColor] NVARCHAR(20) NULL,
    [ParentModuleId] SMALLINT NULL,
    [SortOrder] INT NOT NULL DEFAULT 0,
    [IsActive] BIT NOT NULL DEFAULT 1,
    [RequiresLicense] BIT NOT NULL DEFAULT 0,
    [Description] NVARCHAR(500) NULL,
    CONSTRAINT [PK_ModuleDefinitions] PRIMARY KEY ([ModuleId]),
    CONSTRAINT [FK_ModuleDefinitions_Parent] FOREIGN KEY ([ParentModuleId])
        REFERENCES [Metadata].[ModuleDefinitions]([ModuleId]),
    CONSTRAINT [UK_ModuleDefinitions_Code] UNIQUE ([ModuleCode])
);

-- ============================================
-- Entity Definitions (Tables/Forms)
-- تعريفات الكيانات (الجداول/النماذج)
-- ============================================
CREATE TABLE [Metadata].[EntityDefinitions] (
    [EntityId] INT IDENTITY(1,1) NOT NULL,
    [ModuleId] SMALLINT NOT NULL,
    [EntityCode] NVARCHAR(100) NOT NULL,
    [EntityName] NVARCHAR(200) NOT NULL,
    [EntityName_AR] NVARCHAR(200) NULL,
    [TableName] NVARCHAR(128) NOT NULL,
    [PrimaryKeyField] NVARCHAR(50) NOT NULL,
    [DisplayNameField] NVARCHAR(50) NULL,
    [IconClass] NVARCHAR(50) NULL,
    [AllowCreate] BIT NOT NULL DEFAULT 1,
    [AllowEdit] BIT NOT NULL DEFAULT 1,
    [AllowDelete] BIT NOT NULL DEFAULT 1,
    [AllowExport] BIT NOT NULL DEFAULT 1,
    [AllowImport] BIT NOT NULL DEFAULT 1,
    [RequiresApproval] BIT NOT NULL DEFAULT 0,
    [IsAuditable] BIT NOT NULL DEFAULT 1,
    [IsSoftDeletable] BIT NOT NULL DEFAULT 1,
    [IsLocalizable] BIT NOT NULL DEFAULT 0,
    [Description] NVARCHAR(500) NULL,
    [SortOrder] INT NOT NULL DEFAULT 0,
    CONSTRAINT [PK_EntityDefinitions] PRIMARY KEY ([EntityId]),
    CONSTRAINT [FK_EntityDefinitions_Modules] FOREIGN KEY ([ModuleId])
        REFERENCES [Metadata].[ModuleDefinitions]([ModuleId]),
    CONSTRAINT [UK_EntityDefinitions_Code] UNIQUE ([EntityCode])
);

-- ============================================
-- Field Definitions
-- تعريفات الحقول
-- ============================================
CREATE TABLE [Metadata].[FieldDefinitions] (
    [FieldId] INT IDENTITY(1,1) NOT NULL,
    [EntityId] INT NOT NULL,
    [FieldName] NVARCHAR(100) NOT NULL,
    [FieldLabel] NVARCHAR(200) NOT NULL,
    [FieldLabel_AR] NVARCHAR(200) NULL,
    [DataType] NVARCHAR(50) NOT NULL, -- String, Integer, Decimal, Date, DateTime, Boolean, Lookup, etc.
    [MaxLength] INT NULL,
    [Precision] TINYINT NULL,
    [Scale] TINYINT NULL,
    [IsRequired] BIT NOT NULL DEFAULT 0,
    [IsReadOnly] BIT NOT NULL DEFAULT 0,
    [IsUnique] BIT NOT NULL DEFAULT 0,
    [IsSearchable] BIT NOT NULL DEFAULT 1,
    [IsFilterable] BIT NOT NULL DEFAULT 1,
    [IsSortable] BIT NOT NULL DEFAULT 1,
    [ShowInList] BIT NOT NULL DEFAULT 1,
    [ShowInForm] BIT NOT NULL DEFAULT 1,
    [ShowInFilter] BIT NOT NULL DEFAULT 1,
    [DefaultValue] NVARCHAR(MAX) NULL,
    [ValidationRules] NVARCHAR(MAX) NULL, -- JSON format
    [DisplayFormat] NVARCHAR(100) NULL,
    [PlaceholderText] NVARCHAR(200) NULL,
    [HelpText] NVARCHAR(500) NULL,
    [SortOrder] INT NOT NULL DEFAULT 0,
    [GroupName] NVARCHAR(100) NULL,
    CONSTRAINT [PK_FieldDefinitions] PRIMARY KEY ([FieldId]),
    CONSTRAINT [FK_FieldDefinitions_Entities] FOREIGN KEY ([EntityId])
        REFERENCES [Metadata].[EntityDefinitions]([EntityId]),
    CONSTRAINT [UK_FieldDefinitions] UNIQUE ([EntityId], [FieldName])
);

-- ============================================
-- Lookup Definitions
-- تعريفات القوائم المنسدلة
-- ============================================
CREATE TABLE [Metadata].[LookupDefinitions] (
    [LookupId] INT IDENTITY(1,1) NOT NULL,
    [FieldId] INT NOT NULL,
    [LookupType] NVARCHAR(20) NOT NULL, -- Static, Dynamic, Entity
    [LookupSource] NVARCHAR(200) NULL, -- Entity name or static list name
    [ValueField] NVARCHAR(50) NULL,
    [DisplayField] NVARCHAR(50) NULL,
    [FilterExpression] NVARCHAR(500) NULL,
    [AllowMultiSelect] BIT NOT NULL DEFAULT 0,
    [AllowCustomValue] BIT NOT NULL DEFAULT 0,
    CONSTRAINT [PK_LookupDefinitions] PRIMARY KEY ([LookupId]),
    CONSTRAINT [FK_LookupDefinitions_Fields] FOREIGN KEY ([FieldId])
        REFERENCES [Metadata].[FieldDefinitions]([FieldId])
);

-- ============================================
-- Static Lookup Values
-- قيم القوائم الثابتة
-- ============================================
CREATE TABLE [Metadata].[StaticLookupValues] (
    [ValueId] INT IDENTITY(1,1) NOT NULL,
    [LookupId] INT NOT NULL,
    [ValueCode] NVARCHAR(50) NOT NULL,
    [DisplayText] NVARCHAR(200) NOT NULL,
    [DisplayText_AR] NVARCHAR(200) NULL,
    [SortOrder] INT NOT NULL DEFAULT 0,
    [IsActive] BIT NOT NULL DEFAULT 1,
    CONSTRAINT [PK_StaticLookupValues] PRIMARY KEY ([ValueId]),
    CONSTRAINT [FK_StaticLookupValues_Lookups] FOREIGN KEY ([LookupId])
        REFERENCES [Metadata].[LookupDefinitions]([LookupId]),
    CONSTRAINT [UK_StaticLookupValues] UNIQUE ([LookupId], [ValueCode])
);

-- ============================================
-- Custom Field Definitions (for extensibility)
-- تعريفات الحقول المخصصة (للقابلية للتوسع)
-- ============================================
CREATE TABLE [Metadata].[CustomFieldDefinitions] (
    [CustomFieldId] INT IDENTITY(1,1) NOT NULL,
    [TenantId] UNIQUEIDENTIFIER NOT NULL,
    [EntityId] INT NOT NULL,
    [FieldCode] NVARCHAR(100) NOT NULL,
    [FieldLabel] NVARCHAR(200) NOT NULL,
    [FieldLabel_AR] NVARCHAR(200) NULL,
    [DataType] NVARCHAR(50) NOT NULL,
    [MaxLength] INT NULL,
    [IsRequired] BIT NOT NULL DEFAULT 0,
    [DefaultValue] NVARCHAR(MAX) NULL,
    [ValidationRules] NVARCHAR(MAX) NULL,
    [DisplayFormat] NVARCHAR(100) NULL,
    [SortOrder] INT NOT NULL DEFAULT 0,
    [IsActive] BIT NOT NULL DEFAULT 1,
    [CreatedBy] UNIQUEIDENTIFIER NOT NULL,
    [CreatedDateUtc] DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
    CONSTRAINT [PK_CustomFieldDefinitions] PRIMARY KEY ([CustomFieldId]),
    CONSTRAINT [FK_CustomFieldDefinitions_Entities] FOREIGN KEY ([EntityId])
        REFERENCES [Metadata].[EntityDefinitions]([EntityId]),
    CONSTRAINT [UK_CustomFieldDefinitions] UNIQUE ([TenantId], [EntityId], [FieldCode])
);

-- ============================================
-- Custom Field Values
-- قيم الحقول المخصصة
-- ============================================
CREATE TABLE [Metadata].[CustomFieldValues] (
    [CustomFieldValueId] BIGINT IDENTITY(1,1) NOT NULL,
    [CustomFieldId] INT NOT NULL,
    [EntityRecordId] BIGINT NOT NULL,
    [FieldValue] NVARCHAR(MAX) NULL,
    [CreatedBy] UNIQUEIDENTIFIER NOT NULL,
    [CreatedDateUtc] DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
    [LastModifiedBy] UNIQUEIDENTIFIER NULL,
    [LastModifiedDateUtc] DATETIME2 NULL,
    CONSTRAINT [PK_CustomFieldValues] PRIMARY KEY ([CustomFieldValueId]),
    CONSTRAINT [FK_CustomFieldValues_Definitions] FOREIGN KEY ([CustomFieldId])
        REFERENCES [Metadata].[CustomFieldDefinitions]([CustomFieldId]),
    CONSTRAINT [UK_CustomFieldValues] UNIQUE ([CustomFieldId], [EntityRecordId])
);
```

## 2. Localization Service Implementation

### 2.1 ILocalizationService Interface

```csharp
namespace ERPSaaS.Application.Common.Interfaces
{
    /// <summary>
    /// Service for handling localization and translations
    /// خدمة للتعامل مع الترجمة والتوطين
    /// </summary>
    public interface ILocalizationService
    {
        /// <summary>
        /// Get translated text for a resource key
        /// الحصول على النص المترجم لمفتاح مورد
        /// </summary>
        Task<string> GetStringAsync(
            string resourceKey,
            string languageCode = null,
            params object[] args);
        
        /// <summary>
        /// Get all translations for a module
        /// الحصول على جميع الترجمات لوحدة
        /// </summary>
        Task<Dictionary<string, string>> GetModuleTranslationsAsync(
            string module,
            string languageCode = null);
        
        /// <summary>
        /// Get translation for entity field
        /// الحصول على ترجمة لحقل كيان
        /// </summary>
        Task<string> GetEntityTranslationAsync(
            string entityType,
            long entityId,
            string fieldName,
            string languageCode = null);
        
        /// <summary>
        /// Save or update translation
        /// حفظ أو تحديث ترجمة
        /// </summary>
        Task<bool> SaveTranslationAsync(
            string resourceKey,
            string languageCode,
            string translatedText,
            string module = null,
            string category = null);
        
        /// <summary>
        /// Save entity translation
        /// حفظ ترجمة كيان
        /// </summary>
        Task<bool> SaveEntityTranslationAsync(
            string entityType,
            long entityId,
            string fieldName,
            string languageCode,
            string translatedValue);
        
        /// <summary>
        /// Get current language settings
        /// الحصول على إعدادات اللغة الحالية
        /// </summary>
        Task<LanguageSettings> GetLanguageSettingsAsync(string languageCode);
        
        /// <summary>
        /// Get all active languages
        /// الحصول على جميع اللغات النشطة
        /// </summary>
        Task<List<Language>> GetActiveLanguagesAsync();
        
        /// <summary>
        /// Format number according to locale
        /// تنسيق الرقم حسب اللغة
        /// </summary>
        string FormatNumber(
            decimal number,
            string languageCode = null,
            int decimals = 2);
        
        /// <summary>
        /// Format currency according to locale
        /// تنسيق العملة حسب اللغة
        /// </summary>
        string FormatCurrency(
            decimal amount,
            string currencyCode,
            string languageCode = null);
        
        /// <summary>
        /// Format date according to locale
        /// تنسيق التاريخ حسب اللغة
        /// </summary>
        string FormatDate(
            DateTime date,
            string languageCode = null);
        
        /// <summary>
        /// Format date and time according to locale
        /// تنسيق التاريخ والوقت حسب اللغة
        /// </summary>
        string FormatDateTime(
            DateTime dateTime,
            string languageCode = null);
    }
}
```

### 2.2 LocalizationService Implementation

```csharp
namespace ERPSaaS.Infrastructure.Services
{
    public class LocalizationService : ILocalizationService
    {
        private readonly ApplicationDbContext _context;
        private readonly ICurrentTenant _currentTenant;
        private readonly ICurrentUser _currentUser;
        private readonly IDistributedCache _cache;
        private readonly ILogger<LocalizationService> _logger;
        
        public LocalizationService(
            ApplicationDbContext context,
            ICurrentTenant currentTenant,
            ICurrentUser currentUser,
            IDistributedCache cache,
            ILogger<LocalizationService> logger)
        {
            _context = context;
            _currentTenant = currentTenant;
            _currentUser = currentUser;
            _cache = cache;
            _logger = logger;
        }
        
        public async Task<string> GetStringAsync(
            string resourceKey,
            string languageCode = null,
            params object[] args)
        {
            languageCode ??= _currentUser.Language ?? "en-US";
            
            // Try to get from cache first
            var cacheKey = $"translation:{_currentTenant.TenantId}:{languageCode}:{resourceKey}";
            var cachedValue = await _cache.GetStringAsync(cacheKey);
            
            if (!string.IsNullOrEmpty(cachedValue))
            {
                return FormatString(cachedValue, args);
            }
            
            // Get from database
            var resource = await _context.Set<TranslationResource>()
                .Where(r => r.ResourceKey == resourceKey)
                .Where(r => r.TenantId == null || r.TenantId == _currentTenant.TenantId)
                .FirstOrDefaultAsync();
            
            if (resource == null)
            {
                _logger.LogWarning($"Translation resource not found: {resourceKey}");
                return resourceKey;
            }
            
            var languageId = await GetLanguageIdAsync(languageCode);
            
            var translation = await _context.Set<Translation>()
                .Where(t => t.ResourceId == resource.ResourceId)
                .Where(t => t.LanguageId == languageId)
                .Where(t => t.IsApproved)
                .Select(t => t.TranslatedText)
                .FirstOrDefaultAsync();
            
            var result = translation ?? resource.DefaultText;
            
            // Cache for 1 hour
            await _cache.SetStringAsync(
                cacheKey,
                result,
                new DistributedCacheEntryOptions
                {
                    AbsoluteExpirationRelativeToNow = TimeSpan.FromHours(1)
                });
            
            return FormatString(result, args);
        }
        
        public async Task<Dictionary<string, string>> GetModuleTranslationsAsync(
            string module,
            string languageCode = null)
        {
            languageCode ??= _currentUser.Language ?? "en-US";
            
            var cacheKey = $"translations:{_currentTenant.TenantId}:{module}:{languageCode}";
            var cachedValue = await _cache.GetStringAsync(cacheKey);
            
            if (!string.IsNullOrEmpty(cachedValue))
            {
                return JsonSerializer.Deserialize<Dictionary<string, string>>(cachedValue);
            }
            
            var languageId = await GetLanguageIdAsync(languageCode);
            
            var translations = await _context.Set<TranslationResource>()
                .Where(r => r.Module == module)
                .Where(r => r.TenantId == null || r.TenantId == _currentTenant.TenantId)
                .Select(r => new
                {
                    r.ResourceKey,
                    r.DefaultText,
                    Translation = r.Translations
                        .Where(t => t.LanguageId == languageId && t.IsApproved)
                        .Select(t => t.TranslatedText)
                        .FirstOrDefault()
                })
                .ToDictionaryAsync(
                    x => x.ResourceKey,
                    x => x.Translation ?? x.DefaultText);
            
            // Cache for 1 hour
            await _cache.SetStringAsync(
                cacheKey,
                JsonSerializer.Serialize(translations),
                new DistributedCacheEntryOptions
                {
                    AbsoluteExpirationRelativeToNow = TimeSpan.FromHours(1)
                });
            
            return translations;
        }
        
        public async Task<string> GetEntityTranslationAsync(
            string entityType,
            long entityId,
            string fieldName,
            string languageCode = null)
        {
            languageCode ??= _currentUser.Language ?? "en-US";
            
            var languageId = await GetLanguageIdAsync(languageCode);
            
            var translation = await _context.Set<EntityTranslation>()
                .Where(t => t.TenantId == _currentTenant.TenantId)
                .Where(t => t.EntityType == entityType)
                .Where(t => t.EntityId == entityId)
                .Where(t => t.FieldName == fieldName)
                .Where(t => t.LanguageId == languageId)
                .Select(t => t.TranslatedValue)
                .FirstOrDefaultAsync();
            
            return translation;
        }
        
        public async Task<bool> SaveTranslationAsync(
            string resourceKey,
            string languageCode,
            string translatedText,
            string module = null,
            string category = null)
        {
            try
            {
                var resource = await _context.Set<TranslationResource>()
                    .Where(r => r.ResourceKey == resourceKey)
                    .Where(r => r.TenantId == _currentTenant.TenantId)
                    .FirstOrDefaultAsync();
                
                if (resource == null)
                {
                    // Create new resource
                    var categoryId = await GetCategoryIdAsync(category ?? "UI");
                    
                    resource = new TranslationResource
                    {
                        TenantId = _currentTenant.TenantId,
                        CategoryId = categoryId,
                        ResourceKey = resourceKey,
                        Module = module ?? "Common",
                        DefaultText = translatedText,
                        CreatedBy = _currentUser.UserId.Value
                    };
                    
                    _context.Set<TranslationResource>().Add(resource);
                    await _context.SaveChangesAsync();
                }
                
                var languageId = await GetLanguageIdAsync(languageCode);
                
                var translation = await _context.Set<Translation>()
                    .Where(t => t.ResourceId == resource.ResourceId)
                    .Where(t => t.LanguageId == languageId)
                    .FirstOrDefaultAsync();
                
                if (translation == null)
                {
                    translation = new Translation
                    {
                        ResourceId = resource.ResourceId,
                        LanguageId = languageId,
                        TranslatedText = translatedText,
                        IsApproved = true,
                        TranslationType = "Manual",
                        CreatedBy = _currentUser.UserId.Value
                    };
                    
                    _context.Set<Translation>().Add(translation);
                }
                else
                {
                    translation.TranslatedText = translatedText;
                    translation.LastModifiedBy = _currentUser.UserId;
                    translation.LastModifiedDateUtc = DateTime.UtcNow;
                }
                
                await _context.SaveChangesAsync();
                
                // Invalidate cache
                var cacheKey = $"translation:{_currentTenant.TenantId}:{languageCode}:{resourceKey}";
                await _cache.RemoveAsync(cacheKey);
                
                return true;
            }
            catch (Exception ex)
            {
                _logger.LogError(ex, $"Error saving translation for key: {resourceKey}");
                return false;
            }
        }
        
        public string FormatNumber(
            decimal number,
            string languageCode = null,
            int decimals = 2)
        {
            languageCode ??= _currentUser.Language ?? "en-US";
            var culture = new CultureInfo(languageCode);
            return number.ToString($"N{decimals}", culture);
        }
        
        public string FormatCurrency(
            decimal amount,
            string currencyCode,
            string languageCode = null)
        {
            languageCode ??= _currentUser.Language ?? "en-US";
            var culture = new CultureInfo(languageCode);
            var region = new RegionInfo(culture.Name);
            
            return amount.ToString("C", culture);
        }
        
        public string FormatDate(
            DateTime date,
            string languageCode = null)
        {
            languageCode ??= _currentUser.Language ?? "en-US";
            var culture = new CultureInfo(languageCode);
            return date.ToString("d", culture);
        }
        
        public string FormatDateTime(
            DateTime dateTime,
            string languageCode = null)
        {
            languageCode ??= _currentUser.Language ?? "en-US";
            var culture = new CultureInfo(languageCode);
            return dateTime.ToString("g", culture);
        }
        
        private string FormatString(string format, params object[] args)
        {
            if (args == null || args.Length == 0)
                return format;
            
            try
            {
                return string.Format(format, args);
            }
            catch
            {
                return format;
            }
        }
        
        private async Task<short> GetLanguageIdAsync(string languageCode)
        {
            var cacheKey = $"languageId:{languageCode}";
            var cachedId = await _cache.GetStringAsync(cacheKey);
            
            if (!string.IsNullOrEmpty(cachedId) && short.TryParse(cachedId, out var id))
            {
                return id;
            }
            
            var languageId = await _context.Set<Language>()
                .Where(l => l.LanguageCode == languageCode)
                .Select(l => l.LanguageId)
                .FirstOrDefaultAsync();
            
            if (languageId == 0)
            {
                // Default to English
                languageId = await _context.Set<Language>()
                    .Where(l => l.LanguageCode == "en-US")
                    .Select(l => l.LanguageId)
                    .FirstAsync();
            }
            
            await _cache.SetStringAsync(
                cacheKey,
                languageId.ToString(),
                new DistributedCacheEntryOptions
                {
                    AbsoluteExpirationRelativeToNow = TimeSpan.FromDays(1)
                });
            
            return languageId;
        }
        
        private async Task<short> GetCategoryIdAsync(string categoryCode)
        {
            return await _context.Set<ResourceCategory>()
                .Where(c => c.CategoryCode == categoryCode)
                .Select(c => c.CategoryId)
                .FirstOrDefaultAsync();
        }
    }
}
```

## 3. Frontend Localization Implementation

### 3.1 React i18n Configuration

```typescript
// src/i18n/config.ts
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import Backend from 'i18next-http-backend';
import LanguageDetector from 'i18next-browser-languagedetector';

const RTL_LANGUAGES = ['ar', 'he', 'ur', 'fa'];

i18n
  .use(Backend)
  .use(LanguageDetector)
  .use(initReactI18next)
  .init({
    fallbackLng: 'en',
    debug: process.env.NODE_ENV === 'development',
    
    supportedLngs: [
      'en', 'ar', 'es', 'fr', 'de', 
      'zh', 'ja', 'ru', 'pt', 'hi'
    ],
    
    backend: {
      loadPath: '/api/localization/{{lng}}/{{ns}}',
      requestOptions: {
        mode: 'cors',
        credentials: 'include',
        cache: 'default',
        headers: {
          'X-Tenant-ID': localStorage.getItem('tenantId') || '',
          'Authorization': `Bearer ${localStorage.getItem('token')}` || ''
        }
      }
    },
    
    detection: {
      order: ['localStorage', 'navigator', 'htmlTag'],
      caches: ['localStorage'],
      lookupLocalStorage: 'selectedLanguage'
    },
    
    interpolation: {
      escapeValue: false,
      formatSeparator: ','
    },
    
    react: {
      useSuspense: true,
      bindI18n: 'languageChanged',
      bindI18nStore: '',
      transEmptyNodeValue: '',
      transSupportBasicHtmlNodes: true,
      transKeepBasicHtmlNodesFor: ['br', 'strong', 'i', 'p']
    }
  });

// Set document direction based on language
i18n.on('languageChanged', (lng) => {
  const lang = lng.split('-')[0];
  const isRTL = RTL_LANGUAGES.includes(lang);
  
  document.documentElement.dir = isRTL ? 'rtl' : 'ltr';
  document.documentElement.lang = lng;
  
  // Update HTML class for styling
  document.documentElement.classList.toggle('rtl', isRTL);
});

export default i18n;
```

### 3.2 Localization Hook

```typescript
// src/hooks/useLocalization.ts
import { useTranslation } from 'react-i18next';
import { useCallback, useMemo } from 'react';

interface LocalizationSettings {
  dateFormat: string;
  timeFormat: string;
  numberFormat: string;
  currencyFormat: string;
  decimalSeparator: string;
  thousandsSeparator: string;
  firstDayOfWeek: number;
}

export const useLocalization = () => {
  const { t, i18n } = useTranslation();
  
  const currentLanguage = i18n.language;
  const isRTL = useMemo(() => {
    const lang = currentLanguage.split('-')[0];
    return ['ar', 'he', 'ur', 'fa'].includes(lang);
  }, [currentLanguage]);
  
  const changeLanguage = useCallback(async (lng: string) => {
    await i18n.changeLanguage(lng);
    localStorage.setItem('selectedLanguage', lng);
  }, [i18n]);
  
  const formatNumber = useCallback((
    value: number,
    options?: Intl.NumberFormatOptions
  ): string => {
    return new Intl.NumberFormat(currentLanguage, options).format(value);
  }, [currentLanguage]);
  
  const formatCurrency = useCallback((
    value: number,
    currency: string = 'USD'
  ): string => {
    return new Intl.NumberFormat(currentLanguage, {
      style: 'currency',
      currency: currency
    }).format(value);
  }, [currentLanguage]);
  
  const formatDate = useCallback((
    date: Date | string,
    options?: Intl.DateTimeFormatOptions
  ): string => {
    const dateObj = typeof date === 'string' ? new Date(date) : date;
    return new Intl.DateTimeFormat(currentLanguage, options).format(dateObj);
  }, [currentLanguage]);
  
  const formatDateTime = useCallback((
    date: Date | string
  ): string => {
    return formatDate(date, {
      year: 'numeric',
      month: '2-digit',
      day: '2-digit',
      hour: '2-digit',
      minute: '2-digit'
    });
  }, [formatDate]);
  
  return {
    t,
    currentLanguage,
    isRTL,
    changeLanguage,
    formatNumber,
    formatCurrency,
    formatDate,
    formatDateTime,
    direction: isRTL ? 'rtl' : 'ltr'
  };
};
```

### 3.3 Localizable Form Component

```typescript
// src/components/LocalizableField.tsx
import React, { useState } from 'react';
import { TextField, IconButton, Dialog, DialogTitle, DialogContent, Tabs, Tab } from '@mui/material';
import { Translate as TranslateIcon } from '@mui/icons-material';
import { useLocalization } from '@/hooks/useLocalization';

interface LocalizableFieldProps {
  name: string;
  value: string;
  translations?: Record<string, string>;
  languages?: Array<{ code: string; name: string }>;
  onChange: (value: string, translations?: Record<string, string>) => void;
  label: string;
  required?: boolean;
  multiline?: boolean;
  rows?: number;
}

export const LocalizableField: React.FC<LocalizableFieldProps> = ({
  name,
  value,
  translations = {},
  languages = [],
  onChange,
  label,
  required,
  multiline,
  rows
}) => {
  const { t, currentLanguage } = useLocalization();
  const [dialogOpen, setDialogOpen] = useState(false);
  const [currentTab, setCurrentTab] = useState(0);
  const [localTranslations, setLocalTranslations] = useState(translations);
  
  const handleTranslationChange = (langCode: string, text: string) => {
    const updated = { ...localTranslations, [langCode]: text };
    setLocalTranslations(updated);
  };
  
  const handleSave = () => {
    onChange(value, localTranslations);
    setDialogOpen(false);
  };
  
  return (
    <>
      <TextField
        name={name}
        value={value}
        onChange={(e) => onChange(e.target.value, localTranslations)}
        label={label}
        required={required}
        multiline={multiline}
        rows={rows}
        fullWidth
        InputProps={{
          endAdornment: languages.length > 0 && (
            <IconButton
              onClick={() => setDialogOpen(true)}
              size="small"
              title={t('common.manageTranslations')}
            >
              <TranslateIcon />
            </IconButton>
          )
        }}
      />
      
      <Dialog
        open={dialogOpen}
        onClose={() => setDialogOpen(false)}
        maxWidth="md"
        fullWidth
      >
        <DialogTitle>{t('common.manageTranslations')}</DialogTitle>
        <DialogContent>
          <Tabs
            value={currentTab}
            onChange={(_, newValue) => setCurrentTab(newValue)}
            variant="scrollable"
            scrollButtons="auto"
          >
            {languages.map((lang, index) => (
              <Tab key={lang.code} label={lang.name} />
            ))}
          </Tabs>
          
          {languages.map((lang, index) => (
            <div
              key={lang.code}
              role="tabpanel"
              hidden={currentTab !== index}
              style={{ padding: '20px 0' }}
            >
              {currentTab === index && (
                <TextField
                  value={localTranslations[lang.code] || ''}
                  onChange={(e) => handleTranslationChange(lang.code, e.target.value)}
                  label={`${label} (${lang.name})`}
                  multiline={multiline}
                  rows={rows}
                  fullWidth
                  dir={['ar', 'he', 'ur'].includes(lang.code.split('-')[0]) ? 'rtl' : 'ltr'}
                />
              )}
            </div>
          ))}
          
          <div style={{ marginTop: 20, textAlign: 'right' }}>
            <button onClick={() => setDialogOpen(false)}>
              {t('common.cancel')}
            </button>
            <button onClick={handleSave} style={{ marginLeft: 10 }}>
              {t('common.save')}
            </button>
          </div>
        </DialogContent>
      </Dialog>
    </>
  );
};
```

تابع في الملف الثالث للربط المالي...
