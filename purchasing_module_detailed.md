# 🛒 مديول المشتريات - Purchasing Module
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
مديول المشتريات مسؤول عن إدارة دورة الشراء الكاملة من لحظة تحديد الحاجة للشراء حتى استلام البضاعة ودفع المستحقات، ويشمل:
- إدارة الموردين
- طلبات الشراء (Purchase Requisitions)
- عروض أسعار الموردين (RFQ - Request for Quotation)
- مقارنة العروض واختيار الأفضل
- أوامر الشراء (Purchase Orders)
- استلام البضاعة (GRN - Goods Received Note)
- فواتير الشراء
- المرتجعات للموردين
- المشتريات الخارجية والاستيراد
- إدارة العقود مع الموردين

### النطاق (Scope)
- ✅ مشتريات محلية (Local Purchases)
- ✅ مشتريات خارجية/استيراد (Import)
- ✅ مشتريات خدمات (Services)
- ✅ مشتريات أصول ثابتة
- ✅ Multi-Currency & Multi-Language
- ✅ Multi-Branch & Multi-Company
- ✅ Vendor Portal للموردين

### الأهداف الاستراتيجية
1. **التحكم في التكاليف**: تقليل تكلفة المشتريات
2. **الجودة**: ضمان جودة المواد المشتراة
3. **التوقيت**: استلام المواد في الوقت المناسب
4. **الشفافية**: توثيق كل خطوة في عملية الشراء
5. **الامتثال**: الالتزام بسياسات وإجراءات الشركة

---

<a name="entities"></a>
## 🗂️ 2. الكيانات الأساسية (Core Entities)

### 2.1 الموردين (Suppliers)
```typescript
interface Supplier {
  id: string;
  code: string; // مثل: SUP-001
  
  // معلومات أساسية
  nameAr: string;
  nameEn: string;
  type: 'Individual' | 'Company';
  category: 'LocalSupplier' | 'ImportSupplier' | 'ServiceProvider';
  
  // معلومات الاتصال
  contactInfo: {
    phone: string;
    mobile: string;
    email: string;
    website: string;
    fax?: string;
  };
  
  // العنوان
  address: {
    addressAr: string;
    addressEn: string;
    city: string;
    country: string;
    postalCode: string;
  };
  
  // المعلومات المالية
  financialInfo: {
    taxNumber: string;
    commercialRegister: string;
    paymentTermId: string; // ربط بشروط الدفع
    defaultCurrency: string;
    creditLimit: number;
    currentBalance: number; // المديونية الحالية
  };
  
  // بيانات البنك
  bankAccounts: Array<{
    bankName: string;
    accountNumber: string;
    iban: string;
    swiftCode: string;
    currency: string;
    isDefault: boolean;
  }>;
  
  // التقييم
  rating: {
    qualityScore: number; // 1-5
    deliveryScore: number; // 1-5
    priceScore: number; // 1-5
    overallScore: number; // حساب تلقائي
    lastEvaluationDate: Date;
    evaluatedBy: string;
    notes: string;
  };
  
  // المستندات
  documents: Array<{
    documentType: string; // تسجيل تجاري، بطاقة ضريبية، عقود...
    documentNumber: string;
    issueDate: Date;
    expiryDate?: Date;
    attachmentUrl: string;
  }>;
  
  // جهات الاتصال
  contacts: Array<{
    name: string;
    position: string;
    phone: string;
    email: string;
    isPrimary: boolean;
  }>;
  
  // الحالة والتتبع
  status: 'Active' | 'Inactive' | 'Blocked';
  blockReason?: string;
  isApproved: boolean;
  approvedBy?: string;
  approvedDate?: Date;
  
  // Audit
  createdAt: Date;
  createdBy: string;
  updatedAt: Date;
  updatedBy: string;
  isDeleted: boolean;
}
```

### 2.2 طلب الشراء (Purchase Requisition)
```typescript
interface PurchaseRequisition {
  id: string;
  code: string; // مثل: PR-2025-0001
  
  // معلومات أساسية
  date: Date;
  requiredDate: Date; // التاريخ المطلوب للاستلام
  requestedBy: string; // الموظف الطالب
  department: string;
  priority: 'Low' | 'Medium' | 'High' | 'Urgent';
  
  // الغرض
  purpose: 'Stock' | 'Project' | 'FixedAsset' | 'Maintenance' | 'Service';
  purposeDescription: string;
  projectId?: string; // إذا كان للمشروع
  
  // الأصناف المطلوبة
  items: Array<{
    lineNumber: number;
    productId?: string; // قد يكون صنف جديد
    description: string;
    specifications: string;
    quantity: number;
    unitId: string;
    estimatedPrice: number;
    estimatedTotal: number;
    requiredDate: Date;
    notes: string;
    
    // للربط مع أوامر الشراء
    orderedQuantity: number;
    remainingQuantity: number;
    linkedPurchaseOrders: Array<{
      poId: string;
      poNumber: string;
      quantity: number;
    }>;
  }>;
  
  // الإجماليات
  totals: {
    subtotal: number;
    estimatedTotal: number;
  };
  
  // الموافقات (Approval Workflow)
  approvals: Array<{
    level: number;
    approverRole: string;
    approverId?: string;
    approverName?: string;
    status: 'Pending' | 'Approved' | 'Rejected';
    date?: Date;
    notes?: string;
  }>;
  
  // الحالة
  status: 'Draft' | 'PendingApproval' | 'Approved' | 'PartiallyOrdered' | 'FullyOrdered' | 'Rejected' | 'Cancelled';
  
  // التتبع
  createdAt: Date;
  createdBy: string;
  updatedAt: Date;
  updatedBy: string;
  isDeleted: boolean;
}
```

### 2.3 طلب عرض السعر (RFQ - Request for Quotation)
```typescript
interface RequestForQuotation {
  id: string;
  code: string; // مثل: RFQ-2025-0001
  
  // معلومات أساسية
  date: Date;
  dueDate: Date; // آخر موعد لاستلام العروض
  validUntil: Date; // صلاحية العروض
  
  // المصدر
  sourceType: 'PurchaseRequisition' | 'Manual';
  sourceId?: string; // رقم طلب الشراء
  
  // الموردين المستهدفين
  suppliers: Array<{
    supplierId: string;
    supplierName: string;
    contactPerson: string;
    email: string;
    phone: string;
    sentDate?: Date;
    receivedDate?: Date;
    status: 'Pending' | 'Received' | 'NotReceived' | 'Declined';
  }>;
  
  // الأصناف المطلوب عرض أسعار لها
  items: Array<{
    lineNumber: number;
    productId?: string;
    description: string;
    specifications: string;
    quantity: number;
    unitId: string;
    requiredDate: Date;
    notes: string;
  }>;
  
  // الشروط والملاحظات
  terms: {
    paymentTerms: string;
    deliveryTerms: string;
    warranty: string;
    validityPeriod: number; // days
    otherTerms: string;
  };
  
  // المرفقات
  attachments: Array<{
    fileName: string;
    fileUrl: string;
    uploadedAt: Date;
  }>;
  
  // الحالة
  status: 'Draft' | 'Sent' | 'PartiallyReceived' | 'FullyReceived' | 'Evaluated' | 'Closed' | 'Cancelled';
  
  // التتبع
  createdAt: Date;
  createdBy: string;
  updatedAt: Date;
  updatedBy: string;
  isDeleted: boolean;
}
```

### 2.4 عرض سعر المورد (Supplier Quotation)
```typescript
interface SupplierQuotation {
  id: string;
  code: string; // مثل: SQ-2025-0001
  
  // الربط مع RFQ
  rfqId: string;
  rfqCode: string;
  
  // معلومات المورد
  supplierId: string;
  supplierName: string;
  supplierQuotationNumber: string; // رقم العرض عند المورد
  
  // التواريخ
  date: Date;
  receivedDate: Date;
  validUntil: Date;
  
  // الأصناف
  items: Array<{
    lineNumber: number;
    rfqLineNumber: number; // ربط بسطر RFQ
    productId?: string;
    description: string;
    
    // السعر
    quantity: number;
    unitId: string;
    unitPrice: number;
    discount: number;
    discountAmount: number;
    taxRate: number;
    taxAmount: number;
    totalAmount: number;
    
    // شروط إضافية
    deliveryDays: number; // مدة التوريد
    warranty: string;
    origin: string; // بلد المنشأ
    brand: string;
    notes: string;
    
    // التقييم
    score?: number;
    isSelected: boolean;
  }>;
  
  // الإجماليات
  totals: {
    subtotal: number;
    discountAmount: number;
    taxableAmount: number;
    taxAmount: number;
    total: number;
    currency: string;
  };
  
  // الشروط
  terms: {
    paymentTerms: string;
    deliveryTerms: string;
    warranty: string;
    validity: number; // days
    otherTerms: string;
  };
  
  // التكاليف الإضافية
  additionalCosts: Array<{
    description: string;
    amount: number;
  }>;
  
  // المرفقات
  attachments: Array<{
    fileName: string;
    fileUrl: string;
  }>;
  
  // التقييم
  evaluation: {
    priceScore: number; // 1-5
    qualityScore: number; // 1-5
    deliveryScore: number; // 1-5
    termsScore: number; // 1-5
    totalScore: number; // حساب تلقائي
    evaluatedBy?: string;
    evaluatedDate?: Date;
    notes?: string;
  };
  
  // الحالة
  status: 'Draft' | 'Received' | 'UnderEvaluation' | 'Selected' | 'Rejected' | 'Expired';
  rejectionReason?: string;
  
  // التتبع
  createdAt: Date;
  createdBy: string;
  updatedAt: Date;
  updatedBy: string;
  isDeleted: boolean;
}
```

### 2.5 أمر الشراء (Purchase Order)
```typescript
interface PurchaseOrder {
  id: string;
  code: string; // مثل: PO-2025-0001
  
  // المصدر
  sourceType: 'SupplierQuotation' | 'PurchaseRequisition' | 'Manual';
  sourceId?: string;
  requisitionIds?: string[]; // قد يكون من أكثر من PR
  quotationId?: string;
  
  // معلومات أساسية
  date: Date;
  deliveryDate: Date; // التاريخ المتوقع للاستلام
  
  // المورد
  supplierId: string;
  supplierName: string;
  supplierAddress: string;
  supplierPhone: string;
  
  // التوصيل
  deliveryAddress: {
    warehouseId: string;
    warehouseName: string;
    fullAddress: string;
    contactPerson: string;
    contactPhone: string;
  };
  
  // الأصناف
  items: Array<{
    lineNumber: number;
    productId?: string; // قد يكون صنف جديد
    description: string;
    specifications: string;
    
    // الكميات
    orderedQuantity: number;
    receivedQuantity: number;
    remainingQuantity: number;
    unitId: string;
    
    // الأسعار
    unitPrice: number;
    discount: number;
    discountAmount: number;
    taxRate: number;
    taxAmount: number;
    totalAmount: number;
    
    // التتبع
    requiredDate: Date;
    notes: string;
    
    // الربط مع الاستلام
    linkedGRNs: Array<{
      grnId: string;
      grnNumber: string;
      quantity: number;
      date: Date;
    }>;
    
    // الربط مع الفواتير
    invoicedQuantity: number;
    linkedInvoices: Array<{
      invoiceId: string;
      invoiceNumber: string;
      quantity: number;
      date: Date;
    }>;
  }>;
  
  // الإجماليات
  totals: {
    subtotal: number;
    discountAmount: number;
    taxableAmount: number;
    taxAmount: number;
    total: number;
    currency: string;
    exchangeRate?: number; // للعملات الأجنبية
  };
  
  // التكاليف الإضافية (Shipping, Customs, etc)
  additionalCosts: Array<{
    costType: 'Shipping' | 'Insurance' | 'Customs' | 'Other';
    description: string;
    amount: number;
    accountId?: string; // الحساب المحاسبي
  }>;
  
  // الشروط
  terms: {
    paymentTermId: string;
    paymentTermName: string;
    deliveryTerms: string;
    warranty: string;
    penalty: string; // شرط جزائي
    otherTerms: string;
  };
  
  // الموافقات
  approvals: Array<{
    level: number;
    approverRole: string;
    approverId?: string;
    approverName?: string;
    status: 'Pending' | 'Approved' | 'Rejected';
    date?: Date;
    notes?: string;
  }>;
  
  // المرفقات
  attachments: Array<{
    documentType: string;
    fileName: string;
    fileUrl: string;
    uploadedAt: Date;
  }>;
  
  // الحالة
  status: 'Draft' | 'PendingApproval' | 'Approved' | 'Sent' | 'PartiallyReceived' | 'FullyReceived' | 'PartiallyInvoiced' | 'FullyInvoiced' | 'Closed' | 'Cancelled';
  
  // التواريخ المهمة
  approvedDate?: Date;
  sentDate?: Date;
  firstReceivedDate?: Date;
  lastReceivedDate?: Date;
  closedDate?: Date;
  
  // ملاحظات
  notes: string;
  internalNotes: string; // لا تظهر في الطباعة
  
  // التتبع
  createdAt: Date;
  createdBy: string;
  updatedAt: Date;
  updatedBy: string;
  isDeleted: boolean;
}
```

### 2.6 استلام البضاعة (GRN - Goods Received Note)
```typescript
interface GoodsReceivedNote {
  id: string;
  code: string; // مثل: GRN-2025-0001
  
  // الربط مع أمر الشراء
  purchaseOrderId: string;
  purchaseOrderCode: string;
  
  // معلومات أساسية
  date: Date;
  receiptTime: string; // وقت الاستلام
  
  // المورد
  supplierId: string;
  supplierName: string;
  supplierDeliveryNote?: string; // رقم إذن التوريد من المورد
  
  // المستودع
  warehouseId: string;
  warehouseName: string;
  receivedBy: string; // الموظف المستلم
  
  // الأصناف المستلمة
  items: Array<{
    lineNumber: number;
    poLineNumber: number; // رقم السطر في أمر الشراء
    productId?: string;
    description: string;
    
    // الكميات
    orderedQuantity: number;
    receivedQuantity: number;
    rejectedQuantity: number;
    acceptedQuantity: number; // = received - rejected
    unitId: string;
    
    // فحص الجودة
    qualityStatus: 'Pending' | 'Passed' | 'PartiallyPassed' | 'Failed';
    inspectedBy?: string;
    inspectionDate?: Date;
    inspectionNotes?: string;
    
    // التكلفة
    unitCost: number; // من أمر الشراء
    totalCost: number;
    
    // التتبع (Batch/Serial)
    batches?: Array<{
      batchNumber: string;
      quantity: number;
      manufacturingDate?: Date;
      expiryDate?: Date;
      notes?: string;
    }>;
    
    serialNumbers?: string[]; // للأصناف ذات الرقم التسلسلي
    
    // الموقع في المخزن
    storageLocationId?: string;
    
    notes: string;
  }>;
  
  // الإجماليات
  totals: {
    orderedQuantity: number;
    receivedQuantity: number;
    acceptedQuantity: number;
    rejectedQuantity: number;
    totalCost: number;
  };
  
  // النقل والشحن
  shippingInfo: {
    carrierName?: string;
    vehicleNumber?: string;
    driverName?: string;
    driverPhone?: string;
    deliveryNote?: string;
  };
  
  // فحص الجودة العام
  qualityControl: {
    overallStatus: 'Pending' | 'Passed' | 'PartiallyPassed' | 'Failed';
    inspectorId?: string;
    inspectorName?: string;
    inspectionDate?: Date;
    packagingCondition: 'Good' | 'Damaged' | 'Acceptable';
    notes?: string;
  };
  
  // المرفقات
  attachments: Array<{
    documentType: string; // صور، مستندات التوريد...
    fileName: string;
    fileUrl: string;
    uploadedAt: Date;
  }>;
  
  // الحالة
  status: 'Draft' | 'Completed' | 'QualityPending' | 'QualityApproved' | 'QualityRejected' | 'Returned';
  
  // ملاحظات
  notes: string;
  
  // التتبع
  createdAt: Date;
  createdBy: string;
  updatedAt: Date;
  updatedBy: string;
  isDeleted: boolean;
}
```

### 2.7 فاتورة الشراء (Purchase Invoice)
```typescript
interface PurchaseInvoice {
  id: string;
  code: string; // مثل: PI-2025-0001
  
  // الربط
  purchaseOrderId?: string;
  purchaseOrderCode?: string;
  grnIds: string[]; // قد تكون من عدة استلامات
  
  // معلومات الفاتورة
  date: Date;
  dueDate: Date;
  supplierInvoiceNumber: string; // رقم الفاتورة عند المورد
  supplierInvoiceDate: Date;
  
  // المورد
  supplierId: string;
  supplierName: string;
  supplierAddress: string;
  supplierTaxNumber: string;
  
  // نوع الفاتورة
  invoiceType: 'Regular' | 'Import' | 'Service' | 'FixedAsset';
  
  // الأصناف
  items: Array<{
    lineNumber: number;
    poLineNumber?: number;
    grnLineNumber?: number;
    productId?: string;
    description: string;
    
    // الكميات
    quantity: number;
    unitId: string;
    
    // الأسعار
    unitPrice: number;
    discount: number;
    discountAmount: number;
    taxRate: number;
    taxAmount: number;
    totalAmount: number;
    
    // الربط مع المخزون
    warehouseId?: string;
    batchNumber?: string;
    
    // للمحاسبة
    expenseAccountId?: string; // حساب المصروف
    costCenterId?: string;
    projectId?: string;
    
    notes: string;
  }>;
  
  // الإجماليات
  totals: {
    subtotal: number;
    discountAmount: number;
    taxableAmount: number;
    taxAmount: number;
    total: number;
    currency: string;
    exchangeRate?: number;
    totalInBaseCurrency?: number;
  };
  
  // التكاليف الإضافية (للاستيراد)
  additionalCosts: Array<{
    costType: 'Shipping' | 'Insurance' | 'Customs' | 'Handling' | 'Other';
    description: string;
    amount: number;
    accountId: string;
    distributeToItems: boolean; // توزيع على الأصناف
  }>;
  
  // شروط الدفع
  paymentInfo: {
    paymentTermId: string;
    paymentMethod: 'Cash' | 'Credit' | 'BankTransfer' | 'Check';
    paidAmount: number;
    remainingAmount: number;
    paymentStatus: 'Unpaid' | 'PartiallyPaid' | 'FullyPaid';
  };
  
  // المطابقة الثلاثية (3-Way Matching)
  matching: {
    poMatched: boolean;
    grnMatched: boolean;
    quantityMatched: boolean;
    priceMatched: boolean;
    amountMatched: boolean;
    matchingStatus: 'Perfect' | 'WithinTolerance' | 'Mismatched';
    mismatches?: Array<{
      type: 'Quantity' | 'Price' | 'Amount';
      expected: number;
      actual: number;
      variance: number;
      variancePercent: number;
    }>;
  };
  
  // الموافقات
  approvals: Array<{
    level: number;
    approverRole: string;
    approverId?: string;
    approverName?: string;
    status: 'Pending' | 'Approved' | 'Rejected';
    date?: Date;
    notes?: string;
  }>;
  
  // المرفقات
  attachments: Array<{
    documentType: string;
    fileName: string;
    fileUrl: string;
    uploadedAt: Date;
  }>;
  
  // الحالة
  status: 'Draft' | 'PendingApproval' | 'Approved' | 'Posted' | 'PartiallyPaid' | 'FullyPaid' | 'Cancelled';
  
  // القيود المحاسبية
  journalEntryId?: string;
  journalEntryNumber?: string;
  
  // ملاحظات
  notes: string;
  internalNotes: string;
  
  // التتبع
  createdAt: Date;
  createdBy: string;
  updatedAt: Date;
  updatedBy: string;
  isDeleted: boolean;
}
```

### 2.8 مرتجع المشتريات (Purchase Return)
```typescript
interface PurchaseReturn {
  id: string;
  code: string; // مثل: PR-2025-0001
  
  // الربط
  purchaseInvoiceId?: string;
  purchaseInvoiceCode?: string;
  grnId?: string; // الاستلام الأصلي
  
  // معلومات أساسية
  date: Date;
  returnType: 'QualityIssue' | 'WrongItem' | 'Damaged' | 'Excess' | 'Other';
  returnReason: string;
  
  // المورد
  supplierId: string;
  supplierName: string;
  
  // الأصناف المرتجعة
  items: Array<{
    lineNumber: number;
    invoiceLineNumber?: number;
    productId: string;
    description: string;
    
    // الكميات
    invoicedQuantity: number;
    returnQuantity: number;
    unitId: string;
    
    // الأسعار (من الفاتورة الأصلية)
    unitPrice: number;
    discount: number;
    discountAmount: number;
    taxRate: number;
    taxAmount: number;
    totalAmount: number;
    
    // سبب الإرجاع
    returnReason: string;
    condition: 'Damaged' | 'Defective' | 'Expired' | 'WrongItem' | 'Good';
    
    // التتبع
    batchNumber?: string;
    serialNumber?: string;
    warehouseId: string;
    
    notes: string;
  }>;
  
  // الإجماليات
  totals: {
    subtotal: number;
    discountAmount: number;
    taxableAmount: number;
    taxAmount: number;
    total: number;
  };
  
  // معالجة الإرجاع
  handling: {
    action: 'Refund' | 'CreditNote' | 'Replacement';
    expectedAction: string;
    refundMethod?: 'Cash' | 'BankTransfer' | 'DeductFromNextInvoice';
    refundAmount?: number;
    replacementExpectedDate?: Date;
  };
  
  // الموافقات
  approvals: Array<{
    level: number;
    approverRole: string;
    approverId?: string;
    status: 'Pending' | 'Approved' | 'Rejected';
    date?: Date;
    notes?: string;
  }>;
  
  // المرفقات
  attachments: Array<{
    documentType: string;
    fileName: string;
    fileUrl: string;
    uploadedAt: Date;
  }>;
  
  // الحالة
  status: 'Draft' | 'PendingApproval' | 'Approved' | 'Completed' | 'Rejected' | 'Cancelled';
  
  // القيود المحاسبية
  journalEntryId?: string;
  creditNoteId?: string;
  
  // ملاحظات
  notes: string;
  
  // التتبع
  createdAt: Date;
  createdBy: string;
  updatedAt: Date;
  updatedBy: string;
  isDeleted: boolean;
}
```

---

<a name="workflow"></a>
## 🔄 3. دورة العمل الكاملة (Complete Workflow)

### 3.1 السيناريو الأساسي: الدورة الكاملة

```
┌─────────────────────────────────────────────────────────────────┐
│                    🔄 دورة المشتريات الكاملة                    │
└─────────────────────────────────────────────────────────────────┘

[1] تحديد الحاجة (Identification of Need)
     │
     ├─→ نقص في المخزون (Low Stock Alert)
     ├─→ طلب من قسم معين (Department Request)
     ├─→ مشروع جديد (New Project)
     └─→ أصل ثابت جديد (New Fixed Asset)
     │
     ↓

[2] إنشاء طلب الشراء (Purchase Requisition)
     │
     ├─→ إدخال المواد المطلوبة
     ├─→ تحديد الكميات والمواصفات
     ├─→ تحديد الأولوية والتاريخ المطلوب
     └─→ حفظ الطلب
     │
     ↓

[3] اعتماد طلب الشراء (PR Approval)
     │
     ├─→ Level 1: رئيس القسم
     ├─→ Level 2: مدير المشتريات
     ├─→ Level 3: المدير المالي (للمبالغ الكبيرة)
     └─→ حالة: Approved
     │
     ↓

[4] طلب عروض الأسعار (RFQ)
     │
     ├─→ اختيار الموردين المؤهلين
     ├─→ إرسال RFQ للموردين
     ├─→ تحديد آخر موعد للاستلام
     └─→ انتظار العروض
     │
     ↓

[5] استلام عروض الأسعار (Receive Quotations)
     │
     ├─→ تسجيل عروض الموردين
     ├─→ إدخال الأسعار والشروط
     └─→ إرفاق عروض الموردين
     │
     ↓

[6] مقارنة العروض (Compare Quotations)
     │
     ├─→ مقارنة الأسعار
     ├─→ تقييم الجودة
     ├─→ مقارنة مواعيد التوريد
     ├─→ مقارنة الشروط
     └─→ احتساب النقاط (Scoring)
     │
     ↓

[7] اختيار العرض الفائز (Award Selection)
     │
     ├─→ اختيار أفضل عرض
     ├─→ اعتماد الاختيار
     └─→ إشعار الموردين
     │
     ↓

[8] إنشاء أمر الشراء (Create Purchase Order)
     │
     ├─→ إنشاء PO من العرض الفائز
     ├─→ تحديد شروط التسليم والدفع
     ├─→ تحديد عنوان التوصيل
     └─→ حفظ كمسودة
     │
     ↓

[9] اعتماد أمر الشراء (PO Approval)
     │
     ├─→ Level 1: مدير المشتريات
     ├─→ Level 2: المدير المالي
     └─→ حالة: Approved
     │
     ↓

[10] إرسال أمر الشراء (Send PO)
      │
      ├─→ طباعة أمر الشراء
      ├─→ إرسال للمورد (Email/Portal)
      ├─→ تأكيد الاستلام من المورد
      └─→ حالة: Sent
      │
      ↓

[11] متابعة أمر الشراء (PO Follow-up)
      │
      ├─→ تتبع الحالة
      ├─→ التواصل مع المورد
      ├─→ تأكيد موعد التوريد
      └─→ التنبيهات للتأخير
      │
      ↓

[12] استلام البضاعة (Goods Receipt)
      │
      ├─→ إنشاء GRN
      ├─→ فحص البضاعة
      ├─→ تسجيل الكميات المستلمة
      ├─→ تحديد الكميات المرفوضة
      └─→ إدخال Batch/Serial Numbers
      │
      ↓

[13] فحص الجودة (Quality Inspection)
      │
      ├─→ فحص المواصفات
      ├─→ اختبار الجودة
      ├─→ الموافقة أو الرفض
      └─→ تحديث GRN بالنتائج
      │
      ↓

[14] إدخال للمخزن (Store in Warehouse)
      │
      ├─→ تحديث كميات المخزون
      ├─→ تسجيل الدفعات (Batches)
      ├─→ طباعة ملصقات الباركود
      └─→ تخزين في المواقع المحددة
      │
      ↓

[15] استلام فاتورة المورد (Receive Supplier Invoice)
      │
      ├─→ تسجيل فاتورة المورد
      ├─→ ربط مع PO و GRN
      └─→ إدخال التكاليف الإضافية
      │
      ↓

[16] المطابقة الثلاثية (3-Way Matching)
      │
      ├─→ PO vs Invoice (الطلب مع الفاتورة)
      ├─→ GRN vs Invoice (الاستلام مع الفاتورة)
      ├─→ التحقق من:
      │    ├─→ الكميات
      │    ├─→ الأسعار
      │    └─→ المبالغ
      │
      ├─→ Perfect Match? → الموافقة التلقائية
      └─→ Mismatch? → يحتاج موافقة يدوية
      │
      ↓

[17] اعتماد الفاتورة (Invoice Approval)
      │
      ├─→ Level 1: محاسب الحسابات الدائنة
      ├─→ Level 2: مدير الحسابات الدائنة
      ├─→ Level 3: المدير المالي (للمبالغ الكبيرة)
      └─→ حالة: Approved
      │
      ↓

[18] ترحيل الفاتورة (Post Invoice)
      │
      ├─→ إنشاء قيد محاسبي تلقائي:
      │    ├─→ Debit: حساب المخزون أو المصروف
      │    ├─→ Debit: ضريبة القيمة المضافة (مدخلات)
      │    └─→ Credit: حساب المورد (Accounts Payable)
      │
      └─→ حالة: Posted
      │
      ↓

[19] الدفع للمورد (Payment to Supplier)
      │
      ├─→ تجهيز قائمة المدفوعات
      ├─→ اعتماد المدفوعات
      ├─→ تنفيذ الدفع (نقدي/شيك/تحويل)
      ├─→ إنشاء سند صرف
      │
      └─→ إنشاء قيد محاسبي:
           ├─→ Debit: حساب المورد
           └─→ Credit: البنك/الصندوق
      │
      ↓

[20] إقفال أمر الشراء (Close PO)
      │
      ├─→ التحقق من:
      │    ├─→ استلام كامل البضاعة
      │    ├─→ استلام كل الفواتير
      │    └─→ دفع كل المستحقات
      │
      └─→ حالة: Closed
      │
      ↓

[✓] انتهاء الدورة (End of Cycle)
```

### 3.2 السيناريو البديل: مرتجع مشتريات

```
┌───────────────────────────────────────────────────────────┐
│              📦 دورة مرتجع المشتريات                      │
└───────────────────────────────────────────────────────────┘

[1] اكتشاف المشكلة
     │
     ├─→ عيب في الجودة
     ├─→ صنف خاطئ
     ├─→ تالف أو منتهي الصلاحية
     └─→ كمية زائدة
     │
     ↓

[2] إنشاء مرتجع المشتريات (Purchase Return)
     │
     ├─→ اختيار الفاتورة الأصلية
     ├─→ تحديد الأصناف المراد إرجاعها
     ├─→ تحديد سبب الإرجاع
     ├─→ إرفاق صور/مستندات
     └─→ حفظ المرتجع
     │
     ↓

[3] الموافقة على المرتجع
     │
     ├─→ مراجعة من قسم المشتريات
     ├─→ موافقة مدير المخزون
     └─→ موافقة المدير المالي
     │
     ↓

[4] التواصل مع المورد
     │
     ├─→ إشعار المورد بالإرجاع
     ├─→ طلب إذن إرجاع (RMA)
     └─→ الاتفاق على آلية الإرجاع
     │
     ↓

[5] إرجاع البضاعة
     │
     ├─→ تجهيز البضاعة
     ├─→ شحن للمورد
     └─→ إشعار بالشحن
     │
     ↓

[6] التأثير على المخزون
     │
     ├─→ خصم الكمية من المخزون
     ├─→ تحديث حركة المخزون
     └─→ إلغاء الدفعة (Batch) إن أمكن
     │
     ↓

[7] المعالجة المالية
     │
     ├─→ إنشاء إشعار دائن (Credit Note)
     ├─→ خصم من حساب المورد
     │
     └─→ قيد محاسبي:
          ├─→ Debit: حساب المورد
          ├─→ Credit: حساب المخزون
          └─→ Credit: ضريبة القيمة المضافة
     │
     ↓

[8] الاسترداد أو الاستبدال
     │
     ├─→ استرداد نقدي
     ├─→ خصم من فاتورة قادمة
     └─→ استبدال بصنف جديد
     │
     ↓

[✓] إقفال المرتجع
```

### 3.3 السيناريو: مشتريات خارجية/استيراد

```
┌────────────────────────────────────────────────────────────┐
│            🌍 دورة المشتريات الخارجية والاستيراد           │
└────────────────────────────────────────────────────────────┘

[1] تحديد الحاجة والموافقات الأولية
     │
     ↓

[2] إنشاء طلب الشراء (PR)
     ├─→ تحديد المواصفات الفنية
     └─→ موافقة الإدارة العليا
     │
     ↓

[3] البحث عن موردين خارجيين
     ├─→ معارض دولية
     ├─→ أدلة الموردين
     └─→ وكلاء محليون
     │
     ↓

[4] طلب عروض الأسعار (RFQ)
     ├─→ إرسال المواصفات الفنية
     ├─→ تحديد شروط التوريد (Incoterms)
     └─→ طلب عينات (Samples)
     │
     ↓

[5] تقييم العروض
     ├─→ مقارنة الأسعار
     ├─→ فحص العينات
     ├─→ التحقق من الشهادات
     └─→ حساب التكلفة الإجمالية (Landed Cost)
     │
     ↓

[6] إنشاء أمر الشراء (PO)
     ├─→ تحديد Incoterms (FOB, CIF, etc)
     ├─→ تحديد طريقة الشحن (بحري/جوي)
     ├─→ تحديد شروط الدفع (LC, TT, etc)
     └─→ التأمين على البضاعة
     │
     ↓

[7] فتح اعتماد مستندي (Letter of Credit)
     ├─→ التقديم للبنك
     ├─→ تجهيز المستندات المطلوبة
     ├─→ فتح LC
     └─→ إرسال LC للمورد
     │
     ↓

[8] الشحن من المورد
     ├─→ تجهيز البضاعة
     ├─→ الشحن للميناء
     ├─→ إصدار بوليصة الشحن (Bill of Lading)
     └─→ إرسال المستندات للبنك
     │
     ↓

[9] المستندات وإجراءات التخليص
     ├─→ استلام المستندات من البنك
     ├─→ تعيين مخلص جمركي
     ├─→ تقديم بيان جمركي
     └─→ دفع الرسوم الجمركية
     │
     ↓

[10] التخليص الجمركي
      ├─→ فحص جمركي (إن لزم)
      ├─→ احتساب الرسوم:
      │    ├─→ الرسوم الجمركية
      │    ├─→ ضريبة القيمة المضافة
      │    └─→ رسوم أخرى
      │
      └─→ سداد الرسوم
      │
      ↓

[11] النقل للمخازن
      ├─→ استئجار شاحنة
      ├─→ النقل من الميناء
      └─→ استلام في المخزن
      │
      ↓

[12] استلام البضاعة (GRN)
      ├─→ فحص الكميات
      ├─→ فحص الجودة
      └─→ تسجيل الاستلام
      │
      ↓

[13] حساب التكلفة الفعلية (Landed Cost)
      │
      ├─→ سعر البضاعة (FOB/CIF)
      ├─→ + الشحن
      ├─→ + التأمين
      ├─→ + الرسوم الجمركية
      ├─→ + ضريبة القيمة المضافة
      ├─→ + التخليص الجمركي
      ├─→ + النقل المحلي
      ├─→ + رسوم بنكية (LC)
      └─→ = التكلفة الإجمالية (Landed Cost)
      │
      ↓

[14] توزيع التكاليف على الأصناف
      ├─→ نسبياً حسب القيمة
      ├─→ أو نسبياً حسب الوزن
      └─→ تحديث تكلفة الصنف
      │
      ↓

[15] استلام فاتورة المورد
      ├─→ Invoice من المورد
      ├─→ إضافة التكاليف المحلية
      └─→ احتساب الإجمالي
      │
      ↓

[16] المطابقة والاعتماد
      │
      ↓

[17] الترحيل المحاسبي
      │
      ├─→ قيد الفاتورة:
      │    ├─→ Debit: مخزون
      │    ├─→ Debit: رسوم جمركية
      │    ├─→ Debit: مصاريف شحن
      │    ├─→ Debit: ضريبة قيمة مضافة
      │    └─→ Credit: حساب المورد
      │
      └─→ قيد الدفع للمورد (عبر LC)
      │
      ↓

[18] الدفع والإقفال
      │
      ↓

[✓] انتهاء دورة الاستيراد
```

---

<a name="screens"></a>
## 🖥️ 4. الشاشات التفصيلية (Detailed Screens)

### 4.1 شاشة الموردين (Suppliers)

#### Layout التفصيلي:

```typescript
const supplierFormLayout = {
  // Tab 1: المعلومات الأساسية
  tabs: [
    {
      id: "basic-info",
      titleAr: "المعلومات الأساسية",
      titleEn: "Basic Information",
      sections: [
        {
          titleAr: "البيانات الأساسية",
          titleEn: "Basic Data",
          columns: 2,
          fields: [
            {
              name: "code",
              labelAr: "كود المورد",
              labelEn: "Supplier Code",
              type: "text",
              required: true,
              readOnly: true, // Auto-generated
              width: "half"
            },
            {
              name: "type",
              labelAr: "النوع",
              labelEn: "Type",
              type: "select",
              options: ["Individual", "Company"],
              required: true,
              width: "half"
            },
            {
              name: "nameAr",
              labelAr: "الاسم بالعربي",
              labelEn: "Name in Arabic",
              type: "text",
              required: true,
              width: "half"
            },
            {
              name: "nameEn",
              labelAr: "الاسم بالإنجليزي",
              labelEn: "Name in English",
              type: "text",
              required: true,
              width: "half"
            },
            {
              name: "category",
              labelAr: "التصنيف",
              labelEn: "Category",
              type: "select",
              options: ["LocalSupplier", "ImportSupplier", "ServiceProvider"],
              required: true,
              width: "half"
            },
            {
              name: "status",
              labelAr: "الحالة",
              labelEn: "Status",
              type: "select",
              options: ["Active", "Inactive", "Blocked"],
              required: true,
              width: "half"
            }
          ]
        },
        {
          titleAr: "معلومات الاتصال",
          titleEn: "Contact Information",
          columns: 2,
          fields: [
            { name: "phone", labelAr: "الهاتف", type: "tel", width: "half" },
            { name: "mobile", labelAr: "الجوال", type: "tel", width: "half" },
            { name: "email", labelAr: "البريد الإلكتروني", type: "email", width: "half" },
            { name: "website", labelAr: "الموقع الإلكتروني", type: "url", width: "half" }
          ]
        },
        {
          titleAr: "العنوان",
          titleEn: "Address",
          columns: 2,
          fields: [
            { name: "addressAr", labelAr: "العنوان بالعربي", type: "textarea", width: "full" },
            { name: "city", labelAr: "المدينة", type: "text", width: "half" },
            { name: "country", labelAr: "الدولة", type: "select", width: "half" },
            { name: "postalCode", labelAr: "الرمز البريدي", type: "text", width: "half" }
          ]
        }
      ]
    },
    
    // Tab 2: المعلومات المالية
    {
      id: "financial-info",
      titleAr: "المعلومات المالية",
      titleEn: "Financial Information",
      sections: [
        {
          titleAr: "البيانات الضريبية والتجارية",
          columns: 2,
          fields: [
            { name: "taxNumber", labelAr: "الرقم الضريبي", type: "text", required: true, width: "half" },
            { name: "commercialRegister", labelAr: "السجل التجاري", type: "text", width: "half" }
          ]
        },
        {
          titleAr: "شروط الدفع والائتمان",
          columns: 2,
          fields: [
            { name: "paymentTermId", labelAr: "شروط الدفع", type: "lookup", entity: "PaymentTerms", required: true, width: "half" },
            { name: "defaultCurrency", labelAr: "العملة الافتراضية", type: "select", width: "half" },
            { name: "creditLimit", labelAr: "حد الائتمان", type: "number", width: "half" },
            { name: "currentBalance", labelAr: "الرصيد الحالي", type: "number", readOnly: true, width: "half" }
          ]
        },
        {
          titleAr: "الحسابات البنكية",
          titleEn: "Bank Accounts",
          type: "datagrid",
          columns: [
            { field: "bankName", headerAr: "البنك", type: "text" },
            { field: "accountNumber", headerAr: "رقم الحساب", type: "text" },
            { field: "iban", headerAr: "IBAN", type: "text" },
            { field: "swiftCode", headerAr: "Swift Code", type: "text" },
            { field: "currency", headerAr: "العملة", type: "select" },
            { field: "isDefault", headerAr: "افتراضي", type: "checkbox" }
          ]
        }
      ]
    },
    
    // Tab 3: التقييم والأداء
    {
      id: "rating",
      titleAr: "التقييم والأداء",
      titleEn: "Rating & Performance",
      sections: [
        {
          titleAr: "تقييم المورد",
          columns: 2,
          fields: [
            { name: "qualityScore", labelAr: "تقييم الجودة", type: "rating", max: 5, width: "half" },
            { name: "deliveryScore", labelAr: "تقييم التوريد", type: "rating", max: 5, width: "half" },
            { name: "priceScore", labelAr: "تقييم السعر", type: "rating", max: 5, width: "half" },
            { name: "overallScore", labelAr: "التقييم الإجمالي", type: "number", readOnly: true, width: "half" },
            { name: "lastEvaluationDate", labelAr: "تاريخ آخر تقييم", type: "date", width: "half" },
            { name: "evaluatedBy", labelAr: "تم التقييم بواسطة", type: "text", readOnly: true, width: "half" },
            { name: "ratingNotes", labelAr: "ملاحظات التقييم", type: "textarea", width: "full" }
          ]
        },
        {
          titleAr: "إحصائيات الأداء (آخر سنة)",
          columns: 3,
          type: "statistics",
          stats: [
            { labelAr: "إجمالي المشتريات", field: "totalPurchases", format: "currency" },
            { labelAr: "عدد أوامر الشراء", field: "totalPOs", format: "number" },
            { labelAr: "متوسط مدة التوريد", field: "avgDeliveryDays", format: "number", suffix: "يوم" },
            { labelAr: "نسبة التوريد في الموعد", field: "onTimeDeliveryRate", format: "percent" },
            { labelAr: "نسبة الجودة", field: "qualityRate", format: "percent" },
            { labelAr: "نسبة المرتجعات", field: "returnRate", format: "percent" }
          ]
        }
      ]
    },
    
    // Tab 4: جهات الاتصال
    {
      id: "contacts",
      titleAr: "جهات الاتصال",
      titleEn: "Contacts",
      type: "datagrid",
      columns: [
        { field: "name", headerAr: "الاسم", type: "text", required: true },
        { field: "position", headerAr: "المنصب", type: "text" },
        { field: "phone", headerAr: "الهاتف", type: "tel", required: true },
        { field: "email", headerAr: "البريد الإلكتروني", type: "email" },
        { field: "isPrimary", headerAr: "جهة اتصال أساسية", type: "checkbox" }
      ]
    },
    
    // Tab 5: المستندات
    {
      id: "documents",
      titleAr: "المستندات",
      titleEn: "Documents",
      type: "datagrid",
      columns: [
        { field: "documentType", headerAr: "نوع المستند", type: "select", options: ["CommercialRegister", "TaxCard", "Contract", "Certificate", "Other"] },
        { field: "documentNumber", headerAr: "رقم المستند", type: "text" },
        { field: "issueDate", headerAr: "تاريخ الإصدار", type: "date" },
        { field: "expiryDate", headerAr: "تاريخ الانتهاء", type: "date" },
        { field: "attachment", headerAr: "المرفق", type: "file" }
      ]
    },
    
    // Tab 6: التاريخ
    {
      id: "history",
      titleAr: "التاريخ",
      titleEn: "History",
      sections: [
        {
          titleAr: "سجل المشتريات",
          type: "readonly-list",
          source: "PurchaseOrders",
          filter: { supplierId: "$id" },
          displayFields: ["code", "date", "total", "status"]
        },
        {
          titleAr: "سجل المدفوعات",
          type: "readonly-list",
          source: "Payments",
          filter: { supplierId: "$id" },
          displayFields: ["code", "date", "amount", "paymentMethod"]
        }
      ]
    }
  ],
  
  // Action Buttons
  actions: {
    primary: [
      { id: "save", labelAr: "حفظ", icon: "save" },
      { id: "saveAndNew", labelAr: "حفظ وجديد", icon: "save" }
    ],
    secondary: [
      { id: "print", labelAr: "طباعة", icon: "print" },
      { id: "sendEmail", labelAr: "إرسال بريد", icon: "email" },
      { id: "block", labelAr: "حظر", icon: "block", confirmationRequired: true },
      { id: "delete", labelAr: "حذف", icon: "delete", confirmationRequired: true, role: "Admin" }
    ]
  }
};
```

### 4.2 شاشة أمر الشراء (Purchase Order)

#### Layout التفصيلي:

```typescript
const purchaseOrderFormLayout = {
  tabs: [
    // Tab 1: البيانات الأساسية
    {
      id: "main",
      titleAr: "البيانات الأساسية",
      titleEn: "Main Data",
      sections: [
        {
          titleAr: "معلومات الأمر",
          columns: 3,
          fields: [
            { name: "code", labelAr: "رقم الأمر", type: "text", readOnly: true, width: "third" },
            { name: "date", labelAr: "التاريخ", type: "date", required: true, default: "today", width: "third" },
            { name: "deliveryDate", labelAr: "تاريخ التوريد المتوقع", type: "date", required: true, width: "third" },
            { name: "status", labelAr: "الحالة", type: "badge", readOnly: true, width: "third" },
            { name: "sourceType", labelAr: "المصدر", type: "select", options: ["Manual", "PurchaseRequisition", "SupplierQuotation"], width: "third" },
            { name: "sourceCode", labelAr: "رقم المصدر", type: "text", readOnly: true, width: "third" }
          ]
        },
        {
          titleAr: "بيانات المورد",
          columns: 2,
          fields: [
            { 
              name: "supplierId", 
              labelAr: "المورد", 
              type: "lookup", 
              entity: "Suppliers",
              required: true,
              displayFields: ["code", "nameAr", "phone"],
              searchFields: ["code", "nameAr", "nameEn"],
              width: "full",
              onChange: "loadSupplierData" // حدث لتحميل بيانات المورد
            },
            { name: "supplierAddress", labelAr: "عنوان المورد", type: "textarea", readOnly: true, width: "full" },
            { name: "supplierPhone", labelAr: "هاتف المورد", type: "text", readOnly: true, width: "half" },
            { name: "paymentTerms", labelAr: "شروط الدفع", type: "text", readOnly: true, width: "half" }
          ]
        },
        {
          titleAr: "عنوان التوصيل",
          columns: 2,
          fields: [
            {
              name: "warehouseId",
              labelAr: "المستودع",
              type: "lookup",
              entity: "Warehouses",
              required: true,
              width: "full",
              onChange: "loadWarehouseAddress"
            },
            { name: "deliveryAddress", labelAr: "العنوان الكامل", type: "textarea", readOnly: true, width: "full" },
            { name: "contactPerson", labelAr: "جهة الاتصال", type: "text", width: "half" },
            { name: "contactPhone", labelAr: "هاتف الاتصال", type: "tel", width: "half" }
          ]
        }
      ]
    },
    
    // Tab 2: الأصناف (Items)
    {
      id: "items",
      titleAr: "الأصناف",
      titleEn: "Items",
      type: "datagrid",
      showSummary: true,
      allowAdd: true,
      allowEdit: true,
      allowDelete: true,
      columns: [
        {
          field: "lineNumber",
          headerAr: "م",
          headerEn: "#",
          type: "autoNumber",
          width: 50,
          editable: false
        },
        {
          field: "productId",
          headerAr: "الصنف",
          headerEn: "Product",
          type: "lookup",
          entity: "Products",
          displayFields: ["code", "nameAr"],
          searchFields: ["code", "nameAr", "nameEn", "barcode"],
          required: true,
          width: 250,
          onChange: "loadProductData"
        },
        {
          field: "description",
          headerAr: "الوصف",
          headerEn: "Description",
          type: "text",
          width: 200
        },
        {
          field: "specifications",
          headerAr: "المواصفات",
          headerEn: "Specifications",
          type: "textarea",
          width: 200
        },
        {
          field: "orderedQuantity",
          headerAr: "الكمية المطلوبة",
          headerEn: "Ordered Qty",
          type: "number",
          required: true,
          min: 0.001,
          decimals: 3,
          width: 100,
          onChange: "calculateLineTotal"
        },
        {
          field: "receivedQuantity",
          headerAr: "الكمية المستلمة",
          headerEn: "Received Qty",
          type: "number",
          readOnly: true,
          decimals: 3,
          width: 100
        },
        {
          field: "remainingQuantity",
          headerAr: "الكمية المتبقية",
          headerEn: "Remaining Qty",
          type: "number",
          readOnly: true,
          decimals: 3,
          width: 100,
          formula: "orderedQuantity - receivedQuantity"
        },
        {
          field: "unitId",
          headerAr: "الوحدة",
          headerEn: "Unit",
          type: "select",
          source: "product.units",
          required: true,
          width: 80
        },
        {
          field: "unitPrice",
          headerAr: "السعر",
          headerEn: "Unit Price",
          type: "number",
          required: true,
          min: 0,
          decimals: 2,
          width: 100,
          onChange: "calculateLineTotal"
        },
        {
          field: "discount",
          headerAr: "الخصم %",
          headerEn: "Discount %",
          type: "number",
          min: 0,
          max: 100,
          decimals: 2,
          width: 80,
          onChange: "calculateLineTotal"
        },
        {
          field: "discountAmount",
          headerAr: "قيمة الخصم",
          headerEn: "Discount Amount",
          type: "number",
          readOnly: true,
          decimals: 2,
          width: 100,
          formula: "(orderedQuantity * unitPrice) * (discount / 100)"
        },
        {
          field: "taxRate",
          headerAr: "الضريبة %",
          headerEn: "Tax %",
          type: "number",
          default: 15,
          decimals: 2,
          width: 80,
          onChange: "calculateLineTotal"
        },
        {
          field: "taxAmount",
          headerAr: "قيمة الضريبة",
          headerEn: "Tax Amount",
          type: "number",
          readOnly: true,
          decimals: 2,
          width: 100,
          formula: "((orderedQuantity * unitPrice) - discountAmount) * (taxRate / 100)"
        },
        {
          field: "totalAmount",
          headerAr: "الإجمالي",
          headerEn: "Total",
          type: "number",
          readOnly: true,
          decimals: 2,
          width: 120,
          formula: "(orderedQuantity * unitPrice) - discountAmount + taxAmount",
          summary: "sum"
        },
        {
          field: "requiredDate",
          headerAr: "التاريخ المطلوب",
          headerEn: "Required Date",
          type: "date",
          width: 120
        },
        {
          field: "notes",
          headerAr: "ملاحظات",
          headerEn: "Notes",
          type: "textarea",
          width: 150
        }
      ],
      
      // الإجماليات في أسفل الجدول
      footer: {
        sections: [
          {
            titleAr: "الإجماليات",
            columns: 2,
            fields: [
              { name: "subtotal", labelAr: "المجموع الفرعي", type: "number", readOnly: true, format: "currency" },
              { name: "discountAmount", labelAr: "إجمالي الخصم", type: "number", readOnly: true, format: "currency" },
              { name: "taxableAmount", labelAr: "الوعاء الضريبي", type: "number", readOnly: true, format: "currency" },
              { name: "taxAmount", labelAr: "إجمالي الضريبة", type: "number", readOnly: true, format: "currency" },
              { name: "total", labelAr: "الإجمالي النهائي", type: "number", readOnly: true, format: "currency", highlight: true }
            ]
          }
        ]
      }
    },
    
    // Tab 3: التكاليف الإضافية
    {
      id: "additional-costs",
      titleAr: "التكاليف الإضافية",
      titleEn: "Additional Costs",
      type: "datagrid",
      showTotal: true,
      columns: [
        {
          field: "costType",
          headerAr: "نوع التكلفة",
          type: "select",
          options: ["Shipping", "Insurance", "Customs", "Other"],
          required: true
        },
        {
          field: "description",
          headerAr: "الوصف",
          type: "text",
          required: true
        },
        {
          field: "amount",
          headerAr: "المبلغ",
          type: "number",
          required: true,
          decimals: 2,
          summary: "sum"
        },
        {
          field: "accountId",
          headerAr: "الحساب",
          type: "lookup",
          entity: "ChartOfAccounts"
        }
      ]
    },
    
    // Tab 4: الشروط
    {
      id: "terms",
      titleAr: "الشروط",
      titleEn: "Terms & Conditions",
      sections: [
        {
          columns: 1,
          fields: [
            {
              name: "paymentTerms",
              labelAr: "شروط الدفع",
              type: "richtext",
              height: 150
            },
            {
              name: "deliveryTerms",
              labelAr: "شروط التوريد",
              type: "richtext",
              height: 150
            },
            {
              name: "warranty",
              labelAr: "الضمان",
              type: "richtext",
              height: 100
            },
            {
              name: "penalty",
              labelAr: "الشرط الجزائي",
              type: "richtext",
              height: 100
            },
            {
              name: "otherTerms",
              labelAr: "شروط أخرى",
              type: "richtext",
              height: 150
            }
          ]
        }
      ]
    },
    
    // Tab 5: الموافقات
    {
      id: "approvals",
      titleAr: "الموافقات",
      titleEn: "Approvals",
      type: "workflow",
      levels: [
        {
          level: 1,
          titleAr: "مدير المشتريات",
          role: "PurchasingManager",
          maxAmount: 50000
        },
        {
          level: 2,
          titleAr: "المدير المالي",
          role: "CFO",
          maxAmount: null
        }
      ],
      displayFields: ["level", "approverRole", "approverName", "status", "date", "notes"]
    },
    
    // Tab 6: المرفقات
    {
      id: "attachments",
      titleAr: "المرفقات",
      titleEn: "Attachments",
      type: "file-upload",
      allowMultiple: true,
      maxSize: 10, // MB
      allowedTypes: ["pdf", "doc", "docx", "xlsx", "jpg", "png"],
      columns: [
        { field: "documentType", headerAr: "نوع المستند", type: "select" },
        { field: "fileName", headerAr: "اسم الملف", type: "text" },
        { field: "uploadedAt", headerAr: "تاريخ الرفع", type: "datetime" },
        { field: "uploadedBy", headerAr: "تم الرفع بواسطة", type: "text" },
        { field: "actions", headerAr: "إجراءات", type: "actions" }
      ]
    },
    
    // Tab 7: الاستلامات (GRNs)
    {
      id: "grns",
      titleAr: "الاستلامات",
      titleEn: "Goods Receipts",
      type: "readonly-list",
      source: "GoodsReceivedNotes",
      filter: { purchaseOrderId: "$id" },
      allowNavigate: true,
      columns: [
        { field: "code", headerAr: "رقم الاستلام" },
        { field: "date", headerAr: "التاريخ", type: "date" },
        { field: "receivedQuantity", headerAr: "الكمية المستلمة", type: "number" },
        { field: "acceptedQuantity", headerAr: "الكمية المقبولة", type: "number" },
        { field: "status", headerAr: "الحالة", type: "badge" }
      ],
      actions: [
        { id: "view", labelAr: "عرض", icon: "eye" },
        { id: "print", labelAr: "طباعة", icon: "print" }
      ]
    },
    
    // Tab 8: الفواتير
    {
      id: "invoices",
      titleAr: "الفواتير",
      titleEn: "Invoices",
      type: "readonly-list",
      source: "PurchaseInvoices",
      filter: { purchaseOrderId: "$id" },
      allowNavigate: true,
      columns: [
        { field: "code", headerAr: "رقم الفاتورة" },
        { field: "date", headerAr: "التاريخ", type: "date" },
        { field: "total", headerAr: "المبلغ", type: "currency" },
        { field: "paidAmount", headerAr: "المدفوع", type: "currency" },
        { field: "status", headerAr: "الحالة", type: "badge" }
      ]
    },
    
    // Tab 9: ملاحظات
    {
      id: "notes",
      titleAr: "الملاحظات",
      titleEn: "Notes",
      sections: [
        {
          columns: 1,
          fields: [
            {
              name: "notes",
              labelAr: "ملاحظات عامة",
              type: "richtext",
              height: 200
            },
            {
              name: "internalNotes",
              labelAr: "ملاحظات داخلية (لا تظهر في الطباعة)",
              type: "richtext",
              height: 200,
              hint: "هذه الملاحظات للاستخدام الداخلي فقط"
            }
          ]
        }
      ]
    }
  ],
  
  // الأزرار
  actions: {
    primary: [
      { id: "save", labelAr: "حفظ", icon: "save", enabled: "status === 'Draft'" },
      { id: "saveAndApprove", labelAr: "حفظ واعتماد", icon: "check", enabled: "canApprove" }
    ],
    secondary: [
      { id: "sendForApproval", labelAr: "إرسال للاعتماد", icon: "send", enabled: "status === 'Draft'" },
      { id: "approve", labelAr: "اعتماد", icon: "check", enabled: "status === 'PendingApproval' && canApprove" },
      { id: "reject", labelAr: "رفض", icon: "close", enabled: "status === 'PendingApproval' && canApprove" },
      { id: "send", labelAr: "إرسال للمورد", icon: "email", enabled: "status === 'Approved'" },
      { id: "receiveGoods", labelAr: "استلام بضاعة", icon: "inbox", enabled: "status === 'Sent'" },
      { id: "createInvoice", labelAr: "إنشاء فاتورة", icon: "file", enabled: "receivedQuantity > 0" },
      { id: "close", labelAr: "إقفال الأمر", icon: "lock", enabled: "canClose", confirmationRequired: true },
      { id: "cancel", labelAr: "إلغاء", icon: "cancel", enabled: "status !== 'Closed'", confirmationRequired: true },
      { id: "print", labelAr: "طباعة", icon: "print" },
      { id: "export", labelAr: "تصدير", icon: "download", options: ["PDF", "Excel"] },
      { id: "duplicate", labelAr: "نسخ", icon: "copy" },
      { id: "delete", labelAr: "حذف", icon: "delete", enabled: "status === 'Draft'", confirmationRequired: true, role: "Admin" }
    ]
  },
  
  // شريط الحالة (Status Bar)
  statusBar: {
    left: [
      { field: "status", type: "badge" },
      { field: "createdBy", labelAr: "أنشئ بواسطة" },
      { field: "createdAt", labelAr: "تاريخ الإنشاء", type: "datetime" }
    ],
    right: [
      { field: "currency", type: "text" },
      { field: "total", type: "currency", highlight: true }
    ]
  }
};
```

### 4.3 شاشة استلام البضاعة (Goods Received Note - GRN)

#### Layout التفصيلي:

```typescript
const grnFormLayout = {
  tabs: [
    // Tab 1: البيانات الأساسية
    {
      id: "main",
      titleAr: "البيانات الأساسية",
      sections: [
        {
          titleAr: "معلومات الاستلام",
          columns: 3,
          fields: [
            { name: "code", labelAr: "رقم الاستلام", type: "text", readOnly: true, width: "third" },
            { name: "date", labelAr: "تاريخ الاستلام", type: "date", required: true, default: "today", width: "third" },
            { name: "receiptTime", labelAr: "وقت الاستلام", type: "time", required: true, default: "now", width: "third" },
            {
              name: "purchaseOrderId",
              labelAr: "أمر الشراء",
              type: "lookup",
              entity: "PurchaseOrders",
              required: true,
              filter: { status: ["Approved", "Sent", "PartiallyReceived"] },
              displayFields: ["code", "supplierName", "date"],
              width: "full",
              onChange: "loadPOData"
            },
            { name: "supplierName", labelAr: "المورد", type: "text", readOnly: true, width: "half" },
            { name: "supplierDeliveryNote", labelAr: "رقم إذن التوريد من المورد", type: "text", width: "half" }
          ]
        },
        {
          titleAr: "المستودع والمستلم",
          columns: 2,
          fields: [
            {
              name: "warehouseId",
              labelAr: "المستودع",
              type: "lookup",
              entity: "Warehouses",
              required: true,
              readOnly: true, // من أمر الشراء
              width: "full"
            },
            {
              name: "receivedBy",
              labelAr: "المستلم",
              type: "lookup",
              entity: "Employees",
              required: true,
              default: "currentUser",
              width: "half"
            },
            { name: "status", labelAr: "الحالة", type: "badge", readOnly: true, width: "half" }
          ]
        },
        {
          titleAr: "معلومات النقل",
          columns: 2,
          fields: [
            { name: "carrierName", labelAr: "شركة النقل", type: "text", width: "half" },
            { name: "vehicleNumber", labelAr: "رقم المركبة", type: "text", width: "half" },
            { name: "driverName", labelAr: "السائق", type: "text", width: "half" },
            { name: "driverPhone", labelAr: "هاتف السائق", type: "tel", width: "half" }
          ]
        }
      ]
    },
    
    // Tab 2: الأصناف المستلمة
    {
      id: "items",
      titleAr: "الأصناف المستلمة",
      type: "datagrid",
      showSummary: true,
      allowAdd: false, // يتم تحميلها من أمر الشراء
      allowEdit: true,
      allowDelete: false,
      columns: [
        { field: "lineNumber", headerAr: "م", type: "autoNumber", width: 50, editable: false },
        { field: "productCode", headerAr: "كود الصنف", type: "text", readOnly: true, width: 100 },
        { field: "productName", headerAr: "اسم الصنف", type: "text", readOnly: true, width: 200 },
        { field: "orderedQuantity", headerAr: "الكمية المطلوبة", type: "number", readOnly: true, decimals: 3, width: 100 },
        {
          field: "receivedQuantity",
          headerAr: "الكمية المستلمة",
          type: "number",
          required: true,
          min: 0,
          decimals: 3,
          width: 120,
          highlight: true,
          onChange: "calculateAccepted"
        },
        {
          field: "rejectedQuantity",
          headerAr: "الكمية المرفوضة",
          type: "number",
          min: 0,
          decimals: 3,
          width: 120,
          onChange: "calculateAccepted"
        },
        {
          field: "acceptedQuantity",
          headerAr: "الكمية المقبولة",
          type: "number",
          readOnly: true,
          decimals: 3,
          width: 120,
          formula: "receivedQuantity - rejectedQuantity",
          summary: "sum"
        },
        { field: "unitName", headerAr: "الوحدة", type: "text", readOnly: true, width: 80 },
        {
          field: "qualityStatus",
          headerAr: "حالة الفحص",
          type: "select",
          options: ["Pending", "Passed", "PartiallyPassed", "Failed"],
          required: true,
          width: 120,
          colorCoded: true
        },
        { field: "unitCost", headerAr: "التكلفة", type: "number", readOnly: true, decimals: 2, width: 100 },
        {
          field: "totalCost",
          headerAr: "إجمالي التكلفة",
          type: "number",
          readOnly: true,
          decimals: 2,
          width: 120,
          formula: "acceptedQuantity * unitCost",
          summary: "sum"
        },
        {
          field: "batchTracking",
          headerAr: "الدفعات",
          type: "button",
          icon: "list",
          action: "openBatchDialog",
          visible: "product.hasBatchNumber",
          width: 80
        },
        {
          field: "serialTracking",
          headerAr: "الأرقام التسلسلية",
          type: "button",
          icon: "barcode",
          action: "openSerialDialog",
          visible: "product.hasSerialNumber",
          width: 100
        },
        { field: "notes", headerAr: "ملاحظات", type: "textarea", width: 150 }
      ]
    },
    
    // Tab 3: فحص الجودة
    {
      id: "quality-control",
      titleAr: "فحص الجودة",
      sections: [
        {
          titleAr: "الفحص العام",
          columns: 2,
          fields: [
            {
              name: "overallStatus",
              labelAr: "الحالة العامة",
              type: "select",
              options: ["Pending", "Passed", "PartiallyPassed", "Failed"],
              required: true,
              width: "half"
            },
            {
              name: "inspectorId",
              labelAr: "المفتش",
              type: "lookup",
              entity: "Employees",
              filter: { role: "QualityInspector" },
              width: "half"
            },
            { name: "inspectionDate", labelAr: "تاريخ الفحص", type: "datetime", width: "half" },
            {
              name: "packagingCondition",
              labelAr: "حالة التغليف",
              type: "select",
              options: ["Good", "Damaged", "Acceptable"],
              width: "half"
            },
            { name: "qcNotes", labelAr: "ملاحظات الفحص", type: "richtext", height: 200, width: "full" }
          ]
        }
      ]
    },
    
    // Tab 4: المرفقات
    {
      id: "attachments",
      titleAr: "المرفقات",
      type: "file-upload",
      hint: "صور البضاعة، مستندات التوريد، إلخ",
      allowMultiple: true,
      columns: [
        { field: "documentType", headerAr: "نوع المستند", type: "select", options: ["Photo", "DeliveryNote", "PackingList", "Certificate", "Other"] },
        { field: "fileName", headerAr: "اسم الملف" },
        { field: "uploadedAt", headerAr: "تاريخ الرفع", type: "datetime" }
      ]
    },
    
    // Tab 5: ملاحظات
    {
      id: "notes",
      titleAr: "ملاحظات",
      sections: [
        {
          columns: 1,
          fields: [
            { name: "notes", labelAr: "ملاحظات عامة", type: "richtext", height: 200 }
          ]
        }
      ]
    }
  ],
  
  actions: {
    primary: [
      { id: "save", labelAr: "حفظ", icon: "save" },
      { id: "saveAndComplete", labelAr: "حفظ وإتمام", icon: "check", enabled: "allItemsInspected" }
    ],
    secondary: [
      { id: "complete", labelAr: "إتمام الاستلام", icon: "check", enabled: "status === 'Draft' && allItemsInspected" },
      { id: "approveQuality", labelAr: "اعتماد الجودة", icon: "verified", enabled: "status === 'QualityPending' && canApproveQC" },
      { id: "rejectQuality", labelAr: "رفض الجودة", icon: "cancel", enabled: "status === 'QualityPending' && canApproveQC" },
      { id: "createReturn", labelAr: "إنشاء مرتجع", icon: "return", enabled: "status === 'Completed' && rejectedQuantity > 0" },
      { id: "print", labelAr: "طباعة", icon: "print" },
      { id: "printBarcode", labelAr: "طباعة الباركود", icon: "barcode", enabled: "status === 'Completed'" }
    ]
  },
  
  statusBar: {
    left: [
      { field: "status", type: "badge" },
      { field: "poCode", labelAr: "أمر الشراء" }
    ],
    right: [
      { field: "totalReceivedQuantity", labelAr: "إجمالي الكمية المستلمة" },
      { field: "totalCost", type: "currency", highlight: true }
    ]
  }
};
```

### 4.4 شاشة فاتورة الشراء (Purchase Invoice)

```typescript
const purchaseInvoiceFormLayout = {
  tabs: [
    // Tab 1: البيانات الأساسية
    {
      id: "main",
      titleAr: "البيانات الأساسية",
      sections: [
        {
          titleAr: "معلومات الفاتورة",
          columns: 3,
          fields: [
            { name: "code", labelAr: "رقم الفاتورة", type: "text", readOnly: true, width: "third" },
            { name: "date", labelAr: "التاريخ", type: "date", required: true, default: "today", width: "third" },
            { name: "dueDate", labelAr: "تاريخ الاستحقاق", type: "date", required: true, width: "third" },
            { name: "supplierInvoiceNumber", labelAr: "رقم فاتورة المورد", type: "text", required: true, width: "half" },
            { name: "supplierInvoiceDate", labelAr: "تاريخ فاتورة المورد", type: "date", required: true, width: "half" },
            {
              name: "invoiceType",
              labelAr: "نوع الفاتورة",
              type: "select",
              options: ["Regular", "Import", "Service", "FixedAsset"],
              required: true,
              width: "third"
            },
            { name: "status", labelAr: "الحالة", type: "badge", readOnly: true, width: "third" }
          ]
        },
        {
          titleAr: "المصدر",
          columns: 2,
          fields: [
            {
              name: "purchaseOrderId",
              labelAr: "أمر الشراء",
              type: "lookup",
              entity: "PurchaseOrders",
              required: true,
              width: "full",
              onChange: "loadPOData"
            },
            {
              name: "grnIds",
              labelAr: "استلامات البضاعة",
              type: "multiselect",
              source: "GoodsReceivedNotes",
              filter: { purchaseOrderId: "$purchaseOrderId", status: "Completed" },
              displayField: "code",
              width: "full",
              onChange: "loadGRNItems"
            }
          ]
        },
        {
          titleAr: "بيانات المورد",
          columns: 2,
          fields: [
            { name: "supplierId", labelAr: "المورد", type: "lookup", entity: "Suppliers", required: true, readOnly: true, width: "full" },
            { name: "supplierAddress", labelAr: "العنوان", type: "textarea", readOnly: true, width: "full" },
            { name: "supplierTaxNumber", labelAr: "الرقم الضريبي", type: "text", readOnly: true, width: "half" }
          ]
        }
      ]
    },
    
    // Tab 2: الأصناف
    {
      id: "items",
      titleAr: "الأصناف",
      type: "datagrid",
      showSummary: true,
      columns: [
        { field: "lineNumber", headerAr: "م", type: "autoNumber", width: 50 },
        { field: "productCode", headerAr: "كود الصنف", type: "text", readOnly: true, width: 100 },
        { field: "productName", headerAr: "اسم الصنف", type: "text", readOnly: true, width: 200 },
        { field: "quantity", headerAr: "الكمية", type: "number", required: true, decimals: 3, width: 100 },
        { field: "unitName", headerAr: "الوحدة", type: "text", readOnly: true, width: 80 },
        { field: "unitPrice", headerAr: "السعر", type: "number", required: true, decimals: 2, width: 100, onChange: "calculateLineTotal" },
        { field: "discount", headerAr: "الخصم %", type: "number", decimals: 2, width: 80, onChange: "calculateLineTotal" },
        { field: "discountAmount", headerAr: "قيمة الخصم", type: "number", readOnly: true, decimals: 2, width: 100 },
        { field: "taxRate", headerAr: "الضريبة %", type: "number", default: 15, decimals: 2, width: 80, onChange: "calculateLineTotal" },
        { field: "taxAmount", headerAr: "قيمة الضريبة", type: "number", readOnly: true, decimals: 2, width: 100 },
        { field: "totalAmount", headerAr: "الإجمالي", type: "number", readOnly: true, decimals: 2, width: 120, summary: "sum" },
        {
          name: "expenseAccountId",
          headerAr: "حساب المصروف",
          type: "lookup",
          entity: "ChartOfAccounts",
          filter: { accountType: "Expense" },
          width: 150
        },
        {
          name: "costCenterId",
          headerAr: "مركز التكلفة",
          type: "lookup",
          entity: "CostCenters",
          width: 150
        },
        { field: "notes", headerAr: "ملاحظات", type: "textarea", width: 150 }
      ],
      
      footer: {
        sections: [
          {
            titleAr: "الإجماليات",
            columns: 2,
            fields: [
              { name: "subtotal", labelAr: "المجموع الفرعي", type: "number", readOnly: true, format: "currency" },
              { name: "discountAmount", labelAr: "إجمالي الخصم", type: "number", readOnly: true, format: "currency" },
              { name: "taxableAmount", labelAr: "الوعاء الضريبي", type: "number", readOnly: true, format: "currency" },
              { name: "taxAmount", labelAr: "إجمالي الضريبة", type: "number", readOnly: true, format: "currency" },
              { name: "additionalCostsTotal", labelAr: "إجمالي التكاليف الإضافية", type: "number", readOnly: true, format: "currency" },
              { name: "total", labelAr: "الإجمالي النهائي", type: "number", readOnly: true, format: "currency", highlight: true }
            ]
          }
        ]
      }
    },
    
    // Tab 3: التكاليف الإضافية
    {
      id: "additional-costs",
      titleAr: "التكاليف الإضافية",
      type: "datagrid",
      hint: "للمشتريات الخارجية: شحن، جمارك، تأمين، إلخ",
      columns: [
        {
          field: "costType",
          headerAr: "نوع التكلفة",
          type: "select",
          options: ["Shipping", "Insurance", "Customs", "Handling", "Other"],
          required: true,
          width: 150
        },
        { field: "description", headerAr: "الوصف", type: "text", required: true, width: 200 },
        { field: "amount", headerAr: "المبلغ", type: "number", required: true, decimals: 2, width: 120, summary: "sum" },
        {
          field: "accountId",
          headerAr: "الحساب",
          type: "lookup",
          entity: "ChartOfAccounts",
          required: true,
          width: 200
        },
        {
          field: "distributeToItems",
          headerAr: "توزيع على الأصناف",
          type: "checkbox",
          hint: "سيتم توزيع التكلفة نسبياً على الأصناف",
          width: 100
        }
      ]
    },
    
    // Tab 4: المطابقة (3-Way Matching)
    {
      id: "matching",
      titleAr: "المطابقة الثلاثية",
      type: "readonly",
      hint: "مطابقة الفاتورة مع أمر الشراء والاستلام",
      sections: [
        {
          titleAr: "نتائج المطابقة",
          type: "status-grid",
          items: [
            {
              labelAr: "مطابقة أمر الشراء",
              field: "poMatched",
              type: "boolean-icon",
              trueIcon: "check-circle",
              falseIcon: "warning"
            },
            {
              labelAr: "مطابقة الاستلام",
              field: "grnMatched",
              type: "boolean-icon"
            },
            {
              labelAr: "مطابقة الكميات",
              field: "quantityMatched",
              type: "boolean-icon"
            },
            {
              labelAr: "مطابقة الأسعار",
              field: "priceMatched",
              type: "boolean-icon"
            },
            {
              labelAr: "مطابقة المبالغ",
              field: "amountMatched",
              type: "boolean-icon"
            },
            {
              labelAr: "حالة المطابقة",
              field: "matchingStatus",
              type: "badge",
              colorMap: {
                Perfect: "success",
                WithinTolerance: "warning",
                Mismatched: "error"
              }
            }
          ]
        },
        {
          titleAr: "الانحرافات",
          type: "datagrid",
          visible: "mismatches.length > 0",
          source: "mismatches",
          columns: [
            { field: "type", headerAr: "نوع الانحراف" },
            { field: "expected", headerAr: "المتوقع", type: "number" },
            { field: "actual", headerAr: "الفعلي", type: "number" },
            { field: "variance", headerAr: "الفرق", type: "number" },
            { field: "variancePercent", headerAr: "نسبة الفرق %", type: "percent" }
          ]
        }
      ]
    },
    
    // Tab 5: الموافقات
    {
      id: "approvals",
      titleAr: "الموافقات",
      type: "workflow",
      levels: [
        {
          level: 1,
          titleAr: "محاسب الحسابات الدائنة",
          role: "APClerk"
        },
        {
          level: 2,
          titleAr: "مدير الحسابات الدائنة",
          role: "APManager",
          condition: "total > 10000"
        },
        {
          level: 3,
          titleAr: "المدير المالي",
          role: "CFO",
          condition: "total > 50000 || matchingStatus === 'Mismatched'"
        }
      ]
    },
    
    // Tab 6: الدفع
    {
      id: "payment",
      titleAr: "معلومات الدفع",
      sections: [
        {
          titleAr: "الحالة المالية",
          columns: 2,
          fields: [
            { name: "paymentTermName", labelAr: "شروط الدفع", type: "text", readOnly: true, width: "half" },
            { name: "dueDate", labelAr: "تاريخ الاستحقاق", type: "date", readOnly: true, width: "half" },
            { name: "total", labelAr: "إجمالي الفاتورة", type: "number", readOnly: true, format: "currency", width: "third" },
            { name: "paidAmount", labelAr: "المدفوع", type: "number", readOnly: true, format: "currency", width: "third" },
            { name: "remainingAmount", labelAr: "المتبقي", type: "number", readOnly: true, format: "currency", width: "third", highlight: true },
            {
              name: "paymentStatus",
              labelAr: "حالة الدفع",
              type: "badge",
              readOnly: true,
              width: "half",
              colorMap: {
                Unpaid: "error",
                PartiallyPaid: "warning",
                FullyPaid: "success"
              }
            }
          ]
        },
        {
          titleAr: "سجل المدفوعات",
          type: "readonly-list",
          source: "Payments",
          filter: { invoiceId: "$id" },
          columns: [
            { field: "code", headerAr: "رقم السند" },
            { field: "date", headerAr: "التاريخ", type: "date" },
            { field: "amount", headerAr: "المبلغ", type: "currency" },
            { field: "paymentMethod", headerAr: "طريقة الدفع" }
          ],
          actions: [
            { id: "view", labelAr: "عرض", icon: "eye" }
          ]
        }
      ]
    },
    
    // Tab 7: القيود المحاسبية
    {
      id: "accounting",
      titleAr: "القيود المحاسبية",
      type: "readonly",
      sections: [
        {
          titleAr: "قيد الفاتورة",
          type: "journal-entry-view",
          journalEntryId: "$journalEntryId"
        }
      ]
    },
    
    // Tab 8: المرفقات
    {
      id: "attachments",
      titleAr: "المرفقات",
      type: "file-upload",
      allowMultiple: true,
      columns: [
        { field: "documentType", headerAr: "نوع المستند", type: "select" },
        { field: "fileName", headerAr: "اسم الملف" }
      ]
    }
  ],
  
  actions: {
    primary: [
      { id: "save", labelAr: "حفظ", icon: "save", enabled: "status === 'Draft'" },
      { id: "saveAndApprove", labelAr: "حفظ واعتماد", icon: "check", enabled: "status === 'Draft' && canApprove" }
    ],
    secondary: [
      { id: "sendForApproval", labelAr: "إرسال للاعتماد", icon: "send", enabled: "status === 'Draft'" },
      { id: "approve", labelAr: "اعتماد", icon: "check", enabled: "status === 'PendingApproval' && canApprove" },
      { id: "reject", labelAr: "رفض", icon: "close", enabled: "status === 'PendingApproval' && canApprove" },
      { id: "post", labelAr: "ترحيل", icon: "book", enabled: "status === 'Approved'" },
      { id: "makePayment", labelAr: "دفعة جديدة", icon: "payment", enabled: "status === 'Posted' && remainingAmount > 0" },
      { id: "createReturn", labelAr: "إنشاء مرتجع", icon: "return", enabled: "status === 'Posted'" },
      { id: "print", labelAr: "طباعة", icon: "print" },
      { id: "cancel", labelAr: "إلغاء", icon: "cancel", enabled: "status !== 'Cancelled' && paidAmount === 0", confirmationRequired: true }
    ]
  },
  
  statusBar: {
    left: [
      { field: "status", type: "badge" },
      { field: "matchingStatus", type: "badge", visible: "matchingStatus !== 'Perfect'" }
    ],
    right: [
      { field: "currency", type: "text" },
      { field: "total", type: "currency", highlight: true },
      { field: "remainingAmount", labelAr: "المتبقي", type: "currency", visible: "remainingAmount > 0" }
    ]
  }
};
```

---

<a name="integration"></a>
## 🔗 5. التكامل مع المديولات الأخرى

### 5.1 التكامل مع المحاسبة (Accounting)

#### 5.1.1 عند ترحيل فاتورة شراء:

```typescript
const postPurchaseInvoice = async (invoiceId: string) => {
  const invoice = await getPurchaseInvoice(invoiceId);
  
  // إنشاء قيد محاسبي تلقائي
  const journalEntry = {
    date: invoice.date,
    referenceType: 'PurchaseInvoice',
    referenceId: invoice.id,
    referenceNumber: invoice.code,
    description: `فاتورة شراء من ${invoice.supplierName} - ${invoice.code}`,
    currency: invoice.currency,
    exchangeRate: invoice.exchangeRate || 1,
    
    lines: []
  };
  
  // 1. مدين: حسابات المخزون/المصروفات
  for (const item of invoice.items) {
    if (item.productId) {
      // صنف مخزني
      journalEntry.lines.push({
        accountId: await getProductInventoryAccount(item.productId),
        description: `شراء ${item.description}`,
        debit: item.totalAmount - item.taxAmount,
        credit: 0,
        costCenterId: item.costCenterId,
        projectId: item.projectId
      });
    } else {
      // خدمة أو مصروف
      journalEntry.lines.push({
        accountId: item.expenseAccountId,
        description: item.description,
        debit: item.totalAmount - item.taxAmount,
        credit: 0,
        costCenterId: item.costCenterId,
        projectId: item.projectId
      });
    }
  }
  
  // 2. مدين: ضريبة القيمة المضافة (مدخلات)
  if (invoice.totals.taxAmount > 0) {
    journalEntry.lines.push({
      accountId: await getAccount('InputVAT'),
      description: 'ضريبة قيمة مضافة - مدخلات',
      debit: invoice.totals.taxAmount,
      credit: 0
    });
  }
  
  // 3. مدين: التكاليف الإضافية
  for (const cost of invoice.additionalCosts) {
    journalEntry.lines.push({
      accountId: cost.accountId,
      description: cost.description,
      debit: cost.amount,
      credit: 0
    });
  }
  
  // 4. دائن: حساب المورد (Accounts Payable)
  journalEntry.lines.push({
    accountId: await getSupplierAccount(invoice.supplierId),
    description: `مديونية ${invoice.supplierName}`,
    debit: 0,
    credit: invoice.totals.total
  });
  
  // حفظ القيد
  const je = await createJournalEntry(journalEntry);
  
  // تحديث الفاتورة
  await updatePurchaseInvoice(invoiceId, {
    status: 'Posted',
    journalEntryId: je.id,
    journalEntryNumber: je.code
  });
  
  // تحديث رصيد المورد
  await updateSupplierBalance(invoice.supplierId, invoice.totals.total);
  
  return je;
};
```

#### 5.1.2 عند دفعة للمورد:

```typescript
const recordSupplierPayment = async (paymentData) => {
  // إنشاء سند صرف
  const payment = await createPaymentVoucher({
    date: paymentData.date,
    supplierId: paymentData.supplierId,
    amount: paymentData.amount,
    paymentMethod: paymentData.paymentMethod,
    referenceNumber: paymentData.referenceNumber,
    notes: paymentData.notes,
    
    // توزيع على الفواتير
    invoices: paymentData.invoices.map(inv => ({
      invoiceId: inv.invoiceId,
      invoiceNumber: inv.invoiceNumber,
      invoiceAmount: inv.invoiceAmount,
      paidAmount: inv.paidAmount,
      remainingAmount: inv.remainingAmount
    }))
  });
  
  // إنشاء قيد محاسبي
  const journalEntry = {
    date: paymentData.date,
    referenceType: 'PaymentVoucher',
    referenceId: payment.id,
    referenceNumber: payment.code,
    description: `دفعة للمورد ${paymentData.supplierName}`,
    
    lines: [
      {
        // مدين: حساب المورد (تخفيض المديونية)
        accountId: await getSupplierAccount(paymentData.supplierId),
        description: `دفعة للمورد ${paymentData.supplierName}`,
        debit: paymentData.amount,
        credit: 0
      },
      {
        // دائن: البنك أو الصندوق
        accountId: paymentData.paymentMethod === 'Cash' 
          ? await getAccount('Cash')
          : await getBankAccount(paymentData.bankAccountId),
        description: `دفعة ${paymentData.paymentMethod === 'Cash' ? 'نقدية' : 'بنكية'}`,
        debit: 0,
        credit: paymentData.amount
      }
    ]
  };
  
  await createJournalEntry(journalEntry);
  
  // تحديث حالة الفواتير
  for (const inv of paymentData.invoices) {
    await updateInvoicePaymentStatus(inv.invoiceId, inv.paidAmount);
  }
  
  // تحديث رصيد المورد
  await updateSupplierBalance(paymentData.supplierId, -paymentData.amount);
  
  return payment;
};
```

#### 5.1.3 عند مرتجع مشتريات:

```typescript
const postPurchaseReturn = async (returnId: string) => {
  const purchaseReturn = await getPurchaseReturn(returnId);
  
  // قيد محاسبي عكسي
  const journalEntry = {
    date: purchaseReturn.date,
    referenceType: 'PurchaseReturn',
    referenceId: purchaseReturn.id,
    referenceNumber: purchaseReturn.code,
    description: `مرتجع مشتريات للمورد ${purchaseReturn.supplierName}`,
    
    lines: []
  };
  
  // 1. دائن: حسابات المخزون/المصروفات
  for (const item of purchaseReturn.items) {
    journalEntry.lines.push({
      accountId: await getProductInventoryAccount(item.productId),
      description: `مرتجع ${item.description}`,
      debit: 0,
      credit: item.totalAmount - item.taxAmount
    });
  }
  
  // 2. دائن: ضريبة القيمة المضافة
  if (purchaseReturn.totals.taxAmount > 0) {
    journalEntry.lines.push({
      accountId: await getAccount('InputVAT'),
      description: 'ضريبة قيمة مضافة - مرتجع',
      debit: 0,
      credit: purchaseReturn.totals.taxAmount
    });
  }
  
  // 3. مدين: حساب المورد (تخفيض المديونية)
  journalEntry.lines.push({
    accountId: await getSupplierAccount(purchaseReturn.supplierId),
    description: `مرتجع مشتريات - ${purchaseReturn.supplierName}`,
    debit: purchaseReturn.totals.total,
    credit: 0
  });
  
  await createJournalEntry(journalEntry);
  
  // تحديث رصيد المورد
  await updateSupplierBalance(
    purchaseReturn.supplierId, 
    -purchaseReturn.totals.total
  );
  
  // إنشاء إشعار دائن
  if (purchaseReturn.handling.action === 'CreditNote') {
    await createCreditNote({
      supplierId: purchaseReturn.supplierId,
      amount: purchaseReturn.totals.total,
      referenceType: 'PurchaseReturn',
      referenceId: returnId
    });
  }
};
```

### 5.2 التكامل مع المخزون (Inventory)

#### 5.2.1 عند استلام بضاعة (GRN):

```typescript
const processGRNForInventory = async (grnId: string) => {
  const grn = await getGoodsReceivedNote(grnId);
  
  // إنشاء حركة مخزون (Stock Entry)
  const stockEntry = {
    entryType: 'Receipt',
    purpose: 'Purchase Receipt',
    referenceType: 'GRN',
    referenceId: grn.id,
    referenceNumber: grn.code,
    date: grn.date,
    warehouseId: grn.warehouseId,
    
    items: []
  };
  
  for (const item of grn.items) {
    if (item.acceptedQuantity > 0) {
      const stockItem = {
        productId: item.productId,
        quantity: item.acceptedQuantity,
        unitId: item.unitId,
        costPrice: item.unitCost,
        
        // التتبع
        batchNumber: null,
        serialNumbers: [],
        
        // الموقع
        warehouseId: grn.warehouseId,
        locationId: await assignStorageLocation(
          grn.warehouseId,
          item.productId
        )
      };
      
      // إذا كان الصنف يتطلب دفعات
      if (item.batches && item.batches.length > 0) {
        for (const batch of item.batches) {
          stockEntry.items.push({
            ...stockItem,
            quantity: batch.quantity,
            batchNumber: batch.batchNumber,
            manufacturingDate: batch.manufacturingDate,
            expiryDate: batch.expiryDate
          });
        }
      }
      // إذا كان الصنف يتطلب أرقام تسلسلية
      else if (item.serialNumbers && item.serialNumbers.length > 0) {
        for (const serial of item.serialNumbers) {
          stockEntry.items.push({
            ...stockItem,
            quantity: 1,
            serialNumber: serial
          });
        }
      }
      // صنف عادي بدون تتبع
      else {
        stockEntry.items.push(stockItem);
      }
    }
  }
  
  // حفظ حركة المخزون
  const entry = await createStockEntry(stockEntry);
  
  // تحديث كميات المخزون
  for (const item of stockEntry.items) {
    await updateProductStock({
      productId: item.productId,
      warehouseId: item.warehouseId,
      quantity: item.quantity,
      operation: 'add',
      costPrice: item.costPrice,
      batchNumber: item.batchNumber,
      serialNumber: item.serialNumber
    });
  }
  
  // تحديث آخر سعر شراء
  for (const item of grn.items) {
    await updateProductLastPurchasePrice(
      item.productId,
      item.unitCost,
      grn.date
    );
  }
  
  return entry;
};
```

#### 5.2.2 عند مرتجع للمورد:

```typescript
const processReturnForInventory = async (returnId: string) => {
  const purchaseReturn = await getPurchaseReturn(returnId);
  
  // حركة مخزون عكسية (خروج)
  const stockEntry = {
    entryType: 'Issue',
    purpose: 'Purchase Return',
    referenceType: 'PurchaseReturn',
    referenceId: returnId,
    referenceNumber: purchaseReturn.code,
    date: purchaseReturn.date,
    
    items: purchaseReturn.items.map(item => ({
      productId: item.productId,
      quantity: item.returnQuantity,
      unitId: item.unitId,
      warehouseId: item.warehouseId,
      batchNumber: item.batchNumber,
      serialNumber: item.serialNumber,
      costPrice: item.unitPrice,
      
      // سبب الإرجاع
      reason: item.returnReason,
      condition: item.condition
    }))
  };
  
  const entry = await createStockEntry(stockEntry);
  
  // تحديث الكميات (خصم)
  for (const item of stockEntry.items) {
    await updateProductStock({
      productId: item.productId,
      warehouseId: item.warehouseId,
      quantity: item.quantity,
      operation: 'subtract',
      batchNumber: item.batchNumber,
      serialNumber: item.serialNumber
    });
  }
  
  return entry;
};
```

### 5.3 التكامل مع خدمة التكلفة (Costing Service)

#### إعادة حساب التكلفة عند استلام مشتريات:

```typescript
const recalculateCostOnPurchase = async (invoiceId: string) => {
  const invoice = await getPurchaseInvoice(invoiceId);
  
  for (const item of invoice.items) {
    if (!item.productId) continue;
    
    // حساب التكلفة الفعلية (Landed Cost)
    let landedCost = item.unitPrice;
    
    // إضافة التكاليف الإضافية الموزعة
    const distributedCosts = calculateDistributedCosts(
      invoice.additionalCosts,
      item,
      invoice.items
    );
    
    landedCost += distributedCosts;
    
    // تحديث تكلفة الصنف
    await updateProductCost({
      productId: item.productId,
      newCost: landedCost,
      method: 'WeightedAverage', // أو FIFO حسب الإعدادات
      quantity: item.quantity,
      transactionDate: invoice.date,
      referenceType: 'PurchaseInvoice',
      referenceId: invoiceId
    });
  }
  
  // إطلاق حدث لإعادة حساب تكلفة المبيعات
  await triggerCostRecalculation({
    productIds: invoice.items.map(i => i.productId),
    fromDate: invoice.date
  });
};

const calculateDistributedCosts = (
  additionalCosts: any[],
  item: any,
  allItems: any[]
) => {
  let distributedAmount = 0;
  
  for (const cost of additionalCosts) {
    if (cost.distributeToItems) {
      // التوزيع النسبي حسب القيمة
      const totalValue = allItems.reduce(
        (sum, i) => sum + (i.quantity * i.unitPrice),
        0
      );
      const itemValue = item.quantity * item.unitPrice;
      const proportion = itemValue / totalValue;
      
      distributedAmount += cost.amount * proportion;
    }
  }
  
  return distributedAmount / item.quantity; // التكلفة لكل وحدة
};
```

### 5.4 التكامل مع الموارد البشرية (HR)

#### ربط مع أداء قسم المشتريات:

```typescript
const evaluatePurchasingPerformance = async (
  employeeId: string,
  period: { startDate: Date; endDate: Date }
) => {
  // مؤشرات الأداء
  const kpis = {
    // 1. توفير التكلفة (Cost Savings)
    costSavings: await calculateCostSavings(employeeId, period),
    
    // 2. الالتزام بالميزانية
    budgetCompliance: await calculateBudgetCompliance(employeeId, period),
    
    // 3. مدة دورة الشراء (Procurement Cycle Time)
    avgCycleTime: await calculateAvgProcurementCycle(employeeId, period),
    
    // 4. جودة الموردين
    supplierQuality: await calculateSupplierQuality(employeeId, period),
    
    // 5. التوريد في الموعد
    onTimeDelivery: await calculateOnTimeDeliveryRate(employeeId, period),
    
    // 6. معدل المرتجعات
    returnRate: await calculateReturnRate(employeeId, period)
  };
  
  // حساب النقاط الإجمالية
  const score = (
    kpis.costSavings * 0.25 +
    kpis.budgetCompliance * 0.15 +
    kpis.avgCycleTime * 0.15 +
    kpis.supplierQuality * 0.20 +
    kpis.onTimeDelivery * 0.15 +
    kpis.returnRate * 0.10
  );
  
  // تسجيل التقييم
  await recordPerformanceEvaluation({
    employeeId,
    period,
    department: 'Purchasing',
    kpis,
    score,
    rating: score >= 90 ? 'Excellent' : score >= 75 ? 'Good' : 'NeedsImprovement'
  });
  
  return { kpis, score };
};
```

### 5.5 مصفوفة التكامل الكاملة

| الحدث | المحاسبة | المخزون | التكلفة | الموارد البشرية | الأصول |
|-------|----------|---------|---------|-----------------|--------|
| إنشاء طلب شراء | - | ✅ حجز متوقع | - | ✅ طلب موظف | - |
| اعتماد أمر شراء | ✅ التزام | - | ✅ ميزانية | - | - |
| استلام بضاعة (GRN) | - | ✅ إضافة | ✅ تحديث تكلفة | - | - |
| ترحيل فاتورة | ✅ قيد | - | ✅ حساب Landed Cost | - | ✅ إضافة أصل |
| دفعة للمورد | ✅ قيد دفع | - | - | - | - |
| مرتجع مشتريات | ✅ قيد عكسي | ✅ خصم | ✅ تعديل | - | - |

---

<a name="business-rules"></a>
## ⚖️ 6. قواعد العمل (Business Rules)

### 6.1 قواعد طلب الشراء (Purchase Requisition)

```typescript
interface PurchaseRequisitionRules {
  // قواعد الإنشاء
  creation: {
    requiredFields: ['date', 'requestedBy', 'department', 'items'];
    minimumItems: 1;
    requireJustification: boolean; // للطلبات غير المخزنية
    
    // الموافقة المسبقة
    requireBudgetCheck: true;
    requireStockCheck: true; // التحقق من عدم وجود كمية كافية
  };
  
  // قواعد الموافقة
  approval: {
    levels: [
      {
        role: 'DepartmentHead',
        maxAmount: 10000,
        required: true
      },
      {
        role: 'PurchasingManager',
        maxAmount: 50000,
        required: true
      },
      {
        role: 'CFO',
        maxAmount: Infinity,
        required: true
      }
    ];
    
    // موافقة تلقائية
    autoApproveWhen: {
      amountBelow: 1000,
      fromApprovedVendor: true,
      withinBudget: true
    };
    
    // الموافقة المتوازية
    parallelApproval: false; // تسلسلي
  };
  
  // قواعد الأولوية
  priority: {
    autoSetUrgent: {
      stockBelowSafety: true,
      criticalProject: true,
      productionHalt: true
    };
  };
}
```

### 6.2 قواعد طلب عرض السعر (RFQ)

```typescript
interface RFQRules {
  // قواعد الموردين
  suppliers: {
    minimumSuppliers: 3; // الحد الأدنى من الموردين
    maximumSuppliers: 10;
    
    // معايير الاختيار
    selectionCriteria: {
      mustBeActive: true;
      mustBeApproved: true;
      minRating: 3; // من 5
      mustHaveCategory: true; // في نفس فئة المنتج
    };
    
    // الاستثناءات
    allowSingleSupplier: {
      when: [
        'ExclusiveDealer',
        'TechnicalRequirement',
        'EmergencyPurchase'
      ];
      requireApproval: true;
      approverRole: 'PurchasingManager';
    };
  };
  
  // قواعد الإرسال
  sending: {
    requireTermsAndConditions: true;
    requireDeadline: true;
    minimumResponseDays: 3;
    sendMethod: ['Email', 'Portal', 'Fax'];
    
    // متابعة
    sendReminder: {
      enabled: true;
      daysBefore: 2
    };
  };
  
  // قواعد الاستلام
  receiving: {
    acceptAfterDeadline: false;
    requireAllSuppliers: false; // الانتظار لكل الموردين
    minimumQuotations: 2; // للمضي قدماً
  };
}
```

### 6.3 قواعد المقارنة والاختيار

```typescript
interface QuotationEvaluationRules {
  // معايير المقارنة
  criteria: {
    price: {
      weight: 40, // %
      method: 'LowestWins'
    };
    quality: {
      weight: 25,
      method: 'Rating' // تقييم يدوي
    };
    deliveryTime: {
      weight: 20,
      method: 'ShortestWins'
    };
    paymentTerms: {
      weight: 10,
      method: 'Rating'
    };
    warranty: {
      weight: 5,
      method: 'Rating'
    };
  };
  
  // حساب النقاط التلقائي
  automaticScoring: {
    price: (quotations) => {
      const lowest = Math.min(...quotations.map(q => q.totalPrice));
      return quotations.map(q => ({
        ...q,
        priceScore: (lowest / q.totalPrice) * 100
      }));
    };
    
    delivery: (quotations) => {
      const shortest = Math.min(...quotations.map(q => q.deliveryDays));
      return quotations.map(q => ({
        ...q,
        deliveryScore: (shortest / q.deliveryDays) * 100
      }));
    };
  };
  
  // قواعد الاختيار
  selection: {
    // الفائز التلقائي
    autoSelectWhen: {
      clearWinner: true, // فارق > 10 نقاط
      allCriteriaMet: true
    };
    
    // يتطلب موافقة يدوية
    requireManualApproval: {
      closeScores: true, // فارق < 5 نقاط
      notLowestPrice: true, // اختيار غير الأرخص
      singleQuotation: true
    };
    
    // الرفض التلقائي
    autoReject: {
      priceAboveBudget: true,
      deliveryTooLate: true,
      lowQualityRating: true, // أقل من 3
      supplierBlocked: true
    };
  };
  
  // التوثيق
  documentation: {
    requireComparisonSheet: true;
    requireJustification: {
      when: 'notLowestPrice',
      approverRole: 'PurchasingManager'
    };
    attachQuotations: 'Mandatory';
  };
}
```

### 6.4 قواعد أمر الشراء (Purchase Order)

```typescript
interface PurchaseOrderRules {
  // قواعد الإنشاء
  creation: {
    requireRFQProcess: {
      when: 'amountAbove',
      threshold: 10000
    };
    
    requirePurchaseRequisition: true;
    allowDirectPO: false; // منع الأوامر المباشرة
    
    exceptions: {
      allowDirect: [
        'EmergencyPurchase',
        'RecurringOrder',
        'ServiceContract'
      ];
      requireApproval: true
    };
  };
  
  // قواعد السعر
  pricing: {
    // التحقق من السعر
    checkPriceVariance: true,
    maxVarianceFromQuotation: 5, // %
    maxVarianceFromLastPO: 10, // %
    
    requireApprovalWhen: {
      priceIncrease: 5, // %
      totalAbove: 50000
    };
  };
  
  // قواعد الموافقة
  approval: {
    matrix: [
      {
        role: 'PurchasingManager',
        maxAmount: 50000,
        conditions: {
          hasRFQ: true,
          withinBudget: true
        }
      },
      {
        role: 'CFO',
        maxAmount: 200000
      },
      {
        role: 'CEO',
        maxAmount: Infinity
      }
    ];
    
    // متطلبات إضافية
    additionalApprovals: {
      newSupplier: 'PurchasingManager',
      foreignCurrency: 'CFO',
      capitalExpenditure: 'CFO'
    };
  };
  
  // قواعد الإرسال
  sending: {
    canSendWhen: 'Approved',
    sendMethod: ['Email', 'Portal', 'Print'],
    requireConfirmation: true,
    
    // المستندات المطلوبة
    requiredAttachments: [
      'TechnicalSpecifications',
      'TermsAndConditions'
    ];
  };
  
  // قواعد التعديل
  modification: {
    allowEdit: {
      statuses: ['Draft', 'PendingApproval'],
      fields: 'All'
    };
    
    allowAmendment: {
      statuses: ['Approved', 'Sent'],
      fields: ['deliveryDate', 'deliveryAddress'],
      requireApproval: true
    };
    
    prohibitEdit: {
      statuses: ['PartiallyReceived', 'FullyReceived', 'Closed'],
      action: 'CreateRevision'
    };
  };
  
  // قواعد الإلغاء
  cancellation: {
    allowCancel: {
      statuses: ['Draft', 'PendingApproval', 'Approved'],
      requireReason: true,
      requireApproval: true
    };
    
    prohibitCancel: {
      statuses: ['PartiallyReceived', 'FullyReceived'],
      alternative: 'CreateReturn'
    };
  };
}
```

### 6.5 قواعد استلام البضاعة (GRN)

```typescript
interface GRNRules {
  // قواعد الإنشاء
  creation: {
    requirePurchaseOrder: true,
    allowMultipleGRNsPerPO: true,
    
    // التحقق من الكميات
    checkQuantities: {
      warnIfExceeds: true,
      maxExcessPercent: 5,
      blockIfExceeds: 10 // %
    };
  };
  
  // قواعد فحص الجودة
  qualityControl: {
    mandatory: true,
    requiredFor: 'All', // أو قائمة فئات محددة
    
    inspectionLevels: {
      'HighValue': 'FullInspection', // فحص كامل
      'Normal': 'SampleInspection', // فحص عينة
      'LowRisk': 'VisualInspection' // فحص بصري
    };
    
    // معايير القبول
    acceptanceCriteria: {
      visualDefects: 'MaxAllowed', // عدد العيوب المسموح
      dimensionTolerance: 'Percentage',
      functionalTest: 'Required'
    };
    
    // الرفض
    rejectionHandling: {
      segregate: true, // عزل البضاعة المرفوضة
      requirePhotos: true,
      notifySupplier: 'Automatic',
      createReturnAutomatically: false
    };
  };
  
  // قواعد التتبع
  tracking: {
    batchNumber: {
      required: 'WhenApplicable',
      autoGenerate: true,
      format: 'BATCH-YYYYMMDD-XXX'
    };
    
    serialNumber: {
      required: 'WhenApplicable',
      validateUnique: true
    };
    
    expiryDate: {
      required: 'WhenApplicable',
      warnIfShortShelfLife: true,
      minShelfLifeDays: 90,
      blockIfExpired: true
    };
  };
  
  // قواعد الموقع
  storage: {
    assignLocation: 'Automatic', // أو Manual
    strategy: 'FIFO', // First In First Out
    
    locationRules: {
      separateByBatch: true,
      separateDamaged: true,
      reserveForProjects: true
    };
  };
}
```

### 6.6 قواعد فاتورة الشراء

```typescript
interface PurchaseInvoiceRules {
  // قواعد الإنشاء
  creation: {
    requirePurchaseOrder: true,
    requireGRN: true, // يجب استلام البضاعة أولاً
    allowDirectInvoice: false,
    
    exceptions: {
      services: {
        requireGRN: false,
        requirePOCompletion: true
      };
      utilities: {
        requirePO: false,
        requireApproval: true
      };
    };
  };
  
  // المطابقة الثلاثية (3-Way Matching)
  threeWayMatching: {
    enforce: true,
    tolerance: {
      quantity: 5, // % فرق مسموح
      price: 3,
      amount: 3
    };
    
    matchingChecks: [
      'PO_Exists',
      'GRN_Exists',
      'Supplier_Match',
      'Currency_Match',
      'Quantity_WithinTolerance',
      'Price_WithinTolerance',
      'Amount_WithinTolerance'
    ];
    
    // عند عدم المطابقة
    onMismatch: {
      action: 'RequireApproval',
      notifyRoles: ['PurchasingManager', 'APManager'],
      blockPayment: true
    };
    
    // الموافقة التلقائية
    autoApproveWhen: {
      perfectMatch: true,
      amountBelow: 5000
    };
  };
  
  // قواعد الموافقة
  approval: {
    levels: [
      {
        role: 'APClerk',
        action: 'Review', // مراجعة فقط، لا اعتماد
        checks: ['DataEntry', 'Matching']
      },
      {
        role: 'APManager',
        maxAmount: 50000,
        conditions: {
          matchingStatus: ['Perfect', 'WithinTolerance']
        }
      },
      {
        role: 'CFO',
        maxAmount: Infinity,
        requiredWhen: [
          'amountAbove(50000)',
          'matchingStatus === Mismatched',
          'newSupplier'
        ]
      }
    ];
  };
  
  // قواعد الدفع
  payment: {
    // الحظر
    blockPaymentWhen: {
      notApproved: true,
      mismatchNotResolved: true,
      supplierBlocked: true,
      invoiceDisputed: true,
      missingDocuments: true
    };
    
    // الأولوية
    paymentPriority: {
      earlyPaymentDiscount: 'High',
      dueSoon: 'High', // خلال 3 أيام
      overdue: 'Critical'
    };
    
    // الخصم المبكر
    earlyPaymentDiscount: {
      enabled: true,
      calculateAutomatically: true,
      requireApproval: false
    };
    
    // الدفع الجزئي
    partialPayment: {
      allowed: true,
      requireApproval: true,
      documentReason: true
    };
  };
  
  // قواعد الضريبة
  tax: {
    validateTaxNumber: true,
    validateTaxRate: true,
    
    withholdingTax: {
      applicable: 'WhenRequired',
      rate: 'BySupplierType',
      createJournalEntry: true
    };
  };
  
  // قواعد الإلغاء
  cancellation: {
    allowWhen: {
      notPaid: true,
      notPosted: true
    };
    
    requireApproval: true,
    requireReason: true,
    
    prohibitWhen: {
      partiallyPaid: true,
      fullyPaid: true
    };
  };
}
```

### 6.7 قواعد مرتجع المشتريات

```typescript
interface PurchaseReturnRules {
  // قواعد الإنشاء
  creation: {
    requireInvoice: true,
    requireGRN: true,
    
    // المهلة الزمنية
    timeLimit: {
      enabled: true,
      days: 30, // من تاريخ الاستلام
      exceptions: [
        'QualityDefect',
        'WarrantyIssue'
      ];
    };
    
    // الأسباب المقبولة
    validReasons: [
      'QualityIssue',
      'WrongItem',
      'Damaged',
      'Expired',
      'Excess',
      'NotAsSpecified'
    ];
    
    requireEvidence: {
      photos: true,
      documents: true,
      inspectionReport: true
    };
  };
  
  // قواعد الكمية
  quantity: {
    maxReturnQuantity: 'InvoicedQuantity',
    allowPartialReturn: true,
    
    // التحقق من المخزون
    checkStockAvailability: true,
    requirePhysicalInspection: true
  };
  
  // قواعد الموافقة
  approval: {
    levels: [
      {
        role: 'PurchasingManager',
        maxAmount: 10000
      },
      {
        role: 'CFO',
        maxAmount: Infinity
      }
    ];
    
    additionalApprovals: {
      qualityIssue: 'QualityManager',
      highValue: 'GeneralManager' // فوق 50000
    };
  };
  
  // قواعد المعالجة
  handling: {
    options: ['Refund', 'CreditNote', 'Replacement'],
    
    refund: {
      method: 'SameAsPayment',
      requireSupplierApproval: true,
      timeline: 'AsPerContract' // 14-30 يوم
    };
    
    creditNote: {
      autoCreate: true,
      applyToNextInvoice: true
    };
    
    replacement: {
      createNewPO: true,
      linkToReturn: true,
      timeline: 'Negotiate'
    };
  };
  
  // قواعد الشحن
  shipping: {
    responsibleParty: 'AsPerContract',
    
    supplierFault: {
      shippingCost: 'Supplier',
      arrangementBy: 'Supplier'
    };
    
    buyerFault: {
      shippingCost: 'Buyer',
      requireRestockingFee: 'Possible'
    };
  };
}
```

---

<a name="database"></a>
## 🗄️ 7. Database Schema

### 7.1 الجداول الأساسية

```sql
-- جدول الموردين
CREATE TABLE Suppliers (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    Code NVARCHAR(50) NOT NULL UNIQUE,
    
    -- المعلومات الأساسية
    NameAr NVARCHAR(200) NOT NULL,
    NameEn NVARCHAR(200) NOT NULL,
    Type NVARCHAR(20) NOT NULL CHECK (Type IN ('Individual', 'Company')),
    Category NVARCHAR(50) NOT NULL CHECK (Category IN ('LocalSupplier', 'ImportSupplier', 'ServiceProvider')),
    
    -- معلومات الاتصال
    Phone NVARCHAR(20),
    Mobile NVARCHAR(20),
    Email NVARCHAR(100),
    Website NVARCHAR(200),
    Fax NVARCHAR(20),
    
    -- العنوان
    AddressAr NVARCHAR(500),
    AddressEn NVARCHAR(500),
    City NVARCHAR(100),
    Country NVARCHAR(100),
    PostalCode NVARCHAR(20),
    
    -- المعلومات المالية
    TaxNumber NVARCHAR(50),
    CommercialRegister NVARCHAR(50),
    PaymentTermId UNIQUEIDENTIFIER,
    DefaultCurrency NVARCHAR(3) DEFAULT 'SAR',
    CreditLimit DECIMAL(18,2) DEFAULT 0,
    CurrentBalance DECIMAL(18,2) DEFAULT 0,
    
    -- التقييم
    QualityScore DECIMAL(3,2) DEFAULT 0, -- 0-5
    DeliveryScore DECIMAL(3,2) DEFAULT 0,
    PriceScore DECIMAL(3,2) DEFAULT 0,
    OverallScore DECIMAL(3,2) DEFAULT 0,
    LastEvaluationDate DATETIME2,
    EvaluatedBy UNIQUEIDENTIFIER,
    RatingNotes NVARCHAR(MAX),
    
    -- الحالة
    Status NVARCHAR(20) DEFAULT 'Active' CHECK (Status IN ('Active', 'Inactive', 'Blocked')),
    BlockReason NVARCHAR(500),
    IsApproved BIT DEFAULT 0,
    ApprovedBy UNIQUEIDENTIFIER,
    ApprovedDate DATETIME2,
    
    -- Audit
    TenantId UNIQUEIDENTIFIER NOT NULL,
    CompanyId UNIQUEIDENTIFIER NOT NULL,
    BranchId UNIQUEIDENTIFIER,
    CreatedAt DATETIME2 DEFAULT GETUTCDATE(),
    CreatedBy UNIQUEIDENTIFIER,
    UpdatedAt DATETIME2 DEFAULT GETUTCDATE(),
    UpdatedBy UNIQUEIDENTIFIER,
    IsDeleted BIT DEFAULT 0,
    DeletedAt DATETIME2,
    DeletedBy UNIQUEIDENTIFIER,
    
    FOREIGN KEY (PaymentTermId) REFERENCES PaymentTerms(Id),
    FOREIGN KEY (TenantId) REFERENCES Tenants(Id),
    FOREIGN KEY (CompanyId) REFERENCES Companies(Id),
    INDEX IX_Suppliers_Tenant_Company (TenantId, CompanyId),
    INDEX IX_Suppliers_Code (Code),
    INDEX IX_Suppliers_Status (Status)
);

-- جدول الحسابات البنكية للموردين
CREATE TABLE SupplierBankAccounts (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    SupplierId UNIQUEIDENTIFIER NOT NULL,
    
    BankName NVARCHAR(200) NOT NULL,
    AccountNumber NVARCHAR(50) NOT NULL,
    IBAN NVARCHAR(50),
    SwiftCode NVARCHAR(20),
    Currency NVARCHAR(3) DEFAULT 'SAR',
    IsDefault BIT DEFAULT 0,
    
    -- Audit
    CreatedAt DATETIME2 DEFAULT GETUTCDATE(),
    CreatedBy UNIQUEIDENTIFIER,
    IsDeleted BIT DEFAULT 0,
    
    FOREIGN KEY (SupplierId) REFERENCES Suppliers(Id),
    INDEX IX_SupplierBankAccounts_Supplier (SupplierId)
);

-- جدول جهات الاتصال لدى الموردين
CREATE TABLE SupplierContacts (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    SupplierId UNIQUEIDENTIFIER NOT NULL,
    
    Name NVARCHAR(200) NOT NULL,
    Position NVARCHAR(100),
    Phone NVARCHAR(20) NOT NULL,
    Email NVARCHAR(100),
    IsPrimary BIT DEFAULT 0,
    
    -- Audit
    CreatedAt DATETIME2 DEFAULT GETUTCDATE(),
    CreatedBy UNIQUEIDENTIFIER,
    IsDeleted BIT DEFAULT 0,
    
    FOREIGN KEY (SupplierId) REFERENCES Suppliers(Id),
    INDEX IX_SupplierContacts_Supplier (SupplierId)
);

-- جدول مستندات الموردين
CREATE TABLE SupplierDocuments (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    SupplierId UNIQUEIDENTIFIER NOT NULL,
    
    DocumentType NVARCHAR(50) NOT NULL,
    DocumentNumber NVARCHAR(50),
    IssueDate DATE,
    ExpiryDate DATE,
    AttachmentUrl NVARCHAR(500),
    
    -- Audit
    CreatedAt DATETIME2 DEFAULT GETUTCDATE(),
    CreatedBy UNIQUEIDENTIFIER,
    IsDeleted BIT DEFAULT 0,
    
    FOREIGN KEY (SupplierId) REFERENCES Suppliers(Id),
    INDEX IX_SupplierDocuments_Supplier (SupplierId)
);

-- جدول طلبات الشراء
CREATE TABLE PurchaseRequisitions (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    Code NVARCHAR(50) NOT NULL UNIQUE,
    
    -- المعلومات الأساسية
    Date DATE NOT NULL,
    RequiredDate DATE NOT NULL,
    RequestedBy UNIQUEIDENTIFIER NOT NULL,
    DepartmentId UNIQUEIDENTIFIER,
    Priority NVARCHAR(20) NOT NULL DEFAULT 'Medium' CHECK (Priority IN ('Low', 'Medium', 'High', 'Urgent')),
    
    -- الغرض
    Purpose NVARCHAR(50) NOT NULL CHECK (Purpose IN ('Stock', 'Project', 'FixedAsset', 'Maintenance', 'Service')),
    PurposeDescription NVARCHAR(500),
    ProjectId UNIQUEIDENTIFIER,
    
    -- الحالة
    Status NVARCHAR(50) DEFAULT 'Draft' CHECK (Status IN ('Draft', 'PendingApproval', 'Approved', 'PartiallyOrdered', 'FullyOrdered', 'Rejected', 'Cancelled')),
    
    -- الإجماليات
    EstimatedTotal DECIMAL(18,2) DEFAULT 0,
    
    -- Audit
    TenantId UNIQUEIDENTIFIER NOT NULL,
    CompanyId UNIQUEIDENTIFIER NOT NULL,
    BranchId UNIQUEIDENTIFIER,
    CreatedAt DATETIME2 DEFAULT GETUTCDATE(),
    CreatedBy UNIQUEIDENTIFIER,
    UpdatedAt DATETIME2 DEFAULT GETUTCDATE(),
    UpdatedBy UNIQUEIDENTIFIER,
    IsDeleted BIT DEFAULT 0,
    
    FOREIGN KEY (RequestedBy) REFERENCES Employees(Id),
    FOREIGN KEY (DepartmentId) REFERENCES Departments(Id),
    FOREIGN KEY (ProjectId) REFERENCES Projects(Id),
    INDEX IX_PurchaseRequisitions_Tenant_Company (TenantId, CompanyId),
    INDEX IX_PurchaseRequisitions_Status (Status),
    INDEX IX_PurchaseRequisitions_Date (Date)
);

-- جدول أصناف طلب الشراء
CREATE TABLE PurchaseRequisitionItems (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    PurchaseRequisitionId UNIQUEIDENTIFIER NOT NULL,
    LineNumber INT NOT NULL,
    
    ProductId UNIQUEIDENTIFIER,
    Description NVARCHAR(500) NOT NULL,
    Specifications NVARCHAR(MAX),
    Quantity DECIMAL(18,3) NOT NULL,
    UnitId UNIQUEIDENTIFIER NOT NULL,
    EstimatedPrice DECIMAL(18,2),
    EstimatedTotal DECIMAL(18,2),
    RequiredDate DATE,
    Notes NVARCHAR(500),
    
    -- للربط مع أوامر الشراء
    OrderedQuantity DECIMAL(18,3) DEFAULT 0,
    RemainingQuantity DECIMAL(18,3),
    
    -- Audit
    CreatedAt DATETIME2 DEFAULT GETUTCDATE(),
    IsDeleted BIT DEFAULT 0,
    
    FOREIGN KEY (PurchaseRequisitionId) REFERENCES PurchaseRequisitions(Id),
    FOREIGN KEY (ProductId) REFERENCES Products(Id),
    FOREIGN KEY (UnitId) REFERENCES Units(Id),
    INDEX IX_PRItems_PR (PurchaseRequisitionId)
);

-- جدول موافقات طلب الشراء
CREATE TABLE PurchaseRequisitionApprovals (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    PurchaseRequisitionId UNIQUEIDENTIFIER NOT NULL,
    
    Level INT NOT NULL,
    ApproverRole NVARCHAR(50) NOT NULL,
    ApproverId UNIQUEIDENTIFIER,
    ApproverName NVARCHAR(200),
    Status NVARCHAR(20) DEFAULT 'Pending' CHECK (Status IN ('Pending', 'Approved', 'Rejected')),
    Date DATETIME2,
    Notes NVARCHAR(500),
    
    CreatedAt DATETIME2 DEFAULT GETUTCDATE(),
    
    FOREIGN KEY (PurchaseRequisitionId) REFERENCES PurchaseRequisitions(Id),
    FOREIGN KEY (ApproverId) REFERENCES Users(Id),
    INDEX IX_PRApprovals_PR (PurchaseRequisitionId)
);

-- جدول طلبات عروض الأسعار (RFQ)
CREATE TABLE RequestsForQuotation (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    Code NVARCHAR(50) NOT NULL UNIQUE,
    
    Date DATE NOT NULL,
    DueDate DATE NOT NULL,
    ValidUntil DATE NOT NULL,
    
    -- المصدر
    SourceType NVARCHAR(50) CHECK (SourceType IN ('PurchaseRequisition', 'Manual')),
    SourceId UNIQUEIDENTIFIER,
    
    -- الشروط
    PaymentTerms NVARCHAR(MAX),
    DeliveryTerms NVARCHAR(MAX),
    Warranty NVARCHAR(MAX),
    ValidityPeriod INT,
    OtherTerms NVARCHAR(MAX),
    
    -- الحالة
    Status NVARCHAR(50) DEFAULT 'Draft' CHECK (Status IN ('Draft', 'Sent', 'PartiallyReceived', 'FullyReceived', 'Evaluated', 'Closed', 'Cancelled')),
    
    -- Audit
    TenantId UNIQUEIDENTIFIER NOT NULL,
    CompanyId UNIQUEIDENTIFIER NOT NULL,
    BranchId UNIQUEIDENTIFIER,
    CreatedAt DATETIME2 DEFAULT GETUTCDATE(),
    CreatedBy UNIQUEIDENTIFIER,
    UpdatedAt DATETIME2 DEFAULT GETUTCDATE(),
    UpdatedBy UNIQUEIDENTIFIER,
    IsDeleted BIT DEFAULT 0,
    
    INDEX IX_RFQ_Tenant_Company (TenantId, CompanyId),
    INDEX IX_RFQ_Status (Status)
);

-- جدول الموردين المستهدفين في RFQ
CREATE TABLE RFQSuppliers (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    RFQId UNIQUEIDENTIFIER NOT NULL,
    SupplierId UNIQUEIDENTIFIER NOT NULL,
    
    ContactPerson NVARCHAR(200),
    Email NVARCHAR(100),
    Phone NVARCHAR(20),
    SentDate DATETIME2,
    ReceivedDate DATETIME2,
    Status NVARCHAR(20) DEFAULT 'Pending' CHECK (Status IN ('Pending', 'Received', 'NotReceived', 'Declined')),
    
    CreatedAt DATETIME2 DEFAULT GETUTCDATE(),
    
    FOREIGN KEY (RFQId) REFERENCES RequestsForQuotation(Id),
    FOREIGN KEY (SupplierId) REFERENCES Suppliers(Id),
    INDEX IX_RFQSuppliers_RFQ (RFQId)
);

-- جدول أصناف RFQ
CREATE TABLE RFQItems (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    RFQId UNIQUEIDENTIFIER NOT NULL,
    LineNumber INT NOT NULL,
    
    ProductId UNIQUEIDENTIFIER,
    Description NVARCHAR(500) NOT NULL,
    Specifications NVARCHAR(MAX),
    Quantity DECIMAL(18,3) NOT NULL,
    UnitId UNIQUEIDENTIFIER NOT NULL,
    RequiredDate DATE,
    Notes NVARCHAR(500),
    
    CreatedAt DATETIME2 DEFAULT GETUTCDATE(),
    IsDeleted BIT DEFAULT 0,
    
    FOREIGN KEY (RFQId) REFERENCES RequestsForQuotation(Id),
    FOREIGN KEY (ProductId) REFERENCES Products(Id),
    FOREIGN KEY (UnitId) REFERENCES Units(Id),
    INDEX IX_RFQItems_RFQ (RFQId)
);

-- جدول عروض أسعار الموردين
CREATE TABLE SupplierQuotations (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    Code NVARCHAR(50) NOT NULL UNIQUE,
    
    RFQId UNIQUEIDENTIFIER NOT NULL,
    SupplierId UNIQUEIDENTIFIER NOT NULL,
    SupplierQuotationNumber NVARCHAR(50),
    
    Date DATE NOT NULL,
    ReceivedDate DATE NOT NULL,
    ValidUntil DATE NOT NULL,
    
    -- الإجماليات
    Subtotal DECIMAL(18,2) DEFAULT 0,
    DiscountAmount DECIMAL(18,2) DEFAULT 0,
    TaxableAmount DECIMAL(18,2) DEFAULT 0,
    TaxAmount DECIMAL(18,2) DEFAULT 0,
    Total DECIMAL(18,2) DEFAULT 0,
    Currency NVARCHAR(3) DEFAULT 'SAR',
    
    -- الشروط
    PaymentTerms NVARCHAR(MAX),
    DeliveryTerms NVARCHAR(MAX),
    Warranty NVARCHAR(MAX),
    Validity INT,
    OtherTerms NVARCHAR(MAX),
    
    -- التقييم
    PriceScore DECIMAL(5,2),
    QualityScore DECIMAL(5,2),
    DeliveryScore DECIMAL(5,2),
    TermsScore DECIMAL(5,2),
    TotalScore DECIMAL(5,2),
    EvaluatedBy UNIQUEIDENTIFIER,
    EvaluatedDate DATETIME2,
    EvaluationNotes NVARCHAR(MAX),
    
    -- الحالة
    Status NVARCHAR(50) DEFAULT 'Draft' CHECK (Status IN ('Draft', 'Received', 'UnderEvaluation', 'Selected', 'Rejected', 'Expired')),
    RejectionReason NVARCHAR(500),
    
    -- Audit
    TenantId UNIQUEIDENTIFIER NOT NULL,
    CompanyId UNIQUEIDENTIFIER NOT NULL,
    CreatedAt DATETIME2 DEFAULT GETUTCDATE(),
    CreatedBy UNIQUEIDENTIFIER,
    UpdatedAt DATETIME2 DEFAULT GETUTCDATE(),
    UpdatedBy UNIQUEIDENTIFIER,
    IsDeleted BIT DEFAULT 0,
    
    FOREIGN KEY (RFQId) REFERENCES RequestsForQuotation(Id),
    FOREIGN KEY (SupplierId) REFERENCES Suppliers(Id),
    INDEX IX_SupplierQuotations_RFQ (RFQId),
    INDEX IX_SupplierQuotations_Supplier (SupplierId)
);

-- جدول أصناف عرض السعر
CREATE TABLE SupplierQuotationItems (
    Id UNIQUEIDENTIFIER PRIMARY KEY DEFAULT NEWID(),
    QuotationId UNIQUEIDENTIFIER NOT NULL,
    LineNumber INT NOT NULL,
    RFQLineNumber INT,
    
    ProductId UNIQUEIDENTIFIER,
    Description NVARCHAR(500) NOT NULL,
    
    Quantity DECIMAL(18,3) NOT NULL,
    UnitId UNIQUEIDENTIFIER NOT NULL,
    UnitPrice DECIMAL(18,2) NOT NULL,
    Discount DECIMAL(5,2) DEFAULT 0,
    DiscountAmount DECIMAL(18,2) DEFAULT 0,
    TaxRate DECIMAL(5,2) DEFAULT 0,
    TaxAmount DECIMAL(18,2) DEFAULT 0,
    TotalAmount DECIMAL(18,2) NOT NULL,
    
    DeliveryDays INT,
    Warranty NVARCHAR(200),
    Origin NVARCHAR(100),
    Brand NVARCHAR(100),
    Notes NVARCHAR(500),
    
    Score DECIMAL(5,2),
    IsSelected BIT DEFAULT 0,
    
    CreatedAt DATETIME2 DEFAULT GETUTCDATE(),
    IsDeleted BIT DEFAULT 0,
    
    FOREIGN KEY (QuotationId) REFERENCES SupplierQuotations(Id),
    FOREIGN KEY (ProductId) REFERENCES Products(Id),
    FOREIGN KEY (UnitId) REFERENCES Units(Id),
    INDEX IX_QuotationItems_Quotation (QuotationId)
);

-- سأكمل باقي الجداول في الرسالة التالية...
```

هكمل باقي الـ Database Schema والـ APIs والأمثلة في الرسالة التالية... ⏳