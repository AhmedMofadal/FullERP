# Frontend Component Architecture & Container Pattern
## بنية المكونات للواجهة الأمامية ونمط الحاوية

## 1. Architecture Overview - نظرة عامة على المعمارية

```
┌────────────────────────────────────────────────────────────────┐
│                     Application Shell                          │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐        │
│  │   TopBar     │  │   Sidebar    │  │  Breadcrumb  │        │
│  └──────────────┘  └──────────────┘  └──────────────┘        │
└────────────────────────────────────────────────────────────────┘
                              ↓
┌────────────────────────────────────────────────────────────────┐
│                    Module Container                             │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │                  View Selector                            │ │
│  │  [List] [Kanban] [Calendar] [Gantt] [Cards]              │ │
│  └──────────────────────────────────────────────────────────┘ │
│                              ↓                                  │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │              Dynamic View Renderer                        │ │
│  │  - List View (DataGrid)                                   │ │
│  │  - Kanban View (DragDrop Cards)                          │ │
│  │  - Calendar View                                          │ │
│  │  - Custom Views                                           │ │
│  └──────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────┘
                              ↓ (Click Record)
┌────────────────────────────────────────────────────────────────┐
│                   Form Container (Master)                       │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │                    Action Toolbar                         │ │
│  │ [Save] [Delete] [Archive] [Copy] [New] [Cancel] [Clear]  │ │
│  │ [◄First] [◄Prev] [Next►] [Last►] [Print] [Send] [📎]    │ │
│  └──────────────────────────────────────────────────────────┘ │
│                              ↓                                  │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │              Metadata-Driven Form Renderer                │ │
│  │  ┌────────────────────────────────────────────────────┐  │ │
│  │  │         Dynamic Field Components                    │  │ │
│  │  │  - Text Fields                                      │  │ │
│  │  │  - Number Fields                                    │  │ │
│  │  │  - Date Pickers                                     │  │ │
│  │  │  - Lookups/Dropdowns                               │  │ │
│  │  │  - Rich Text                                        │  │ │
│  │  │  - Custom Components                                │  │ │
│  │  └────────────────────────────────────────────────────┘  │ │
│  │  ┌────────────────────────────────────────────────────┐  │ │
│  │  │         Tabs (if applicable)                        │  │ │
│  │  │  [Tab 1] [Tab 2] [Tab 3]                           │  │ │
│  │  └────────────────────────────────────────────────────┘  │ │
│  │  ┌────────────────────────────────────────────────────┐  │ │
│  │  │         DataGrid/Details (if applicable)            │  │ │
│  │  │  - Invoice Lines                                    │  │ │
│  │  │  - Order Items                                      │  │ │
│  │  └────────────────────────────────────────────────────┘  │ │
│  └──────────────────────────────────────────────────────────┘ │
└────────────────────────────────────────────────────────────────┘
```

## 2. Core Components Library - مكتبة المكونات الأساسية

### 2.1 Directory Structure

```
src/
├── components/
│   ├── core/                      # Core reusable components
│   │   ├── buttons/
│   │   │   ├── ActionButton.tsx
│   │   │   ├── SaveButton.tsx
│   │   │   ├── DeleteButton.tsx
│   │   │   ├── NavigationButton.tsx
│   │   │   └── index.ts
│   │   ├── fields/
│   │   │   ├── TextField.tsx
│   │   │   ├── NumberField.tsx
│   │   │   ├── DateField.tsx
│   │   │   ├── LookupField.tsx
│   │   │   ├── BooleanField.tsx
│   │   │   ├── RichTextField.tsx
│   │   │   └── index.ts
│   │   ├── layout/
│   │   │   ├── FormSection.tsx
│   │   │   ├── FormGroup.tsx
│   │   │   ├── TabPanel.tsx
│   │   │   └── index.ts
│   │   └── feedback/
│   │       ├── LoadingSpinner.tsx
│   │       ├── ErrorBoundary.tsx
│   │       ├── Toast.tsx
│   │       └── index.ts
│   │
│   ├── containers/                # Container components
│   │   ├── FormContainer.tsx      # Master form container
│   │   ├── ListContainer.tsx      # List view container
│   │   ├── KanbanContainer.tsx    # Kanban view container
│   │   └── index.ts
│   │
│   ├── dynamic/                   # Dynamic/Metadata-driven components
│   │   ├── DynamicField.tsx
│   │   ├── DynamicForm.tsx
│   │   ├── DynamicGrid.tsx
│   │   ├── DynamicView.tsx
│   │   └── index.ts
│   │
│   └── modules/                   # Module-specific components
│       ├── sales/
│       │   ├── InvoiceForm.tsx    # Specific layout for invoice
│       │   ├── InvoiceLineGrid.tsx
│       │   └── index.ts
│       └── finance/
│           ├── JournalEntryForm.tsx
│           └── index.ts
│
├── hooks/                         # Custom hooks
│   ├── useFormContainer.ts
│   ├── useMetadata.ts
│   ├── useCRUD.ts
│   ├── useNavigation.ts
│   └── index.ts
│
└── types/
    ├── metadata.types.ts
    ├── form.types.ts
    └── index.ts
```

## 3. Form Container - الحاوية الرئيسية للنماذج

### 3.1 FormContainer Component

```typescript
// src/components/containers/FormContainer.tsx
import React, { useState, useCallback, useEffect } from 'react';
import { Box, Paper } from '@mui/material';
import { ActionToolbar } from './ActionToolbar';
import { DynamicForm } from '../dynamic/DynamicForm';
import { useFormContainer } from '@/hooks/useFormContainer';
import { FormMetadata, FormMode } from '@/types/metadata.types';

interface FormContainerProps {
  entityType: string;
  entityId?: number | string;
  mode?: FormMode;
  onSave?: (data: any) => void;
  onClose?: () => void;
  metadata?: FormMetadata;
  customLayout?: React.ComponentType<any>;
}

export const FormContainer: React.FC<FormContainerProps> = ({
  entityType,
  entityId,
  mode = 'view',
  onSave,
  onClose,
  metadata,
  customLayout
}) => {
  const {
    // Data
    formData,
    setFormData,
    originalData,
    isDirty,
    isLoading,
    isSaving,
    errors,
    
    // Metadata
    formMetadata,
    fieldMetadata,
    
    // Actions
    handleSave,
    handleDelete,
    handleArchive,
    handleCopy,
    handleNew,
    handleCancel,
    handleClear,
    handlePrint,
    handleSend,
    handleAttachments,
    
    // Navigation
    handleFirst,
    handlePrevious,
    handleNext,
    handleLast,
    canNavigatePrevious,
    canNavigateNext,
    
    // Permissions
    canEdit,
    canDelete,
    canCreate,
    canPrint,
    
    // Status
    currentMode,
    setCurrentMode
  } = useFormContainer({
    entityType,
    entityId,
    initialMode: mode,
    metadata,
    onSaveCallback: onSave
  });

  // Render custom layout if provided, otherwise use dynamic form
  const FormContent = customLayout || DynamicForm;

  return (
    <Box sx={{ height: '100%', display: 'flex', flexDirection: 'column' }}>
      {/* Action Toolbar */}
      <ActionToolbar
        mode={currentMode}
        isDirty={isDirty}
        isLoading={isLoading}
        isSaving={isSaving}
        canEdit={canEdit}
        canDelete={canDelete}
        canCreate={canCreate}
        canPrint={canPrint}
        canNavigatePrevious={canNavigatePrevious}
        canNavigateNext={canNavigateNext}
        onSave={handleSave}
        onDelete={handleDelete}
        onArchive={handleArchive}
        onCopy={handleCopy}
        onNew={handleNew}
        onCancel={handleCancel}
        onClear={handleClear}
        onPrint={handlePrint}
        onSend={handleSend}
        onAttachments={handleAttachments}
        onFirst={handleFirst}
        onPrevious={handlePrevious}
        onNext={handleNext}
        onLast={handleLast}
        onModeChange={setCurrentMode}
      />

      {/* Form Content */}
      <Paper sx={{ flex: 1, overflow: 'auto', p: 2 }}>
        {isLoading ? (
          <LoadingSpinner />
        ) : (
          <FormContent
            entityType={entityType}
            data={formData}
            onChange={setFormData}
            metadata={formMetadata}
            fieldMetadata={fieldMetadata}
            mode={currentMode}
            errors={errors}
          />
        )}
      </Paper>
    </Box>
  );
};
```

### 3.2 ActionToolbar Component

```typescript
// src/components/containers/ActionToolbar.tsx
import React from 'react';
import { Box, Toolbar, Divider } from '@mui/material';
import { 
  SaveButton, 
  DeleteButton, 
  ArchiveButton,
  CopyButton,
  NewButton,
  CancelButton,
  ClearButton,
  PrintButton,
  SendButton,
  AttachmentsButton,
  NavigationButtons,
  EditButton,
  ViewButton
} from '../core/buttons';
import { FormMode } from '@/types/metadata.types';

interface ActionToolbarProps {
  mode: FormMode;
  isDirty: boolean;
  isLoading: boolean;
  isSaving: boolean;
  canEdit: boolean;
  canDelete: boolean;
  canCreate: boolean;
  canPrint: boolean;
  canNavigatePrevious: boolean;
  canNavigateNext: boolean;
  onSave: () => void;
  onDelete: () => void;
  onArchive: () => void;
  onCopy: () => void;
  onNew: () => void;
  onCancel: () => void;
  onClear: () => void;
  onPrint: () => void;
  onSend: () => void;
  onAttachments: () => void;
  onFirst: () => void;
  onPrevious: () => void;
  onNext: () => void;
  onLast: () => void;
  onModeChange: (mode: FormMode) => void;
}

export const ActionToolbar: React.FC<ActionToolbarProps> = (props) => {
  const {
    mode,
    isDirty,
    isSaving,
    canEdit,
    canDelete,
    canCreate,
    canPrint,
    canNavigatePrevious,
    canNavigateNext,
    onModeChange,
    ...actions
  } = props;

  return (
    <Toolbar sx={{ gap: 1, borderBottom: 1, borderColor: 'divider' }}>
      {/* Main Actions Group */}
      <Box sx={{ display: 'flex', gap: 0.5 }}>
        {mode === 'view' && canEdit && (
          <EditButton onClick={() => onModeChange('edit')} />
        )}
        
        {(mode === 'edit' || mode === 'new') && (
          <>
            <SaveButton 
              onClick={actions.onSave}
              disabled={!isDirty || isSaving}
              loading={isSaving}
            />
            <CancelButton 
              onClick={actions.onCancel}
              disabled={isSaving}
            />
          </>
        )}
        
        {mode === 'view' && (
          <>
            <DeleteButton 
              onClick={actions.onDelete}
              disabled={!canDelete}
            />
            <ArchiveButton onClick={actions.onArchive} />
          </>
        )}
      </Box>

      <Divider orientation="vertical" flexItem />

      {/* Document Actions */}
      <Box sx={{ display: 'flex', gap: 0.5 }}>
        <CopyButton 
          onClick={actions.onCopy}
          disabled={mode === 'new'}
        />
        <NewButton 
          onClick={actions.onNew}
          disabled={!canCreate}
        />
        {mode === 'edit' && (
          <ClearButton onClick={actions.onClear} />
        )}
      </Box>

      <Divider orientation="vertical" flexItem />

      {/* Navigation */}
      {mode === 'view' && (
        <NavigationButtons
          onFirst={actions.onFirst}
          onPrevious={actions.onPrevious}
          onNext={actions.onNext}
          onLast={actions.onLast}
          canNavigatePrevious={canNavigatePrevious}
          canNavigateNext={canNavigateNext}
        />
      )}

      <Box sx={{ flex: 1 }} />

      {/* Utility Actions */}
      <Box sx={{ display: 'flex', gap: 0.5 }}>
        <PrintButton 
          onClick={actions.onPrint}
          disabled={!canPrint || mode === 'new'}
        />
        <SendButton 
          onClick={actions.onSend}
          disabled={mode === 'new'}
        />
        <AttachmentsButton 
          onClick={actions.onAttachments}
          disabled={mode === 'new'}
        />
      </Box>
    </Toolbar>
  );
};
```

## 4. Reusable Button Components - مكونات الأزرار القابلة لإعادة الاستخدام

### 4.1 Base Action Button

```typescript
// src/components/core/buttons/ActionButton.tsx
import React from 'react';
import { Button, ButtonProps, Tooltip, CircularProgress } from '@mui/material';
import { useLocalization } from '@/hooks/useLocalization';

interface ActionButtonProps extends Omit<ButtonProps, 'onClick'> {
  label?: string;
  icon?: React.ReactNode;
  tooltip?: string;
  loading?: boolean;
  confirmMessage?: string;
  onClick: () => void | Promise<void>;
  shortcut?: string;
}

export const ActionButton: React.FC<ActionButtonProps> = ({
  label,
  icon,
  tooltip,
  loading,
  confirmMessage,
  onClick,
  shortcut,
  disabled,
  ...buttonProps
}) => {
  const { t } = useLocalization();
  const [isProcessing, setIsProcessing] = React.useState(false);

  const handleClick = async () => {
    if (confirmMessage) {
      const confirmed = await window.confirm(confirmMessage);
      if (!confirmed) return;
    }

    setIsProcessing(true);
    try {
      await onClick();
    } finally {
      setIsProcessing(false);
    }
  };

  // Register keyboard shortcut
  React.useEffect(() => {
    if (!shortcut || disabled) return;

    const handleKeyPress = (e: KeyboardEvent) => {
      const keys = shortcut.split('+');
      const ctrl = keys.includes('Ctrl') && (e.ctrlKey || e.metaKey);
      const shift = keys.includes('Shift') && e.shiftKey;
      const alt = keys.includes('Alt') && e.altKey;
      const key = keys[keys.length - 1].toLowerCase();

      if (ctrl === e.ctrlKey && 
          shift === e.shiftKey && 
          alt === e.altKey && 
          e.key.toLowerCase() === key) {
        e.preventDefault();
        handleClick();
      }
    };

    window.addEventListener('keydown', handleKeyPress);
    return () => window.removeEventListener('keydown', handleKeyPress);
  }, [shortcut, disabled]);

  const button = (
    <Button
      variant="contained"
      disabled={disabled || loading || isProcessing}
      onClick={handleClick}
      startIcon={
        (loading || isProcessing) ? (
          <CircularProgress size={20} />
        ) : icon
      }
      {...buttonProps}
    >
      {label}
    </Button>
  );

  if (tooltip) {
    return (
      <Tooltip title={`${tooltip}${shortcut ? ` (${shortcut})` : ''}`}>
        <span>{button}</span>
      </Tooltip>
    );
  }

  return button;
};
```

### 4.2 Specific Action Buttons

```typescript
// src/components/core/buttons/SaveButton.tsx
import React from 'react';
import { Save as SaveIcon } from '@mui/icons-material';
import { ActionButton } from './ActionButton';
import { useLocalization } from '@/hooks/useLocalization';

interface SaveButtonProps {
  onClick: () => void | Promise<void>;
  loading?: boolean;
  disabled?: boolean;
}

export const SaveButton: React.FC<SaveButtonProps> = (props) => {
  const { t } = useLocalization();
  
  return (
    <ActionButton
      label={t('common.save')}
      icon={<SaveIcon />}
      tooltip={t('common.saveTooltip')}
      shortcut="Ctrl+S"
      color="primary"
      {...props}
    />
  );
};

// src/components/core/buttons/DeleteButton.tsx
import React from 'react';
import { Delete as DeleteIcon } from '@mui/icons-material';
import { ActionButton } from './ActionButton';
import { useLocalization } from '@/hooks/useLocalization';

interface DeleteButtonProps {
  onClick: () => void | Promise<void>;
  disabled?: boolean;
}

export const DeleteButton: React.FC<DeleteButtonProps> = (props) => {
  const { t } = useLocalization();
  
  return (
    <ActionButton
      label={t('common.delete')}
      icon={<DeleteIcon />}
      tooltip={t('common.deleteTooltip')}
      confirmMessage={t('common.deleteConfirm')}
      color="error"
      {...props}
    />
  );
};

// Similar pattern for all other buttons:
// - ArchiveButton
// - CopyButton
// - NewButton
// - CancelButton
// - ClearButton
// - PrintButton
// - SendButton
// - AttachmentsButton
// - EditButton
// - ViewButton
```

### 4.3 Navigation Buttons

```typescript
// src/components/core/buttons/NavigationButtons.tsx
import React from 'react';
import { Box, IconButton, Tooltip } from '@mui/material';
import {
  FirstPage,
  NavigateBefore,
  NavigateNext,
  LastPage
} from '@mui/icons-material';
import { useLocalization } from '@/hooks/useLocalization';

interface NavigationButtonsProps {
  onFirst: () => void;
  onPrevious: () => void;
  onNext: () => void;
  onLast: () => void;
  canNavigatePrevious: boolean;
  canNavigateNext: boolean;
}

export const NavigationButtons: React.FC<NavigationButtonsProps> = ({
  onFirst,
  onPrevious,
  onNext,
  onLast,
  canNavigatePrevious,
  canNavigateNext
}) => {
  const { t } = useLocalization();

  return (
    <Box sx={{ display: 'flex', gap: 0.5 }}>
      <Tooltip title={t('common.firstRecord')}>
        <span>
          <IconButton
            size="small"
            onClick={onFirst}
            disabled={!canNavigatePrevious}
          >
            <FirstPage />
          </IconButton>
        </span>
      </Tooltip>

      <Tooltip title={t('common.previousRecord')}>
        <span>
          <IconButton
            size="small"
            onClick={onPrevious}
            disabled={!canNavigatePrevious}
          >
            <NavigateBefore />
          </IconButton>
        </span>
      </Tooltip>

      <Tooltip title={t('common.nextRecord')}>
        <span>
          <IconButton
            size="small"
            onClick={onNext}
            disabled={!canNavigateNext}
          >
            <NavigateNext />
          </IconButton>
        </span>
      </Tooltip>

      <Tooltip title={t('common.lastRecord')}>
        <span>
          <IconButton
            size="small"
            onClick={onLast}
            disabled={!canNavigateNext}
          >
            <LastPage />
          </IconButton>
        </span>
      </Tooltip>
    </Box>
  );
};
```

## 5. Dynamic Form Renderer - عارض النماذج الديناميكي

### 5.1 DynamicForm Component

```typescript
// src/components/dynamic/DynamicForm.tsx
import React from 'react';
import { Grid, Tabs, Tab, Box } from '@mui/material';
import { DynamicField } from './DynamicField';
import { DynamicGrid } from './DynamicGrid';
import { FormSection } from '../core/layout/FormSection';
import { FormMetadata, FieldMetadata, FormMode } from '@/types/metadata.types';

interface DynamicFormProps {
  entityType: string;
  data: any;
  onChange: (data: any) => void;
  metadata: FormMetadata;
  fieldMetadata: FieldMetadata[];
  mode: FormMode;
  errors?: Record<string, string>;
}

export const DynamicForm: React.FC<DynamicFormProps> = ({
  entityType,
  data,
  onChange,
  metadata,
  fieldMetadata,
  mode,
  errors
}) => {
  const [activeTab, setActiveTab] = React.useState(0);

  const handleFieldChange = (fieldName: string, value: any) => {
    onChange({ ...data, [fieldName]: value });
  };

  // Group fields by section and tab
  const groupedFields = React.useMemo(() => {
    const groups: Record<string, Record<string, FieldMetadata[]>> = {};
    
    fieldMetadata.forEach(field => {
      if (!field.showInForm) return;
      
      const tab = field.tabName || 'main';
      const section = field.sectionName || 'general';
      
      if (!groups[tab]) groups[tab] = {};
      if (!groups[tab][section]) groups[tab][section] = [];
      
      groups[tab][section].push(field);
    });
    
    return groups;
  }, [fieldMetadata]);

  const tabs = Object.keys(groupedFields);
  const hasMultipleTabs = tabs.length > 1;

  // Render fields in a section
  const renderSection = (sectionFields: FieldMetadata[]) => {
    // Sort by display order
    const sortedFields = [...sectionFields].sort((a, b) => 
      (a.displayOrder || 0) - (b.displayOrder || 0)
    );

    return (
      <Grid container spacing={2}>
        {sortedFields.map(field => (
          <Grid 
            item 
            xs={field.columnSpan || 12} 
            key={field.fieldName}
          >
            <DynamicField
              field={field}
              value={data[field.fieldName]}
              onChange={(value) => handleFieldChange(field.fieldName, value)}
              mode={mode}
              error={errors?.[field.fieldName]}
            />
          </Grid>
        ))}
      </Grid>
    );
  };

  // Render a tab content
  const renderTab = (tabName: string) => {
    const sections = groupedFields[tabName];
    
    return (
      <Box sx={{ p: 2 }}>
        {Object.entries(sections).map(([sectionName, sectionFields]) => (
          <FormSection
            key={sectionName}
            title={sectionName !== 'general' ? sectionName : undefined}
            collapsible={sectionName !== 'general'}
          >
            {renderSection(sectionFields)}
          </FormSection>
        ))}
        
        {/* Render child grids if any */}
        {metadata.childGrids?.map(gridConfig => (
          <DynamicGrid
            key={gridConfig.entityType}
            config={gridConfig}
            parentData={data}
            mode={mode}
          />
        ))}
      </Box>
    );
  };

  if (!hasMultipleTabs) {
    return renderTab(tabs[0]);
  }

  return (
    <Box>
      <Tabs 
        value={activeTab} 
        onChange={(_, newValue) => setActiveTab(newValue)}
      >
        {tabs.map((tab, index) => (
          <Tab 
            key={tab} 
            label={metadata.tabLabels?.[tab] || tab}
            value={index}
          />
        ))}
      </Tabs>
      
      {tabs.map((tab, index) => (
        <Box
          key={tab}
          role="tabpanel"
          hidden={activeTab !== index}
        >
          {activeTab === index && renderTab(tab)}
        </Box>
      ))}
    </Box>
  );
};
```

### 5.2 DynamicField Component

```typescript
// src/components/dynamic/DynamicField.tsx
import React from 'react';
import { FieldMetadata, FormMode } from '@/types/metadata.types';
import { 
  TextField,
  NumberField,
  DateField,
  BooleanField,
  LookupField,
  RichTextField
} from '../core/fields';

interface DynamicFieldProps {
  field: FieldMetadata;
  value: any;
  onChange: (value: any) => void;
  mode: FormMode;
  error?: string;
}

export const DynamicField: React.FC<DynamicFieldProps> = ({
  field,
  value,
  onChange,
  mode,
  error
}) => {
  const isReadOnly = mode === 'view' || field.isReadOnly;

  const commonProps = {
    label: field.fieldLabel,
    value: value,
    onChange: onChange,
    required: field.isRequired,
    disabled: isReadOnly,
    error: error,
    helperText: error || field.helpText,
    placeholder: field.placeholderText
  };

  switch (field.dataType) {
    case 'String':
    case 'Text':
      return field.maxLength && field.maxLength > 500 ? (
        <RichTextField {...commonProps} />
      ) : (
        <TextField 
          {...commonProps}
          maxLength={field.maxLength}
          multiline={field.isMultiline}
          rows={field.rows}
        />
      );

    case 'Integer':
    case 'Decimal':
      return (
        <NumberField
          {...commonProps}
          precision={field.precision}
          scale={field.scale}
          min={field.minValue}
          max={field.maxValue}
          format={field.displayFormat}
        />
      );

    case 'Date':
      return (
        <DateField
          {...commonProps}
          format={field.displayFormat}
        />
      );

    case 'DateTime':
      return (
        <DateField
          {...commonProps}
          showTime
          format={field.displayFormat}
        />
      );

    case 'Boolean':
      return (
        <BooleanField
          {...commonProps}
          displayAs={field.booleanDisplayType} // checkbox, switch, radio
        />
      );

    case 'Lookup':
      return (
        <LookupField
          {...commonProps}
          lookupConfig={field.lookupConfig}
          allowMultiple={field.allowMultiple}
          allowCustomValue={field.allowCustomValue}
        />
      );

    default:
      return <TextField {...commonProps} />;
  }
};
```

تابع في الجزء التالي...
