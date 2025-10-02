# Remaining Modules Summary & Integration Guide
## ملخص الوحدات المتبقية ودليل التكامل

## 1. Purchasing Module Summary - ملخص وحدة المشتريات

### 1.1 Core Tables Structure

```sql
-- Supplier Management
[Purchasing].[SupplierGroups]
[Purchasing].[Suppliers]
[Purchasing].[SupplierContacts]
[Purchasing].[SupplierBankAccounts]

-- Purchase Requests
[Purchasing].[PurchaseRequests]
[Purchasing].[PurchaseRequestLines]

-- Purchase Quotations
[Purchasing].[PurchaseQuotations]
[Purchasing].[PurchaseQuotationLines]

-- Purchase Orders
[Purchasing].[PurchaseOrders]
[Purchasing].[PurchaseOrderLines]

-- Goods Receipt
[Purchasing].[GoodsReceipts]
[Purchasing].[GoodsReceiptLines]

-- Purchase Invoices
[Purchasing].[PurchaseInvoices]
[Purchasing].[PurchaseInvoiceLines]

-- Supplier Payments
[Purchasing].[SupplierPayments]
[Purchasing].[PaymentAllocations]

-- Import Letters (LC)
[Purchasing].[ImportLetters]
[Purchasing].[ImportLetterItems]
[Purchasing].[ImportCosts]
```

### 1.2 Financial Integration Points

```csharp
// Purchase Invoice Posting
public async Task<JournalEntry> PostPurchaseInvoiceAsync(long invoiceId)
{
    var invoice = await GetInvoiceWithLinesAsync(invoiceId);
    var postingRequest = new FinancialPostingRequest
    {
        SourceModule = "Purchasing",
        DocumentType = "PurchaseInvoice",
        DocumentId = invoice.InvoiceId,
        DocumentNumber = invoice.InvoiceNumber,
        PostingDate = invoice.InvoiceDate,
        Lines = new List<PostingLineRequest>
        {
            // DR: Inventory/Expense Account
            new PostingLineRequest
            {
                AccountCode = "INVENTORY", // or EXPENSE based on item type
                DebitAmount = invoice.SubTotal
            },
            // DR: VAT Input
            new PostingLineRequest
            {
                AccountCode = "VAT_INPUT",
                DebitAmount = invoice.TotalTax
            },
            // CR: Accounts Payable
            new PostingLineRequest
            {
                AccountCode = "AP",
                CreditAmount = invoice.TotalAmount
            }
        }
    };
    
    return await _financialPostingService.PostDocumentAsync(postingRequest);
}
```

## 2. Inventory Module Summary - ملخص وحدة المخزون

### 2.1 Core Tables Structure

```sql
-- Item Master
[Inventory].[ItemCategories]
[Inventory].[Items]
[Inventory].[ItemTranslations]
[Inventory].[ItemUnits] -- Multiple UOMs per item
[Inventory].[ItemBarcodes]
[Inventory].[ItemImages]
[Inventory].[ItemAlternatives]

-- Warehouses
[Inventory].[Warehouses]
[Inventory].[WarehouseLocations]
[Inventory].[WarehouseBins]

-- Stock Management
[Inventory].[ItemStock] -- Current stock levels
[Inventory].[ItemStockByLocation]
[Inventory].[ItemLots] -- Serial/Batch tracking
[Inventory].[ItemLotTransactions]

-- Stock Movements
[Inventory].[StockTransactions]
[Inventory].[StockTransactionLines]
[Inventory].[StockAdjustments]
[Inventory].[StockAdjustmentLines]
[Inventory].[StockTransfers]
[Inventory].[StockTransferLines]

-- Stock Taking (Physical Inventory)
[Inventory].[StockTakings]
[Inventory].[StockTakingLines]
[Inventory].[StockTakingResults]

-- Item Costing
[Inventory].[ItemCostHistory]
[Inventory].[ItemCostCalculations]

-- Reservations
[Inventory].[StockReservations]
```

### 2.2 Costing Methods Implementation

```csharp
public interface IInventoryCostingService
{
    Task<decimal> CalculateItemCostAsync(
        long itemId,
        long warehouseId,
        decimal quantity,
        CostingMethod method);
    
    Task RecalculateItemCostsAsync(
        long itemId,
        DateTime fromDate);
}

public class InventoryCostingService : IInventoryCostingService
{
    public async Task<decimal> CalculateItemCostAsync(
        long itemId,
        long warehouseId,
        decimal quantity,
        CostingMethod method)
    {
        return method switch
        {
            CostingMethod.FIFO => await CalculateFIFOCostAsync(itemId, warehouseId, quantity),
            CostingMethod.LIFO => await CalculateLIFOCostAsync(itemId, warehouseId, quantity),
            CostingMethod.WeightedAverage => await CalculateWeightedAverageCostAsync(itemId, warehouseId),
            CostingMethod.MovingAverage => await CalculateMovingAverageCostAsync(itemId, warehouseId),
            CostingMethod.StandardCost => await GetStandardCostAsync(itemId),
            _ => throw new NotSupportedException($"Costing method {method} not supported")
        };
    }
    
    private async Task<decimal> CalculateFIFOCostAsync(
        long itemId,
        long warehouseId,
        decimal quantity)
    {
        // Get oldest stock lots first
        var lots = await _context.Set<ItemLot>()
            .Where(l => l.ItemId == itemId && l.WarehouseId == warehouseId)
            .Where(l => l.QuantityOnHand > 0)
            .OrderBy(l => l.ReceiptDate)
            .ToListAsync();
        
        decimal totalCost = 0;
        decimal remainingQty = quantity;
        
        foreach (var lot in lots)
        {
            if (remainingQty <= 0) break;
            
            var qtyFromLot = Math.Min(remainingQty, lot.QuantityOnHand);
            totalCost += qtyFromLot * lot.UnitCost;
            remainingQty -= qtyFromLot;
        }
        
        return quantity > 0 ? totalCost / quantity : 0;
    }
}
```

### 2.3 Stock Movement Financial Integration

```csharp
public async Task PostInventoryTransactionAsync(StockTransaction transaction)
{
    var postingRequest = new FinancialPostingRequest
    {
        SourceModule = "Inventory",
        DocumentType = transaction.TransactionType,
        DocumentId = transaction.TransactionId,
        PostingDate = transaction.TransactionDate
    };
    
    foreach (var line in transaction.Lines)
    {
        var item = await GetItemAsync(line.ItemId);
        var totalCost = line.Quantity * line.UnitCost;
        
        switch (transaction.TransactionType)
        {
            case "Receipt":
                // DR: Inventory, CR: GRN Clearing
                postingRequest.Lines.Add(new PostingLineRequest
                {
                    AccountId = item.InventoryAccountId,
                    DebitAmount = totalCost
                });
                postingRequest.Lines.Add(new PostingLineRequest
                {
                    AccountId = GetGRNAccountId(),
                    CreditAmount = totalCost
                });
                break;
                
            case "Issue":
                // DR: COGS/Expense, CR: Inventory
                postingRequest.Lines.Add(new PostingLineRequest
                {
                    AccountId = GetCOGSAccountId(),
                    DebitAmount = totalCost,
                    CostCenterId = transaction.CostCenterId,
                    ProjectId = transaction.ProjectId
                });
                postingRequest.Lines.Add(new PostingLineRequest
                {
                    AccountId = item.InventoryAccountId,
                    CreditAmount = totalCost
                });
                break;
                
            case "Adjustment":
                // DR/CR based on adjustment type
                if (line.Quantity > 0)
                {
                    // Increase: DR Inventory, CR Adjustment Account
                    postingRequest.Lines.Add(new PostingLineRequest
                    {
                        AccountId = item.InventoryAccountId,
                        DebitAmount = totalCost
                    });
                    postingRequest.Lines.Add(new PostingLineRequest
                    {
                        AccountId = GetInventoryAdjustmentAccountId(),
                        CreditAmount = totalCost
                    });
                }
                else
                {
                    // Decrease: DR Adjustment Account, CR Inventory
                    postingRequest.Lines.Add(new PostingLineRequest
                    {
                        AccountId = GetInventoryAdjustmentAccountId(),
                        DebitAmount = Math.Abs(totalCost)
                    });
                    postingRequest.Lines.Add(new PostingLineRequest
                    {
                        AccountId = item.InventoryAccountId,
                        CreditAmount = Math.Abs(totalCost)
                    });
                }
                break;
        }
    }
    
    await _financialPostingService.PostDocumentAsync(postingRequest);
}
```

## 3. HR & Payroll Module Summary - ملخص وحدة الموارد البشرية

### 3.1 Core Tables

```sql
-- Employee Management
[HR].[Departments]
[HR].[Positions]
[HR].[Employees]
[HR].[EmployeeContracts]
[HR].[EmployeeDocuments]

-- Payroll Structure
[HR].[SalaryComponents] -- Basic, Allowances, Deductions
[HR].[SalaryStructures]
[HR].[SalaryStructureLines]

-- Attendance
[HR].[AttendanceRecords]
[HR].[LeaveTypes]
[HR].[LeaveRequests]
[HR].[LeaveBalances]
[HR].[Overtime]

-- Payroll Processing
[HR].[PayrollRuns]
[HR].[PayrollRunLines]
[HR].[PayrollTransactions]
[HR].[LoanRequests]
[HR].[LoanInstallments]

-- Benefits & Insurance
[HR].[EmployeeBenefits]
[HR].[InsurancePolicies]
[HR].[EndOfServiceProvision]
```

### 3.2 Payroll Calculation Service

```csharp
public class PayrollCalculationService
{
    public async Task<PayrollRun> ProcessPayrollAsync(int year, int month)
    {
        var employees = await GetActiveEmployeesAsync();
        var payrollRun = new PayrollRun
        {
            Year = year,
            Month = month,
            PayrollDate = new DateTime(year, month, DateTime.DaysInMonth(year, month)),
            Status = "Draft"
        };
        
        foreach (var employee in employees)
        {
            var payrollLine = await CalculateEmployeePayrollAsync(employee, year, month);
            payrollRun.Lines.Add(payrollLine);
        }
        
        payrollRun.TotalGross = payrollRun.Lines.Sum(l => l.GrossSalary);
        payrollRun.TotalDeductions = payrollRun.Lines.Sum(l => l.TotalDeductions);
        payrollRun.TotalNet = payrollRun.Lines.Sum(l => l.NetSalary);
        
        await SavePayrollRunAsync(payrollRun);
        return payrollRun;
    }
    
    private async Task<PayrollRunLine> CalculateEmployeePayrollAsync(
        Employee employee,
        int year,
        int month)
    {
        var line = new PayrollRunLine
        {
            EmployeeId = employee.EmployeeId,
            EmployeeCode = employee.EmployeeCode,
            EmployeeName = employee.FullName
        };
        
        // 1. Basic Salary
        line.BasicSalary = employee.BasicSalary;
        
        // 2. Allowances
        var allowances = await CalculateAllowancesAsync(employee, year, month);
        line.TotalAllowances = allowances.Sum(a => a.Amount);
        
        // 3. Overtime
        var overtime = await CalculateOvertimeAsync(employee.EmployeeId, year, month);
        line.OvertimeAmount = overtime;
        
        // 4. Gross Salary
        line.GrossSalary = line.BasicSalary + line.TotalAllowances + line.OvertimeAmount;
        
        // 5. Deductions
        line.SocialInsurance = CalculateSocialInsurance(line.GrossSalary);
        line.IncomeTax = CalculateIncomeTax(line.GrossSalary);
        line.LoanDeduction = await CalculateLoanDeductionAsync(employee.EmployeeId, year, month);
        line.AbsenceDeduction = await CalculateAbsenceDeductionAsync(employee.EmployeeId, year, month);
        line.TotalDeductions = line.SocialInsurance + line.IncomeTax + 
                               line.LoanDeduction + line.AbsenceDeduction;
        
        // 6. Net Salary
        line.NetSalary = line.GrossSalary - line.TotalDeductions;
        
        return line;
    }
}
```

### 3.3 Payroll Financial Posting

```csharp
public async Task PostPayrollToGLAsync(long payrollRunId)
{
    var payrollRun = await GetPayrollRunWithLinesAsync(payrollRunId);
    
    var postingRequest = new FinancialPostingRequest
    {
        SourceModule = "HR",
        DocumentType = "Payroll",
        DocumentId = payrollRun.PayrollRunId,
        PostingDate = payrollRun.PayrollDate,
        Lines = new List<PostingLineRequest>()
    };
    
    // Group by department/cost center
    var byDepartment = payrollRun.Lines
        .GroupBy(l => l.Employee.DepartmentId)
        .ToList();
    
    foreach (var group in byDepartment)
    {
        var totalGross = group.Sum(l => l.GrossSalary);
        var totalDeductions = group.Sum(l => l.TotalDeductions);
        var totalNet = group.Sum(l => l.NetSalary);
        var department = await GetDepartmentAsync(group.Key);
        
        // DR: Salary Expense
        postingRequest.Lines.Add(new PostingLineRequest
        {
            AccountCode = "SALARY_EXPENSE",
            DebitAmount = totalGross,
            CostCenterId = department.CostCenterId,
            Description = $"Payroll for {payrollRun.Year}-{payrollRun.Month:D2}"
        });
        
        // CR: Salary Payable
        postingRequest.Lines.Add(new PostingLineRequest
        {
            AccountCode = "SALARY_PAYABLE",
            CreditAmount = totalNet
        });
        
        // CR: Social Insurance Payable
        var socialInsurance = group.Sum(l => l.SocialInsurance);
        postingRequest.Lines.Add(new PostingLineRequest
        {
            AccountCode = "SOCIAL_INS_PAYABLE",
            CreditAmount = socialInsurance
        });
        
        // CR: Tax Payable
        var tax = group.Sum(l => l.IncomeTax);
        postingRequest.Lines.Add(new PostingLineRequest
        {
            AccountCode = "TAX_PAYABLE",
            CreditAmount = tax
        });
    }
    
    await _financialPostingService.PostDocumentAsync(postingRequest);
}
```

## 4. Fixed Assets Module Summary - ملخص وحدة الأصول الثابتة

### 4.1 Core Tables

```sql
-- Asset Categories
[FixedAssets].[AssetCategories]
[FixedAssets].[DepreciationMethods]

-- Assets
[FixedAssets].[FixedAssets]
[FixedAssets].[AssetDetails]
[FixedAssets].[AssetImages]
[FixedAssets].[AssetDocuments]
[FixedAssets].[AssetLocations]

-- Depreciation
[FixedAssets].[DepreciationSchedules]
[FixedAssets].[DepreciationTransactions]

-- Asset Movements
[FixedAssets].[AssetTransfers]
[FixedAssets].[AssetDisposals]
[FixedAssets].[AssetRevaluations]

-- Maintenance
[FixedAssets].[MaintenanceSchedules]
[FixedAssets].[MaintenanceRecords]

-- Assets Under Construction
[FixedAssets].[AssetsUnderConstruction]
[FixedAssets].[AUCTransactions]
[FixedAssets].[AUCCapitalization]
```

### 4.2 Depreciation Calculation Service

```csharp
public interface IDepreciationService
{
    Task CalculateMonthlyDepreciationAsync(int year, int month);
    Task<List<DepreciationTransaction>> GetDepreciationScheduleAsync(long assetId);
    Task RecalculateDepreciationAsync(long assetId);
}

public class DepreciationService : IDepreciationService
{
    public async Task CalculateMonthlyDepreciationAsync(int year, int month)
    {
        var assets = await GetActiveAssetsForDepreciationAsync();
        
        foreach (var asset in assets)
        {
            var depreciationAmount = CalculateDepreciation(asset, year, month);
            
            if (depreciationAmount > 0)
            {
                var transaction = new DepreciationTransaction
                {
                    AssetId = asset.AssetId,
                    Year = year,
                    Month = month,
                    DepreciationAmount = depreciationAmount,
                    AccumulatedDepreciation = asset.AccumulatedDepreciation + depreciationAmount,
                    BookValue = asset.Cost - (asset.AccumulatedDepreciation + depreciationAmount),
                    TransactionDate = new DateTime(year, month, DateTime.DaysInMonth(year, month))
                };
                
                await SaveDepreciationTransactionAsync(transaction);
                await PostDepreciationToGLAsync(transaction);
            }
        }
    }
    
    private decimal CalculateDepreciation(FixedAsset asset, int year, int month)
    {
        return asset.DepreciationMethod switch
        {
            "StraightLine" => CalculateStraightLineDepreciation(asset),
            "DecliningBalance" => CalculateDecliningBalanceDepreciation(asset),
            "UnitsOfProduction" => CalculateUnitsOfProductionDepreciation(asset),
            _ => 0
        };
    }
    
    private decimal CalculateStraightLineDepreciation(FixedAsset asset)
    {
        var depreciableAmount = asset.Cost - asset.SalvageValue;
        var monthlyDepreciation = depreciableAmount / (asset.UsefulLifeMonths);
        
        // Check if already fully depreciated
        if (asset.AccumulatedDepreciation + monthlyDepreciation > depreciableAmount)
        {
            return depreciableAmount - asset.AccumulatedDepreciation;
        }
        
        return monthlyDepreciation;
    }
}
```

### 4.3 Asset Depreciation Financial Posting

```csharp
public async Task PostDepreciationToGLAsync(DepreciationTransaction transaction)
{
    var asset = await GetAssetAsync(transaction.AssetId);
    
    var postingRequest = new FinancialPostingRequest
    {
        SourceModule = "FixedAssets",
        DocumentType = "Depreciation",
        DocumentId = transaction.TransactionId,
        PostingDate = transaction.TransactionDate,
        Lines = new List<PostingLineRequest>
        {
            // DR: Depreciation Expense
            new PostingLineRequest
            {
                AccountId = asset.DepreciationExpenseAccountId,
                DebitAmount = transaction.DepreciationAmount,
                CostCenterId = asset.CostCenterId,
                Description = $"Depreciation for {asset.AssetCode}"
            },
            // CR: Accumulated Depreciation
            new PostingLineRequest
            {
                AccountId = asset.AccumulatedDepreciationAccountId,
                CreditAmount = transaction.DepreciationAmount
            }
        }
    };
    
    await _financialPostingService.PostDocumentAsync(postingRequest);
}
```

## 5. Inter-Module Integration Patterns

### 5.1 Integration Events

```csharp
// Sales to Inventory
public class SalesInvoicePostedEvent : IntegrationEvent
{
    public long InvoiceId { get; set; }
    public List<SalesInvoiceLineDto> Lines { get; set; }
}

public class SalesInvoicePostedEventHandler : IIntegrationEventHandler<SalesInvoicePostedEvent>
{
    public async Task HandleAsync(SalesInvoicePostedEvent @event)
    {
        // Create inventory issue transaction
        foreach (var line in @event.Lines)
        {
            await _inventoryService.IssueStockAsync(new IssueStockRequest
            {
                ItemId = line.ItemId,
                Quantity = line.Quantity,
                WarehouseId = line.WarehouseId,
                SourceModule = "Sales",
                SourceDocumentId = @event.InvoiceId,
                CostingMethod = CostingMethod.FIFO
            });
        }
    }
}

// Inventory to Finance
public class StockIssuedEvent : IntegrationEvent
{
    public long TransactionId { get; set; }
    public decimal TotalCost { get; set; }
}

public class StockIssuedEventHandler : IIntegrationEventHandler<StockIssuedEvent>
{
    public async Task HandleAsync(StockIssuedEvent @event)
    {
        // Post to general ledger
        await _inventoryFinancialService.PostInventoryTransactionAsync(@event.TransactionId);
    }
}
```

### 5.2 Module Dependencies Map

```
┌─────────────────────────────────────────────────┐
│              Core Shared Services               │
│  (Localization, Metadata, NumberSequence, etc)  │
└─────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────┐
│           Finance Module (Central Hub)          │
│   - Chart of Accounts                           │
│   - Journal Entries                             │
│   - General Ledger                              │
└─────────────────────────────────────────────────┘
         ↑           ↑           ↑           ↑
         │           │           │           │
    ┌────┴───┐  ┌───┴────┐ ┌───┴────┐ ┌────┴─────┐
    │ Sales  │  │Purchase│ │Inventory│ │    HR    │
    └────┬───┘  └───┬────┘ └───┬────┘ └────┬─────┘
         │          │           │           │
         └──────────┴───────────┴───────────┘
                        ↓
         ┌──────────────────────────┐
         │  Fixed Assets, Projects, │
         │  Fleet, Manufacturing    │
         └──────────────────────────┘
```

## 6. Background Jobs Configuration

### 6.1 Recurring Jobs Setup

```csharp
public class BackgroundJobsSetup
{
    public void ConfigureRecurringJobs()
    {
        // Daily Jobs
        RecurringJob.AddOrUpdate<AgingCalculationService>(
            "calculate-customer-aging",
            service => service.CalculateCustomerAgingAsync(),
            "0 2 * * *", // Daily at 2 AM
            TimeZoneInfo.Utc);
        
        RecurringJob.AddOrUpdate<AgingCalculationService>(
            "calculate-supplier-aging",
            service => service.CalculateSupplierAgingAsync(),
            "0 3 * * *",
            TimeZoneInfo.Utc);
        
        // Monthly Jobs
        RecurringJob.AddOrUpdate<DepreciationService>(
            "calculate-monthly-depreciation",
            service => service.CalculateMonthlyDepreciationAsync(
                DateTime.Now.Year, 
                DateTime.Now.Month),
            "0 1 1 * *", // 1st day of month at 1 AM
            TimeZoneInfo.Utc);
        
        RecurringJob.AddOrUpdate<PayrollCalculationService>(
            "process-monthly-payroll",
            service => service.ProcessPayrollReminderAsync(),
            "0 9 25 * *", // 25th of month at 9 AM
            TimeZoneInfo.Utc);
        
        // Weekly Jobs
        RecurringJob.AddOrUpdate<InventoryCostingService>(
            "recalculate-inventory-costs",
            service => service.RecalculateAllItemCostsAsync(),
            "0 0 * * 0", // Sunday at midnight
            TimeZoneInfo.Utc);
        
        RecurringJob.AddOrUpdate<FinancialReportService>(
            "generate-weekly-reports",
            service => service.GenerateScheduledReportsAsync(),
            "0 6 * * 1", // Monday at 6 AM
            TimeZoneInfo.Utc);
    }
}
```

## 7. API Controllers Structure Pattern

### 7.1 Standard REST API Pattern

```csharp
[ApiController]
[ApiVersion("1.0")]
[Route("api/v{version:apiVersion}/sales/[controller]")]
[Authorize]
public class InvoicesController : ControllerBase
{
    private readonly IMediator _mediator;
    
    [HttpGet]
    [RequirePermission("sales.invoice.view")]
    public async Task<ActionResult<PagedResult<SalesInvoiceDto>>> GetInvoices(
        [FromQuery] GetSalesInvoicesQuery query)
    {
        var result = await _mediator.Send(query);
        return Ok(result);
    }
    
    [HttpGet("{id}")]
    [RequirePermission("sales.invoice.view")]
    public async Task<ActionResult<SalesInvoiceDto>> GetInvoice(long id)
    {
        var result = await _mediator.Send(new GetSalesInvoiceByIdQuery { InvoiceId = id });
        return result.IsSuccess ? Ok(result.Value) : NotFound();
    }
    
    [HttpPost]
    [RequirePermission("sales.invoice.create")]
    public async Task<ActionResult<SalesInvoiceDto>> CreateInvoice(
        [FromBody] CreateSalesInvoiceCommand command)
    {
        var result = await _mediator.Send(command);
        return result.IsSuccess 
            ? CreatedAtAction(nameof(GetInvoice), new { id = result.Value.InvoiceId }, result.Value)
            : BadRequest(result.Errors);
    }
    
    [HttpPut("{id}")]
    [RequirePermission("sales.invoice.edit")]
    public async Task<ActionResult<SalesInvoiceDto>> UpdateInvoice(
        long id,
        [FromBody] UpdateSalesInvoiceCommand command)
    {
        command.InvoiceId = id;
        var result = await _mediator.Send(command);
        return result.IsSuccess ? Ok(result.Value) : BadRequest(result.Errors);
    }
    
    [HttpPost("{id}/post")]
    [RequirePermission("sales.invoice.post")]
    public async Task<ActionResult> PostInvoice(long id)
    {
        var result = await _mediator.Send(new PostSalesInvoiceCommand { InvoiceId = id });
        return result.IsSuccess ? Ok() : BadRequest(result.Errors);
    }
    
    [HttpDelete("{id}")]
    [RequirePermission("sales.invoice.delete")]
    public async Task<ActionResult> DeleteInvoice(long id)
    {
        var result = await _mediator.Send(new DeleteSalesInvoiceCommand { InvoiceId = id });
        return result.IsSuccess ? NoContent() : BadRequest(result.Errors);
    }
}
```

## 8. Frontend Component Pattern

### 8.1 React Data Grid Component

```typescript
// src/modules/sales/components/InvoiceList.tsx
import React, { useState, useEffect } from 'react';
import { DataGrid, GridColDef } from '@mui/x-data-grid';
import { useLocalization } from '@/hooks/useLocalization';
import { useSalesInvoices } from '../hooks/useSalesInvoices';

export const InvoiceList: React.FC = () => {
  const { t, formatCurrency, formatDate } = useLocalization();
  const { invoices, loading, fetchInvoices } = useSalesInvoices();
  
  const columns: GridColDef[] = [
    { 
      field: 'invoiceNumber', 
      headerName: t('sales.invoice.number'),
      width: 150 
    },
    { 
      field: 'invoiceDate', 
      headerName: t('sales.invoice.date'),
      width: 120,
      valueFormatter: (params) => formatDate(params.value)
    },
    { 
      field: 'customerName', 
      headerName: t('sales.customer'),
      width: 200 
    },
    { 
      field: 'totalAmount', 
      headerName: t('sales.invoice.total'),
      width: 130,
      align: 'right',
      valueFormatter: (params) => formatCurrency(params.value, params.row.currencyCode)
    },
    { 
      field: 'remainingAmount', 
      headerName: t('sales.invoice.remaining'),
      width: 130,
      align: 'right',
      valueFormatter: (params) => formatCurrency(params.value, params.row.currencyCode)
    },
    { 
      field: 'paymentStatus', 
      headerName: t('sales.invoice.paymentStatus'),
      width: 120,
      renderCell: (params) => (
        <StatusChip status={params.value} />
      )
    },
    {
      field: 'actions',
      headerName: t('common.actions'),
      width: 150,
      renderCell: (params) => (
        <ActionButtons 
          invoiceId={params.row.invoiceId}
          status={params.row.status}
        />
      )
    }
  ];
  
  useEffect(() => {
    fetchInvoices();
  }, []);
  
  return (
    <DataGrid
      rows={invoices}
      columns={columns}
      loading={loading}
      pageSize={25}
      rowsPerPageOptions={[25, 50, 100]}
      checkboxSelection
      disableSelectionOnClick
      getRowId={(row) => row.invoiceId}
    />
  );
};
```

---

## الخلاصة النهائية

هذا التصميم المعماري الشامل يغطي:

✅ **البنية الأساسية الكاملة** - Core infrastructure with multi-tenancy
✅ **نظام Metadata متطور** - Dynamic forms & localization
✅ **إطار الربط المالي** - Complete financial integration framework
✅ **الوحدات الأساسية** - Finance, Sales, Purchasing, Inventory, HR, Fixed Assets
✅ **أنماط التكامل** - Integration events & patterns
✅ **خدمات الخلفية** - Background jobs configuration
✅ **API & Frontend** - Complete implementation patterns

**الخطوات التالية للتنفيذ:**

1. إنشاء Core Infrastructure (4 أسابيع)
2. تطوير Finance Module (6 أسابيع)
3. تطوير Sales & Inventory (8 أسابيع)
4. تطوير Purchasing & HR (8 أسابيع)
5. تطوير Fixed Assets & Reports (4 أسابيع)
6. Integration Testing & Deployment (4 أسابيع)

**المجموع: حوالي 34 أسبوع (8 أشهر) للنظام الكامل**
