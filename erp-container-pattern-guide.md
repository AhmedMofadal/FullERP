# Container Pattern - الدليل الشامل
## نمط الحاوية للنماذج الموحدة في أنظمة ERP

---

## 📋 جدول المحتويات

1. [مقدمة - ما هو Container Pattern؟](#مقدمة)
2. [الفكرة الأساسية](#الفكرة-الأساسية)
3. [البنية المعمارية](#البنية-المعمارية)
4. [كيف يعمل Container مع الشاشات المختلفة](#كيف-يعمل-container)
5. [نظام الـ Metadata](#نظام-metadata)
6. [أمثلة تطبيقية](#أمثلة-تطبيقية)
7. [جدول المقارنة](#جدول-المقارنة)
8. [الفوائد والمزايا](#الفوائد)
9. [التطبيق العملي](#التطبيق-العملي)

---

## 🎯 مقدمة - ما هو Container Pattern؟ {#مقدمة}

**Container Pattern** هو نمط معماري يعتمد على فكرة إنشاء **حاوية واحدة موحدة** تحتوي على جميع الوظائف المشتركة بين الشاشات، بينما يتم تغيير **المحتوى الداخلي فقط** بناءً على نوع البيانات المعروضة.

### المشكلة التقليدية:
في الأنظمة التقليدية، يتم إنشاء كل شاشة بشكل منفصل:
- شاشة فاتورة مبيعات → كود كامل منفصل
- شاشة عميل → كود كامل منفصل
- شاشة منتج → كود كامل منفصل
- **النتيجة:** تكرار كود هائل، صعوبة في الصيانة، عدم توحيد

### الحل باستخدام Container Pattern:
**حاوية واحدة** تستخدم في جميع الشاشات، والاختلاف فقط في:
- البيانات المعروضة (تأتي من Metadata)
- طريقة عرض البيانات (حقول فقط، أو حقول + جدول)

---

## 💡 الفكرة الأساسية {#الفكرة-الأساسية}

```
┌────────────────────────────────────────────────────────────┐
│                    CONTAINER (ثابت)                        │
│  ┌──────────────────────────────────────────────────────┐  │
│  │      Action Toolbar - شريط الأدوات                  │  │
│  │  [حفظ] [حذف] [نسخ] [جديد] [طباعة] [إرسال] [مرفقات] │  │
│  │  [◄ أول] [◄ سابق] [تالي ►] [آخر ►]                  │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │              TABS - التبويبات                        │  │
│  │  [البيانات الأساسية] [السطور] [الإجماليات] [ملاحظات]│  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │         CONTENT - المحتوى (متغير حسب الكيان)        │  │
│  │                                                        │  │
│  │  🔸 فاتورة مبيعات:                                   │  │
│  │     - حقول الفاتورة (رقم، تاريخ، عميل، مبلغ)        │  │
│  │     - جدول السطور (المنتجات)                         │  │
│  │                                                        │  │
│  │  🔸 عميل:                                             │  │
│  │     - حقول العميل فقط (اسم، هاتف، عنوان)            │  │
│  │     - بدون جدول                                       │  │
│  │                                                        │  │
│  │  🔸 منتج:                                             │  │
│  │     - حقول المنتج (كود، اسم، سعر)                   │  │
│  │     - صور المنتج                                      │  │
│  │                                                        │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                             │
│  ┌──────────────────────────────────────────────────────┐  │
│  │       Status Bar - شريط الحالة                       │  │
│  │  الوضع: [عرض] | تغييرات غير محفوظة | آخر تعديل      │  │
│  └──────────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────────┘
```

---

## 🏗️ البنية المعمارية {#البنية-المعمارية}

### 1. المكونات الثابتة (في كل الشاشات):

```typescript
// Container Components (ثابتة لكل الشاشات)

1. ActionToolbar
   ├── SaveButton
   ├── DeleteButton
   ├── CopyButton
   ├── NewButton
   ├── NavigationButtons (First, Previous, Next, Last)
   ├── PrintButton
   ├── SendButton
   └── AttachmentsButton

2. FormTabs
   ├── Tab 1: البيانات الأساسية
   ├── Tab 2: السطور (إذا كان موجود)
   ├── Tab 3: الإجماليات (إذا كان موجود)
   └── Tab N: تبويبات إضافية

3. StatusBar
   ├── CurrentMode (عرض / تعديل / جديد)
   ├── DirtyIndicator (يوجد تغييرات غير محفوظة)
   └── LastModified (آخر تعديل)
```

### 2. المكونات المتغيرة (حسب نوع الشاشة):

```typescript
// Dynamic Components (تأتي من Metadata)

1. FormFields
   ├── TextField
   ├── NumberField
   ├── DateField
   ├── LookupField
   ├── BooleanField
   └── CustomFields

2. DataGrid (إذا كان موجود)
   ├── Columns (من Metadata)
   ├── EditableRows
   └── Calculations

3. CustomSections (حسب الحاجة)
   ├── ImageGallery (للمنتجات)
   ├── AttachmentsList (للموظفين)
   └── RelatedRecords
```

---

## 🔄 كيف يعمل Container مع الشاشات المختلفة {#كيف-يعمل-container}

### مثال 1️⃣: فاتورة مبيعات

```typescript
// استخدام Container لعرض فاتورة مبيعات
<FormContainer
  entityType="SalesInvoice"    // نوع الكيان
  entityId={invoiceId}          // رقم الفاتورة
  mode="view"                   // وضع العرض
/>

// ماذا يحدث داخلياً؟
// 1. Container يستدعي API لجلب metadata
GET /api/metadata/SalesInvoice

// 2. Container يستدعي API لجلب البيانات
GET /api/sales/invoices/{invoiceId}

// 3. Container يعرض:
//    ✅ Toolbar بكل الأزرار
//    ✅ Tabs: [البيانات الأساسية] [السطور] [الإجماليات] [ملاحظات]
//    ✅ Form Fields: رقم الفاتورة، التاريخ، العميل، المبلغ
//    ✅ DataGrid: جدول سطور الفاتورة (المنتجات)
//    ✅ Totals Section: الإجمالي الفرعي، الضريبة، الإجمالي
//    ✅ Navigation Buttons: للتنقل بين الفواتير
```

**الشكل النهائي:**
```
┌─────────────────────────────────────────────────────┐
│ [حفظ] [حذف] [طباعة] [◄◄] [◄] [►] [►►]              │
├─────────────────────────────────────────────────────┤
│ [البيانات] [السطور] [الإجماليات] [ملاحظات]        │
├─────────────────────────────────────────────────────┤
│ رقم الفاتورة: INV-001      التاريخ: 2025-01-15     │
│ العميل: شركة التقنية       المبلغ: 15,500 ر.س      │
│                                                      │
│ 📊 جدول السطور:                                     │
│ ┌────┬──────────┬──────┬────────┬──────────┐       │
│ │ # │ المنتج   │ الكمية│ السعر │ الإجمالي │       │
│ ├────┼──────────┼──────┼────────┼──────────┤       │
│ │ 1 │ لابتوب    │  5   │ 2,500 │ 12,500  │       │
│ │ 2 │ ماوس      │ 10   │   50  │    500  │       │
│ └────┴──────────┴──────┴────────┴──────────┘       │
└─────────────────────────────────────────────────────┘
```

---

### مثال 2️⃣: عميل (نفس الـ Container!)

```typescript
// نفس الـ Container بالضبط، لكن entityType مختلف
<FormContainer
  entityType="Customer"         // نوع مختلف
  entityId={customerId}
  mode="view"
/>

// ماذا يحدث؟
// 1. Container يجلب metadata للعميل
GET /api/metadata/Customer

// 2. Container يجلب بيانات العميل
GET /api/sales/customers/{customerId}

// 3. Container يعرض:
//    ✅ نفس الـ Toolbar (حفظ، حذف، طباعة...)
//    ✅ Tabs: [البيانات الأساسية] فقط (بدون سطور)
//    ✅ Form Fields: اسم العميل، الهاتف، العنوان، البريد
//    ❌ بدون DataGrid (لأن العميل ليس له سطور)
//    ✅ نفس الـ Navigation Buttons
```

**الشكل النهائي:**
```
┌─────────────────────────────────────────────────────┐
│ [حفظ] [حذف] [طباعة] [◄◄] [◄] [►] [►►]              │
├─────────────────────────────────────────────────────┤
│ [البيانات الأساسية]                                 │
├─────────────────────────────────────────────────────┤
│ كود العميل: C-001          اسم العميل: شركة التقنية│
│ الهاتف: 0501234567          البريد: info@tech.com  │
│ العنوان: الرياض، المملكة العربية السعودية          │
│ حد الائتمان: 100,000 ر.س   الرصيد: 25,000 ر.س     │
│                                                      │
│ (بدون جدول - لأن العميل ليس له سطور)               │
└─────────────────────────────────────────────────────┘
```

---

### مثال 3️⃣: منتج (نفس الـ Container!)

```typescript
<FormContainer
  entityType="Product"
  entityId={productId}
  mode="view"
/>

// Container يعرض:
//    ✅ نفس الـ Toolbar
//    ✅ Tabs: [البيانات] [الأسعار] [الصور] [البدائل]
//    ✅ Form Fields: كود المنتج، الاسم، الوصف، السعر
//    ✅ Image Gallery: صور المنتج (component خاص)
//    ✅ Price Grid: جدول الأسعار حسب الكمية
```

**الشكل النهائي:**
```
┌─────────────────────────────────────────────────────┐
│ [حفظ] [حذف] [طباعة] [◄◄] [◄] [►] [►►]              │
├─────────────────────────────────────────────────────┤
│ [البيانات] [الأسعار] [الصور] [البدائل]             │
├─────────────────────────────────────────────────────┤
│ كود المنتج: P-001           اسم المنتج: لابتوب     │
│ الفئة: أجهزة كمبيوتر        السعر: 2,500 ر.س       │
│                                                      │
│ 🖼️ معرض الصور:                                      │
│ [صورة 1] [صورة 2] [صورة 3]                         │
│                                                      │
│ 💰 جدول الأسعار:                                     │
│ من 1-10 قطع → 2,500 ر.س                            │
│ من 11-50 قطع → 2,300 ر.س                           │
│ أكثر من 50 → 2,100 ر.س                              │
└─────────────────────────────────────────────────────┘
```

---

### مثال 4️⃣: قيد يومية (نفس الـ Container!)

```typescript
<FormContainer
  entityType="JournalEntry"
  entityId={entryId}
  mode="view"
/>

// Container يعرض:
//    ✅ نفس الـ Toolbar
//    ✅ Tabs: [البيانات] [السطور]
//    ✅ Form Fields: رقم القيد، التاريخ، الوصف
//    ✅ DataGrid: جدول سطور القيد (مدين/دائن)
//    ✅ Validation: مدين = دائن
//    ✅ Footer: إجمالي مدين، إجمالي دائن
```

**الشكل النهائي:**
```
┌─────────────────────────────────────────────────────┐
│ [حفظ] [حذف] [طباعة] [◄◄] [◄] [►] [►►]              │
├─────────────────────────────────────────────────────┤
│ [البيانات الأساسية] [السطور]                       │
├─────────────────────────────────────────────────────┤
│ رقم القيد: JV-001          التاريخ: 2025-01-15      │
│ الوصف: قيد افتتاحي         الحالة: مرحل             │
│                                                      │
│ 📊 سطور القيد:                                      │
│ ┌────┬──────────────┬──────────┬──────────┐         │
│ │ # │ الحساب       │ مدين     │ دائن     │         │
│ ├────┼──────────────┼──────────┼──────────┤         │
│ │ 1 │ الصندوق      │ 50,000  │    -     │         │
│ │ 2 │ رأس المال    │    -     │ 50,000  │         │
│ ├────┴──────────────┼──────────┼──────────┤         │
│ │      الإجمالي    │ 50,000  │ 50,000  │         │
│ └──────────────────┴──────────┴──────────┘         │
│                                                      │
│ ✅ القيد متوازن (مدين = دائن)                       │
└─────────────────────────────────────────────────────┘
```

---

### مثال 5️⃣: موظف (نفس الـ Container!)

```typescript
<FormContainer
  entityType="Employee"
  entityId={employeeId}
  mode="view"
/>

// Container يعرض:
//    ✅ نفس الـ Toolbar
//    ✅ Tabs: [بيانات شخصية] [معلومات وظيفية] [الرواتب] [المرفقات]
//    ✅ Form Fields: الاسم، الهوية، تاريخ الميلاد، الوظيفة
//    ✅ Attachments Section: السيرة الذاتية، العقد، الشهادات
```

---

## 📊 نظام الـ Metadata {#نظام-metadata}

### كيف يعرف الـ Container ماذا يعرض؟

**الإجابة: من الـ Metadata المخزنة في قاعدة البيانات!**

### مثال: Metadata لفاتورة المبيعات

```json
{
  "entityType": "SalesInvoice",
  "entityName": "فاتورة مبيعات",
  "entityName_AR": "فاتورة مبيعات",
  "tableName": "Sales.SalesInvoices",
  "primaryKey": "InvoiceId",
  "displayField": "InvoiceNumber",
  
  "permissions": {
    "canCreate": true,
    "canEdit": true,
    "canDelete": true,
    "canPrint": true,
    "canExport": true
  },
  
  "fields": [
    {
      "fieldName": "invoiceNumber",
      "fieldLabel": "رقم الفاتورة",
      "fieldLabel_AR": "رقم الفاتورة",
      "dataType": "String",
      "maxLength": 50,
      "isRequired": true,
      "isReadOnly": true,
      "showInForm": true,
      "showInList": true,
      "displayOrder": 1,
      "tabName": "main",
      "columnSpan": 6
    },
    {
      "fieldName": "invoiceDate",
      "fieldLabel": "تاريخ الفاتورة",
      "dataType": "Date",
      "isRequired": true,
      "showInForm": true,
      "showInList": true,
      "displayOrder": 2,
      "tabName": "main",
      "columnSpan": 6
    },
    {
      "fieldName": "customerId",
      "fieldLabel": "العميل",
      "dataType": "Lookup",
      "isRequired": true,
      "lookupConfig": {
        "lookupType": "Entity",
        "lookupSource": "Customer",
        "valueField": "CustomerId",
        "displayField": "CustomerName",
        "searchFields": ["CustomerCode", "CustomerName"]
      },
      "showInForm": true,
      "showInList": true,
      "displayOrder": 3,
      "tabName": "main",
      "columnSpan": 12
    },
    {
      "fieldName": "totalAmount",
      "fieldLabel": "الإجمالي",
      "dataType": "Decimal",
      "precision": 19,
      "scale": 2,
      "isReadOnly": true,
      "displayFormat": "currency",
      "currencyCode": "SAR",
      "showInForm": true,
      "showInList": true,
      "displayOrder": 4,
      "tabName": "main",
      "columnSpan": 6
    }
  ],
  
  "tabs": [
    {
      "tabName": "main",
      "tabLabel": "البيانات الأساسية",
      "tabLabel_AR": "البيانات الأساسية",
      "displayOrder": 1
    },
    {
      "tabName": "lines",
      "tabLabel": "السطور",
      "tabLabel_AR": "السطور",
      "displayOrder": 2
    },
    {
      "tabName": "totals",
      "tabLabel": "الإجماليات",
      "tabLabel_AR": "الإجماليات",
      "displayOrder": 3
    },
    {
      "tabName": "notes",
      "tabLabel": "ملاحظات",
      "tabLabel_AR": "ملاحظات",
      "displayOrder": 4
    }
  ],
  
  "childGrids": [
    {
      "gridName": "lines",
      "entityType": "SalesInvoiceLine",
      "parentField": "InvoiceId",
      "displayInTab": "lines",
      "allowAdd": true,
      "allowEdit": true,
      "allowDelete": true,
      "columns": [
        {
          "fieldName": "itemId",
          "fieldLabel": "المنتج",
          "dataType": "Lookup",
          "width": 200
        },
        {
          "fieldName": "quantity",
          "fieldLabel": "الكمية",
          "dataType": "Decimal",
          "width": 100
        },
        {
          "fieldName": "unitPrice",
          "fieldLabel": "السعر",
          "dataType": "Decimal",
          "width": 120
        },
        {
          "fieldName": "lineTotal",
          "fieldLabel": "الإجمالي",
          "dataType": "Decimal",
          "width": 120,
          "isReadOnly": true
        }
      ]
    }
  ],
  
  "availableViews": ["list", "kanban", "calendar"],
  
  "listViewConfig": {
    "defaultSort": "InvoiceDate DESC",
    "pageSize": 25,
    "allowExport": true,
    "allowPrint": true
  },
  
  "kanbanConfig": {
    "groupByField": "Status",
    "columns": [
      { "id": "Draft", "title": "مسودة", "color": "secondary" },
      { "id": "Posted", "title": "مرحل", "color": "primary" },
      { "id": "Paid", "title": "مدفوع", "color": "success" }
    ]
  }
}
```

### مثال: Metadata للعميل

```json
{
  "entityType": "Customer",
  "entityName": "عميل",
  "tableName": "Sales.Customers",
  "primaryKey": "CustomerId",
  
  "fields": [
    {
      "fieldName": "customerCode",
      "fieldLabel": "كود العميل",
      "dataType": "String",
      "isRequired": true,
      "showInForm": true,
      "showInList": true
    },
    {
      "fieldName": "customerName",
      "fieldLabel": "اسم العميل",
      "dataType": "String",
      "isRequired": true,
      "isLocalizable": true,
      "showInForm": true,
      "showInList": true
    },
    {
      "fieldName": "phone",
      "fieldLabel": "الهاتف",
      "dataType": "String",
      "showInForm": true,
      "showInList": true
    },
    {
      "fieldName": "email",
      "fieldLabel": "البريد الإلكتروني",
      "dataType": "String",
      "validationRules": ["email"],
      "showInForm": true,
      "showInList": true
    }
  ],
  
  "tabs": [
    {
      "tabName": "main",
      "tabLabel": "البيانات الأساسية"
    }
  ],
  
  "childGrids": [],  // ⚠️ لا يوجد جدول سطور!
  
  "availableViews": ["list", "kanban", "grid"]
}
```

---

## 📊 جدول المقارنة {#جدول-المقارنة}

| الشاشة | نفس الـ Toolbar؟ | نفس الـ Navigation؟ | الحقول | هل يوجد Grid؟ | التبويبات |
|--------|------------------|--------------------|--------------------|--------------|-----------|
| **فاتورة مبيعات** | ✅ نعم | ✅ نعم | رقم، تاريخ، عميل، مبلغ | ✅ نعم (سطور الفاتورة) | 4 تبويبات |
| **عميل** | ✅ نعم | ✅ نعم | كود، اسم، هاتف، عنوان | ❌ لا | تبويب واحد |
| **منتج** | ✅ نعم | ✅ نعم | كود، اسم، سعر، وصف | ✅ نعم (الأسعار) | 4 تبويبات |
| **موظف** | ✅ نعم | ✅ نعم | اسم، هوية، وظيفة، راتب | ❌ لا | 4 تبويبات |
| **قيد يومية** | ✅ نعم | ✅ نعم | رقم، تاريخ، وصف | ✅ نعم (مدين/دائن) | 2 تبويبات |
| **أمر شراء** | ✅ نعم | ✅ نعم | رقم، تاريخ، مورد | ✅ نعم (أصناف الشراء) | 3 تبويبات |
| **مشروع** | ✅ نعم | ✅ نعم | كود، اسم، تواريخ | ✅ نعم (المهام) | 5 تبويبات |

**النتيجة:** كلهم يستخدمون **نفس الـ `<FormContainer>`!**

---

## ✨ الفوائد والمزايا {#الفوائد}

### 1. **كتابة الكود مرة واحدة**
```typescript
// بدلاً من كتابة 100 شاشة منفصلة:
<InvoiceScreen />
<CustomerScreen />
<ProductScreen />
// ... 97 شاشة أخرى

// نكتب Container واحد فقط:
<FormContainer entityType="SalesInvoice" />
<FormContainer entityType="Customer" />
<FormContainer entityType="Product" />
// نفس الكود، entityType مختلف فقط!
```

### 2. **Consistency - التوحيد الكامل**
- كل الشاشات بنفس الشكل
- نفس الأزرار في نفس الأماكن
- نفس سلوك التنقل
- نفس اختصارات الكيبورد
- تجربة مستخدم موحدة 100%

### 3. **سهولة الصيانة**
```typescript
// تعديل واحد في Container:
const ActionToolbar = () => {
  return (
    <div>
      <SaveButton />
      <DeleteButton />
      <NewFeatureButton />  // ✅ زر جديد!
    </div>
  );
};

// النتيجة: الزر الجديد يظهر في كل 100 شاشة تلقائياً!
```

### 4. **Metadata-Driven Development**
```typescript
// إضافة شاشة جديدة = إضافة metadata فقط
// لا يوجد كود برمجي جديد!

// خطوات إضافة شاشة "طلب صيانة":
// 1. إدخال metadata في قاعدة البيانات
INSERT INTO Metadata.EntityDefinitions 
VALUES ('MaintenanceRequest', 'طلب صيانة', ...)

// 2. إدخال الحقول
INSERT INTO Metadata.FieldDefinitions 
VALUES ('MaintenanceRequest', 'requestNumber', 'رقم الطلب', ...)

// 3. استخدام Container
<FormContainer entityType="MaintenanceRequest" />

// ✅ انتهى! الشاشة جاهزة بدون كتابة أي كود!
```

### 5. **Reusability - إعادة الاستخدام**
```typescript
// كل component قابل لإعادة الاستخدام:
<SaveButton />        // استخدمه في أي مكان
<DateField />         // استخدمه في أي مكان
<DataGrid />          // استخدمه في أي مكان
<ActionToolbar />     // استخدمه في أي مكان
```

### 6. **سهولة التطوير**
```typescript
// المطور الجديد:
// ❌ قديماً: يحتاج تعلم 100 شاشة مختلفة
// ✅ حديثاً: يتعلم Container واحد فقط!

// إضافة ميزة جديدة:
// ❌ قديماً: تعديل 100 ملف
// ✅ حديثاً: تعديل ملف واحد
```

### 7. **Testing سهل**
```typescript
// ❌ قديماً: اختبار 100 شاشة منفصلة
describe('All Screens', () => {
  it('should test invoice screen', () => {...});
  it('should test customer screen', () => {...});
  // ... 98 اختبار آخر
});

// ✅ حديثاً: اختبار Container واحد
describe('FormContainer', () => {
  it('should work with any entityType', () => {
    render(<FormContainer entityType="SalesInvoice" />);
    render(<FormContainer entityType="Customer" />);
    render(<FormContainer entityType="Product" />);
    // ✅ كلهم يعملون!
  });
});
```

---

## 💻 التطبيق العملي {#التطبيق-العملي}

### 1. تعريف الـ Container

```typescript
// src/components/containers/FormContainer.tsx
interface FormContainerProps {
  entityType: string;    // "SalesInvoice", "Customer", "Product", etc.
  entityId?: number;     // ID السجل (undefined للسجل الجديد)
  mode?: FormMode;       // "view", "edit", "new"
}

export const FormContainer: React.FC<FormContainerProps> = ({
  entityType,
  entityId,
  mode = 'view'
}) => {
  // 1. جلب Metadata
  const { metadata, loading } = useMetadata(entityType);
  
  // 2. جلب البيانات
  const { data, save, delete } = useCRUD(entityType, entityId);
  
  // 3. العرض
  return (
    <div>
      <ActionToolbar {...toolbarProps} />
      <FormTabs tabs={metadata.tabs}>
        <DynamicForm 
          fields={metadata.fields}
          data={data}
        />
        {metadata.hasChildGrid && (
          <DataGrid config={metadata.childGridConfig} />
        )}
      </FormTabs>
      <StatusBar {...statusProps} />
    </div>
  );
};
```

### 2. الاستخدام في الصفحات

```typescript
// src/pages/sales/InvoicePage.tsx
export const InvoicePage = () => {
  const { id } = useParams();
  
  return (
    <FormContainer
      entityType="SalesInvoice"
      entityId={id}
    />
  );
};

// src/pages/sales/CustomerPage.tsx
export const CustomerPage = () => {
  const { id } = useParams();
  
  return (
    <FormContainer
      entityType="Customer"
      entityId={id}
    />
  );
};

// src/pages/inventory/ProductPage.tsx
export const ProductPage = () => {
  const { id } = useParams();
  
  return (
    <FormContainer
      entityType="Product"
      entityId={id}
    />
  );
};

// ✅ ثلاث صفحات، نفس Container!
```

### 3. الـ Routing

```typescript
// src/routes/index.tsx
const routes = [
  {
    path: '/sales/invoices/:id',
    element: <FormContainer entityType="SalesInvoice" />
  },
  {
    path: '/sales/customers/:id',
    element: <FormContainer entityType="Customer" />
  },
  {
    path: '/inventory/products/:id',
    element: <FormContainer entityType="Product" />
  },
  {
    path: '/hr/employees/:id',
    element: <FormContainer entityType="Employee" />
  },
  {
    path: '/finance/journal-entries/:id',
    element: <FormContainer entityType="JournalEntry" />
  }
  // ... باقي الـ routes
];
```

---

## 🎯 الخلاصة

### ما هو Container Pattern؟
**حاوية واحدة موحدة** تحتوي على:
- ✅ Action Toolbar (حفظ، حذف، طباعة...)
- ✅ Navigation Buttons (أول، سابق، تالي، آخر)
- ✅ Tabs System
- ✅ Status Bar
- ✅ Dynamic Form Renderer
- ✅ Optional DataGrid

### كيف يختلف المحتوى؟
من خلال **Metadata** المخزنة في قاعدة البيانات:
- الحقول المعروضة
- التبويبات
- هل يوجد DataGrid أم لا
- طريقة العرض المتاحة (List, Kanban, Calendar)

### لماذا هذا النمط؟
- **كتابة مرة واحدة** → استخدام في 100 شاشة
- **Consistency** → تجربة موحدة
- **Easy Maintenance** → تعديل في مكان واحد
- **Metadata-Driven** → إضافة شاشات بدون كود
- **Reusability** → كل component قابل لإعادة الاستخدام

---

## 📚 المراجع والموارد

### ملفات مرتبطة:
- `01 - System Architecture & Core Infrastructure Blueprint`
- `02 - Metadata Framework & Localization Engine`
- `08 - Frontend Component Architecture & Container Pattern`
- `09 - Field Components & Multi-View System`

### مفاهيم مرتبطة:
- Metadata-Driven UI
- Component-Based Architecture
- Dynamic Forms
- Single Responsibility Principle
- DRY (Don't Repeat Yourself)

---

**تم إنشاء هذا المستند بواسطة:** Claude Sonnet 4.5  
**التاريخ:** 2025-01-02  
**النسخة:** 1.0

