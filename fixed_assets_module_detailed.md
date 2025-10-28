# 🏢 مديول الأصول الثابتة - Fixed Assets Module
## التحليل التفصيلي الكامل والمعماري

---

## 📑 جدول المحتويات

1. [نظرة عامة](#overview)
2. [الكيانات الأساسية](#entities)
3. [دورة العمل الكاملة](#workflow)
4. [الشاشات التفصيلية](#screens)
5. [التكامل مع المديولات](#integration)
6. [قواعد العمل](#business-rules)
7. [Database Schema](#database)
8. [APIs المطلوبة](#apis)
9. [الأمثلة العملية](#examples)

---

<a name="overview"></a>
## 🎯 1. نظرة عامة على المديول

### الهدف الرئيسي
مديول الأصول الثابتة مسؤول عن إدارة دورة حياة الأصول الكاملة من لحظة الاقتناء حتى التخلص منها، ويشمل:
- إدارة الأصول الثابتة
- الاقتناء (شراء، تحويل من أصول تحت التشغيل، هبات)
- الإهلاك بطرق متعددة
- الصيانة والإصلاحات
- إعادة التقييم
- النقل بين المواقع
- البيع والتخريد
- الجرد الدوري
- تقارير تفصيلية

### النطاق (Scope)
- ✅ إدارة دورة حياة الأصل الكاملة
- ✅ طرق إهلاك متعددة (قسط ثابت، متناقص، وحدات إنتاج)
- ✅ الإهلاك الضريبي والمحاسبي المنفصل
- ✅ تحويل أصول تحت التشغيل → أصول ثابتة
- ✅ الصيانة الوقائية والتصحيحية
- ✅ إعادة التقييم والتحسينات
- ✅ البيع والتخريد
- ✅ Multi-Location & Multi-Company
- ✅ التتبع بالباركود/RFID
- ✅ الجرد والتسويات

### الميزات الرئيسية
1. **إدارة متقدمة للأصول**
   - تصنيف هرمي للأصول
   - تتبع تفصيلي لكل أصل
   - إدارة المواصفات والمستندات
   - تاريخ كامل للأصل

2. **نظام إهلاك شامل**
   - طرق إهلاك متعددة
   - احتساب آلي شهري/سنوي
   - إهلاك محاسبي وضريبي منفصل
   - معالجة الإهلاك المتراكم

3. **إدارة الصيانة**
   - صيانة وقائية مجدولة
   - صيانة تصحيحية
   - تتبع تكاليف الصيانة
   - تاريخ الصيانة

4. **العمليات المالية**
   - الشراء والبيع
   - إعادة التقييم
   - التحسينات الرأسمالية
   - معالجة الربح/الخسارة

---

<a name="entities"></a>
## 🔑 2. الكيانات الأساسية

### 2.1 Asset (الأصل)
الكيان الرئيسي الذي يمثل الأصل الثابت نفسه

**الخصائص الرئيسية:**
- معلومات الأصل الأساسية (الكود، الاسم، الوصف)
- التصنيف (فئة، نوع، مجموعة)
- معلومات الاقتناء (التاريخ، التكلفة، المورد)
- معلومات الإهلاك (الطريقة، المعدل، العمر الإنتاجي)
- الموقع والحالة
- القيم المالية (التكلفة، الإهلاك المتراكم، القيمة الدفترية)

**الحالات (States):**
- Draft (مسودة)
- Under Construction (تحت التشغيل)
- Active (نشط)
- Under Maintenance (تحت الصيانة)
- Disposed (تم التخلص منه)
- Sold (مباع)
- Written Off (مشطوب)

### 2.2 Asset Category (فئة الأصل)
تصنيف الأصول إلى فئات

**أمثلة:**
- المباني والإنشاءات
- الأثاث والتجهيزات
- المعدات والآلات
- السيارات والمركبات
- الأجهزة الإلكترونية
- البرمجيات

**كل فئة تحتوي على:**
- طريقة الإهلاك الافتراضية
- معدل الإهلاك
- العمر الإنتاجي
- الحسابات المحاسبية المرتبطة

### 2.3 Depreciation (الإهلاك)
سجل عمليات الإهلاك

**أنواع:**
- Accounting Depreciation (إهلاك محاسبي)
- Tax Depreciation (إهلاك ضريبي)

**الطرق:**
- Straight Line (القسط الثابت)
- Declining Balance (القسط المتناقص)
- Units of Production (وحدات الإنتاج)
- Sum of Years Digits (مجموع أرقام السنوات)

### 2.4 Asset Maintenance (صيانة الأصل)
سجل عمليات الصيانة

**الأنواع:**
- Preventive (وقائية)
- Corrective (تصحيحية)
- Predictive (تنبؤية)
- Emergency (طارئة)

### 2.5 Asset Transfer (نقل الأصل)
سجل نقل الأصول بين المواقع

### 2.6 Asset Revaluation (إعادة تقييم الأصل)
سجل إعادة تقييم الأصول

### 2.7 Asset Disposal (التخلص من الأصل)
سجل بيع أو تخريد الأصول

---

<a name="workflow"></a>
## 🔄 3. دورة العمل الكاملة

### المسار الأساسي: دورة حياة الأصل

```
┌─────────────────────────────────────────────────────────────────┐
│                    دورة حياة الأصل الثابت                      │
└─────────────────────────────────────────────────────────────────┘

1️⃣ الاقتناء (Acquisition)
   ├── شراء مباشر
   ├── تحويل من أصول تحت التشغيل
   ├── هبات/تبرعات
   └── إيجار تمويلي
              ↓
2️⃣ التفعيل (Activation)
   ├── تسجيل البيانات الأساسية
   ├── تحديد طريقة الإهلاك
   ├── تعيين الموقع والمستخدم
   └── البدء في الإهلاك
              ↓
3️⃣ التشغيل (Operation)
   ├── الإهلاك الدوري
   ├── الصيانة والإصلاحات
   ├── النقل بين المواقع
   └── التحسينات والإضافات
              ↓
4️⃣ إعادة التقييم (Revaluation)
   ├── زيادة القيمة
   ├── تخفيض القيمة
   └── تعديل الإهلاك
              ↓
5️⃣ التخلص (Disposal)
   ├── البيع
   ├── التخريد
   └── الشطب
```

### 3.1 مسار الاقتناء (Acquisition Flow)

```
📦 شراء أصل جديد
    ↓
┌─────────────────────────┐
│  1. أمر شراء            │ ← من مديول المشتريات
│     Purchase Order      │
└─────────────────────────┘
    ↓
┌─────────────────────────┐
│  2. استلام الأصل        │
│     GRN                 │
└─────────────────────────┘
    ↓
┌─────────────────────────┐
│  3. فاتورة الشراء       │
│     Purchase Invoice    │
└─────────────────────────┘
    ↓
┌─────────────────────────┐
│  4. تسجيل الأصل         │
│     Asset Registration  │
│     - التكلفة          │
│     - الموقع          │
│     - المستخدم        │
└─────────────────────────┘
    ↓
┌─────────────────────────┐
│  5. بدء الإهلاك        │
│     Start Depreciation  │
└─────────────────────────┘

القيد المحاسبي:
Dr. Fixed Assets          xxx
    Cr. Accounts Payable      xxx
```

### 3.2 مسار تحويل أصول تحت التشغيل

```
🏗️ مشروع قيد التنفيذ
    ↓
┌─────────────────────────────────┐
│  أصول تحت التشغيل               │
│  Assets Under Construction      │
│  (CIP - Construction in         │
│   Progress)                     │
│                                 │
│  - تكاليف مباشرة               │
│  - تكاليف غير مباشرة           │
│  - تكاليف التمويل              │
└─────────────────────────────────┘
    ↓
┌─────────────────────────────────┐
│  اكتمال المشروع                │
│  Project Completion             │
└─────────────────────────────────┘
    ↓
┌─────────────────────────────────┐
│  تحويل → أصل ثابت               │
│  Transfer to Fixed Asset        │
│                                 │
│  - تجميع التكاليف              │
│  - توزيع على الأصول            │
│  - تسجيل الأصل الجديد           │
└─────────────────────────────────┘
    ↓
┌─────────────────────────────────┐
│  بدء الإهلاك                   │
│  Start Depreciation             │
└─────────────────────────────────┘

القيد المحاسبي:
Dr. Fixed Assets - Building    xxx
    Cr. CIP - Building Project     xxx
```

### 3.3 مسار الإهلاك (Depreciation Flow)

```
📅 نهاية الشهر/السنة
    ↓
┌──────────────────────────────────┐
│  Depreciation Service            │
│  (خدمة الإهلاك الآلية)          │
└──────────────────────────────────┘
    ↓
┌──────────────────────────────────┐
│  1. جلب الأصول القابلة للإهلاك  │
│     - Active Status              │
│     - StartDate <= CurrentDate   │
│     - Not Fully Depreciated      │
└──────────────────────────────────┘
    ↓
┌──────────────────────────────────┐
│  2. حساب الإهلاك لكل أصل        │
│     - حسب الطريقة المحددة       │
│     - المحاسبي والضريبي         │
└──────────────────────────────────┘
    ↓
┌──────────────────────────────────┐
│  3. إنشاء سجلات الإهلاك         │
│     Depreciation Entries         │
└──────────────────────────────────┘
    ↓
┌──────────────────────────────────┐
│  4. ترحيل للمحاسبة              │
│     Post to Accounting           │
└──────────────────────────────────┘

القيد المحاسبي:
Dr. Depreciation Expense      xxx
    Cr. Accumulated Depreciation  xxx
```

### 3.4 مسار الصيانة (Maintenance Flow)

```
🔧 طلب صيانة
    ↓
┌──────────────────────────┐
│  1. إنشاء أمر صيانة      │
│     Maintenance Order    │
│     - النوع             │
│     - الأولوية          │
│     - الوصف            │
└──────────────────────────┘
    ↓
┌──────────────────────────┐
│  2. التخصيص              │
│     Assignment           │
│     - الفني             │
│     - الموعد           │
└──────────────────────────┘
    ↓
┌──────────────────────────┐
│  3. التنفيذ             │
│     Execution           │
│     - قطع الغيار       │
│     - ساعات العمل      │
└──────────────────────────┘
    ↓
┌──────────────────────────┐
│  4. الإنجاز             │
│     Completion          │
│     - تقرير الصيانة    │
│     - التكاليف         │
└──────────────────────────┘
    ↓
┌──────────────────────────┐
│  5. القيد المحاسبي      │
│     Accounting Entry    │
└──────────────────────────┘

إذا كانت صيانة عادية:
Dr. Maintenance Expense       xxx
    Cr. Cash/Inventory/AP        xxx

إذا كانت تحسين رأسمالي:
Dr. Fixed Assets              xxx
    Cr. Cash/Inventory/AP        xxx
```

### 3.5 مسار البيع (Sale/Disposal Flow)

```
💰 قرار بيع الأصل
    ↓
┌─────────────────────────────┐
│  1. تقييم الأصل             │
│     - القيمة الدفترية      │
│     - الإهلاك المتراكم     │
│     - سعر البيع المتوقع    │
└─────────────────────────────┘
    ↓
┌─────────────────────────────┐
│  2. الموافقات               │
│     Approvals               │
└─────────────────────────────┘
    ↓
┌─────────────────────────────┐
│  3. إيقاف الإهلاك          │
│     Stop Depreciation       │
└─────────────────────────────┘
    ↓
┌─────────────────────────────┐
│  4. البيع                  │
│     - فاتورة البيع         │
│     - تحويل الملكية        │
└─────────────────────────────┘
    ↓
┌─────────────────────────────┐
│  5. حساب الربح/الخسارة     │
│     Gain/Loss Calculation   │
│                             │
│     Sale Price              │
│     - Book Value            │
│     = Gain/Loss             │
└─────────────────────────────┘
    ↓
┌─────────────────────────────┐
│  6. القيد المحاسبي         │
│     Accounting Entry        │
└─────────────────────────────┘

القيد:
Dr. Cash/AR                   xxx
Dr. Accumulated Depreciation  xxx
Dr. Loss on Sale             xxx (if loss)
    Cr. Fixed Assets             xxx
    Cr. Gain on Sale             xxx (if gain)
```

---

<a name="screens"></a>
## 🖥️ 4. الشاشات التفصيلية

### 4.1 شاشة الأصل (Asset Screen)

```typescript
interface AssetScreen {
  // التبويبات
  tabs: [
    'General',        // معلومات عامة
    'Financial',      // معلومات مالية
    'Depreciation',   // الإهلاك
    'Maintenance',    // الصيانة
    'Documents',      // المستندات
    'History'         // التاريخ
  ];
  
  // Tab 1: General
  general: {
    sections: [
      {
        title: 'Basic Information',
        fields: [
          { name: 'code', type: 'text', required: true, readOnly: true },
          { name: 'nameAr', type: 'text', required: true },
          { name: 'nameEn', type: 'text', required: true },
          { name: 'categoryId', type: 'dropdown', required: true, lookup: 'AssetCategories' },
          { name: 'typeId', type: 'dropdown', lookup: 'AssetTypes' },
          { name: 'description', type: 'textarea' },
          { name: 'serialNumber', type: 'text' },
          { name: 'barcode', type: 'text' },
          { name: 'rfidTag', type: 'text' },
          { name: 'manufacturer', type: 'text' },
          { name: 'model', type: 'text' },
          { name: 'yearOfManufacture', type: 'number' }
        ]
      },
      {
        title: 'Location & Assignment',
        fields: [
          { name: 'companyId', type: 'dropdown', required: true },
          { name: 'branchId', type: 'dropdown' },
          { name: 'departmentId', type: 'dropdown' },
          { name: 'locationId', type: 'dropdown' },
          { name: 'roomNumber', type: 'text' },
          { name: 'assignedTo', type: 'dropdown', lookup: 'Employees' },
          { name: 'custodian', type: 'dropdown', lookup: 'Employees' }
        ]
      },
      {
        title: 'Status',
        fields: [
          { name: 'status', type: 'dropdown', required: true,
            options: ['Draft', 'Under Construction', 'Active', 'Under Maintenance', 'Disposed', 'Sold', 'Written Off']
          },
          { name: 'condition', type: 'dropdown',
            options: ['Excellent', 'Good', 'Fair', 'Poor', 'Critical']
          },
          { name: 'warrantyStartDate', type: 'date' },
          { name: 'warrantyEndDate', type: 'date' },
          { name: 'insurancePolicyNumber', type: 'text' },
          { name: 'insuranceExpiry', type: 'date' }
        ]
      }
    ]
  };
  
  // Tab 2: Financial
  financial: {
    sections: [
      {
        title: 'Acquisition',
        fields: [
          { name: 'acquisitionMethod', type: 'dropdown', required: true,
            options: ['Purchase', 'Transfer from CIP', 'Donation', 'Lease']
          },
          { name: 'acquisitionDate', type: 'date', required: true },
          { name: 'supplierId', type: 'dropdown', lookup: 'Suppliers' },
          { name: 'purchaseOrderId', type: 'dropdown', lookup: 'PurchaseOrders' },
          { name: 'purchaseInvoiceId', type: 'dropdown', lookup: 'PurchaseInvoices' },
          { name: 'originalCost', type: 'money', required: true },
          { name: 'currency', type: 'text' },
          { name: 'additionalCosts', type: 'money' },
          { name: 'totalCost', type: 'money', calculated: true }
        ]
      },
      {
        title: 'Current Value',
        fields: [
          { name: 'bookValue', type: 'money', readOnly: true },
          { name: 'accumulatedDepreciation', type: 'money', readOnly: true },
          { name: 'netBookValue', type: 'money', readOnly: true },
          { name: 'fairMarketValue', type: 'money' },
          { name: 'salvageValue', type: 'money' }
        ]
      },
      {
        title: 'Accounting',
        fields: [
          { name: 'assetAccountId', type: 'dropdown', required: true, lookup: 'ChartOfAccounts' },
          { name: 'depreciationAccountId', type: 'dropdown', required: true, lookup: 'ChartOfAccounts' },
          { name: 'accumulatedDepreciationAccountId', type: 'dropdown', required: true, lookup: 'ChartOfAccounts' },
          { name: 'costCenterId', type: 'dropdown', lookup: 'CostCenters' },
          { name: 'projectId', type: 'dropdown', lookup: 'Projects' }
        ]
      }
    ]
  };
  
  // Tab 3: Depreciation
  depreciation: {
    sections: [
      {
        title: 'Depreciation Settings',
        fields: [
          { name: 'depreciable', type: 'checkbox' },
          { name: 'depreciationMethod', type: 'dropdown',
            options: ['Straight Line', 'Declining Balance', 'Units of Production', 'Sum of Years Digits']
          },
          { name: 'usefulLife', type: 'number', suffix: 'years' },
          { name: 'usefulLifeMonths', type: 'number', suffix: 'months' },
          { name: 'depreciationRate', type: 'percentage' },
          { name: 'depreciationStartDate', type: 'date' },
          { name: 'salvageValue', type: 'money' },
          { name: 'totalUnitsToDepreciate', type: 'number' } // for Units of Production
        ]
      },
      {
        title: 'Tax Depreciation',
        fields: [
          { name: 'useTaxDepreciation', type: 'checkbox' },
          { name: 'taxDepreciationMethod', type: 'dropdown' },
          { name: 'taxUsefulLife', type: 'number' },
          { name: 'taxDepreciationRate', type: 'percentage' }
        ]
      },
      {
        title: 'Depreciation Summary',
        fields: [
          { name: 'totalDepreciation', type: 'money', readOnly: true },
          { name: 'yearToDateDepreciation', type: 'money', readOnly: true },
          { name: 'remainingDepreciation', type: 'money', readOnly: true },
          { name: 'fullyDepreciatedDate', type: 'date', readOnly: true }
        ]
      }
    ],
    
    // جدول سجلات الإهلاك
    grid: {
      title: 'Depreciation History',
      columns: [
        { field: 'period', header: 'Period', width: '100px' },
        { field: 'depreciationDate', header: 'Date', width: '120px' },
        { field: 'depreciationType', header: 'Type', width: '120px' },
        { field: 'depreciationAmount', header: 'Amount', width: '120px', align: 'right' },
        { field: 'accumulatedDepreciation', header: 'Accumulated', width: '120px', align: 'right' },
        { field: 'netBookValue', header: 'NBV', width: '120px', align: 'right' },
        { field: 'posted', header: 'Posted', width: '80px', type: 'boolean' }
      ],
      actions: ['view', 'reverse']
    }
  };
  
  // Tab 4: Maintenance
  maintenance: {
    grid: {
      title: 'Maintenance History',
      toolbar: {
        buttons: [
          { text: 'New Maintenance', icon: 'add', action: 'create' },
          { text: 'Schedule Preventive', icon: 'calendar', action: 'schedule' }
        ]
      },
      columns: [
        { field: 'code', header: 'Code', width: '120px' },
        { field: 'maintenanceDate', header: 'Date', width: '120px' },
        { field: 'maintenanceType', header: 'Type', width: '120px' },
        { field: 'description', header: 'Description', width: '200px' },
        { field: 'performedBy', header: 'Technician', width: '150px' },
        { field: 'cost', header: 'Cost', width: '120px', align: 'right' },
        { field: 'status', header: 'Status', width: '100px' }
      ],
      actions: ['view', 'edit', 'complete']
    }
  };
  
  // Tab 5: Documents
  documents: {
    sections: [
      {
        title: 'Attachments',
        component: 'FileUpload',
        allowedTypes: ['pdf', 'jpg', 'png', 'doc', 'docx', 'xls', 'xlsx'],
        maxSize: 10, // MB
        categories: [
          'Purchase Invoice',
          'Warranty Card',
          'Insurance Policy',
          'User Manual',
          'Specifications',
          'Photos',
          'Other'
        ]
      }
    ]
  };
  
  // Tab 6: History
  history: {
    grid: {
      title: 'Asset History',
      columns: [
        { field: 'transactionDate', header: 'Date', width: '120px' },
        { field: 'transactionType', header: 'Type', width: '150px' },
        { field: 'description', header: 'Description', width: '300px' },
        { field: 'amount', header: 'Amount', width: '120px', align: 'right' },
        { field: 'performedBy', header: 'User', width: '150px' }
      ]
    },
    
    timeline: {
      events: [
        'Acquisition',
        'Activation',
        'Transfer',
        'Revaluation',
        'Maintenance',
        'Depreciation',
        'Disposal'
      ]
    }
  };
  
  // Action Toolbar
  toolbar: {
    mainActions: [
      { text: 'Save', icon: 'save', action: 'save', shortcut: 'Ctrl+S' },
      { text: 'Post', icon: 'check', action: 'post', condition: 'status == Draft' },
      { text: 'Print', icon: 'print', action: 'print' },
      { text: 'Delete', icon: 'delete', action: 'delete', condition: 'status == Draft' }
    ],
    
    moreActions: [
      { text: 'Transfer', icon: 'swap', action: 'transfer', condition: 'status == Active' },
      { text: 'Revalue', icon: 'money', action: 'revalue', condition: 'status == Active' },
      { text: 'Create Maintenance', icon: 'build', action: 'maintenance' },
      { text: 'Calculate Depreciation', icon: 'calculate', action: 'depreciate' },
      { text: 'Sell', icon: 'sell', action: 'sell', condition: 'status == Active' },
      { text: 'Dispose', icon: 'delete-forever', action: 'dispose', condition: 'status == Active' },
      { text: 'Duplicate', icon: 'copy', action: 'duplicate' },
      { text: 'Export', icon: 'download', action: 'export' }
    ]
  };
}
```

### 4.2 شاشة فئات الأصول (Asset Categories Screen)

```typescript
interface AssetCategoryScreen {
  layout: 'master-detail';
  
  master: {
    title: 'Asset Categories',
    treeView: true,
    hierarchical: true,
    
    toolbar: {
      search: true,
      buttons: [
        { text: 'New Category', icon: 'add' },
        { text: 'New Sub-Category', icon: 'add-child' },
        { text: 'Expand All', icon: 'expand' },
        { text: 'Collapse All', icon: 'collapse' }
      ]
    },
    
    columns: [
      { field: 'code', header: 'Code', width: '100px' },
      { field: 'name', header: 'Name', width: '250px' },
      { field: 'depreciationMethod', header: 'Depreciation', width: '150px' },
      { field: 'usefulLife', header: 'Useful Life', width: '100px' },
      { field: 'assetCount', header: 'Assets', width: '80px', align: 'right' }
    ]
  };
  
  detail: {
    tabs: ['General', 'Depreciation', 'Accounting', 'Assets'];
    
    general: {
      fields: [
        { name: 'code', type: 'text', required: true },
        { name: 'nameAr', type: 'text', required: true },
        { name: 'nameEn', type: 'text', required: true },
        { name: 'parentCategoryId', type: 'dropdown', lookup: 'AssetCategories' },
        { name: 'description', type: 'textarea' },
        { name: 'isActive', type: 'checkbox' }
      ]
    };
    
    depreciation: {
      fields: [
        { name: 'defaultDepreciationMethod', type: 'dropdown', required: true },
        { name: 'defaultUsefulLife', type: 'number', suffix: 'years' },
        { name: 'defaultDepreciationRate', type: 'percentage' },
        { name: 'defaultSalvageValuePercentage', type: 'percentage' },
        { name: 'taxDepreciationMethod', type: 'dropdown' },
        { name: 'taxUsefulLife', type: 'number' }
      ]
    };
    
    accounting: {
      fields: [
        { name: 'assetAccountId', type: 'dropdown', lookup: 'ChartOfAccounts' },
        { name: 'depreciationAccountId', type: 'dropdown', lookup: 'ChartOfAccounts' },
        { name: 'accumulatedDepreciationAccountId', type: 'dropdown', lookup: 'ChartOfAccounts' },
        { name: 'disposalAccountId', type: 'dropdown', lookup: 'ChartOfAccounts' },
        { name: 'gainOnDisposalAccountId', type: 'dropdown', lookup: 'ChartOfAccounts' },
        { name: 'lossOnDisposalAccountId', type: 'dropdown', lookup: 'ChartOfAccounts' }
      ]
    };
    
    assets: {
      grid: {
        title: 'Assets in this Category',
        columns: [
          { field: 'code', header: 'Code' },
          { field: 'name', header: 'Name' },
          { field: 'acquisitionDate', header: 'Acquisition Date' },
          { field: 'cost', header: 'Cost' },
          { field: 'netBookValue', header: 'NBV' },
          { field: 'status', header: 'Status' }
        ]
      }
    }
  };
}
```

### 4.3 شاشة الإهلاك (Depreciation Screen)

```typescript
interface DepreciationScreen {
  // شاشة تشغيل الإهلاك الدفعي
  batchDepreciation: {
    wizard: {
      steps: [
        'Selection Criteria',
        'Preview & Calculate',
        'Review',
        'Post'
      ]
    };
    
    step1_criteria: {
      fields: [
        { name: 'depreciationPeriod', type: 'month-picker', required: true },
        { name: 'depreciationType', type: 'radio',
          options: ['Accounting', 'Tax', 'Both'],
          default: 'Accounting'
        },
        { name: 'categoryId', type: 'multiselect', lookup: 'AssetCategories' },
        { name: 'branchId', type: 'multiselect', lookup: 'Branches' },
        { name: 'includeNewAssets', type: 'checkbox', default: true },
        { name: 'excludeFullyDepreciated', type: 'checkbox', default: true }
      ],
      
      summary: {
        totalAssets: 0,
        totalCost: 0,
        estimatedDepreciation: 0
      }
    };
    
    step2_calculate: {
      grid: {
        selectable: true,
        columns: [
          { field: 'select', type: 'checkbox', width: '40px' },
          { field: 'assetCode', header: 'Asset Code', width: '120px' },
          { field: 'assetName', header: 'Asset Name', width: '200px' },
          { field: 'category', header: 'Category', width: '150px' },
          { field: 'method', header: 'Method', width: '120px' },
          { field: 'cost', header: 'Cost', width: '120px', align: 'right' },
          { field: 'accumulated', header: 'Accumulated', width: '120px', align: 'right' },
          { field: 'nbv', header: 'NBV', width: '120px', align: 'right' },
          { field: 'depreciation', header: 'Depreciation', width: '120px', align: 'right', editable: true },
          { field: 'newNbv', header: 'New NBV', width: '120px', align: 'right' }
        ]
      },
      
      footer: {
        totalDepreciation: 0,
        selectedAssets: 0
      }
    };
    
    step3_review: {
      summary: {
        period: '',
        assetsCount: 0,
        totalDepreciation: 0,
        accountingEntries: []
      },
      
      validation: {
        checks: [
          'All selected assets are eligible',
          'No duplicate entries for the period',
          'Accounting entries are balanced',
          'No negative NBV values'
        ]
      }
    };
    
    step4_post: {
      options: {
        postToAccounting: true,
        generateReport: true,
        sendNotification: true
      },
      
      progress: {
        current: 0,
        total: 0,
        status: 'In Progress'
      }
    }
  };
  
  // شاشة عرض سجلات الإهلاك
  depreciationHistory: {
    filters: {
      period: 'month-range',
      depreciationType: 'dropdown',
      categoryId: 'multiselect',
      status: 'dropdown'
    };
    
    grid: {
      groupBy: 'period',
      columns: [
        { field: 'period', header: 'Period', width: '100px' },
        { field: 'batchNumber', header: 'Batch', width: '120px' },
        { field: 'assetCount', header: 'Assets', width: '80px', align: 'right' },
        { field: 'totalDepreciation', header: 'Total', width: '120px', align: 'right' },
        { field: 'posted', header: 'Posted', width: '80px', type: 'boolean' },
        { field: 'postedDate', header: 'Posted Date', width: '120px' },
        { field: 'postedBy', header: 'Posted By', width: '150px' }
      ],
      
      actions: ['view', 'reverse', 'print', 'export']
    }
  };
}
```

### 4.4 شاشة الصيانة (Maintenance Screen)

```typescript
interface MaintenanceScreen {
  tabs: ['General', 'Parts & Labor', 'Schedule', 'Documents'];
  
  general: {
    sections: [
      {
        title: 'Maintenance Information',
        fields: [
          { name: 'code', type: 'text', required: true, readOnly: true },
          { name: 'assetId', type: 'dropdown', required: true, lookup: 'Assets' },
          { name: 'maintenanceType', type: 'dropdown', required: true,
            options: ['Preventive', 'Corrective', 'Predictive', 'Emergency']
          },
          { name: 'priority', type: 'dropdown',
            options: ['Low', 'Medium', 'High', 'Critical']
          },
          { name: 'scheduledDate', type: 'datetime' },
          { name: 'actualStartDate', type: 'datetime' },
          { name: 'actualEndDate', type: 'datetime' },
          { name: 'description', type: 'textarea', required: true },
          { name: 'problemDescription', type: 'textarea' },
          { name: 'workPerformed', type: 'textarea' }
        ]
      },
      {
        title: 'Assignment',
        fields: [
          { name: 'assignedTo', type: 'dropdown', lookup: 'Employees' },
          { name: 'assignedTeam', type: 'dropdown', lookup: 'Teams' },
          { name: 'requestedBy', type: 'dropdown', lookup: 'Employees' },
          { name: 'approvedBy', type: 'dropdown', lookup: 'Employees' }
        ]
      },
      {
        title: 'Status',
        fields: [
          { name: 'status', type: 'dropdown',
            options: ['Draft', 'Scheduled', 'In Progress', 'Completed', 'Cancelled']
          },
          { name: 'completionPercentage', type: 'percentage' }
        ]
      }
    ]
  };
  
  partsAndLabor: {
    sections: [
      {
        title: 'Parts Used',
        grid: {
          columns: [
            { field: 'lineNumber', header: '#', width: '50px' },
            { field: 'productId', header: 'Part', width: '200px', editable: true },
            { field: 'quantity', header: 'Qty', width: '100px', editable: true },
            { field: 'unitId', header: 'Unit', width: '100px' },
            { field: 'unitCost', header: 'Unit Cost', width: '120px', align: 'right' },
            { field: 'totalCost', header: 'Total', width: '120px', align: 'right' }
          ],
          actions: ['add', 'delete']
        }
      },
      {
        title: 'Labor',
        grid: {
          columns: [
            { field: 'lineNumber', header: '#', width: '50px' },
            { field: 'employeeId', header: 'Technician', width: '200px', editable: true },
            { field: 'hours', header: 'Hours', width: '100px', editable: true },
            { field: 'hourlyRate', header: 'Rate', width: '120px', align: 'right' },
            { field: 'totalCost', header: 'Total', width: '120px', align: 'right' }
          ],
          actions: ['add', 'delete']
        }
      },
      {
        title: 'Other Costs',
        fields: [
          { name: 'externalServiceCost', type: 'money' },
          { name: 'transportationCost', type: 'money' },
          { name: 'otherCosts', type: 'money' }
        ]
      },
      {
        title: 'Total Cost',
        summary: {
          partsCost: 0,
          laborCost: 0,
          otherCosts: 0,
          totalCost: 0
        }
      }
    ]
  };
  
  schedule: {
    fields: [
      { name: 'isRecurring', type: 'checkbox' },
      { name: 'frequency', type: 'dropdown',
        options: ['Daily', 'Weekly', 'Monthly', 'Quarterly', 'Yearly'],
        condition: 'isRecurring == true'
      },
      { name: 'interval', type: 'number', condition: 'isRecurring == true' },
      { name: 'startDate', type: 'date', condition: 'isRecurring == true' },
      { name: 'endDate', type: 'date', condition: 'isRecurring == true' },
      { name: 'nextMaintenanceDate', type: 'date', readOnly: true }
    ],
    
    calendar: {
      title: 'Scheduled Maintenance',
      view: ['month', 'week', 'day']
    }
  };
}
```

---

<a name="integration"></a>
## 🔗 5. التكامل مع المديولات الأخرى

### 5.1 التكامل مع مديول المشتريات

```typescript
interface PurchasingIntegration {
  // عند شراء أصل جديد
  onPurchaseInvoicePosted: {
    trigger: 'PurchaseInvoice.Posted',
    condition: 'invoice.items.any(i => i.product.isFixedAsset == true)',
    
    action: async (invoice) => {
      for (const item of invoice.items.filter(i => i.product.isFixedAsset)) {
        // إنشاء أصل جديد تلقائياً أو ربط بأصل موجود
        await assetService.createFromPurchase({
          productId: item.productId,
          acquisitionDate: invoice.invoiceDate,
          originalCost: item.totalAmount,
          supplierId: invoice.supplierId,
          purchaseInvoiceId: invoice.id,
          categoryId: item.product.assetCategoryId,
          status: 'Draft'
        });
      }
    }
  };
  
  // الحسابات المتأثرة
  accountingImpact: {
    // عند استلام الأصل
    onGRN: {
      debit: 'Assets Under Construction', // if CIP
      // or
      debit: 'Fixed Assets',
      credit: 'GRN Not Invoiced'
    };
    
    // عند الفاتورة
    onInvoice: {
      debit: 'GRN Not Invoiced',
      credit: 'Accounts Payable'
    }
  };
}
```

### 5.2 التكامل مع مديول الحسابات

```typescript
interface AccountingIntegration {
  // قيود الأصول الثابتة
  journalEntries: {
    // 1. الاقتناء
    acquisition: {
      debit: [
        {
          account: 'Fixed Assets - [Category]',
          amount: 'asset.totalCost',
          costCenter: 'asset.costCenterId'
        }
      ],
      credit: [
        {
          account: 'Accounts Payable' | 'Cash' | 'CIP',
          amount: 'asset.totalCost'
        }
      ]
    };
    
    // 2. الإهلاك
    depreciation: {
      monthly: {
        debit: [
          {
            account: 'Depreciation Expense - [Category]',
            amount: 'depreciation.amount',
            costCenter: 'asset.costCenterId'
          }
        ],
        credit: [
          {
            account: 'Accumulated Depreciation - [Category]',
            amount: 'depreciation.amount'
          }
        ]
      }
    };
    
    // 3. الصيانة
    maintenance: {
      revenue: {
        // صيانة عادية (مصروف)
        debit: [
          {
            account: 'Maintenance Expense',
            amount: 'maintenance.totalCost',
            costCenter: 'asset.costCenterId'
          }
        ],
        credit: [
          {
            account: 'Cash' | 'Accounts Payable' | 'Inventory',
            amount: 'maintenance.totalCost'
          }
        ]
      },
      
      capital: {
        // تحسين رأسمالي (يضاف للأصل)
        debit: [
          {
            account: 'Fixed Assets - [Category]',
            amount: 'improvement.cost',
            costCenter: 'asset.costCenterId'
          }
        ],
        credit: [
          {
            account: 'Cash' | 'Accounts Payable' | 'Inventory',
            amount: 'improvement.cost'
          }
        ]
      }
    };
    
    // 4. إعادة التقييم
    revaluation: {
      increase: {
        debit: [
          {
            account: 'Fixed Assets - [Category]',
            amount: 'revaluation.increaseAmount'
          }
        ],
        credit: [
          {
            account: 'Revaluation Surplus (Equity)',
            amount: 'revaluation.increaseAmount'
          }
        ]
      },
      
      decrease: {
        debit: [
          {
            account: 'Revaluation Loss (Expense)',
            amount: 'revaluation.decreaseAmount'
          }
        ],
        credit: [
          {
            account: 'Fixed Assets - [Category]',
            amount: 'revaluation.decreaseAmount'
          }
        ]
      }
    };
    
    // 5. البيع
    disposal: {
      // عكس قيمة الأصل
      step1_removeAsset: {
        debit: [
          {
            account: 'Accumulated Depreciation',
            amount: 'asset.accumulatedDepreciation'
          }
        ],
        credit: [
          {
            account: 'Fixed Assets',
            amount: 'asset.originalCost'
          }
        ]
      },
      
      // تسجيل البيع
      step2_recordSale: {
        debit: [
          {
            account: 'Cash' | 'Accounts Receivable',
            amount: 'disposal.salePrice'
          }
        ],
        credit: [
          {
            account: 'Gain on Sale (if gain)',
            amount: 'disposal.gain'
          }
        ]
      },
      
      // أو تسجيل الخسارة
      step2_recordLoss: {
        debit: [
          {
            account: 'Loss on Sale',
            amount: 'disposal.loss'
          },
          {
            account: 'Cash' | 'Accounts Receivable',
            amount: 'disposal.salePrice'
          }
        ]
      }
    };
    
    // 6. التخريد
    writeOff: {
      debit: [
        {
          account: 'Accumulated Depreciation',
          amount: 'asset.accumulatedDepreciation'
        },
        {
          account: 'Loss on Write-off',
          amount: 'asset.netBookValue'
        }
      ],
      credit: [
        {
          account: 'Fixed Assets',
          amount: 'asset.originalCost'
        }
      ]
    }
  };
  
  // التأثير على القوائم المالية
  financialStatements: {
    balanceSheet: {
      assets: {
        nonCurrentAssets: {
          fixedAssets: 'SUM(assets.cost)',
          lessAccumulatedDepreciation: '-SUM(assets.accumulatedDepreciation)',
          netFixedAssets: 'fixedAssets + lessAccumulatedDepreciation'
        }
      },
      
      equity: {
        revaluationSurplus: 'SUM(revaluations.surplusAmount)'
      }
    };
    
    incomeStatement: {
      expenses: {
        depreciationExpense: 'SUM(depreciation.amount)',
        maintenanceExpense: 'SUM(maintenance.cost where not capital)',
        gainLossOnDisposal: 'SUM(disposal.gainLoss)'
      }
    };
    
    cashFlow: {
      operatingActivities: {
        depreciationAddBack: 'SUM(depreciation.amount)'
      },
      investingActivities: {
        purchaseOfFixedAssets: '-SUM(acquisition.cost)',
        proceedsFromSale: 'SUM(disposal.salePrice)'
      }
    }
  };
}
```

### 5.3 التكامل مع مديول المخزون

```typescript
interface InventoryIntegration {
  // قطع الغيار للصيانة
  maintenanceParts: {
    onMaintenanceOrder: {
      // حجز قطع الغيار
      reserveItems: async (maintenance) => {
        for (const part of maintenance.parts) {
          await inventoryService.reserve({
            productId: part.productId,
            quantity: part.quantity,
            warehouseId: maintenance.warehouseId,
            referenceType: 'Maintenance',
            referenceId: maintenance.id
          });
        }
      }
    };
    
    onMaintenanceComplete: {
      // صرف قطع الغيار
      issueItems: async (maintenance) => {
        const transaction = await inventoryService.issue({
          transactionType: 'Maintenance Issue',
          warehouseId: maintenance.warehouseId,
          costCenterId: maintenance.asset.costCenterId,
          items: maintenance.parts.map(p => ({
            productId: p.productId,
            quantity: p.quantity,
            unitCost: p.unitCost
          }))
        });
        
        // تحديث تكلفة الصيانة
        await maintenanceService.updateCost(maintenance.id, {
          partsCost: transaction.totalCost
        });
      }
    }
  };
  
  // تتبع الأصول ذات الرقم التسلسلي
  serialTracking: {
    // ربط الأصل بالسيريال نمبر في المخزون
    onAssetActivation: async (asset) => {
      if (asset.serialNumber) {
        await inventoryService.linkSerialToAsset({
          productId: asset.productId,
          serialNumber: asset.serialNumber,
          assetId: asset.id
        });
      }
    }
  };
}
```

### 5.4 التكامل مع مديول شئون العاملين (HR)

```typescript
interface HRIntegration {
  // تعيين الأصول للموظفين
  assetCustody: {
    onAssignment: {
      notification: async (asset, employee) => {
        await notificationService.send({
          to: employee.email,
          subject: 'Asset Assignment',
          template: 'asset-custody',
          data: {
            assetName: asset.name,
            assetCode: asset.code,
            serialNumber: asset.serialNumber,
            assignmentDate: new Date(),
            returnDate: asset.expectedReturnDate
          }
        });
      },
      
      document: async (asset, employee) => {
        // إنشاء نموذج عهدة
        await documentService.generate({
          template: 'custody-form',
          data: { asset, employee },
          requireSignature: true
        });
      }
    };
    
    onEmployeeTermination: {
      // التحقق من الأصول المعينة
      checkAssignedAssets: async (employee) => {
        const assets = await assetService.getByEmployee(employee.id);
        
        if (assets.length > 0) {
          // تنبيه HR
          await notificationService.send({
            to: 'hr@company.com',
            subject: 'Employee Termination - Assets Check',
            message: `Employee ${employee.name} has ${assets.length} assigned assets that need to be returned.`
          });
          
          // منع إتمام الإنهاء حتى إرجاع الأصول
          return {
            blocked: true,
            reason: 'Assigned assets pending return',
            assets: assets
          };
        }
      }
    }
  };
  
  // سائقي الأسطول
  fleetDrivers: {
    vehicleAssignment: {
      onAssign: async (vehicle, driver) => {
        await assetService.update(vehicle.assetId, {
          assignedTo: driver.employeeId,
          assignmentDate: new Date()
        });
      }
    }
  };
}
```

### 5.5 التكامل مع مديول المشاريع

```typescript
interface ProjectIntegration {
  // أصول تحت التشغيل
  constructionInProgress: {
    // تجميع تكاليف المشروع
    accumulateCosts: async (project) => {
      const cip = await assetService.getCIP(project.id);
      
      // تكاليف مباشرة
      const directCosts = await projectService.getDirectCosts(project.id);
      // تكاليف غير مباشرة
      const indirectCosts = await projectService.getIndirectCosts(project.id);
      // تكاليف التمويل
      const financingCosts = await projectService.getFinancingCosts(project.id);
      
      const totalCost = directCosts + indirectCosts + financingCosts;
      
      await assetService.updateCIP(cip.id, {
        accumulatedCost: totalCost
      });
    };
    
    // تحويل لأصل ثابت عند الاكتمال
    onProjectComplete: async (project) => {
      const cip = await assetService.getCIP(project.id);
      
      // إنشاء الأصل الثابت
      const asset = await assetService.create({
        name: project.name,
        categoryId: project.assetCategoryId,
        acquisitionMethod: 'Transfer from CIP',
        acquisitionDate: new Date(),
        originalCost: cip.accumulatedCost,
        cipId: cip.id,
        projectId: project.id,
        status: 'Active'
      });
      
      // بدء الإهلاك
      await depreciationService.startDepreciation(asset.id);
      
      // القيد المحاسبي
      await journalService.post({
        description: `Transfer from CIP - ${project.name}`,
        entries: [
          {
            account: asset.assetAccountId,
            debit: cip.accumulatedCost
          },
          {
            account: cip.cipAccountId,
            credit: cip.accumulatedCost
          }
        ]
      });
      
      // تحديث حالة CIP
      await assetService.updateCIP(cip.id, {
        status: 'Transferred',
        transferredToAssetId: asset.id
      });
    }
  };
  
  // تخصيص الأصول للمشاريع
  projectAssets: {
    allocateAsset: async (assetId, projectId) => {
      await assetService.update(assetId, {
        projectId: projectId,
        costCenterId: project.costCenterId
      });
      
      // تحويل الإهلاك للمشروع
      await depreciationService.updateAllocation(assetId, {
        costCenterId: project.costCenterId,
        projectId: projectId
      });
    }
  };
}
```

---

<a name="business-rules"></a>
## ⚙️ 6. قواعد العمل (Business Rules)

### 6.1 قواعد الاقتناء

```typescript
interface AcquisitionRules {
  // قواعد الإنشاء
  creation: {
    requiredFields: ['name', 'categoryId', 'acquisitionDate', 'originalCost'];
    
    codeGeneration: {
      pattern: '{prefix}-{category}-{year}-{sequence}',
      prefix: 'AST',
      sequenceLength: 5,
      example: 'AST-BLD-2025-00001'
    };
    
    validation: {
      acquisitionDate: {
        notFuture: true,
        notBeforeCompanyStartDate: true
      };
      
      originalCost: {
        minimumValue: 0,
        maximumValue: null,
        requireApprovalAbove: 100000
      };
      
      category: {
        mustBeActive: true,
        mustBeLeafNode: true  // لا يمكن اختيار فئة رئيسية
      }
    }
  };
  
  // طرق الاقتناء
  acquisitionMethods: {
    purchase: {
      requirePurchaseOrder: false,  // optional
      requireInvoice: true,
      requireSupplier: true,
      
      costComponents: [
        'Invoice Amount',
        'Shipping Cost',
        'Installation Cost',
        'Customs Duty',
        'Other Direct Costs'
      ]
    };
    
    transferFromCIP: {
      requireProject: true,
      requireCIPAsset: true,
      validateCIPCompleted: true,
      
      costCalculation: 'accumulated_cost_from_cip',
      
      onTransfer: {
        closeCIP: true,
        updateProjectStatus: true,
        startDepreciation: true
      }
    };
    
    donation: {
      requireDonationDocument: true,
      valuationMethod: 'fair_market_value',
      requireApproval: true,
      
      accounting: {
        debit: 'Fixed Assets',
        credit: 'Donation Income'
      }
    };
    
    financialLease: {
      requireLeaseAgreement: true,
      calculatePresentValue: true,
      recognizeAsset: true,
      recognizeLiability: true
    }
  };
}
```

### 6.2 قواعد الإهلاك

```typescript
interface DepreciationRules {
  // متى يبدأ الإهلاك
  depreciationStart: {
    condition: 'asset.status == Active',
    startDate: {
      options: [
        'acquisition_date',
        'first_day_of_next_month',
        'custom_date'
      ],
      default: 'first_day_of_next_month'
    };
    
    proRata: {
      enabled: true,
      calculation: 'days_basis'  // or 'months_basis'
    }
  };
  
  // طرق الإهلاك
  methods: {
    straightLine: {
      formula: '(cost - salvage_value) / useful_life',
      monthlyAmount: 'annual_depreciation / 12',
      
      example: {
        cost: 100000,
        salvageValue: 10000,
        usefulLife: 10,  // years
        annualDepreciation: 9000,  // (100000 - 10000) / 10
        monthlyDepreciation: 750   // 9000 / 12
      }
    };
    
    decliningBalance: {
      formula: 'net_book_value * depreciation_rate',
      doubleDecline: 'rate = 2 / useful_life',
      
      example: {
        cost: 100000,
        rate: 20,  // %
        year1: 20000,  // 100000 * 20%
        year2: 16000,  // 80000 * 20%
        year3: 12800   // 64000 * 20%
      },
      
      switchToStraightLine: {
        when: 'declining_amount < straight_line_amount',
        ensure: 'salvage_value_not_exceeded'
      }
    };
    
    unitsOfProduction: {
      formula: '(cost - salvage_value) * (units_produced / total_units)',
      
      tracking: {
        requireProductionData: true,
        frequency: 'monthly',
        updateDepreciationBasedOnActual: true
      },
      
      example: {
        cost: 100000,
        salvageValue: 10000,
        totalUnits: 100000,  // hours/km/units
        costPerUnit: 0.9,    // (100000 - 10000) / 100000
        month1Units: 500,
        month1Depreciation: 450  // 500 * 0.9
      }
    };
    
    sumOfYearsDigits: {
      formula: '(cost - salvage_value) * (remaining_life / sum_of_years)',
      
      example: {
        cost: 100000,
        salvageValue: 10000,
        usefulLife: 5,
        sumOfYears: 15,  // 5+4+3+2+1
        year1: 30000,    // 90000 * (5/15)
        year2: 24000,    // 90000 * (4/15)
        year3: 18000     // 90000 * (3/15)
      }
    }
  };
  
  // الإهلاك الضريبي
  taxDepreciation: {
    separateFromAccounting: true,
    useTaxRates: true,
    
    temporaryDifferences: {
      track: true,
      account: 'Deferred Tax Asset/Liability'
    }
  };
  
  // قواعد التوقف
  stopDepreciation: {
    when: [
      'asset.status == Disposed',
      'asset.status == Sold',
      'asset.netBookValue <= asset.salvageValue',
      'asset.status == Under Maintenance && stopDuringMaintenance == true'
    ]
  };
  
  // التحقق من الصحة
  validation: {
    notExceedCost: 'accumulated_depreciation <= (cost - salvage_value)',
    notNegativeNBV: 'net_book_value >= salvage_value',
    noDuplicateForPeriod: 'one_depreciation_per_asset_per_period',
    balancedEntries: 'debit_amount == credit_amount'
  };
}
```

### 6.3 قواعد الصيانة

```typescript
interface MaintenanceRules {
  // أنواع الصيانة
  maintenanceTypes: {
    preventive: {
      description: 'Scheduled maintenance to prevent failures',
      scheduling: 'automated',
      capitalization: false,  // مصروف
      
      triggers: [
        'time_based',      // كل X شهور
        'usage_based',     // كل X ساعات/كم
        'condition_based'  // عند وصول شرط معين
      ]
    };
    
    corrective: {
      description: 'Repair after failure',
      scheduling: 'on_demand',
      
      capitalization: {
        rule: 'if_extends_life_or_improves_capacity',
        threshold: 1000,  // مبلغ معين
        requireApproval: true
      }
    };
    
    predictive: {
      description: 'Maintenance based on condition monitoring',
      requiresIoT: true,
      useAI: true
    };
    
    emergency: {
      description: 'Urgent unplanned maintenance',
      priority: 'critical',
      immediate: true
    }
  };
  
  // رسملة الصيانة
  capitalization: {
    criteria: {
      or: [
        {
          type: 'extends_useful_life',
          condition: 'maintenance extends asset useful life'
        },
        {
          type: 'increases_capacity',
          condition: 'maintenance increases production capacity'
        },
        {
          type: 'improves_quality',
          condition: 'maintenance improves output quality'
        },
        {
          type: 'major_overhaul',
          minAmount: 5000,
          condition: 'cost > threshold'
        }
      ]
    };
    
    accounting: {
      capitalize: {
        debit: 'Fixed Assets',
        credit: 'Cash/AP/Inventory',
        
        effect: {
          increaseAssetCost: true,
          recalculateDepreciation: true,
          extendUsefulLife: 'optional'
        }
      };
      
      expense: {
        debit: 'Maintenance Expense',
        credit: 'Cash/AP/Inventory',
        
        effect: {
          noAssetChange: true,
          currentPeriodExpense: true
        }
      }
    }
  };
  
  // الجدولة التلقائية
  scheduling: {
    preventiveMaintenance: {
      basedonTimeBased: {
        interval: 'months',
        frequency: 3,  // كل 3 أشهر
        createWorkOrder: 'auto',
        notifyInAdvance: 7  // days
      };
      
      basedOnUsage: {
        metric: 'hours' | 'kilometers' | 'cycles',
        threshold: 5000,
        trackUsage: true,
        alertWhenNear: 0.9  // 90% of threshold
      }
    }
  };
}
```

### 6.4 قواعد النقل

```typescript
interface TransferRules {
  // قواعد النقل
  transfer: {
    requiredFields: ['fromLocation', 'toLocation', 'transferDate', 'reason'];
    
    validation: {
      assetMustBeActive: true,
      fromLocationNotEqualToLocation: true,
      notUnderMaintenance: true,
      noPendingTransfers: true
    };
    
    approval: {
      requireApproval: true,
      approvalLevels: [
        {
          role: 'Asset Manager',
          condition: 'same_branch'
        },
        {
          role: 'CFO',
          condition: 'different_branch || different_company'
        }
      ]
    };
    
    effects: {
      updateLocation: true,
      updateCostCenter: 'if_different',
      updateCustodian: 'if_specified',
      reallocateDepreciation: true,
      
      notifications: [
        'from_custodian',
        'to_custodian',
        'asset_manager'
      ]
    }
  };
  
  // النقل بين الشركات
  intercompanyTransfer: {
    requireValuation: true,
    createSaleTransaction: true,
    createPurchaseTransaction: true,
    
    accounting: {
      sellingCompany: {
        removeAsset: true,
        recognizeGainLoss: true
      },
      buyingCompany: {
        recordAsset: true,
        atTransferPrice: true
      }
    }
  };
}
```

### 6.5 قواعد إعادة التقييم

```typescript
interface RevaluationRules {
  // متى تتم إعادة التقييم
  when: {
    triggers: [
      'significant_market_change',
      'periodic_review',  // سنوياً مثلاً
      'impairment_indication',
      'pre_sale_assessment'
    ]
  };
  
  // طرق التقييم
  valuationMethods: {
    fairMarketValue: {
      source: 'independent_appraiser',
      required: true
    };
    
    replacementCost: {
      approach: 'current_cost_to_replace'
    };
    
    netRealizableValue: {
      calculation: 'estimated_selling_price - costs_to_sell'
    }
  };
  
  // المعالجة المحاسبية
  accounting: {
    increase: {
      // زيادة القيمة
      ifFirstTime: {
        debit: 'Fixed Assets',
        credit: 'Revaluation Surplus (OCI)',
        
        effect: {
          increaseAssetCost: true,
          increaseEquity: true,
          recalculateDepreciation: true
        }
      };
      
      ifPreviousDecrease: {
        // عكس الخسارة السابقة أولاً
        reverseUpToPreviousLoss: true,
        excessToRevaluationSurplus: true
      }
    };
    
    decrease: {
      // تخفيض القيمة
      ifPreviousIncrease: {
        // تخفيض من فائض إعادة التقييم أولاً
        reduceRevaluationSurplus: true,
        excessToExpense: true
      };
      
      ifNoPreviousIncrease: {
        debit: 'Impairment Loss (Expense)',
        credit: 'Fixed Assets',
        
        effect: {
          decreaseAssetCost: true,
          recognizeLoss: true,
          recalculateDepreciation: true
        }
      }
    }
  };
  
  // تأثير على الإهلاك
  depreciationImpact: {
    recalculate: true,
    basis: 'revalued_amount - salvage_value',
    remainingLife: 'unchanged' | 'reassessed',
    prospective: true  // تأثير مستقبلي فقط
  };
}
```

### 6.6 قواعد البيع والتخلص

```typescript
interface DisposalRules {
  // أنواع التخلص
  disposalTypes: {
    sale: {
      requireBuyer: true,
      requireSalePrice: true,
      requireInvoice: true,
      calculateGainLoss: true
    };
    
    scrap: {
      requireScrapValue: false,
      scrapValueUsually: 0,
      requireApproval: true,
      writeOffRemainingValue: true
    };
    
    donation: {
      requireRecipient: true,
      requireValuation: true,
      noProceeds: true
    };
    
    exchange: {
      requireNewAsset: true,
      calculateBootAmount: true,
      fairValueMeasurement: true
    }
  };
  
  // الموافقات
  approval: {
    required: true,
    levels: [
      {
        role: 'Asset Manager',
        condition: 'nbv < 10000'
      },
      {
        role: 'CFO',
        condition: 'nbv >= 10000 && nbv < 100000'
      },
      {
        role: 'Board',
        condition: 'nbv >= 100000'
      }
    ]
  };
  
  // قبل البيع
  preDisposal: {
    checks: [
      'no_pending_maintenance',
      'clear_custody',
      'remove_from_insurance',
      'settle_any_liens'
    ];
    
    stopDepreciation: {
      at: 'disposal_date',
      calculateFinalDepreciation: true
    }
  };
  
  // حساب الربح/الخسارة
  gainLossCalculation: {
    formula: 'sale_price - net_book_value',
    
    netBookValue: 'original_cost - accumulated_depreciation',
    
    example: {
      originalCost: 100000,
      accumulatedDepreciation: 60000,
      netBookValue: 40000,
      salePrice: 45000,
      gain: 5000  // 45000 - 40000
    }
  };
  
  // المعالجة المحاسبية
  accounting: {
    sale: {
      step1_removeCost: {
        debit: 'Accumulated Depreciation - 60,000',
        credit: 'Fixed Assets - 100,000'
      };
      
      step2_recordSale: {
        debit: 'Cash/AR - 45,000',
        // الفرق هو الربح/الخسارة
      };
      
      step3_recognizeGain: {
        credit: 'Gain on Sale - 5,000'
      };
      
      // أو في حالة الخسارة
      step3_recognizeLoss: {
        debit: 'Loss on Sale - xxx'
      }
    };
    
    writeOff: {
      debit: [
        'Accumulated Depreciation',
        'Loss on Write-off'  // = NBV
      ],
      credit: [
        'Fixed Assets'
      ]
    }
  };
  
  // ما بعد التخلص
  postDisposal: {
    updateStatus: 'Disposed' | 'Sold',
    archiveRecords: true,
    retainHistory: true,
    removeFromActiveReports: true,
    notifyStakeholders: true
  };
}
```

---

<a name="database"></a>
## 🗄️ 7. Database Schema

### 7.1 الجداول الأساسية

```sql
-- جدول فئات الأصول
CREATE TABLE AssetCategories (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    Code NVARCHAR(50) NOT NULL,
    NameAr NVARCHAR(200) NOT NULL,
    NameEn NVARCHAR(200) NOT NULL,
    
    ParentCategoryId UNIQUEIDENTIFIER,
    Level INT NOT NULL DEFAULT 0,
    Path NVARCHAR(500), -- للبحث الهرمي
    
    Description NVARCHAR(1000),
    
    -- إعدادات الإهلاك الافتراضية
    DefaultDepreciationMethod NVARCHAR(50) CHECK (DefaultDepreciationMethod IN ('Straight Line', 'Declining Balance', 'Units of Production', 'Sum of Years Digits')),
    DefaultUsefulLife INT, -- بالسنوات
    DefaultUsefulLifeMonths INT,
    DefaultDepreciationRate DECIMAL(5,2),
    DefaultSalvageValuePercentage DECIMAL(5,2),
    
    -- الإهلاك الضريبي
    TaxDepreciationMethod NVARCHAR(50),
    TaxUsefulLife INT,
    TaxDepreciationRate DECIMAL(5,2),
    
    -- الحسابات المحاسبية
    AssetAccountId UNIQUEIDENTIFIER,
    DepreciationAccountId UNIQUEIDENTIFIER,
    AccumulatedDepreciationAccountId UNIQUEIDENTIFIER,
    DisposalAccountId UNIQUEIDENTIFIER,
    GainOnDisposalAccountId UNIQUEIDENTIFIER,
    LossOnDisposalAccountId UNIQUEIDENTIFIER,
    
    IsActive BIT DEFAULT 1,
    DisplayOrder INT,
    
    -- Audit
    TenantId UNIQUEIDENTIFIER NOT NULL,
    CreatedAt DATETIME2 DEFAULT GETUTCDATE(),
    CreatedBy UNIQUEIDENTIFIER,
    UpdatedAt DATETIME2 DEFAULT GETUTCDATE(),
    UpdatedBy UNIQUEIDENTIFIER,
    IsDeleted BIT DEFAULT 0,
    
    FOREIGN KEY (ParentCategoryId) REFERENCES AssetCategories(Id),
    FOREIGN KEY (AssetAccountId) REFERENCES ChartOfAccounts(Id),
    FOREIGN KEY (DepreciationAccountId) REFERENCES ChartOfAccounts(Id),
    FOREIGN KEY (AccumulatedDepreciationAccountId) REFERENCES ChartOfAccounts(Id),
    
    INDEX IX_AssetCategories_Parent (ParentCategoryId),
    UNIQUE INDEX UX_AssetCategories_Code (TenantId, Code, IsDeleted)
);

-- جدول الأصول
CREATE TABLE Assets (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    Code NVARCHAR(50) NOT NULL,
    NameAr NVARCHAR(200) NOT NULL,
    NameEn NVARCHAR(200) NOT NULL,
    Description NVARCHAR(1000),
    
    -- التصنيف
    CategoryId UNIQUEIDENTIFIER NOT NULL,
    TypeId UNIQUEIDENTIFIER,
    
    -- معلومات المنتج
    SerialNumber NVARCHAR(100),
    Barcode NVARCHAR(100),
    RFIDTag NVARCHAR(100),
    Manufacturer NVARCHAR(200),
    Model NVARCHAR(200),
    YearOfManufacture INT,
    
    -- الموقع والتعيين
    CompanyId UNIQUEIDENTIFIER NOT NULL,
    BranchId UNIQUEIDENTIFIER,
    DepartmentId UNIQUEIDENTIFIER,
    LocationId UNIQUEIDENTIFIER,
    RoomNumber NVARCHAR(50),
    
    AssignedTo UNIQUEIDENTIFIER, -- Employee
    Custodian UNIQUEIDENTIFIER, -- Employee
    AssignmentDate DATE,
    
    -- الاقتناء
    AcquisitionMethod NVARCHAR(50) NOT NULL CHECK (AcquisitionMethod IN ('Purchase', 'Transfer from CIP', 'Donation', 'Lease', 'Exchange')),
    AcquisitionDate DATE NOT NULL,
    
    SupplierId UNIQUEIDENTIFIER,
    PurchaseOrderId UNIQUEIDENTIFIER,
    PurchaseInvoiceId UNIQUEIDENTIFIER,
    CIPAssetId UNIQUEIDENTIFIER, -- إذا كان محول من CIP
    ProjectId UNIQUEIDENTIFIER,
    
    -- التكلفة
    Currency NVARCHAR(3) NOT NULL DEFAULT 'SAR',
    ExchangeRate DECIMAL(18,6) DEFAULT 1,
    OriginalCost DECIMAL(18,2) NOT NULL,
    AdditionalCosts DECIMAL(18,2) DEFAULT 0,
    TotalCost AS (OriginalCost + AdditionalCosts) PERSISTED,
    
    -- القيمة الحالية
    AccumulatedDepreciation DECIMAL(18,2) DEFAULT 0,
    NetBookValue AS (TotalCost - AccumulatedDepreciation) PERSISTED,
    FairMarketValue DECIMAL(18,2),
    SalvageValue DECIMAL(18,2) DEFAULT 0,
    
    -- الإهلاك
    Depreciable BIT DEFAULT 1,
    DepreciationMethod NVARCHAR(50),
    UsefulLife INT, -- بالسنوات
    UsefulLifeMonths INT,
    DepreciationRate DECIMAL(5,2),
    DepreciationStartDate DATE,
    
    -- الإهلاك الضريبي
    UseTaxDepreciation BIT DEFAULT 0,
    TaxDepreciationMethod NVARCHAR(50),
    TaxUsefulLife INT,
    TaxDepreciationRate DECIMAL(5,2),
    
    -- وحدات الإنتاج (للإهلاك)
    TotalUnitsToDepreciate DECIMAL(18,3),
    UnitsDepreciatedToDate DECIMAL(18,3) DEFAULT 0,
    
    -- الحسابات المحاسبية
    AssetAccountId UNIQUEIDENTIFIER,
    DepreciationAccountId UNIQUEIDENTIFIER,
    AccumulatedDepreciationAccountId UNIQUEIDENTIFIER,
    CostCenterId UNIQUEIDENTIFIER,
    
    -- الضمان والتأمين
    WarrantyStartDate DATE,
    WarrantyEndDate DATE,
    InsurancePolicyNumber NVARCHAR(100),
    InsuranceExpiry DATE,
    InsuredValue DECIMAL(18,2),
    
    -- الحالة
    Status NVARCHAR(50) NOT NULL DEFAULT 'Draft' CHECK (Status IN ('Draft', 'Under Construction', 'Active', 'Under Maintenance', 'Disposed', 'Sold', 'Written Off')),
    Condition NVARCHAR(50) CHECK (Condition IN ('Excellent', 'Good', 'Fair', 'Poor', 'Critical')),
    
    -- التخلص
    DisposalDate DATE,
    DisposalMethod NVARCHAR(50),
    DisposalPrice DECIMAL(18,2),
    GainLossOnDisposal DECIMAL(18,2),
    
    Notes NVARCHAR(MAX),
    
    -- Audit
    TenantId UNIQUEIDENTIFIER NOT NULL,
    CreatedAt DATETIME2 DEFAULT GETUTCDATE(),
    CreatedBy UNIQUEIDENTIFIER,
    UpdatedAt DATETIME2 DEFAULT GETUTCDATE(),
    UpdatedBy UNIQUEIDENTIFIER,
    IsDeleted BIT DEFAULT 0,
    
    FOREIGN KEY (CategoryId) REFERENCES AssetCategories(Id),
    FOREIGN KEY (CompanyId) REFERENCES Companies(Id),
    FOREIGN KEY (BranchId) REFERENCES Branches(Id),
    FOREIGN KEY (LocationId) REFERENCES Locations(Id),
    FOREIGN KEY (SupplierId) REFERENCES Suppliers(Id),
    FOREIGN KEY (CostCenterId) REFERENCES CostCenters(Id),
    FOREIGN KEY (ProjectId) REFERENCES Projects(Id),
    FOREIGN KEY (AssetAccountId) REFERENCES ChartOfAccounts(Id),
    
    INDEX IX_Assets_Category (CategoryId),
    INDEX IX_Assets_Company (CompanyId),
    INDEX IX_Assets_Status (Status),
    INDEX IX_Assets_SerialNumber (SerialNumber),
    INDEX IX_Assets_Barcode (Barcode),
    UNIQUE INDEX UX_Assets_Code (TenantId, CompanyId, Code, IsDeleted)
);

-- جدول أصول تحت التشغيل (CIP)
CREATE TABLE AssetsUnderConstruction (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    Code NVARCHAR(50) NOT NULL,
    NameAr NVARCHAR(200) NOT NULL,
    NameEn NVARCHAR(200) NOT NULL,
    Description NVARCHAR(1000),
    
    ProjectId UNIQUEIDENTIFIER,
    CategoryId UNIQUEIDENTIFIER NOT NULL,
    
    CompanyId UNIQUEIDENTIFIER NOT NULL,
    BranchId UNIQUEIDENTIFIER,
    
    StartDate DATE NOT NULL,
    EstimatedCompletionDate DATE,
    ActualCompletionDate DATE,
    
    -- التكاليف المتراكمة
    AccumulatedCost DECIMAL(18,2) DEFAULT 0,
    DirectCosts DECIMAL(18,2) DEFAULT 0,
    IndirectCosts DECIMAL(18,2) DEFAULT 0,
    FinancingCosts DECIMAL(18,2) DEFAULT 0,
    
    CIPAccountId UNIQUEIDENTIFIER,
    
    Status NVARCHAR(50) NOT NULL DEFAULT 'In Progress' CHECK (Status IN ('In Progress', 'Completed', 'Transferred', 'Cancelled')),
    
    -- عند التحويل
    TransferredToAssetId UNIQUEIDENTIFIER,
    TransferredDate DATE,
    
    Notes NVARCHAR(MAX),
    
    -- Audit
    TenantId UNIQUEIDENTIFIER NOT NULL,
    CreatedAt DATETIME2 DEFAULT GETUTCDATE(),
    CreatedBy UNIQUEIDENTIFIER,
    UpdatedAt DATETIME2 DEFAULT GETUTCDATE(),
    UpdatedBy UNIQUEIDENTIFIER,
    IsDeleted BIT DEFAULT 0,
    
    FOREIGN KEY (ProjectId) REFERENCES Projects(Id),
    FOREIGN KEY (CategoryId) REFERENCES AssetCategories(Id),
    FOREIGN KEY (CompanyId) REFERENCES Companies(Id),
    FOREIGN KEY (TransferredToAssetId) REFERENCES Assets(Id),
    
    INDEX IX_CIP_Project (ProjectId),
    INDEX IX_CIP_Status (Status),
    UNIQUE INDEX UX_CIP_Code (TenantId, CompanyId, Code, IsDeleted)
);

-- جدول الإهلاك
CREATE TABLE Depreciations (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    
    AssetId UNIQUEIDENTIFIER NOT NULL,
    
    DepreciationPeriod DATE NOT NULL, -- أول يوم من الشهر
    DepreciationType NVARCHAR(50) NOT NULL CHECK (DepreciationType IN ('Accounting', 'Tax')),
    
    DepreciationMethod NVARCHAR(50) NOT NULL,
    
    -- القيم في بداية الفترة
    OpeningCost DECIMAL(18,2) NOT NULL,
    OpeningAccumulatedDepreciation DECIMAL(18,2) NOT NULL,
    OpeningNetBookValue DECIMAL(18,2) NOT NULL,
    
    -- الإهلاك للفترة
    DepreciationAmount DECIMAL(18,2) NOT NULL,
    
    -- وحدات الإنتاج
    UnitsProduced DECIMAL(18,3),
    
    -- القيم في نهاية الفترة
    ClosingAccumulatedDepreciation DECIMAL(18,2) NOT NULL,
    ClosingNetBookValue DECIMAL(18,2) NOT NULL,
    
    IsFullyDepreciated BIT DEFAULT 0,
    
    -- الترحيل
    Posted BIT DEFAULT 0,
    PostedAt DATETIME2,
    PostedBy UNIQUEIDENTIFIER,
    JournalEntryId UNIQUEIDENTIFIER,
    
    -- العكس
    IsReversed BIT DEFAULT 0,
    ReversedAt DATETIME2,
    ReversedBy UNIQUEIDENTIFIER,
    ReversalReason NVARCHAR(500),
    
    Notes NVARCHAR(500),
    
    -- Audit
    TenantId UNIQUEIDENTIFIER NOT NULL,
    CompanyId UNIQUEIDENTIFIER NOT NULL,
    CreatedAt DATETIME2 DEFAULT GETUTCDATE(),
    CreatedBy UNIQUEIDENTIFIER,
    
    FOREIGN KEY (AssetId) REFERENCES Assets(Id),
    FOREIGN KEY (JournalEntryId) REFERENCES JournalEntries(Id),
    
    INDEX IX_Depreciations_Asset (AssetId),
    INDEX IX_Depreciations_Period (DepreciationPeriod),
    INDEX IX_Depreciations_Posted (Posted),
    UNIQUE INDEX UX_Depreciations (TenantId, AssetId, DepreciationPeriod, DepreciationType, IsReversed)
);

-- جدول دفعات الإهلاك
CREATE TABLE DepreciationBatches (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    BatchNumber NVARCHAR(50) NOT NULL,
    
    DepreciationPeriod DATE NOT NULL,
    DepreciationType NVARCHAR(50) NOT NULL,
    
    TotalAssets INT NOT NULL,
    TotalDepreciation DECIMAL(18,2) NOT NULL,
    
    Status NVARCHAR(50) NOT NULL DEFAULT 'Draft' CHECK (Status IN ('Draft', 'Calculated', 'Posted', 'Reversed')),
    
    CalculatedAt DATETIME2,
    CalculatedBy UNIQUEIDENTIFIER,
    
    Posted BIT DEFAULT 0,
    PostedAt DATETIME2,
    PostedBy UNIQUEIDENTIFIER,
    
    Notes NVARCHAR(1000),
    
    -- Audit
    TenantId UNIQUEIDENTIFIER NOT NULL,
    CompanyId UNIQUEIDENTIFIER NOT NULL,
    CreatedAt DATETIME2 DEFAULT GETUTCDATE(),
    CreatedBy UNIQUEIDENTIFIER,
    
    INDEX IX_DepreciationBatches_Period (DepreciationPeriod),
    UNIQUE INDEX UX_DepreciationBatches (TenantId, CompanyId, BatchNumber)
);

-- جدول الصيانة
CREATE TABLE AssetMaintenances (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    Code NVARCHAR(50) NOT NULL,
    
    AssetId UNIQUEIDENTIFIER NOT NULL,
    
    MaintenanceType NVARCHAR(50) NOT NULL CHECK (MaintenanceType IN ('Preventive', 'Corrective', 'Predictive', 'Emergency')),
    Priority NVARCHAR(50) CHECK (Priority IN ('Low', 'Medium', 'High', 'Critical')),
    
    ScheduledDate DATETIME2,
    ActualStartDate DATETIME2,
    ActualEndDate DATETIME2,
    
    Description NVARCHAR(1000) NOT NULL,
    ProblemDescription NVARCHAR(1000),
    WorkPerformed NVARCHAR(MAX),
    
    -- التعيين
    AssignedTo UNIQUEIDENTIFIER,
    AssignedTeam UNIQUEIDENTIFIER,
    RequestedBy UNIQUEIDENTIFIER,
    ApprovedBy UNIQUEIDENTIFIER,
    
    -- التكاليف
    PartsCost DECIMAL(18,2) DEFAULT 0,
    LaborCost DECIMAL(18,2) DEFAULT 0,
    ExternalServiceCost DECIMAL(18,2) DEFAULT 0,
    TransportationCost DECIMAL(18,2) DEFAULT 0,
    OtherCosts DECIMAL(18,2) DEFAULT 0,
    TotalCost AS (PartsCost + LaborCost + ExternalServiceCost + TransportationCost + OtherCosts) PERSISTED,
    
    -- الرسملة
    IsCapitalized BIT DEFAULT 0,
    CapitalizationReason NVARCHAR(500),
    
    -- الحالة
    Status NVARCHAR(50) NOT NULL DEFAULT 'Draft' CHECK (Status IN ('Draft', 'Scheduled', 'In Progress', 'Completed', 'Cancelled')),
    CompletionPercentage DECIMAL(5,2) DEFAULT 0,
    
    -- الترحيل
    Posted BIT DEFAULT 0,
    PostedAt DATETIME2,
    PostedBy UNIQUEIDENTIFIER,
    JournalEntryId UNIQUEIDENTIFIER,
    
    -- الصيانة المتكررة
    IsRecurring BIT DEFAULT 0,
    Frequency NVARCHAR(50),
    Interval INT,
    NextMaintenanceDate DATE,
    
    Notes NVARCHAR(MAX),
    
    -- Audit
    TenantId UNIQUEIDENTIFIER NOT NULL,
    CompanyId UNIQUEIDENTIFIER NOT NULL,
    CreatedAt DATETIME2 DEFAULT GETUTCDATE(),
    CreatedBy UNIQUEIDENTIFIER,
    UpdatedAt DATETIME2 DEFAULT GETUTCDATE(),
    UpdatedBy UNIQUEIDENTIFIER,
    IsDeleted BIT DEFAULT 0,
    
    FOREIGN KEY (AssetId) REFERENCES Assets(Id),
    FOREIGN KEY (JournalEntryId) REFERENCES JournalEntries(Id),
    
    INDEX IX_Maintenances_Asset (AssetId),
    INDEX IX_Maintenances_ScheduledDate (ScheduledDate),
    INDEX IX_Maintenances_Status (Status),
    UNIQUE INDEX UX_Maintenances_Code (TenantId, CompanyId, Code, IsDeleted)
);

-- جدول قطع غيار الصيانة
CREATE TABLE AssetMaintenanceParts (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    MaintenanceId UNIQUEIDENTIFIER NOT NULL,
    LineNumber INT NOT NULL,
    
    ProductId UNIQUEIDENTIFIER NOT NULL,
    Quantity DECIMAL(18,3) NOT NULL,
    UnitId UNIQUEIDENTIFIER NOT NULL,
    UnitCost DECIMAL(18,2) NOT NULL,
    TotalCost DECIMAL(18,2) NOT NULL,
    
    WarehouseId UNIQUEIDENTIFIER,
    
    Notes NVARCHAR(500),
    
    CreatedAt DATETIME2 DEFAULT GETUTCDATE(),
    
    FOREIGN KEY (MaintenanceId) REFERENCES AssetMaintenances(Id),
    FOREIGN KEY (ProductId) REFERENCES Products(Id),
    FOREIGN KEY (UnitId) REFERENCES Units(Id),
    
    INDEX IX_MaintenanceParts_Maintenance (MaintenanceId)
);

-- جدول أيدي عاملة الصيانة
CREATE TABLE AssetMaintenanceLabor (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    MaintenanceId UNIQUEIDENTIFIER NOT NULL,
    LineNumber INT NOT NULL,
    
    EmployeeId UNIQUEIDENTIFIER NOT NULL,
    Hours DECIMAL(5,2) NOT NULL,
    HourlyRate DECIMAL(18,2) NOT NULL,
    TotalCost DECIMAL(18,2) NOT NULL,
    
    WorkDate DATE,
    
    Notes NVARCHAR(500),
    
    CreatedAt DATETIME2 DEFAULT GETUTCDATE(),
    
    FOREIGN KEY (MaintenanceId) REFERENCES AssetMaintenances(Id),
    FOREIGN KEY (EmployeeId) REFERENCES Employees(Id),
    
    INDEX IX_MaintenanceLabor_Maintenance (MaintenanceId)
);

-- جدول نقل الأصول
CREATE TABLE AssetTransfers (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    Code NVARCHAR(50) NOT NULL,
    
    AssetId UNIQUEIDENTIFIER NOT NULL,
    TransferDate DATE NOT NULL,
    
    -- من
    FromCompanyId UNIQUEIDENTIFIER,
    FromBranchId UNIQUEIDENTIFIER,
    FromLocationId UNIQUEIDENTIFIER,
    FromCustodian UNIQUEIDENTIFIER,
    
    -- إلى
    ToCompanyId UNIQUEIDENTIFIER,
    ToBranchId UNIQUEIDENTIFIER,
    ToLocationId UNIQUEIDENTIFIER,
    ToCustodian UNIQUEIDENTIFIER,
    
    Reason NVARCHAR(500),
    
    -- الحالة
    Status NVARCHAR(50) NOT NULL DEFAULT 'Draft' CHECK (Status IN ('Draft', 'Pending Approval', 'Approved', 'In Transit', 'Completed', 'Cancelled')),
    
    RequestedBy UNIQUEIDENTIFIER,
    ApprovedBy UNIQUEIDENTIFIER,
    ApprovedAt DATETIME2,
    
    CompletedAt DATETIME2,
    CompletedBy UNIQUEIDENTIFIER,
    
    Notes NVARCHAR(1000),
    
    -- Audit
    TenantId UNIQUEIDENTIFIER NOT NULL,
    CreatedAt DATETIME2 DEFAULT GETUTCDATE(),
    CreatedBy UNIQUEIDENTIFIER,
    UpdatedAt DATETIME2 DEFAULT GETUTCDATE(),
    UpdatedBy UNIQUEIDENTIFIER,
    IsDeleted BIT DEFAULT 0,
    
    FOREIGN KEY (AssetId) REFERENCES Assets(Id),
    FOREIGN KEY (FromCompanyId) REFERENCES Companies(Id),
    FOREIGN KEY (ToCompanyId) REFERENCES Companies(Id),
    
    INDEX IX_Transfers_Asset (AssetId),
    INDEX IX_Transfers_Date (TransferDate),
    UNIQUE INDEX UX_Transfers_Code (TenantId, Code, IsDeleted)
);

-- جدول إعادة تقييم الأصول
CREATE TABLE AssetRevaluations (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    Code NVARCHAR(50) NOT NULL,
    
    AssetId UNIQUEIDENTIFIER NOT NULL,
    RevaluationDate DATE NOT NULL,
    
    -- القيم قبل إعادة التقييم
    PreviousCost DECIMAL(18,2) NOT NULL,
    PreviousAccumulatedDepreciation DECIMAL(18,2) NOT NULL,
    PreviousNetBookValue DECIMAL(18,2) NOT NULL,
    
    -- القيمة الجديدة
    RevaluedAmount DECIMAL(18,2) NOT NULL,
    
    -- الفرق
    RevaluationDifference DECIMAL(18,2) NOT NULL,
    IsIncrease BIT NOT NULL,
    
    -- طريقة التقييم
    ValuationMethod NVARCHAR(50),
    Appraiser NVARCHAR(200),
    AppraisalReportNumber NVARCHAR(100),
    
    Reason NVARCHAR(500),
    
    -- الحالة
    Status NVARCHAR(50) NOT NULL DEFAULT 'Draft' CHECK (Status IN ('Draft', 'Pending Approval', 'Approved', 'Posted', 'Cancelled')),
    
    ApprovedBy UNIQUEIDENTIFIER,
    ApprovedAt DATETIME2,
    
    Posted BIT DEFAULT 0,
    PostedAt DATETIME2,
    PostedBy UNIQUEIDENTIFIER,
    JournalEntryId UNIQUEIDENTIFIER,
    
    Notes NVARCHAR(1000),
    
    -- Audit
    TenantId UNIQUEIDENTIFIER NOT NULL,
    CompanyId UNIQUEIDENTIFIER NOT NULL,
    CreatedAt DATETIME2 DEFAULT GETUTCDATE(),
    CreatedBy UNIQUEIDENTIFIER,
    UpdatedAt DATETIME2 DEFAULT GETUTCDATE(),
    UpdatedBy UNIQUEIDENTIFIER,
    IsDeleted BIT DEFAULT 0,
    
    FOREIGN KEY (AssetId) REFERENCES Assets(Id),
    FOREIGN KEY (JournalEntryId) REFERENCES JournalEntries(Id),
    
    INDEX IX_Revaluations_Asset (AssetId),
    INDEX IX_Revaluations_Date (RevaluationDate),
    UNIQUE INDEX UX_Revaluations_Code (TenantId, CompanyId, Code, IsDeleted)
);

-- جدول التخلص من الأصول
CREATE TABLE AssetDisposals (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    Code NVARCHAR(50) NOT NULL,
    
    AssetId UNIQUEIDENTIFIER NOT NULL,
    DisposalDate DATE NOT NULL,
    
    DisposalMethod NVARCHAR(50) NOT NULL CHECK (DisposalMethod IN ('Sale', 'Scrap', 'Donation', 'Exchange', 'Write Off')),
    
    -- القيم عند التخلص
    AssetCost DECIMAL(18,2) NOT NULL,
    AccumulatedDepreciation DECIMAL(18,2) NOT NULL,
    NetBookValue DECIMAL(18,2) NOT NULL,
    
    -- التخلص
    DisposalPrice DECIMAL(18,2) DEFAULT 0,
    DisposalCosts DECIMAL(18,2) DEFAULT 0,
    NetProceeds AS (DisposalPrice - DisposalCosts) PERSISTED,
    
    -- الربح/الخسارة
    GainLoss AS (DisposalPrice - DisposalCosts - NetBookValue) PERSISTED,
    IsGain AS (CASE WHEN (DisposalPrice - DisposalCosts - NetBookValue) > 0 THEN 1 ELSE 0 END) PERSISTED,
    
    -- معلومات المشتري/المتلقي
    BuyerName NVARCHAR(200),
    BuyerTaxNumber NVARCHAR(50),
    SalesInvoiceId UNIQUEIDENTIFIER,
    
    Reason NVARCHAR(500),
    
    -- الحالة
    Status NVARCHAR(50) NOT NULL DEFAULT 'Draft' CHECK (Status IN ('Draft', 'Pending Approval', 'Approved', 'Posted', 'Cancelled')),
    
    ApprovedBy UNIQUEIDENTIFIER,
    ApprovedAt DATETIME2,
    
    Posted BIT DEFAULT 0,
    PostedAt DATETIME2,
    PostedBy UNIQUEIDENTIFIER,
    JournalEntryId UNIQUEIDENTIFIER,
    
    Notes NVARCHAR(1000),
    
    -- Audit
    TenantId UNIQUEIDENTIFIER NOT NULL,
    CompanyId UNIQUEIDENTIFIER NOT NULL,
    CreatedAt DATETIME2 DEFAULT GETUTCDATE(),
    CreatedBy UNIQUEIDENTIFIER,
    UpdatedAt DATETIME2 DEFAULT GETUTCDATE(),
    UpdatedBy UNIQUEIDENTIFIER,
    IsDeleted BIT DEFAULT 0,
    
    FOREIGN KEY (AssetId) REFERENCES Assets(Id),
    FOREIGN KEY (JournalEntryId) REFERENCES JournalEntries(Id),
    
    INDEX IX_Disposals_Asset (AssetId),
    INDEX IX_Disposals_Date (DisposalDate),
    UNIQUE INDEX UX_Disposals_Code (TenantId, CompanyId, Code, IsDeleted)
);

-- جدول المستندات المرفقة
CREATE TABLE AssetDocuments (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    AssetId UNIQUEIDENTIFIER NOT NULL,
    
    DocumentType NVARCHAR(100) NOT NULL,
    FileName NVARCHAR(500) NOT NULL,
    FileUrl NVARCHAR(1000) NOT NULL,
    FileSize BIGINT,
    MimeType NVARCHAR(100),
    
    Category NVARCHAR(100),
    Description NVARCHAR(500),
    
    UploadedAt DATETIME2 DEFAULT GETUTCDATE(),
    UploadedBy UNIQUEIDENTIFIER,
    
    IsDeleted BIT DEFAULT 0,
    
    FOREIGN KEY (AssetId) REFERENCES Assets(Id),
    INDEX IX_AssetDocuments_Asset (AssetId)
);

-- جدول تاريخ الأصول (Audit Trail)
CREATE TABLE AssetHistory (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    AssetId UNIQUEIDENTIFIER NOT NULL,
    
    TransactionDate DATETIME2 NOT NULL DEFAULT GETUTCDATE(),
    TransactionType NVARCHAR(100) NOT NULL,
    
    Description NVARCHAR(1000),
    
    -- القيم قبل وبعد
    OldValue NVARCHAR(MAX),
    NewValue NVARCHAR(MAX),
    
    Amount DECIMAL(18,2),
    
    ReferenceType NVARCHAR(100),
    ReferenceId UNIQUEIDENTIFIER,
    
    PerformedBy UNIQUEIDENTIFIER,
    
    TenantId UNIQUEIDENTIFIER NOT NULL,
    
    FOREIGN KEY (AssetId) REFERENCES Assets(Id),
    INDEX IX_AssetHistory_Asset (AssetId),
    INDEX IX_AssetHistory_Date (TransactionDate)
);

-- جدول جرد الأصول
CREATE TABLE AssetPhysicalCounts (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    Code NVARCHAR(50) NOT NULL,
    
    CountDate DATE NOT NULL,
    CompanyId UNIQUEIDENTIFIER NOT NULL,
    BranchId UNIQUEIDENTIFIER,
    LocationId UNIQUEIDENTIFIER,
    
    Description NVARCHAR(500),
    
    Status NVARCHAR(50) NOT NULL DEFAULT 'Draft' CHECK (Status IN ('Draft', 'In Progress', 'Completed', 'Approved', 'Posted')),
    
    CountedBy UNIQUEIDENTIFIER,
    ApprovedBy UNIQUEIDENTIFIER,
    ApprovedAt DATETIME2,
    
    Notes NVARCHAR(1000),
    
    -- Audit
    TenantId UNIQUEIDENTIFIER NOT NULL,
    CreatedAt DATETIME2 DEFAULT GETUTCDATE(),
    CreatedBy UNIQUEIDENTIFIER,
    UpdatedAt DATETIME2 DEFAULT GETUTCDATE(),
    UpdatedBy UNIQUEIDENTIFIER,
    IsDeleted BIT DEFAULT 0,
    
    INDEX IX_PhysicalCounts_Date (CountDate),
    UNIQUE INDEX UX_PhysicalCounts_Code (TenantId, CompanyId, Code, IsDeleted)
);

-- جدول تفاصيل الجرد
CREATE TABLE AssetPhysicalCountLines (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    PhysicalCountId UNIQUEIDENTIFIER NOT NULL,
    
    AssetId UNIQUEIDENTIFIER NOT NULL,
    
    ExpectedLocation UNIQUEIDENTIFIER,
    ActualLocation UNIQUEIDENTIFIER,
    
    ExpectedCustodian UNIQUEIDENTIFIER,
    ActualCustodian UNIQUEIDENTIFIER,
    
    ExpectedCondition NVARCHAR(50),
    ActualCondition NVARCHAR(50),
    
    Found BIT NOT NULL,
    ConditionChanged BIT DEFAULT 0,
    LocationChanged BIT DEFAULT 0,
    CustodianChanged BIT DEFAULT 0,
    
    Remarks NVARCHAR(500),
    
    CountedAt DATETIME2,
    CountedBy UNIQUEIDENTIFIER,
    
    FOREIGN KEY (PhysicalCountId) REFERENCES AssetPhysicalCounts(Id),
    FOREIGN KEY (AssetId) REFERENCES Assets(Id),
    
    INDEX IX_PhysicalCountLines_Count (PhysicalCountId)
);
```

---

<a name="apis"></a>
## 🔌 8. APIs المطلوبة

### 8.1 Asset APIs

```typescript
// GET /api/v1/assets
GET /api/v1/assets?page=1&pageSize=20&categoryId=&status=Active&search=

// GET /api/v1/assets/{id}
GET /api/v1/assets/550e8400-e29b-41d4-a716-446655440000

// POST /api/v1/assets
POST /api/v1/assets
{
  "code": "AST-BLD-2025-00001",
  "nameAr": "مبنى المكاتب الرئيسي",
  "nameEn": "Main Office Building",
  "categoryId": "guid",
  "acquisitionMethod": "Purchase",
  "acquisitionDate": "2025-10-01",
  "originalCost": 5000000,
  "supplierId": "guid",
  "depreciationMethod": "Straight Line",
  "usefulLife": 25
}

// PUT /api/v1/assets/{id}
PUT /api/v1/assets/550e8400-e29b-41d4-a716-446655440000

// DELETE /api/v1/assets/{id}
DELETE /api/v1/assets/550e8400-e29b-41d4-a716-446655440000

// POST /api/v1/assets/{id}/activate
POST /api/v1/assets/550e8400-e29b-41d4-a716-446655440000/activate

// POST /api/v1/assets/{id}/assign
POST /api/v1/assets/550e8400-e29b-41d4-a716-446655440000/assign
{
  "assignedTo": "employee-guid",
  "assignmentDate": "2025-10-06"
}

// GET /api/v1/assets/{id}/history
GET /api/v1/assets/550e8400-e29b-41d4-a716-446655440000/history

// GET /api/v1/assets/{id}/depreciation-schedule
GET /api/v1/assets/550e8400-e29b-41d4-a716-446655440000/depreciation-schedule
```

### 8.2 Asset Category APIs

```typescript
// GET /api/v1/asset-categories (tree structure)
GET /api/v1/asset-categories?hierarchical=true

// GET /api/v1/asset-categories/{id}
GET /api/v1/asset-categories/{id}

// POST /api/v1/asset-categories
POST /api/v1/asset-categories
{
  "code": "CAT-001",
  "nameAr": "المباني",
  "nameEn": "Buildings",
  "parentCategoryId": null,
  "defaultDepreciationMethod": "Straight Line",
  "defaultUsefulLife": 25,
  "assetAccountId": "guid"
}

// PUT /api/v1/asset-categories/{id}
PUT /api/v1/asset-categories/{id}

// DELETE /api/v1/asset-categories/{id}
DELETE /api/v1/asset-categories/{id}
```

### 8.3 CIP (Construction in Progress) APIs

```typescript
// GET /api/v1/assets-under-construction
GET /api/v1/assets-under-construction?status=In Progress

// GET /api/v1/assets-under-construction/{id}
GET /api/v1/assets-under-construction/{id}

// POST /api/v1/assets-under-construction
POST /api/v1/assets-under-construction
{
  "code": "CIP-2025-001",
  "nameAr": "مشروع مبنى الفرع الجديد",
  "projectId": "guid",
  "categoryId": "guid",
  "startDate": "2025-01-01",
  "estimatedCompletionDate": "2025-12-31"
}

// POST /api/v1/assets-under-construction/{id}/add-cost
POST /api/v1/assets-under-construction/{id}/add-cost
{
  "costType": "Direct" | "Indirect" | "Financing",
  "amount": 50000,
  "description": "Construction materials",
  "referenceId": "invoice-guid"
}

// POST /api/v1/assets-under-construction/{id}/transfer-to-asset
POST /api/v1/assets-under-construction/{id}/transfer-to-asset
{
  "transferDate": "2025-10-01",
  "assetCategoryId": "guid",
  "depreciationMethod": "Straight Line",
  "usefulLife": 25
}

// GET /api/v1/assets-under-construction/{id}/cost-breakdown
GET /api/v1/assets-under-construction/{id}/cost-breakdown
```

### 8.4 Depreciation APIs

```typescript
// POST /api/v1/depreciation/calculate-batch
POST /api/v1/depreciation/calculate-batch
{
  "period": "2025-10-01",
  "depreciationType": "Accounting",
  "categoryIds": ["guid1", "guid2"],
  "branchIds": ["guid1"],
  "includeNewAssets": true
}

Response: {
  "batchId": "guid",
  "totalAssets": 150,
  "totalDepreciation": 125000,
  "assets": [...]
}

// POST /api/v1/depreciation/post-batch
POST /api/v1/depreciation/post-batch/{batchId}

// POST /api/v1/depreciation/reverse
POST /api/v1/depreciation/reverse/{depreciationId}
{
  "reason": "Calculation error"
}

// GET /api/v1/depreciation/history
GET /api/v1/depreciation/history?assetId=guid&from=2025-01-01&to=2025-12-31

// GET /api/v1/depreciation/schedule
GET /api/v1/depreciation/schedule/{assetId}

// POST /api/v1/depreciation/recalculate
POST /api/v1/depreciation/recalculate/{assetId}
{
  "fromDate": "2025-10-01",
  "reason": "Asset revaluation"
}
```

### 8.5 Maintenance APIs

```typescript
// GET /api/v1/asset-maintenance
GET /api/v1/asset-maintenance?assetId=guid&status=Scheduled

// GET /api/v1/asset-maintenance/{id}
GET /api/v1/asset-maintenance/{id}

// POST /api/v1/asset-maintenance
POST /api/v1/asset-maintenance
{
  "assetId": "guid",
  "maintenanceType": "Preventive",
  "priority": "Medium",
  "scheduledDate": "2025-10-15T09:00:00Z",
  "description": "Annual preventive maintenance",
  "assignedTo": "technician-guid"
}

// PUT /api/v1/asset-maintenance/{id}
PUT /api/v1/asset-maintenance/{id}

// POST /api/v1/asset-maintenance/{id}/start
POST /api/v1/asset-maintenance/{id}/start

// POST /api/v1/asset-maintenance/{id}/complete
POST /api/v1/asset-maintenance/{id}/complete
{
  "actualEndDate": "2025-10-15T15:30:00Z",
  "workPerformed": "Replaced oil and filters, checked all systems",
  "parts": [...],
  "labor": [...],
  "isCapitalized": false
}

// POST /api/v1/asset-maintenance/{id}/post
POST /api/v1/asset-maintenance/{id}/post

// GET /api/v1/asset-maintenance/schedule
GET /api/v1/asset-maintenance/schedule?from=2025-10-01&to=2025-10-31

// POST /api/v1/asset-maintenance/schedule-recurring
POST /api/v1/asset-maintenance/schedule-recurring
{
  "assetId": "guid",
  "maintenanceType": "Preventive",
  "frequency": "Monthly",
  "interval": 3,
  "startDate": "2025-10-01",
  "description": "Quarterly maintenance"
}
```

### 8.6 Transfer APIs

```typescript
// POST /api/v1/asset-transfers
POST /api/v1/asset-transfers
{
  "assetId": "guid",
  "transferDate": "2025-10-10",
  "fromLocationId": "guid",
  "toLocationId": "guid",
  "toCustodian": "employee-guid",
  "reason": "Department relocation"
}

// POST /api/v1/asset-transfers/{id}/approve
POST /api/v1/asset-transfers/{id}/approve

// POST /api/v1/asset-transfers/{id}/complete
POST /api/v1/asset-transfers/{id}/complete
{
  "completedDate": "2025-10-10",
  "notes": "Transfer completed successfully"
}

// POST /api/v1/asset-transfers/{id}/cancel
POST /api/v1/asset-transfers/{id}/cancel
```

### 8.7 Revaluation APIs

```typescript
// POST /api/v1/asset-revaluations
POST /api/v1/asset-revaluations
{
  "assetId": "guid",
  "revaluationDate": "2025-10-01",
  "revaluedAmount": 4500000,
  "valuationMethod": "Fair Market Value",
  "appraiser": "ABC Appraisal Company",
  "reason": "Market value increase"
}

// POST /api/v1/asset-revaluations/{id}/approve
POST /api/v1/asset-revaluations/{id}/approve

// POST /api/v1/asset-revaluations/{id}/post
POST /api/v1/asset-revaluations/{id}/post
```

### 8.8 Disposal APIs

```typescript
// POST /api/v1/asset-disposals
POST /api/v1/asset-disposals
{
  "assetId": "guid",
  "disposalDate": "2025-10-15",
  "disposalMethod": "Sale",
  "disposalPrice": 150000,
  "buyerName": "XYZ Company",
  "reason": "Upgrading to new equipment"
}

// POST /api/v1/asset-disposals/{id}/approve
POST /api/v1/asset-disposals/{id}/approve

// POST /api/v1/asset-disposals/{id}/post
POST /api/v1/asset-disposals/{id}/post

// GET /api/v1/asset-disposals/{id}/gain-loss-calculation
GET /api/v1/asset-disposals/{id}/gain-loss-calculation
```

### 8.9 Physical Count APIs

```typescript
// POST /api/v1/asset-physical-counts
POST /api/v1/asset-physical-counts
{
  "countDate": "2025-10-20",
  "branchId": "guid",
  "locationId": "guid",
  "description": "Annual asset verification"
}

// POST /api/v1/asset-physical-counts/{id}/add-assets
POST /api/v1/asset-physical-counts/{id}/add-assets
{
  "assetIds": ["guid1", "guid2", "guid3"]
}

// PUT /api/v1/asset-physical-counts/{id}/lines/{lineId}
PUT /api/v1/asset-physical-counts/{id}/lines/{lineId}
{
  "found": true,
  "actualLocation": "guid",
  "actualCondition": "Good",
  "remarks": "Asset found in good condition"
}

// POST /api/v1/asset-physical-counts/{id}/complete
POST /api/v1/asset-physical-counts/{id}/complete

// POST /api/v1/asset-physical-counts/{id}/approve
POST /api/v1/asset-physical-counts/{id}/approve

// GET /api/v1/asset-physical-counts/{id}/discrepancies
GET /api/v1/asset-physical-counts/{id}/discrepancies
```

---

<a name="examples"></a>
## 📋 9. الأمثلة العملية

### مثال 1: دورة حياة أصل كاملة (شراء → تشغيل → بيع)

#### الخطوة 1: شراء الأصل

```http
POST /api/v1/assets
{
  "code": "AST-VEH-2025-00001",
  "nameAr": "سيارة تويوتا كامري 2025",
  "nameEn": "Toyota Camry 2025",
  "description": "Company vehicle for sales manager",
  "categoryId": "vehicles-category-guid",
  "serialNumber": "VIN123456789",
  "manufacturer": "Toyota",
  "model": "Camry GLX",
  "yearOfManufacture": 2025,
  
  "companyId": "company-guid",
  "branchId": "riyadh-branch-guid",
  "departmentId": "sales-dept-guid",
  
  "acquisitionMethod": "Purchase",
  "acquisitionDate": "2025-10-01",
  "supplierId": "toyota-dealer-guid",
  "purchaseInvoiceId": "invoice-guid",
  
  "originalCost": 120000,
  "additionalCosts": 5000,
  
  "depreciable": true,
  "depreciationMethod": "Declining Balance",
  "usefulLife": 5,
  "depreciationRate": 40,
  "salvageValue": 20000,
  "depreciationStartDate": "2025-11-01",
  
  "assetAccountId": "1310-vehicles-guid",
  "depreciationAccountId": "5210-depreciation-expense-guid",
  "accumulatedDepreciationAccountId": "1311-accumulated-dep-guid",
  "costCenterId": "sales-cc-guid",
  
  "warrantyStartDate": "2025-10-01",
  "warrantyEndDate": "2028-10-01",
  "insurancePolicyNumber": "INS-2025-456",
  "insuranceExpiry": "2026-10-01",
  
  "status": "Draft"
}

Response: 201 Created
{
  "id": "asset-001",
  "code": "AST-VEH-2025-00001",
  "totalCost": 125000,
  "status": "Draft"
}
```

#### الخطوة 2: تفعيل الأصل

```http
POST /api/v1/assets/asset-001/activate

Response: 200 OK
{
  "status": "Active",
  "activatedAt": "2025-10-01T10:00:00Z",
  "depreciationStartDate": "2025-11-01",
  "accountingEntry": {
    "description": "Asset Acquisition - Toyota Camry",
    "entries": [
      {
        "account": "1310 - Vehicles",
        "debit": 125000
      },
      {
        "account": "2110 - Accounts Payable",
        "credit": 125000
      }
    ]
  }
}
```

#### الخطوة 3: تعيين الأصل لموظف

```http
POST /api/v1/assets/asset-001/assign
{
  "assignedTo": "employee-ahmed-guid",
  "custodian": "employee-ahmed-guid",
  "assignmentDate": "2025-10-01"
}

Response: 200 OK
{
  "assigned": true,
  "assignedTo": {
    "id": "employee-ahmed-guid",
    "name": "Ahmed Mohammed"
  },
  "custodyDocument": {
    "documentId": "custody-doc-001",
    "url": "/documents/custody-forms/asset-001.pdf",
    "requiresSignature": true
  }
}
```

#### الخطوة 4: الإهلاك الشهري (تلقائي)

```http
// يتم تشغيله في نهاية كل شهر تلقائياً
POST /api/v1/depreciation/calculate-batch
{
  "period": "2025-11-01",
  "depreciationType": "Accounting",
  "categoryIds": ["vehicles-category-guid"]
}

Response: 200 OK
{
  "batchId": "dep-batch-202511",
  "totalAssets": 1,
  "totalDepreciation": 4166.67,
  "assets": [
    {
      "assetId": "asset-001",
      "assetCode": "AST-VEH-2025-00001",
      "method": "Declining Balance",
      "openingCost": 125000,
      "openingAccumulatedDepreciation": 0,
      "openingNetBookValue": 125000,
      "depreciationAmount": 4166.67,  // (125000 * 40%) / 12
      "closingAccumulatedDepreciation": 4166.67,
      "closingNetBookValue": 120833.33
    }
  ]
}

// ترحيل الإهلاك
POST /api/v1/depreciation/post-batch/dep-batch-202511

Response: 200 OK
{
  "posted": true,
  "journalEntry": {
    "description": "Depreciation for November 2025",
    "entries": [
      {
        "account": "5210 - Depreciation Expense",
        "debit": 4166.67,
        "costCenter": "sales-cc-guid"
      },
      {
        "account": "1311 - Accumulated Depreciation - Vehicles",
        "credit": 4166.67
      }
    ]
  }
}
```

#### الخطوة 5: صيانة دورية

```http
POST /api/v1/asset-maintenance
{
  "assetId": "asset-001",
  "maintenanceType": "Preventive",
  "priority": "Medium",
  "scheduledDate": "2025-12-15T09:00:00Z",
  "description": "10,000 KM service - oil change and inspection",
  "assignedTo": "technician-guid"
}

Response: 201 Created
{
  "id": "maintenance-001",
  "code": "MNT-2025-001",
  "status": "Scheduled"
}

// إنجاز الصيانة
POST /api/v1/asset-maintenance/maintenance-001/complete
{
  "actualEndDate": "2025-12-15T11:30:00Z",
  "workPerformed": "- Oil and filter change\n- Tire rotation\n- Brake inspection\n- All systems checked",
  "parts": [
    {
      "productId": "oil-filter-guid",
      "quantity": 1,
      "unitCost": 50,
      "totalCost": 50
    },
    {
      "productId": "engine-oil-guid",
      "quantity": 5,
      "unitCost": 30,
      "totalCost": 150
    }
  ],
  "labor": [
    {
      "employeeId": "technician-guid",
      "hours": 2,
      "hourlyRate": 100,
      "totalCost": 200
    }
  ],
  "externalServiceCost": 0,
  "otherCosts": 50,
  "isCapitalized": false
}

Response: 200 OK
{
  "status": "Completed",
  "totalCost": 450,
  "breakdown": {
    "parts": 200,
    "labor": 200,
    "other": 50
  }
}

// ترحيل الصيانة
POST /api/v1/asset-maintenance/maintenance-001/post

Response: 200 OK
{
  "posted": true,
  "journalEntry": {
    "description": "Vehicle Maintenance - AST-VEH-2025-00001",
    "entries": [
      {
        "account": "5310 - Maintenance Expense",
        "debit": 450,
        "costCenter": "sales-cc-guid"
      },
      {
        "account": "1410 - Inventory",
        "credit": 200
      },
      {
        "account": "1010 - Cash",
        "credit": 250
      }
    ]
  }
}
```

#### الخطوة 6: بيع الأصل بعد 3 سنوات

```http
// أولاً: حساب الإهلاك المتراكم بعد 3 سنوات
GET /api/v1/assets/asset-001

Response:
{
  "originalCost": 125000,
  "accumulatedDepreciation": 76800,  // بعد 3 سنوات
  "netBookValue": 48200,
  "currentCondition": "Good"
}

// إنشاء عملية البيع
POST /api/v1/asset-disposals
{
  "assetId": "asset-001",
  "disposalDate": "2028-10-01",
  "disposalMethod": "Sale",
  "disposalPrice": 55000,
  "disposalCosts": 1000,
  "buyerName": "Mohammed Ali",
  "buyerTaxNumber": "300123456789",
  "reason": "Upgrading fleet"
}

Response: 201 Created
{
  "id": "disposal-001",
  "code": "DSP-2028-001",
  "assetCost": 125000,
  "accumulatedDepreciation": 76800,
  "netBookValue": 48200,
  "disposalPrice": 55000,
  "disposalCosts": 1000,
  "netProceeds": 54000,
  "gainLoss": 5800,  // 54000 - 48200
  "isGain": true,
  "status": "Draft"
}

// الموافقة
POST /api/v1/asset-disposals/disposal-001/approve

// الترحيل
POST /api/v1/asset-disposals/disposal-001/post

Response: 200 OK
{
  "posted": true,
  "journalEntry": {
    "description": "Asset Disposal - Sale of Toyota Camry",
    "entries": [
      {
        "account": "1010 - Cash",
        "debit": 54000
      },
      {
        "account": "1311 - Accumulated Depreciation - Vehicles",
        "debit": 76800
      },
      {
        "account": "1310 - Vehicles",
        "credit": 125000
      },
      {
        "account": "4510 - Gain on Asset Sale",
        "credit": 5800
      }
    ]
  },
  "assetStatus": "Sold"
}
```

---

### مثال 2: تحويل من أصول تحت التشغيل (CIP)

#### الخطوة 1: إنشاء مشروع CIP

```http
POST /api/v1/assets-under-construction
{
  "code": "CIP-2025-001",
  "nameAr": "مشروع مبنى الفرع الجديد - جدة",
  "nameEn": "New Branch Building Project - Jeddah",
  "description": "Construction of new 5-story office building",
  "projectId": "project-jeddah-building-guid",
  "categoryId": "buildings-category-guid",
  "companyId": "company-guid",
  "branchId": "jeddah-branch-guid",
  "startDate": "2025-01-01",
  "estimatedCompletionDate": "2026-06-30",
  "cipAccountId": "1320-cip-buildings-guid"
}

Response: 201 Created
{
  "id": "cip-001",
  "code": "CIP-2025-001",
  "status": "In Progress",
  "accumulatedCost": 0
}
```

#### الخطوة 2: إضافة تكاليف للمشروع (على مدار السنة)

```http
// تكلفة مباشرة - مواد بناء
POST /api/v1/assets-under-construction/cip-001/add-cost
{
  "costType": "Direct",
  "amount": 500000,
  "description": "Construction materials - Phase 1",
  "referenceType": "PurchaseInvoice",
  "referenceId": "invoice-001-guid"
}

// تكلفة مباشرة - عمالة
POST /api/v1/assets-under-construction/cip-001/add-cost
{
  "costType": "Direct",
  "amount": 300000,
  "description": "Labor costs - Q1 2025",
  "referenceType": "PayrollBatch",
  "referenceId": "payroll-q1-guid"
}

// تكلفة غير مباشرة - إشراف هندسي
POST /api/v1/assets-under-construction/cip-001/add-cost
{
  "costType": "Indirect",
  "amount": 50000,
  "description": "Engineering supervision",
  "referenceType": "Expense",
  "referenceId": "expense-001-guid"
}

// تكلفة تمويل - فوائد القرض
POST /api/v1/assets-under-construction/cip-001/add-cost
{
  "costType": "Financing",
  "amount": 25000,
  "description": "Interest on construction loan - Q1",
  "referenceType": "JournalEntry",
  "referenceId": "je-interest-guid"
}

// الحصول على ملخص التكاليف
GET /api/v1/assets-under-construction/cip-001/cost-breakdown

Response:
{
  "cipId": "cip-001",
  "code": "CIP-2025-001",
  "status": "In Progress",
  "costBreakdown": {
    "directCosts": 3500000,
    "indirectCosts": 350000,
    "financingCosts": 150000,
    "totalAccumulatedCost": 4000000
  },
  "timeline": {
    "startDate": "2025-01-01",
    "estimatedCompletion": "2026-06-30",
    "progressPercentage": 75
  }
}
```

#### الخطوة 3: اكتمال المشروع والتحويل لأصل ثابت

```http
POST /api/v1/assets-under-construction/cip-001/transfer-to-asset
{
  "transferDate": "2026-06-30",
  "actualCompletionDate": "2026-06-30",
  "assetCategoryId": "buildings-category-guid",
  "depreciationMethod": "Straight Line",
  "usefulLife": 25,
  "salvageValue": 400000,
  "depreciationStartDate": "2026-07-01"
}

Response: 200 OK
{
  "cipStatus": "Transferred",
  "newAsset": {
    "id": "asset-building-001",
    "code": "AST-BLD-2026-00001",
    "nameAr": "مبنى الفرع الجديد - جدة",
    "totalCost": 4000000,
    "categoryId": "buildings-category-guid",
    "status": "Active",
    "depreciationStartDate": "2026-07-01",
    "annualDepreciation": 144000,  // (4000000 - 400000) / 25
    "monthlyDepreciation": 12000
  },
  "journalEntry": {
    "description": "Transfer from CIP to Fixed Asset - Jeddah Building",
    "entries": [
      {
        "account": "1300 - Buildings",
        "debit": 4000000
      },
      {
        "account": "1320 - CIP - Buildings",
        "credit": 4000000
      }
    ]
  }
}
```

---

### مثال 3: إعادة تقييم أصل

```http
POST /api/v1/asset-revaluations
{
  "assetId": "asset-building-001",
  "revaluationDate": "2027-07-01",
  "revaluedAmount": 4500000,
  "valuationMethod": "Fair Market Value",
  "appraiser": "Professional Valuation Company",
  "appraisalReportNumber": "VAL-2027-789",
  "reason": "Significant increase in real estate market value"
}

Response: 201 Created
{
  "id": "revaluation-001",
  "code": "RVL-2027-001",
  "previousCost": 4000000,
  "previousAccumulatedDepreciation": 144000,  // سنة واحدة
  "previousNetBookValue": 3856000,
  "revaluedAmount": 4500000,
  "revaluationDifference": 644000,  // 4500000 - 3856000
  "isIncrease": true,
  "status": "Draft"
}

// الموافقة والترحيل
POST /api/v1/asset-revaluations/revaluation-001/approve
POST /api/v1/asset-revaluations/revaluation-001/post

Response: 200 OK
{
  "posted": true,
  "journalEntry": {
    "description": "Asset Revaluation - Jeddah Building",
    "entries": [
      {
        "account": "1300 - Buildings",
        "debit": 644000
      },
      {
        "account": "3510 - Revaluation Surplus (OCI)",
        "credit": 644000
      }
    ]
  },
  "depreciationImpact": {
    "newBasis": 4500000,
    "salvageValue": 400000,
    "remainingLife": 24,  // years
    "newAnnualDepreciation": 170833.33,  // (4500000 - 400000) / 24
    "previousAnnualDepreciation": 144000,
    "increase": 26833.33
  }
}
```

---

### مثال 4: صيانة رأسمالية (تضاف للأصل)

```http
POST /api/v1/asset-maintenance
{
  "assetId": "asset-building-001",
  "maintenanceType": "Corrective",
  "priority": "High",
  "scheduledDate": "2028-03-01T08:00:00Z",
  "description": "Major HVAC system upgrade and roof replacement",
  "assignedTeam": "maintenance-team-guid"
}

Response: 201 Created
{
  "id": "maintenance-major-001",
  "code": "MNT-2028-045",
  "status": "Draft"
}

// إتمام الصيانة
POST /api/v1/asset-maintenance/maintenance-major-001/complete
{
  "actualEndDate": "2028-03-15T17:00:00Z",
  "workPerformed": "Complete HVAC system replacement with high-efficiency units. Full roof membrane replacement.",
  "parts": [
    {
      "productId": "hvac-unit-guid",
      "quantity": 10,
      "unitCost": 15000,
      "totalCost": 150000
    },
    {
      "productId": "roofing-material-guid",
      "quantity": 1,
      "unitCost": 80000,
      "totalCost": 80000
    }
  ],
  "labor": [
    {
      "employeeId": "team-lead-guid",
      "hours": 120,
      "hourlyRate": 150,
      "totalCost": 18000
    }
  ],
  "externalServiceCost": 50000,
  "otherCosts": 12000,
  "isCapitalized": true,
  "capitalizationReason": "This upgrade significantly extends the building's useful life and improves energy efficiency"
}

Response: 200 OK
{
  "status": "Completed",
  "totalCost": 310000,
  "isCapitalized": true
}

// الترحيل
POST /api/v1/asset-maintenance/maintenance-major-001/post

Response: 200 OK
{
  "posted": true,
  "journalEntry": {
    "description": "Capitalized Maintenance - Building Upgrade",
    "entries": [
      {
        "account": "1300 - Buildings",
        "debit": 310000
      },
      {
        "account": "1410 - Inventory",
        "credit": 230000
      },
      {
        "account": "1010 - Cash",
        "credit": 80000
      }
    ]
  },
  "assetUpdated": {
    "previousCost": 4500000,
    "capitalizedAmount": 310000,
    "newCost": 4810000,
    "depreciationRecalculated": true,
    "newAnnualDepreciation": 184583.33
  }
}
```

---

### مثال 5: جرد الأصول

```http
// إنشاء عملية جرد
POST /api/v1/asset-physical-counts
{
  "countDate": "2026-12-31",
  "companyId": "company-guid",
  "branchId": "riyadh-branch-guid",
  "description": "Annual physical verification of all assets"
}

Response: 201 Created
{
  "id": "count-2026-001",
  "code": "CNT-2026-001",
  "status": "Draft"
}

// إضافة الأصول المراد جردها
POST /api/v1/asset-physical-counts/count-2026-001/add-assets
{
  "assetIds": [
    "asset-001",
    "asset-002",
    "asset-003",
    // ... جميع الأصول في الفرع
  ]
}

// البدء في الجرد
POST /api/v1/asset-physical-counts/count-2026-001/start

// تحديث حالة كل أصل
PUT /api/v1/asset-physical-counts/count-2026-001/lines/line-001
{
  "found": true,
  "actualLocation": "expected-location-guid",
  "actualCustodian": "expected-custodian-guid",
  "actualCondition": "Good",
  "remarks": "Asset found in expected location and good condition"
}

PUT /api/v1/asset-physical-counts/count-2026-001/lines/line-002
{
  "found": true,
  "actualLocation": "different-location-guid",
  "actualCustodian": "expected-custodian-guid",
  "actualCondition": "Fair",
  "conditionChanged": true,
  "locationChanged": true,
  "remarks": "Asset found in different location. Condition downgraded to Fair due to visible wear."
}

PUT /api/v1/asset-physical-counts/count-2026-001/lines/line-003
{
  "found": false,
  "remarks": "Asset not found during physical count. Further investigation required."
}

// الحصول على المعلومات
GET /api/v1/asset-physical-counts/count-2026-001/discrepancies

Response:
{
  "totalAssets": 150,
  "assetsFound": 148,
  "assetsNotFound": 2,
  "locationDiscrepancies": 5,
  "conditionDiscrepancies": 8,
  "custodianDiscrepancies": 3,
  "discrepancies": [
    {
      "assetCode": "AST-VEH-2025-00003",
      "issue": "Not Found",
      "severity": "High"
    },
    {
      "assetCode": "AST-FURN-2024-00045",
      "issue": "Location Changed",
      "expectedLocation": "Office 201",
      "actualLocation": "Office 305",
      "severity": "Medium"
    }
  ]
}

// إكمال الجرد
POST /api/v1/asset-physical-counts/count-2026-001/complete

// الموافقة
POST /api/v1/asset-physical-counts/count-2026-001/approve
```

---

## 📊 10. التقارير الأساسية

### 10.1 تقارير الأصول

1. **Asset Register**
   - سجل شامل لجميع الأصول
   - حسب الفئة/الموقع/الحالة

2. **Asset List by Category**
   - تصنيف الأصول حسب الفئة
   - مع القيم والإهلاك

3. **Asset Location Report**
   - الأصول حسب الموقع
   - مع المسؤول

4. **Asset Custody Report**
   - الأصول المعينة للموظفين
   - عهد الموظفين

5. **Asset Acquisition Report**
   - الأصول المقتناة خلال فترة
   - حسب طريقة الاقتناء

6. **Assets Near End of Life**
   - الأصول القريبة من نهاية العمر الإنتاجي

7. **Asset Condition Report**
   - حالة الأصول
   - الأصول التي تحتاج صيانة

### 10.2 تقارير الإهلاك

1. **Depreciation Schedule**
   - جدول الإهلاك لكل أصل
   - الإهلاك المتوقع

2. **Depreciation Summary**
   - ملخص الإهلاك الشهري/السنوي
   - حسب الفئة

3. **Fully Depreciated Assets**
   - الأصول المستهلكة بالكامل

4. **Depreciation Expense Report**
   - مصروف الإهلاك حسب مركز التكلفة

5. **Book vs. Tax Depreciation**
   - مقارنة الإهلاك المحاسبي والضريبي

### 10.3 تقارير الصيانة

1. **Maintenance History**
   - سجل الصيانة لكل أصل

2. **Maintenance Cost Analysis**
   - تحليل تكاليف الصيانة

3. **Scheduled Maintenance**
   - الصيانة المجدولة القادمة

4. **Overdue Maintenance**
   - الصيانة المتأخرة

5. **Maintenance by Asset Category**
   - الصيانة حسب نوع الأصل

### 10.4 التقارير المالية

1. **Fixed Assets Summary**
   - ملخص الأصول الثابتة
   - التكلفة والإهلاك والقيمة الدفترية

2. **Asset Movement Report**
   - حركة الأصول خلال فترة
   - (إضافات، تخلص، تحويلات)

3. **Gain/Loss on Disposal**
   - الأرباح والخسائر من بيع الأصول

4. **Asset Revaluation Report**
   - إعادة التقييمات
   - التأثير على القوائم المالية

5. **CIP Status Report**
   - حالة الأصول تحت التشغيل
   - التكاليف المتراكمة

### 10.5 تقارير الجرد

1. **Physical Count Summary**
   - ملخص عمليات الجرد

2. **Asset Discrepancies**
   - الفروقات المكتشفة

3. **Missing Assets Report**
   - الأصول المفقودة

---

## 🎯 الخلاصة والنقاط المهمة

### العمليات الأساسية
1. ✅ إدارة دورة حياة الأصل الكاملة
2. ✅ نظام إهلاك شامل (4 طرق)
3. ✅ الإهلاك المحاسبي والضريبي المنفصل
4. ✅ تحويل CIP لأصول ثابتة
5. ✅ إدارة الصيانة (وقائية وتصحيحية)
6. ✅ الرسملة الذكية للصيانة
7. ✅ إعادة التقييم
8. ✅ البيع والتخلص

### التكامل
- 🔄 تكامل مع المشتريات (اقتناء الأصول)
- 🔄 تكامل مع الحسابات (قيود آلية)
- 🔄 تكامل مع المخزون (قطع الغيار)
- 🔄 تكامل مع HR (تعيين الأصول)
- 🔄 تكامل مع المشاريع (CIP)

### الميزات المتقدمة
- 🎯 4 طرق للإهلاك
- 🎯 المطابقة الثلاثية للصيانة الرأسمالية
- 🎯 التتبع بالباركود/RFID
- 🎯 الجرد الفعلي
- 🎯 سجل تاريخي كامل
- 🎯 Multi-Location & Multi-Company

### خدمات مستقلة (Services)
- ⚙️ **Depreciation Service**: حساب وترحيل الإهلاك تلقائياً
- ⚙️ **CIP Transfer Service**: تحويل الأصول من CIP
- ⚙️ **Maintenance Scheduler**: جدولة الصيانة الدورية
- ⚙️ **Asset Tracking Service**: تتبع الأصول والموقع

---

**تم إعداد هذا المستند بواسطة:** Claude Sonnet 4.5  
**التاريخ:** 2025-10-06  
**النسخة:** 1.0  
**الحالة:** ✅ مكتمل