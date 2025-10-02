# Field Components & Multi-View System
## مكونات الحقول ونظام العروض المتعددة

## 1. Core Field Components - مكونات الحقول الأساسية

### 1.1 TextField Component

```typescript
// src/components/core/fields/TextField.tsx
import React from 'react';
import { TextField as MUITextField, TextFieldProps } from '@mui/material';
import { useLocalization } from '@/hooks/useLocalization';

interface CustomTextFieldProps extends Omit<TextFieldProps, 'onChange'> {
  value: string;
  onChange: (value: string) => void;
  maxLength?: number;
  localizable?: boolean;
  onTranslate?: () => void;
}

export const TextField: React.FC<CustomTextFieldProps> = ({
  value,
  onChange,
  maxLength,
  localizable,
  onTranslate,
  ...props
}) => {
  const { t, isRTL } = useLocalization();

  return (
    <MUITextField
      fullWidth
      value={value || ''}
      onChange={(e) => onChange(e.target.value)}
      inputProps={{
        maxLength,
        dir: isRTL ? 'rtl' : 'ltr'
      }}
      InputProps={{
        endAdornment: localizable && (
          <IconButton size="small" onClick={onTranslate}>
            <TranslateIcon />
          </IconButton>
        )
      }}
      {...props}
    />
  );
};
```

### 1.2 NumberField Component

```typescript
// src/components/core/fields/NumberField.tsx
import React from 'react';
import { TextField, InputAdornment } from '@mui/material';
import { useLocalization } from '@/hooks/useLocalization';

interface NumberFieldProps {
  label: string;
  value: number | null;
  onChange: (value: number | null) => void;
  precision?: number;
  scale?: number;
  min?: number;
  max?: number;
  format?: string;
  currency?: string;
  disabled?: boolean;
  required?: boolean;
  error?: string;
}

export const NumberField: React.FC<NumberFieldProps> = ({
  label,
  value,
  onChange,
  precision = 18,
  scale = 2,
  min,
  max,
  format,
  currency,
  disabled,
  required,
  error
}) => {
  const { formatNumber, currentLanguage } = useLocalization();
  const [displayValue, setDisplayValue] = React.useState('');

  React.useEffect(() => {
    if (value !== null && value !== undefined) {
      setDisplayValue(formatNumber(value, { 
        minimumFractionDigits: scale,
        maximumFractionDigits: scale 
      }));
    } else {
      setDisplayValue('');
    }
  }, [value, scale]);

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const input = e.target.value;
    setDisplayValue(input);

    // Parse the number considering locale
    const parsed = parseFloat(input.replace(/[^\d.-]/g, ''));
    
    if (!isNaN(parsed)) {
      // Validate range
      if (min !== undefined && parsed < min) return;
      if (max !== undefined && parsed > max) return;
      
      onChange(parsed);
    } else if (input === '' || input === '-') {
      onChange(null);
    }
  };

  const handleBlur = () => {
    // Reformat on blur
    if (value !== null && value !== undefined) {
      setDisplayValue(formatNumber(value, { 
        minimumFractionDigits: scale,
        maximumFractionDigits: scale 
      }));
    }
  };

  return (
    <TextField
      fullWidth
      label={label}
      value={displayValue}
      onChange={handleChange}
      onBlur={handleBlur}
      disabled={disabled}
      required={required}
      error={!!error}
      helperText={error}
      InputProps={{
        startAdornment: currency && (
          <InputAdornment position="start">{currency}</InputAdornment>
        ),
        inputProps: {
          inputMode: 'decimal',
          style: { textAlign: 'right' }
        }
      }}
    />
  );
};
```

### 1.3 DateField Component

```typescript
// src/components/core/fields/DateField.tsx
import React from 'react';
import { DatePicker, DateTimePicker } from '@mui/x-date-pickers';
import { TextField } from '@mui/material';
import { useLocalization } from '@/hooks/useLocalization';
import { parseISO, format as formatDate } from 'date-fns';
import { arSA, enUS } from 'date-fns/locale';

interface DateFieldProps {
  label: string;
  value: Date | string | null;
  onChange: (value: Date | null) => void;
  showTime?: boolean;
  format?: string;
  minDate?: Date;
  maxDate?: Date;
  disabled?: boolean;
  required?: boolean;
  error?: string;
  timezone?: string;
}

export const DateField: React.FC<DateFieldProps> = ({
  label,
  value,
  onChange,
  showTime,
  format: customFormat,
  minDate,
  maxDate,
  disabled,
  required,
  error,
  timezone
}) => {
  const { currentLanguage, isRTL } = useLocalization();
  
  const locale = currentLanguage.startsWith('ar') ? arSA : enUS;
  const format = customFormat || (showTime ? 'dd/MM/yyyy HH:mm' : 'dd/MM/yyyy');

  const dateValue = React.useMemo(() => {
    if (!value) return null;
    return typeof value === 'string' ? parseISO(value) : value;
  }, [value]);

  const Picker = showTime ? DateTimePicker : DatePicker;

  return (
    <Picker
      label={label}
      value={dateValue}
      onChange={onChange}
      format={format}
      minDate={minDate}
      maxDate={maxDate}
      disabled={disabled}
      slotProps={{
        textField: {
          fullWidth: true,
          required,
          error: !!error,
          helperText: error,
          InputProps: {
            dir: isRTL ? 'rtl' : 'ltr'
          }
        }
      }}
    />
  );
};
```

### 1.4 LookupField Component

```typescript
// src/components/core/fields/LookupField.tsx
import React from 'react';
import { Autocomplete, TextField, CircularProgress } from '@mui/material';
import { useLookupData } from '@/hooks/useLookupData';
import { LookupConfig } from '@/types/metadata.types';

interface LookupFieldProps {
  label: string;
  value: any;
  onChange: (value: any) => void;
  lookupConfig: LookupConfig;
  allowMultiple?: boolean;
  allowCustomValue?: boolean;
  disabled?: boolean;
  required?: boolean;
  error?: string;
}

export const LookupField: React.FC<LookupFieldProps> = ({
  label,
  value,
  onChange,
  lookupConfig,
  allowMultiple,
  allowCustomValue,
  disabled,
  required,
  error
}) => {
  const [inputValue, setInputValue] = React.useState('');
  
  const {
    options,
    loading,
    fetchOptions
  } = useLookupData(lookupConfig);

  React.useEffect(() => {
    fetchOptions(inputValue);
  }, [inputValue]);

  return (
    <Autocomplete
      fullWidth
      multiple={allowMultiple}
      value={value}
      onChange={(_, newValue) => onChange(newValue)}
      inputValue={inputValue}
      onInputChange={(_, newInputValue) => setInputValue(newInputValue)}
      options={options}
      getOptionLabel={(option) => 
        typeof option === 'string' ? option : option[lookupConfig.displayField]
      }
      loading={loading}
      disabled={disabled}
      freeSolo={allowCustomValue}
      renderInput={(params) => (
        <TextField
          {...params}
          label={label}
          required={required}
          error={!!error}
          helperText={error}
          InputProps={{
            ...params.InputProps,
            endAdornment: (
              <>
                {loading && <CircularProgress size={20} />}
                {params.InputProps.endAdornment}
              </>
            )
          }}
        />
      )}
    />
  );
};
```

## 2. View Types System - نظام أنواع العروض

### 2.1 ViewSelector Component

```typescript
// src/components/views/ViewSelector.tsx
import React from 'react';
import { ToggleButtonGroup, ToggleButton, Tooltip } from '@mui/material';
import {
  ViewList,
  ViewKanban,
  ViewModule,
  CalendarMonth,
  Timeline
} from '@mui/icons-material';
import { ViewType } from '@/types/view.types';

interface ViewSelectorProps {
  currentView: ViewType;
  availableViews: ViewType[];
  onViewChange: (view: ViewType) => void;
}

export const ViewSelector: React.FC<ViewSelectorProps> = ({
  currentView,
  availableViews,
  onViewChange
}) => {
  const viewIcons: Record<ViewType, { icon: React.ReactNode; label: string }> = {
    list: { icon: <ViewList />, label: 'List View' },
    kanban: { icon: <ViewKanban />, label: 'Kanban View' },
    grid: { icon: <ViewModule />, label: 'Grid View' },
    calendar: { icon: <CalendarMonth />, label: 'Calendar View' },
    gantt: { icon: <Timeline />, label: 'Gantt View' }
  };

  return (
    <ToggleButtonGroup
      value={currentView}
      exclusive
      onChange={(_, newView) => newView && onViewChange(newView)}
      size="small"
    >
      {availableViews.map(view => (
        <ToggleButton key={view} value={view}>
          <Tooltip title={viewIcons[view].label}>
            {viewIcons[view].icon}
          </Tooltip>
        </ToggleButton>
      ))}
    </ToggleButtonGroup>
  );
};
```

### 2.2 ListView Component

```typescript
// src/components/views/ListView.tsx
import React from 'react';
import { DataGrid, GridColDef, GridRowParams } from '@mui/x-data-grid';
import { Box, IconButton } from '@mui/material';
import { Edit, Delete, Visibility } from '@mui/icons-material';
import { useLocalization } from '@/hooks/useLocalization';
import { ViewMetadata } from '@/types/view.types';

interface ListViewProps {
  entityType: string;
  metadata: ViewMetadata;
  data: any[];
  loading: boolean;
  onRowClick: (id: string | number) => void;
  onEdit?: (id: string | number) => void;
  onDelete?: (id: string | number) => void;
  onRefresh: () => void;
}

export const ListView: React.FC<ListViewProps> = ({
  entityType,
  metadata,
  data,
  loading,
  onRowClick,
  onEdit,
  onDelete,
  onRefresh
}) => {
  const { t, formatDate, formatNumber, formatCurrency } = useLocalization();

  // Build columns from metadata
  const columns: GridColDef[] = React.useMemo(() => {
    const cols = metadata.fields
      .filter(f => f.showInList)
      .sort((a, b) => (a.displayOrder || 0) - (b.displayOrder || 0))
      .map(field => ({
        field: field.fieldName,
        headerName: field.fieldLabel,
        width: field.columnWidth || 150,
        sortable: field.isSortable,
        filterable: field.isFilterable,
        valueFormatter: (params: any) => {
          if (!params.value) return '';
          
          switch (field.dataType) {
            case 'Date':
            case 'DateTime':
              return formatDate(params.value);
            case 'Decimal':
            case 'Integer':
              return field.displayFormat === 'currency'
                ? formatCurrency(params.value, field.currencyCode || 'SAR')
                : formatNumber(params.value, { maximumFractionDigits: field.scale });
            case 'Boolean':
              return params.value ? t('common.yes') : t('common.no');
            default:
              return params.value;
          }
        }
      }));

    // Add actions column
    cols.push({
      field: 'actions',
      headerName: t('common.actions'),
      width: 120,
      sortable: false,
      filterable: false,
      renderCell: (params) => (
        <Box>
          <IconButton
            size="small"
            onClick={(e) => {
              e.stopPropagation();
              onRowClick(params.row.id);
            }}
          >
            <Visibility />
          </IconButton>
          {onEdit && (
            <IconButton
              size="small"
              onClick={(e) => {
                e.stopPropagation();
                onEdit(params.row.id);
              }}
            >
              <Edit />
            </IconButton>
          )}
          {onDelete && (
            <IconButton
              size="small"
              onClick={(e) => {
                e.stopPropagation();
                onDelete(params.row.id);
              }}
            >
              <Delete />
            </IconButton>
          )}
        </Box>
      )
    });

    return cols;
  }, [metadata.fields]);

  return (
    <DataGrid
      rows={data}
      columns={columns}
      loading={loading}
      pagination
      pageSizeOptions={[25, 50, 100]}
      checkboxSelection
      disableRowSelectionOnClick
      onRowClick={(params: GridRowParams) => onRowClick(params.row.id)}
      sx={{
        '& .MuiDataGrid-row': {
          cursor: 'pointer'
        }
      }}
    />
  );
};
```

### 2.3 KanbanView Component

```typescript
// src/components/views/KanbanView.tsx
import React from 'react';
import { DragDropContext, Droppable, Draggable } from '@hello-pangea/dnd';
import { Box, Paper, Typography, Card, CardContent, Chip } from '@mui/material';
import { ViewMetadata, KanbanColumn } from '@/types/view.types';

interface KanbanViewProps {
  entityType: string;
  metadata: ViewMetadata;
  data: any[];
  columns: KanbanColumn[];
  groupByField: string;
  onCardClick: (id: string | number) => void;
  onCardMove: (cardId: string | number, newStatus: string) => void;
}

export const KanbanView: React.FC<KanbanViewProps> = ({
  entityType,
  metadata,
  data,
  columns,
  groupByField,
  onCardClick,
  onCardMove
}) => {
  // Group data by status/column
  const groupedData = React.useMemo(() => {
    const groups: Record<string, any[]> = {};
    
    columns.forEach(col => {
      groups[col.id] = data.filter(item => item[groupByField] === col.id);
    });
    
    return groups;
  }, [data, columns, groupByField]);

  const handleDragEnd = (result: any) => {
    if (!result.destination) return;
    
    const { draggableId, destination } = result;
    onCardMove(draggableId, destination.droppableId);
  };

  return (
    <DragDropContext onDragEnd={handleDragEnd}>
      <Box sx={{ display: 'flex', gap: 2, overflowX: 'auto', height: '100%' }}>
        {columns.map(column => (
          <Box key={column.id} sx={{ minWidth: 300, flex: '0 0 auto' }}>
            <Paper sx={{ p: 2, height: '100%', display: 'flex', flexDirection: 'column' }}>
              {/* Column Header */}
              <Box sx={{ mb: 2, display: 'flex', justifyContent: 'space-between' }}>
                <Typography variant="h6">
                  {column.title}
                </Typography>
                <Chip 
                  label={groupedData[column.id]?.length || 0}
                  size="small"
                  color={column.color}
                />
              </Box>

              {/* Cards */}
              <Droppable droppableId={column.id}>
                {(provided, snapshot) => (
                  <Box
                    ref={provided.innerRef}
                    {...provided.droppableProps}
                    sx={{
                      flex: 1,
                      overflowY: 'auto',
                      bgcolor: snapshot.isDraggingOver ? 'action.hover' : 'transparent',
                      borderRadius: 1,
                      p: 1
                    }}
                  >
                    {groupedData[column.id]?.map((item, index) => (
                      <Draggable
                        key={item.id}
                        draggableId={item.id.toString()}
                        index={index}
                      >
                        {(provided, snapshot) => (
                          <Card
                            ref={provided.innerRef}
                            {...provided.draggableProps}
                            {...provided.dragHandleProps}
                            onClick={() => onCardClick(item.id)}
                            sx={{
                              mb: 1,
                              cursor: 'pointer',
                              opacity: snapshot.isDragging ? 0.8 : 1,
                              '&:hover': {
                                boxShadow: 3
                              }
                            }}
                          >
                            <CardContent>
                              <Typography variant="subtitle2" gutterBottom>
                                {item[metadata.displayField]}
                              </Typography>
                              {metadata.kanbanCardFields?.map(field => (
                                <Typography 
                                  key={field.fieldName}
                                  variant="body2" 
                                  color="text.secondary"
                                >
                                  {field.fieldLabel}: {item[field.fieldName]}
                                </Typography>
                              ))}
                            </CardContent>
                          </Card>
                        )}
                      </Draggable>
                    ))}
                    {provided.placeholder}
                  </Box>
                )}
              </Droppable>
            </Paper>
          </Box>
        ))}
      </Box>
    </DragDropContext>
  );
};
```

### 2.4 CalendarView Component

```typescript
// src/components/views/CalendarView.tsx
import React from 'react';
import { Calendar, momentLocalizer, Event } from 'react-big-calendar';
import moment from 'moment';
import 'react-big-calendar/lib/css/react-big-calendar.css';
import { ViewMetadata } from '@/types/view.types';

const localizer = momentLocalizer(moment);

interface CalendarViewProps {
  entityType: string;
  metadata: ViewMetadata;
  data: any[];
  startDateField: string;
  endDateField?: string;
  titleField: string;
  onEventClick: (id: string | number) => void;
  onSlotSelect?: (start: Date, end: Date) => void;
}

export const CalendarView: React.FC<CalendarViewProps> = ({
  entityType,
  metadata,
  data,
  startDateField,
  endDateField,
  titleField,
  onEventClick,
  onSlotSelect
}) => {
  // Transform data to calendar events
  const events: Event[] = React.useMemo(() => {
    return data.map(item => ({
      id: item.id,
      title: item[titleField],
      start: new Date(item[startDateField]),
      end: endDateField ? new Date(item[endDateField]) : new Date(item[startDateField]),
      resource: item
    }));
  }, [data, startDateField, endDateField, titleField]);

  return (
    <Calendar
      localizer={localizer}
      events={events}
      startAccessor="start"
      endAccessor="end"
      style={{ height: '100%' }}
      onSelectEvent={(event) => onEventClick(event.id)}
      onSelectSlot={({ start, end }) => onSlotSelect?.(start, end)}
      selectable
    />
  );
};
```

## 3. useFormContainer Hook - الـ Hook الرئيسي

```typescript
// src/hooks/useFormContainer.ts
import { useState, useEffect, useCallback } from 'react';
import { useMetadata } from './useMetadata';
import { useCRUD } from './useCRUD';
import { useNavigation } from './useNavigation';
import { usePermissions } from './usePermissions';
import { FormMode, FormMetadata } from '@/types/metadata.types';

interface UseFormContainerProps {
  entityType: string;
  entityId?: number | string;
  initialMode?: FormMode;
  metadata?: FormMetadata;
  onSaveCallback?: (data: any) => void;
}

export const useFormContainer = ({
  entityType,
  entityId,
  initialMode = 'view',
  metadata: providedMetadata,
  onSaveCallback
}: UseFormContainerProps) => {
  const [currentMode, setCurrentMode] = useState<FormMode>(initialMode);
  const [formData, setFormData] = useState<any>({});
  const [originalData, setOriginalData] = useState<any>({});
  const [errors, setErrors] = useState<Record<string, string>>({});

  // Fetch metadata
  const { 
    metadata: formMetadata,
    fieldMetadata,
    loading: metadataLoading
  } = useMetadata(entityType, providedMetadata);

  // CRUD operations
  const {
    data,
    loading: dataLoading,
    saving,
    create,
    read,
    update,
    remove,
    archive
  } = useCRUD(entityType);

  // Navigation
  const {
    canNavigatePrevious,
    canNavigateNext,
    navigateFirst,
    navigatePrevious,
    navigateNext,
    navigateLast
  } = useNavigation(entityType, entityId);

  // Permissions
  const {
    canEdit,
    canDelete,
    canCreate,
    canPrint
  } = usePermissions(entityType);

  // Load data
  useEffect(() => {
    if (entityId && entityId !== 'new') {
      loadData();
    } else if (entityId === 'new') {
      setCurrentMode('new');
      setFormData(getDefaultValues());
    }
  }, [entityId]);

  const loadData = async () => {
    const result = await read(entityId as number);
    if (result) {
      setFormData(result);
      setOriginalData(result);
    }
  };

  const getDefaultValues = () => {
    const defaults: any = {};
    fieldMetadata.forEach(field => {
      if (field.defaultValue) {
        defaults[field.fieldName] = field.defaultValue;
      }
    });
    return defaults;
  };

  // Check if form is dirty
  const isDirty = JSON.stringify(formData) !== JSON.stringify(originalData);

  // Validate form
  const validate = (): boolean => {
    const newErrors: Record<string, string> = {};

    fieldMetadata.forEach(field => {
      if (field.isRequired && !formData[field.fieldName]) {
        newErrors[field.fieldName] = `${field.fieldLabel} is required`;
      }

      if (field.validationRules) {
        // Apply custom validation rules
        // TODO: Implement validation engine
      }
    });

    setErrors(newErrors);
    return Object.keys(newErrors).length === 0;
  };

  // Actions
  const handleSave = async () => {
    if (!validate()) return;

    try {
      let result;
      if (currentMode === 'new') {
        result = await create(formData);
      } else {
        result = await update(entityId as number, formData);
      }

      if (result) {
        setOriginalData(result);
        setFormData(result);
        setCurrentMode('view');
        onSaveCallback?.(result);
      }
    } catch (error) {
      console.error('Save error:', error);
    }
  };

  const handleDelete = async () => {
    if (!window.confirm('Are you sure you want to delete this record?')) return;

    const success = await remove(entityId as number);
    if (success) {
      // Navigate to list or close
      window.history.back();
    }
  };

  const handleArchive = async () => {
    const success = await archive(entityId as number);
    if (success) {
      await loadData();
    }
  };

  const handleCopy = () => {
    setCurrentMode('new');
    const copiedData = { ...formData };
    delete copiedData.id;
    setFormData(copiedData);
  };

  const handleNew = () => {
    setCurrentMode('new');
    setFormData(getDefaultValues());
    setOriginalData({});
  };

  const handleCancel = () => {
    if (isDirty && !window.confirm('Discard changes?')) return;

    if (currentMode === 'new') {
      window.history.back();
    } else {
      setFormData(originalData);
      setCurrentMode('view');
    }
  };

  const handleClear = () => {
    setFormData(getDefaultValues());
  };

  const handlePrint = () => {
    // TODO: Implement print functionality
    window.print();
  };

  const handleSend = () => {
    // TODO: Implement send functionality
  };

  const handleAttachments = () => {
    // TODO: Implement attachments dialog
  };

  // Navigation actions
  const handleFirst = () => {
    const firstId = navigateFirst();
    if (firstId) {
      window.location.href = `/${entityType}/${firstId}`;
    }
  };

  const handlePrevious = () => {
    const prevId = navigatePrevious();
    if (prevId) {
      window.location.href = `/${entityType}/${prevId}`;
    }
  };

  const handleNext = () => {
    const nextId = navigateNext();
    if (nextId) {
      window.location.href = `/${entityType}/${nextId}`;
    }
  };

  const handleLast = () => {
    const lastId = navigateLast();
    if (lastId) {
      window.location.href = `/${entityType}/${lastId}`;
    }
  };

  return {
    // Data
    formData,
    setFormData,
    originalData,
    isDirty,
    isLoading: metadataLoading || dataLoading,
    isSaving: saving,
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

    // Mode
    currentMode,
    setCurrentMode
  };
};
```

## 4. Usage Examples - أمثلة الاستخدام

### 4.1 Simple Form (Customer)

```typescript
// src/modules/sales/pages/CustomerPage.tsx
import React from 'react';
import { useParams } from 'react-router-dom';
import { FormContainer } from '@/components/containers/FormContainer';

export const CustomerPage: React.FC = () => {
  const { id } = useParams();

  return (
    <FormContainer
      entityType="Customer"
      entityId={id}
    />
  );
};
```

### 4.2 Complex Form with Custom Layout (Invoice)

```typescript
// src/modules/sales/pages/InvoicePage.tsx
import React from 'react';
import { useParams } from 'react-router-dom';
import { FormContainer } from '@/components/containers/FormContainer';
import { InvoiceFormLayout } from '../components/InvoiceFormLayout';

export const InvoicePage: React.FC = () => {
  const { id } = useParams();

  return (
    <FormContainer
      entityType="SalesInvoice"
      entityId={id}
      customLayout={InvoiceFormLayout}
    />
  );
};

// Custom layout component
// src/modules/sales/components/InvoiceFormLayout.tsx
export const InvoiceFormLayout: React.FC<any> = ({
  data,
  onChange,
  metadata,
  fieldMetadata,
  mode,
  errors
}) => {
  return (
    <Box>
      {/* Header Section */}
      <FormSection title="Invoice Header">
        <Grid container spacing={2}>
          <Grid item xs={6}>
            <DynamicField
              field={fieldMetadata.find(f => f.fieldName === 'invoiceNumber')}
              value={data.invoiceNumber}
              onChange={(v) => onChange({ ...data, invoiceNumber: v })}
              mode={mode}
            />
          </Grid>
          <Grid item xs={6}>
            <DynamicField
              field={fieldMetadata.find(f => f.fieldName === 'invoiceDate')}
              value={data.invoiceDate}
              onChange={(v) => onChange({ ...data, invoiceDate: v })}
              mode={mode}
            />
          </Grid>
          {/* ... more fields */}
        </Grid>
      </FormSection>

      {/* Invoice Lines Grid */}
      <FormSection title="Invoice Lines">
        <InvoiceLineGrid
          lines={data.lines || []}
          onChange={(lines) => onChange({ ...data, lines })}
          mode={mode}
        />
      </FormSection>

      {/* Totals Section */}
      <FormSection title="Totals">
        <InvoiceTotals data={data} />
      </FormSection>
    </Box>
  );
};
```

### 4.3 List Page with Multiple Views

```typescript
// src/modules/sales/pages/InvoicesListPage.tsx
import React, { useState } from 'react';
import { Box } from '@mui/material';
import { ViewSelector } from '@/components/views/ViewSelector';
import { ListView } from '@/components/views/ListView';
import { KanbanView } from '@/components/views/KanbanView';
import { useViewData } from '@/hooks/useViewData';
import { ViewType } from '@/types/view.types';

export const InvoicesListPage: React.FC = () => {
  const [currentView, setCurrentView] = useState<ViewType>('list');
  
  const {
    data,
    loading,
    metadata,
    refresh
  } = useViewData('SalesInvoice');

  const availableViews: ViewType[] = ['list', 'kanban', 'calendar'];

  const handleRowClick = (id: number) => {
    window.location.href = `/sales/invoices/${id}`;
  };

  return (
    <Box sx={{ height: '100%', display: 'flex', flexDirection: 'column' }}>
      {/* View Selector */}
      <Box sx={{ p: 2, borderBottom: 1, borderColor: 'divider' }}>
        <ViewSelector
          currentView={currentView}
          availableViews={availableViews}
          onViewChange={setCurrentView}
        />
      </Box>

      {/* View Content */}
      <Box sx={{ flex: 1, overflow: 'auto', p: 2 }}>
        {currentView === 'list' && (
          <ListView
            entityType="SalesInvoice"
            metadata={metadata}
            data={data}
            loading={loading}
            onRowClick={handleRowClick}
            onRefresh={refresh}
          />
        )}

        {currentView === 'kanban' && (
          <KanbanView
            entityType="SalesInvoice"
            metadata={metadata}
            data={data}
            columns={[
              { id: 'Draft', title: 'Draft', color: 'default' },
              { id: 'Posted', title: 'Posted', color: 'primary' },
              { id: 'Paid', title: 'Paid', color: 'success' }
            ]}
            groupByField="status"
            onCardClick={handleRowClick}
            onCardMove={async (id, status) => {
              // Update invoice status
            }}
          />
        )}
      </Box>
    </Box>
  );
};
```

---

هل تريد المزيد من التفاصيل عن أي جزء معين؟ 🚀
