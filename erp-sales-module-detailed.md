# Sales Module - Complete Specification
## وحدة المبيعات - المواصفات الكاملة

## 1. Module Features Overview

```
Sales Module
├── 1. Customer Management
│   ├── Customer Master Data
│   ├── Customer Groups
│   ├── Credit Limits
│   ├── Price Lists per Customer
│   └── Customer Aging
│
├── 2. Sales Quotations
│   ├── Quotation Creation
│   ├── Quotation Approval
│   ├── Convert to Order
│   ├── Quotation Expiry
│   └── Quotation Revisions
│
├── 3. Sales Orders
│   ├── Order Creation
│   ├── Order Approval
│   ├── Order Confirmation
│   ├── Partial Delivery
│   ├── Order Cancellation
│   └── Backorders
│
├── 4. Delivery Notes
│   ├── Delivery Creation from Order
│   ├── Partial Deliveries
│   ├── Delivery Confirmation
│   └── Return Delivery
│
├── 5. Sales Invoices
│   ├── Invoice from Order/Delivery
│   ├── Direct Invoice
│   ├── Advance Invoice
│   ├── Credit Notes
│   ├── Debit Notes
│   └── Invoice Approval
│
├── 6. Payment Management
│   ├── Payment Collection
│   ├── Payment Allocation
│   ├── Partial Payments
│   ├── Payment Methods
│   └── Receipt Vouchers
│
├── 7. Price Management
│   ├── Price Lists
│   ├── Volume Discounts
│   ├── Promotional Prices
│   ├── Customer-Specific Prices
│   └── Price History
│
├── 8. Discounts & Promotions
│   ├── Discount Rules
│   ├── Promotional Campaigns
│   ├── Bundle Offers
│   └── Loyalty Programs
│
├── 9. Point of Sale (POS)
│   ├── POS Interface
│   ├── Quick Sale
│   ├── Split Payments
│   ├── Returns & Refunds
│   └── Cash Management
│
├── 10. Sales Returns
│   ├── Return Authorization
│   ├── Return Processing
│   ├── Return to Stock
│   └── Refund/Credit Note
│
└── 11. Sales Reports
    ├── Sales Summary
    ├── Sales by Customer
    ├── Sales by Product
    ├── Sales by Salesperson
    ├── Sales Aging
    └── Sales Forecast
```

## 2. Complete Database Schema

### 2.1 Customer Master

```sql
-- ============================================
-- Customer Groups
-- مجموعات العملاء
-- ============================================
CREATE TABLE [Sales].[CustomerGroups] (
    [CustomerGroupId] INT IDENTITY(1,1) NOT NULL,
    [TenantId] UNIQUEIDENTIFIER NOT NULL,
    [CompanyId] UNIQUEIDENTIFIER NOT NULL,
    [GroupCode] NVARCHAR(20) NOT NULL,
    [GroupName] NVARCHAR(100) NOT NULL,
    [GroupName_AR] NVARCHAR(100) NULL,
    [DefaultDiscountPercent] DECIMAL(5,2) NULL,
    [DefaultPaymentTermId] INT NULL,
    [DefaultPriceListId] INT NULL,
    [DefaultTaxGroupId] INT NULL,
    [CreditLimitAmount] DECIMAL(19,4) NULL,
    [IsActive] BIT NOT NULL DEFAULT 1,
    [CreatedBy] UNIQUEIDENTIFIER NOT NULL,
    [CreatedDateUtc] DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
    CONSTRAINT [PK_CustomerGroups] PRIMARY KEY ([CustomerGroupId]),
    CONSTRAINT [UK_CustomerGroups] UNIQUE ([TenantId], [CompanyId], [GroupCode])
);

-- ============================================
-- Customers
-- العملاء
-- ============================================
CREATE TABLE [Sales].[Customers] (
    [CustomerId] BIGINT IDENTITY(1,1) NOT NULL,
    [TenantId] UNIQUEIDENTIFIER NOT NULL,
    [CompanyId] UNIQUEIDENTIFIER NOT NULL,
    [CustomerCode] NVARCHAR(20) NOT NULL,
    [CustomerName] NVARCHAR(200) NOT NULL,
    [CustomerName_AR] NVARCHAR(200) NULL,
    [CustomerGroupId] INT NULL,
    [CustomerType] NVARCHAR(20) NOT NULL DEFAULT 'Regular', -- Regular, VIP, Wholesale, Retail
    
    -- Contact Information
    [ContactPerson] NVARCHAR(100) NULL,
    [Phone1] NVARCHAR(20) NULL,
    [Phone2] NVARCHAR(20) NULL,
    [Mobile] NVARCHAR(20) NULL,
    [Email] NVARCHAR(100) NULL,
    [Website] NVARCHAR(200) NULL,
    
    -- Address
    [Address] NVARCHAR(500) NULL,
    [City] NVARCHAR(100) NULL,
    [State] NVARCHAR(100) NULL,
    [CountryCode] CHAR(2) NULL,
    [PostalCode] NVARCHAR(20) NULL,
    
    -- Tax Information
    [TaxRegistrationNumber] NVARCHAR(50) NULL,
    [TaxGroupId] INT NULL,
    [IsTaxExempt] BIT NOT NULL DEFAULT 0,
    
    -- Financial Settings
    [CurrencyCode] CHAR(3) NOT NULL,
    [PaymentTermId] INT NULL,
    [PriceListId] INT NULL,
    [CreditLimit] DECIMAL(19,4) NULL,
    [CurrentBalance] DECIMAL(19,4) NOT NULL DEFAULT 0,
    [CreditStatus] NVARCHAR(20) NOT NULL DEFAULT 'Good', -- Good, Warning, Blocked
    
    -- AR Account Link
    [ARAccountId] BIGINT NULL,
    
    -- Salesperson
    [SalespersonId] BIGINT NULL,
    
    -- Settings
    [AllowDiscount] BIT NOT NULL DEFAULT 1,
    [DefaultDiscountPercent] DECIMAL(5,2) NULL,
    [IsActive] BIT NOT NULL DEFAULT 1,
    [BlockReason] NVARCHAR(500) NULL,
    
    -- Notes
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
    
    CONSTRAINT [PK_Customers] PRIMARY KEY ([CustomerId]),
    CONSTRAINT [FK_Customers_Group] FOREIGN KEY ([CustomerGroupId])
        REFERENCES [Sales].[CustomerGroups]([CustomerGroupId]),
    CONSTRAINT [FK_Customers_ARAccount] FOREIGN KEY ([ARAccountId])
        REFERENCES [Finance].[ChartOfAccounts]([AccountId]),
    CONSTRAINT [UK_Customers_Code] UNIQUE ([TenantId], [CompanyId], [CustomerCode])
);

CREATE NONCLUSTERED INDEX [IX_Customers_Tenant_Active]
    ON [Sales].[Customers]([TenantId], [CompanyId], [IsActive])
    INCLUDE ([CustomerCode], [CustomerName], [CurrentBalance]);

CREATE NONCLUSTERED INDEX [IX_Customers_Name]
    ON [Sales].[Customers]([CustomerName])
    WHERE [IsDeleted] = 0;
```

### 2.2 Sales Quotations

```sql
-- ============================================
-- Sales Quotations Header
-- رأس عروض الأسعار
-- ============================================
CREATE TABLE [Sales].[SalesQuotations] (
    [QuotationId] BIGINT IDENTITY(1,1) NOT NULL,
    [TenantId] UNIQUEIDENTIFIER NOT NULL,
    [CompanyId] UNIQUEIDENTIFIER NOT NULL,
    [BranchId] UNIQUEIDENTIFIER NULL,
    [QuotationNumber] NVARCHAR(50) NOT NULL,
    [QuotationDate] DATE NOT NULL,
    [ExpiryDate] DATE NULL,
    [CustomerId] BIGINT NOT NULL,
    [CustomerName] NVARCHAR(200) NOT NULL, -- Denormalized for performance
    
    -- Reference
    [CustomerReference] NVARCHAR(50) NULL,
    [SalespersonId] BIGINT NULL,
    
    -- Financial
    [CurrencyCode] CHAR(3) NOT NULL,
    [ExchangeRate] DECIMAL(10,6) NOT NULL DEFAULT 1,
    [SubTotal] DECIMAL(19,4) NOT NULL DEFAULT 0,
    [TotalDiscount] DECIMAL(19,4) NOT NULL DEFAULT 0,
    [TotalTax] DECIMAL(19,4) NOT NULL DEFAULT 0,
    [TotalAmount] DECIMAL(19,4) NOT NULL DEFAULT 0,
    
    -- Payment & Delivery
    [PaymentTermId] INT NULL,
    [DeliveryTerms] NVARCHAR(500) NULL,
    [DeliveryDate] DATE NULL,
    
    -- Status & Workflow
    [Status] NVARCHAR(20) NOT NULL DEFAULT 'Draft', -- Draft, Submitted, Approved, Converted, Expired, Cancelled
    [ApprovalStatus] NVARCHAR(20) NULL,
    [ApprovedBy] UNIQUEIDENTIFIER NULL,
    [ApprovedDate] DATETIME2 NULL,
    [ConvertedToOrderId] BIGINT NULL,
    [ConvertedDate] DATETIME2 NULL,
    
    -- Notes
    [Remarks] NVARCHAR(MAX) NULL,
    [Terms] NVARCHAR(MAX) NULL,
    [InternalNotes] NVARCHAR(MAX) NULL,
    
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
    
    CONSTRAINT [PK_SalesQuotations] PRIMARY KEY ([QuotationId]),
    CONSTRAINT [FK_Quotations_Customer] FOREIGN KEY ([CustomerId])
        REFERENCES [Sales].[Customers]([CustomerId]),
    CONSTRAINT [UK_Quotations_Number] UNIQUE ([TenantId], [CompanyId], [QuotationNumber])
);

CREATE NONCLUSTERED INDEX [IX_Quotations_Customer_Date]
    ON [Sales].[SalesQuotations]([CustomerId], [QuotationDate] DESC)
    INCLUDE ([QuotationNumber], [TotalAmount], [Status]);

-- ============================================
-- Sales Quotation Lines
-- سطور عروض الأسعار
-- ============================================
CREATE TABLE [Sales].[SalesQuotationLines] (
    [LineId] BIGINT IDENTITY(1,1) NOT NULL,
    [QuotationId] BIGINT NOT NULL,
    [LineNumber] INT NOT NULL,
    [ItemId] BIGINT NOT NULL,
    [ItemCode] NVARCHAR(50) NOT NULL,
    [ItemName] NVARCHAR(200) NOT NULL,
    [ItemName_AR] NVARCHAR(200) NULL,
    [Description] NVARCHAR(500) NULL,
    
    -- Quantity & UOM
    [Quantity] DECIMAL(18,6) NOT NULL,
    [UnitId] INT NOT NULL,
    [UnitName] NVARCHAR(50) NOT NULL,
    
    -- Pricing
    [UnitPrice] DECIMAL(19,4) NOT NULL,
    [DiscountPercent] DECIMAL(5,2) NOT NULL DEFAULT 0,
    [DiscountAmount] DECIMAL(19,4) NOT NULL DEFAULT 0,
    [NetPrice] AS ([UnitPrice] - [DiscountAmount]) PERSISTED,
    [LineTotal] AS (([UnitPrice] - [DiscountAmount]) * [Quantity]) PERSISTED,
    
    -- Tax
    [TaxGroupId] INT NULL,
    [TaxPercent] DECIMAL(5,2) NOT NULL DEFAULT 0,
    [TaxAmount] DECIMAL(19,4) NOT NULL DEFAULT 0,
    [LineTotalWithTax] AS ((([UnitPrice] - [DiscountAmount]) * [Quantity]) + [TaxAmount]) PERSISTED,
    
    -- Delivery
    [RequestedDeliveryDate] DATE NULL,
    [WarehouseId] BIGINT NULL,
    
    -- Notes
    [Notes] NVARCHAR(500) NULL,
    
    CONSTRAINT [PK_SalesQuotationLines] PRIMARY KEY ([LineId]),
    CONSTRAINT [FK_QuotationLines_Header] FOREIGN KEY ([QuotationId])
        REFERENCES [Sales].[SalesQuotations]([QuotationId]) ON DELETE CASCADE,
    CONSTRAINT [CK_QuotationLines_Quantity] CHECK ([Quantity] > 0)
);

CREATE NONCLUSTERED INDEX [IX_QuotationLines_Item]
    ON [Sales].[SalesQuotationLines]([ItemId], [QuotationId]);
```

### 2.3 Sales Orders

```sql
-- ============================================
-- Sales Orders Header
-- رأس أوامر المبيعات
-- ============================================
CREATE TABLE [Sales].[SalesOrders] (
    [OrderId] BIGINT IDENTITY(1,1) NOT NULL,
    [TenantId] UNIQUEIDENTIFIER NOT NULL,
    [CompanyId] UNIQUEIDENTIFIER NOT NULL,
    [BranchId] UNIQUEIDENTIFIER NULL,
    [OrderNumber] NVARCHAR(50) NOT NULL,
    [OrderDate] DATE NOT NULL,
    [CustomerId] BIGINT NOT NULL,
    [CustomerName] NVARCHAR(200) NOT NULL,
    
    -- Source Reference
    [QuotationId] BIGINT NULL,
    [QuotationNumber] NVARCHAR(50) NULL,
    [CustomerPO] NVARCHAR(50) NULL,
    [SalespersonId] BIGINT NULL,
    
    -- Financial
    [CurrencyCode] CHAR(3) NOT NULL,
    [ExchangeRate] DECIMAL(10,6) NOT NULL DEFAULT 1,
    [SubTotal] DECIMAL(19,4) NOT NULL DEFAULT 0,
    [TotalDiscount] DECIMAL(19,4) NOT NULL DEFAULT 0,
    [TotalTax] DECIMAL(19,4) NOT NULL DEFAULT 0,
    [TotalAmount] DECIMAL(19,4) NOT NULL DEFAULT 0,
    
    -- Payment
    [PaymentTermId] INT NULL,
    [AdvancePaymentAmount] DECIMAL(19,4) NULL,
    [AdvancePaymentDate] DATE NULL,
    
    -- Delivery
    [RequestedDeliveryDate] DATE NULL,
    [PromisedDeliveryDate] DATE NULL,
    [ShippingAddress] NVARCHAR(500) NULL,
    [ShippingMethod] NVARCHAR(100) NULL,
    [ShippingCost] DECIMAL(19,4) NULL,
    
    -- Status & Workflow
    [Status] NVARCHAR(20) NOT NULL DEFAULT 'Draft', -- Draft, Confirmed, InProgress, PartiallyDelivered, Delivered, Invoiced, Cancelled
    [ApprovalStatus] NVARCHAR(20) NULL,
    [ApprovedBy] UNIQUEIDENTIFIER NULL,
    [ApprovedDate] DATETIME2 NULL,
    [ConfirmedBy] UNIQUEIDENTIFIER NULL,
    [ConfirmedDate] DATETIME2 NULL,
    
    -- Fulfillment Tracking
    [TotalQuantityOrdered] DECIMAL(18,6) NOT NULL DEFAULT 0,
    [TotalQuantityDelivered] DECIMAL(18,6) NOT NULL DEFAULT 0,
    [TotalQuantityInvoiced] DECIMAL(18,6) NOT NULL DEFAULT 0,
    [IsFulfilled] AS (CASE WHEN [TotalQuantityOrdered] = [TotalQuantityDelivered] THEN 1 ELSE 0 END) PERSISTED,
    
    -- Notes
    [Remarks] NVARCHAR(MAX) NULL,
    [InternalNotes] NVARCHAR(MAX) NULL,
    
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
    
    CONSTRAINT [PK_SalesOrders] PRIMARY KEY ([OrderId]),
    CONSTRAINT [FK_Orders_Customer] FOREIGN KEY ([CustomerId])
        REFERENCES [Sales].[Customers]([CustomerId]),
    CONSTRAINT [FK_Orders_Quotation] FOREIGN KEY ([QuotationId])
        REFERENCES [Sales].[SalesQuotations]([QuotationId]),
    CONSTRAINT [UK_Orders_Number] UNIQUE ([TenantId], [CompanyId], [OrderNumber])
);

CREATE NONCLUSTERED INDEX [IX_Orders_Customer_Status]
    ON [Sales].[SalesOrders]([CustomerId], [Status], [OrderDate] DESC)
    INCLUDE ([OrderNumber], [TotalAmount]);

CREATE NONCLUSTERED INDEX [IX_Orders_Status_Delivery]
    ON [Sales].[SalesOrders]([Status], [RequestedDeliveryDate])
    WHERE [Status] IN ('Confirmed', 'InProgress', 'PartiallyDelivered');

-- ============================================
-- Sales Order Lines
-- سطور أوامر المبيعات
-- ============================================
CREATE TABLE [Sales].[SalesOrderLines] (
    [LineId] BIGINT IDENTITY(1,1) NOT NULL,
    [OrderId] BIGINT NOT NULL,
    [LineNumber] INT NOT NULL,
    [ItemId] BIGINT NOT NULL,
    [ItemCode] NVARCHAR(50) NOT NULL,
    [ItemName] NVARCHAR(200) NOT NULL,
    [ItemName_AR] NVARCHAR(200) NULL,
    [Description] NVARCHAR(500) NULL,
    
    -- Source Reference
    [QuotationLineId] BIGINT NULL,
    
    -- Quantity & UOM
    [QuantityOrdered] DECIMAL(18,6) NOT NULL,
    [QuantityDelivered] DECIMAL(18,6) NOT NULL DEFAULT 0,
    [QuantityInvoiced] DECIMAL(18,6) NOT NULL DEFAULT 0,
    [QuantityBackordered] AS ([QuantityOrdered] - [QuantityDelivered]) PERSISTED,
    [UnitId] INT NOT NULL,
    [UnitName] NVARCHAR(50) NOT NULL,
    
    -- Pricing
    [UnitPrice] DECIMAL(19,4) NOT NULL,
    [DiscountPercent] DECIMAL(5,2) NOT NULL DEFAULT 0,
    [DiscountAmount] DECIMAL(19,4) NOT NULL DEFAULT 0,
    [NetPrice] AS ([UnitPrice] - [DiscountAmount]) PERSISTED,
    [LineTotal] AS (([UnitPrice] - [DiscountAmount]) * [QuantityOrdered]) PERSISTED,
    
    -- Tax
    [TaxGroupId] INT NULL,
    [TaxPercent] DECIMAL(5,2) NOT NULL DEFAULT 0,
    [TaxAmount] DECIMAL(19,4) NOT NULL DEFAULT 0,
    [LineTotalWithTax] AS ((([UnitPrice] - [DiscountAmount]) * [QuantityOrdered]) + [TaxAmount]) PERSISTED,
    
    -- Delivery
    [RequestedDeliveryDate] DATE NULL,
    [PromisedDeliveryDate] DATE NULL,
    [WarehouseId] BIGINT NULL,
    [ReservedQuantity] DECIMAL(18,6) NOT NULL DEFAULT 0,
    
    -- Status
    [LineStatus] NVARCHAR(20) NOT NULL DEFAULT 'Open', -- Open, PartiallyDelivered, Delivered, Cancelled
    [IsClosed] BIT NOT NULL DEFAULT 0,
    
    -- Notes
    [Notes] NVARCHAR(500) NULL,
    
    CONSTRAINT [PK_SalesOrderLines] PRIMARY KEY ([LineId]),
    CONSTRAINT [FK_OrderLines_Header] FOREIGN KEY ([OrderId])
        REFERENCES [Sales].[SalesOrders]([OrderId]) ON DELETE CASCADE,
    CONSTRAINT [CK_OrderLines_Quantity] CHECK ([QuantityOrdered] > 0),
    CONSTRAINT [CK_OrderLines_Delivered] CHECK ([QuantityDelivered] <= [QuantityOrdered])
);

CREATE NONCLUSTERED INDEX [IX_OrderLines_Item_Status]
    ON [Sales].[SalesOrderLines]([ItemId], [LineStatus])
    INCLUDE ([QuantityOrdered], [QuantityDelivered]);
```

### 2.4 Sales Invoices

```sql
-- ============================================
-- Sales Invoices Header
-- رأس فواتير المبيعات
-- ============================================
CREATE TABLE [Sales].[SalesInvoices] (
    [InvoiceId] BIGINT IDENTITY(1,1) NOT NULL,
    [TenantId] UNIQUEIDENTIFIER NOT NULL,
    [CompanyId] UNIQUEIDENTIFIER NOT NULL,
    [BranchId] UNIQUEIDENTIFIER NULL,
    [InvoiceNumber] NVARCHAR(50) NOT NULL,
    [InvoiceDate] DATE NOT NULL,
    [DueDate] DATE NULL,
    [InvoiceType] NVARCHAR(20) NOT NULL DEFAULT 'Standard', -- Standard, Advance, CreditNote, DebitNote
    [CustomerId] BIGINT NOT NULL,
    [CustomerName] NVARCHAR(200) NOT NULL,
    
    -- Source Reference
    [OrderId] BIGINT NULL,
    [OrderNumber] NVARCHAR(50) NULL,
    [DeliveryNoteId] BIGINT NULL,
    [DeliveryNoteNumber] NVARCHAR(50) NULL,
    [CustomerPO] NVARCHAR(50) NULL,
    [SalespersonId] BIGINT NULL,
    
    -- Financial
    [CurrencyCode] CHAR(3) NOT NULL,
    [ExchangeRate] DECIMAL(10,6) NOT NULL DEFAULT 1,
    [SubTotal] DECIMAL(19,4) NOT NULL DEFAULT 0,
    [TotalDiscount] DECIMAL(19,4) NOT NULL DEFAULT 0,
    [TotalTax] DECIMAL(19,4) NOT NULL DEFAULT 0,
    [ShippingCost] DECIMAL(19,4) NOT NULL DEFAULT 0,
    [TotalAmount] DECIMAL(19,4) NOT NULL DEFAULT 0,
    
    -- Payment Tracking
    [PaidAmount] DECIMAL(19,4) NOT NULL DEFAULT 0,
    [RemainingAmount] AS ([TotalAmount] - [PaidAmount]) PERSISTED,
    [PaymentStatus] NVARCHAR(20) NOT NULL DEFAULT 'Unpaid', -- Unpaid, PartiallyPaid, Paid, Overdue
    [PaymentTermId] INT NULL,
    
    -- Status & Workflow
    [Status] NVARCHAR(20) NOT NULL DEFAULT 'Draft', -- Draft, Posted, Paid, Cancelled
    [ApprovalStatus] NVARCHAR(20) NULL,
    [ApprovedBy] UNIQUEIDENTIFIER NULL,
    [ApprovedDate] DATETIME2 NULL,
    [PostedBy] UNIQUEIDENTIFIER NULL,
    [PostedDate] DATETIME2 NULL,
    
    -- Financial Integration
    [IsPostedToGL] BIT NOT NULL DEFAULT 0,
    [JournalEntryId] BIGINT NULL,
    [PostingError] NVARCHAR(MAX) NULL,
    
    -- Original Invoice (for Credit/Debit Notes)
    [OriginalInvoiceId] BIGINT NULL,
    [OriginalInvoiceNumber] NVARCHAR(50) NULL,
    
    -- Notes
    [Remarks] NVARCHAR(MAX) NULL,
    [InternalNotes] NVARCHAR(MAX) NULL,
    
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
    
    CONSTRAINT [PK_SalesInvoices] PRIMARY KEY ([InvoiceId]),
    CONSTRAINT [FK_Invoices_Customer] FOREIGN KEY ([CustomerId])
        REFERENCES [Sales].[Customers]([CustomerId]),
    CONSTRAINT [FK_Invoices_Order] FOREIGN KEY ([OrderId])
        REFERENCES [Sales].[SalesOrders]([OrderId]),
    CONSTRAINT [FK_Invoices_Original] FOREIGN KEY ([OriginalInvoiceId])
        REFERENCES [Sales].[SalesInvoices]([InvoiceId]),
    CONSTRAINT [UK_Invoices_Number] UNIQUE ([TenantId], [CompanyId], [InvoiceNumber])
);

CREATE NONCLUSTERED INDEX [IX_Invoices_Customer_Date]
    ON [Sales].[SalesInvoices]([CustomerId], [InvoiceDate] DESC)
    INCLUDE ([InvoiceNumber], [TotalAmount], [RemainingAmount], [PaymentStatus]);

CREATE NONCLUSTERED INDEX [IX_Invoices_Payment_Status]
    ON [Sales].[SalesInvoices]([PaymentStatus], [DueDate])
    WHERE [Status] = 'Posted' AND [PaymentStatus] IN ('Unpaid', 'PartiallyPaid');

CREATE NONCLUSTERED INDEX [IX_Invoices_GL_Posting]
    ON [Sales].[SalesInvoices]([TenantId], [CompanyId], [IsPostedToGL])
    WHERE [Status] = 'Posted' AND [IsPostedToGL] = 0;

-- ============================================
-- Sales Invoice Lines
-- سطور فواتير المبيعات
-- ============================================
CREATE TABLE [Sales].[SalesInvoiceLines] (
    [LineId] BIGINT IDENTITY(1,1) NOT NULL,
    [InvoiceId] BIGINT NOT NULL,
    [LineNumber] INT NOT NULL,
    [ItemId] BIGINT NOT NULL,
    [ItemCode] NVARCHAR(50) NOT NULL,
    [ItemName] NVARCHAR(200) NOT NULL,
    [ItemName_AR] NVARCHAR(200) NULL,
    [Description] NVARCHAR(500) NULL,
    
    -- Source Reference
    [OrderLineId] BIGINT NULL,
    [DeliveryLineId] BIGINT NULL,
    
    -- Quantity & UOM
    [Quantity] DECIMAL(18,6) NOT NULL,
    [UnitId] INT NOT NULL,
    [UnitName] NVARCHAR(50) NOT NULL,
    
    -- Pricing
    [UnitPrice] DECIMAL(19,4) NOT NULL,
    [DiscountPercent] DECIMAL(5,2) NOT NULL DEFAULT 0,
    [DiscountAmount] DECIMAL(19,4) NOT NULL DEFAULT 0,
    [NetPrice] AS ([UnitPrice] - [DiscountAmount]) PERSISTED,
    [LineTotal] AS (([UnitPrice] - [DiscountAmount]) * [Quantity]) PERSISTED,
    
    -- Tax
    [TaxGroupId] INT NULL,
    [TaxPercent] DECIMAL(5,2) NOT NULL DEFAULT 0,
    [TaxAmount] DECIMAL(19,4) NOT NULL DEFAULT 0,
    [LineTotalWithTax] AS ((([UnitPrice] - [DiscountAmount]) * [Quantity]) + [TaxAmount]) PERSISTED,
    
    -- Cost & Inventory
    [CostPrice] DECIMAL(19,4) NULL,
    [TotalCost] AS ([CostPrice] * [Quantity]) PERSISTED,
    [GrossProfit] AS ((([UnitPrice] - [DiscountAmount]) - [CostPrice]) * [Quantity]) PERSISTED,
    [WarehouseId] BIGINT NULL,
    
    -- Financial Posting
    [RevenueAccountId] BIGINT NULL,
    [COGSAccountId] BIGINT NULL,
    [InventoryAccountId] BIGINT NULL,
    
    -- Notes
    [Notes] NVARCHAR(500) NULL,
    
    CONSTRAINT [PK_SalesInvoiceLines] PRIMARY KEY ([LineId]),
    CONSTRAINT [FK_InvoiceLines_Header] FOREIGN KEY ([InvoiceId])
        REFERENCES [Sales].[SalesInvoices]([InvoiceId]) ON DELETE CASCADE,
    CONSTRAINT [FK_InvoiceLines_OrderLine] FOREIGN KEY ([OrderLineId])
        REFERENCES [Sales].[SalesOrderLines]([LineId]),
    CONSTRAINT [CK_InvoiceLines_Quantity] CHECK ([Quantity] <> 0) -- Allow negative for credit notes
);

CREATE NONCLUSTERED INDEX [IX_InvoiceLines_Item]
    ON [Sales].[SalesInvoiceLines]([ItemId], [InvoiceId])
    INCLUDE ([Quantity], [UnitPrice], [LineTotal]);
```

### 2.5 Customer Payments

```sql
-- ============================================
-- Customer Payments
-- دفعات العملاء
-- ============================================
CREATE TABLE [Sales].[CustomerPayments] (
    [PaymentId] BIGINT IDENTITY(1,1) NOT NULL,
    [TenantId] UNIQUEIDENTIFIER NOT NULL,
    [CompanyId] UNIQUEIDENTIFIER NOT NULL,
    [BranchId] UNIQUEIDENTIFIER NULL,
    [PaymentNumber] NVARCHAR(50) NOT NULL,
    [PaymentDate] DATE NOT NULL,
    [CustomerId] BIGINT NOT NULL,
    [CustomerName] NVARCHAR(200) NOT NULL,
    
    -- Payment Details
    [PaymentMethodId] INT NOT NULL,
    [PaymentAmount] DECIMAL(19,4) NOT NULL,
    [CurrencyCode] CHAR(3) NOT NULL,
    [ExchangeRate] DECIMAL(10,6) NOT NULL DEFAULT 1,
    [BaseAmount] AS ([PaymentAmount] * [ExchangeRate]) PERSISTED,
    
    -- Reference
    [ReferenceNumber] NVARCHAR(50) NULL, -- Check number, transfer ref, etc.
    [BankAccountId] BIGINT NULL,
    [ChequeDate] DATE NULL,
    [ChequeStatus] NVARCHAR(20) NULL, -- Received, Deposited, Cleared, Bounced
    
    -- Allocation
    [AllocatedAmount] DECIMAL(19,4) NOT NULL DEFAULT 0,
    [UnallocatedAmount] AS ([PaymentAmount] - [AllocatedAmount]) PERSISTED,
    [IsFullyAllocated] AS (CASE WHEN ABS([PaymentAmount] - [AllocatedAmount]) < 0.01 THEN 1 ELSE 0 END) PERSISTED,
    
    -- Status
    [Status] NVARCHAR(20) NOT NULL DEFAULT 'Draft', -- Draft, Posted, Cleared, Cancelled
    [PostedBy] UNIQUEIDENTIFIER NULL,
    [PostedDate] DATETIME2 NULL,
    
    -- Financial Integration
    [IsPostedToGL] BIT NOT NULL DEFAULT 0,
    [JournalEntryId] BIGINT NULL,
    
    -- Cash Account
    [CashAccountId] BIGINT NULL,
    
    -- Notes
    [Remarks] NVARCHAR(MAX) NULL,
    
    -- Audit
    [CreatedBy] UNIQUEIDENTIFIER NOT NULL,
    [CreatedDateUtc] DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
    [CreatedTimezone] NVARCHAR(50) NOT NULL,
    [CreatedDateLocal] DATETIME2 NOT NULL,
    [LastModifiedBy] UNIQUEIDENTIFIER NULL,
    [LastModifiedDateUtc] DATETIME2 NULL,
    [IsDeleted] BIT NOT NULL DEFAULT 0,
    [RowVersion] ROWVERSION NOT NULL,
    
    CONSTRAINT [PK_CustomerPayments] PRIMARY KEY ([PaymentId]),
    CONSTRAINT [FK_Payments_Customer] FOREIGN KEY ([CustomerId])
        REFERENCES [Sales].[Customers]([CustomerId]),
    CONSTRAINT [UK_Payments_Number] UNIQUE ([TenantId], [CompanyId], [PaymentNumber])
);

-- ============================================
-- Payment Allocations
-- توزيع الدفعات
-- ============================================
CREATE TABLE [Sales].[PaymentAllocations] (
    [AllocationId] BIGINT IDENTITY(1,1) NOT NULL,
    [PaymentId] BIGINT NOT NULL,
    [InvoiceId] BIGINT NOT NULL,
    [InvoiceNumber] NVARCHAR(50) NOT NULL,
    [InvoiceAmount] DECIMAL(19,4) NOT NULL,
    [AllocatedAmount] DECIMAL(19,4) NOT NULL,
    [AllocationDate] DATE NOT NULL,
    [Notes] NVARCHAR(500) NULL,
    [CreatedBy] UNIQUEIDENTIFIER NOT NULL,
    [CreatedDateUtc] DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
    
    CONSTRAINT [PK_PaymentAllocations] PRIMARY KEY ([AllocationId]),
    CONSTRAINT [FK_Allocations_Payment] FOREIGN KEY ([PaymentId])
        REFERENCES [Sales].[CustomerPayments]([PaymentId]) ON DELETE CASCADE,
    CONSTRAINT [FK_Allocations_Invoice] FOREIGN KEY ([InvoiceId])
        REFERENCES [Sales].[SalesInvoices]([InvoiceId]),
    CONSTRAINT [CK_Allocations_Amount] CHECK ([AllocatedAmount] > 0)
);

CREATE NONCLUSTERED INDEX [IX_Allocations_Invoice]
    ON [Sales].[PaymentAllocations]([InvoiceId])
    INCLUDE ([AllocatedAmount]);
```

### 2.6 Price Lists

```sql
-- ============================================
-- Price Lists
-- قوائم الأسعار
-- ============================================
CREATE TABLE [Sales].[PriceLists] (
    [PriceListId] INT IDENTITY(1,1) NOT NULL,
    [TenantId] UNIQUEIDENTIFIER NOT NULL,
    [CompanyId] UNIQUEIDENTIFIER NOT NULL,
    [PriceListCode] NVARCHAR(20) NOT NULL,
    [PriceListName] NVARCHAR(100) NOT NULL,
    [PriceListName_AR] NVARCHAR(100) NULL,
    [CurrencyCode] CHAR(3) NOT NULL,
    [EffectiveFromDate] DATE NOT NULL,
    [EffectiveToDate] DATE NULL,
    [IsDefault] BIT NOT NULL DEFAULT 0,
    [IsActive] BIT NOT NULL DEFAULT 1,
    [CreatedBy] UNIQUEIDENTIFIER NOT NULL,
    [CreatedDateUtc] DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME(),
    CONSTRAINT [PK_PriceLists] PRIMARY KEY ([PriceListId]),
    CONSTRAINT [UK_PriceLists] UNIQUE ([TenantId], [CompanyId], [PriceListCode])
);

-- ============================================
-- Price List Items
-- عناصر قائمة الأسعار
-- ============================================
CREATE TABLE [Sales].[PriceListItems] (
    [PriceListItemId] BIGINT IDENTITY(1,1) NOT NULL,
    [PriceListId] INT NOT NULL,
    [ItemId] BIGINT NOT NULL,
    [UnitId] INT NOT NULL,
    [Price] DECIMAL(19,4) NOT NULL,
    [MinimumQuantity] DECIMAL(18,6) NULL,
    [MaximumQuantity] DECIMAL(18,6) NULL,
    [EffectiveFromDate] DATE NOT NULL,
    [EffectiveToDate] DATE NULL,
    [IsActive] BIT NOT NULL DEFAULT 1,
    
    CONSTRAINT [PK_PriceListItems] PRIMARY KEY ([PriceListItemId]),
    CONSTRAINT [FK_PriceItems_List] FOREIGN KEY ([PriceListId])
        REFERENCES [Sales].[PriceLists]([PriceListId]) ON DELETE CASCADE,
    CONSTRAINT [UK_PriceListItems] UNIQUE ([PriceListId], [ItemId], [UnitId], [EffectiveFromDate])
);

CREATE NONCLUSTERED INDEX [IX_PriceListItems_Lookup]
    ON [Sales].[PriceListItems]([PriceListId], [ItemId], [EffectiveFromDate] DESC)
    INCLUDE ([Price], [IsActive]);
```

## 3. Business Logic & Workflows

### 3.1 Sales Order Creation Workflow

```csharp
namespace ERPSaaS.Modules.Sales.Application.Features.Orders
{
    public class CreateSalesOrderCommandHandler : IRequestHandler<CreateSalesOrderCommand, Result<SalesOrderDto>>
    {
        public async Task<Result<SalesOrderDto>> Handle(
            CreateSalesOrderCommand request,
            CancellationToken cancellationToken)
        {
            using var transaction = await _context.Database.BeginTransactionAsync(cancellationToken);
            
            try
            {
                // 1. Validate customer
                var customer = await ValidateCustomerAsync(request.CustomerId);
                if (customer == null)
                    return Result<SalesOrderDto>.Failure("Customer not found");
                
                if (!customer.IsActive)
                    return Result<SalesOrderDto>.Failure("Customer is not active");
                
                // 2. Check credit limit
                if (customer.CreditLimit.HasValue)
                {
                    var creditCheck = await CheckCreditLimitAsync(
                        customer.CustomerId,
                        customer.CreditLimit.Value,
                        request.TotalAmount);
                    
                    if (!creditCheck.IsValid)
                        return Result<SalesOrderDto>.Failure(creditCheck.Message);
                }
                
                // 3. Generate order number
                var orderNumber = await _numberSequenceService.GetNextNumberAsync(
                    _currentTenant.TenantId,
                    request.CompanyId,
                    "SalesOrder");
                
                // 4. Create order header
                var order = new SalesOrder
                {
                    TenantId = _currentTenant.TenantId,
                    CompanyId = request.CompanyId,
                    BranchId = request.BranchId,
                    OrderNumber = orderNumber,
                    OrderDate = request.OrderDate,
                    CustomerId = request.CustomerId,
                    CustomerName = customer.CustomerName,
                    QuotationId = request.QuotationId,
                    CustomerPO = request.CustomerPO,
                    SalespersonId = request.SalespersonId,
                    CurrencyCode = request.CurrencyCode ?? customer.CurrencyCode,
                    ExchangeRate = request.ExchangeRate,
                    PaymentTermId = request.PaymentTermId ?? customer.PaymentTermId,
                    RequestedDeliveryDate = request.RequestedDeliveryDate,
                    ShippingAddress = request.ShippingAddress,
                    Status = "Draft",
                    Remarks = request.Remarks,
                    CreatedBy = _currentUser.UserId.Value
                };
                
                // 5. Add order lines
                decimal subTotal = 0;
                decimal totalDiscount = 0;
                decimal totalTax = 0;
                int lineNumber = 0;
                
                foreach (var lineRequest in request.Lines)
                {
                    lineNumber++;
                    
                    // Get item details
                    var item = await _context.Set<Item>()
                        .FirstOrDefaultAsync(i => i.ItemId == lineRequest.ItemId, cancellationToken);
                    
                    if (item == null)
                        return Result<SalesOrderDto>.Failure($"Item not found: {lineRequest.ItemId}");
                    
                    // Get price
                    var price = await GetItemPriceAsync(
                        lineRequest.ItemId,
                        customer.PriceListId,
                        lineRequest.Quantity);
                    
                    // Calculate amounts
                    var lineTotal = (price - lineRequest.DiscountAmount) * lineRequest.Quantity;
                    var taxAmount = lineTotal * (lineRequest.TaxPercent / 100);
                    
                    var orderLine = new SalesOrderLine
                    {
                        LineNumber = lineNumber,
                        ItemId = lineRequest.ItemId,
                        ItemCode = item.ItemCode,
                        ItemName = item.ItemName,
                        ItemName_AR = item.ItemName_AR,
                        Description = lineRequest.Description,
                        QuantityOrdered = lineRequest.Quantity,
                        UnitId = lineRequest.UnitId,
                        UnitName = lineRequest.UnitName,
                        UnitPrice = price,
                        DiscountPercent = lineRequest.DiscountPercent,
                        DiscountAmount = lineRequest.DiscountAmount,
                        TaxGroupId = lineRequest.TaxGroupId,
                        TaxPercent = lineRequest.TaxPercent,
                        TaxAmount = taxAmount,
                        RequestedDeliveryDate = lineRequest.RequestedDeliveryDate,
                        WarehouseId = lineRequest.WarehouseId,
                        Notes = lineRequest.Notes
                    };
                    
                    order.Lines.Add(orderLine);
                    
                    subTotal += lineTotal;
                    totalDiscount += lineRequest.DiscountAmount * lineRequest.Quantity;
                    totalTax += taxAmount;
                }
                
                // 6. Update order totals
                order.SubTotal = subTotal;
                order.TotalDiscount = totalDiscount;
                order.TotalTax = totalTax;
                order.TotalAmount = subTotal + totalTax;
                order.TotalQuantityOrdered = order.Lines.Sum(l => l.QuantityOrdered);
                
                // 7. Save order
                _context.Set<SalesOrder>().Add(order);
                await _context.SaveChangesAsync(cancellationToken);
                
                // 8. Reserve inventory if configured
                if (request.ReserveInventory)
                {
                    await ReserveInventoryAsync(order);
                }
                
                // 9. Create approval workflow if required
                if (RequiresApproval(order))
                {
                    await CreateApprovalWorkflowAsync(order);
                }
                
                // 10. Update quotation status if created from quotation
                if (order.QuotationId.HasValue)
                {
                    await UpdateQuotationStatusAsync(order.QuotationId.Value, "Converted");
                }
                
                await transaction.CommitAsync(cancellationToken);
                
                // 11. Publish integration event
                await _eventBus.PublishAsync(new SalesOrderCreatedEvent
                {
                    OrderId = order.OrderId,
                    OrderNumber = order.OrderNumber,
                    CustomerId = order.CustomerId,
                    TotalAmount = order.TotalAmount
                });
                
                return Result<SalesOrderDto>.Success(_mapper.Map<SalesOrderDto>(order));
            }
            catch (Exception ex)
            {
                await transaction.RollbackAsync(cancellationToken);
                _logger.LogError(ex, "Error creating sales order");
                return Result<SalesOrderDto>.Failure($"Error creating order: {ex.Message}");
            }
        }
    }
}
```

تابع في الملف السابع لوحدة المشتريات...
