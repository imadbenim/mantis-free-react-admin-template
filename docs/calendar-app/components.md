# Component Breakdown

## Overview

This document details all components used in the NPO Calendar application, including existing Mantis components and new custom components to be built.

## Component Categories

1. **Existing Mantis Components** - Reused from template
2. **New Calendar Components** - Custom components for calendar functionality
3. **Shared Components** - Reusable components across the app

## Existing Mantis Components Used

### From `/components`

#### 1. MainCard
**Location**: `components/MainCard.jsx`
**Usage**: Primary container for calendar views, event forms, admin panels

```javascript
import MainCard from 'components/MainCard';

// Calendar container
<MainCard title="Calendar" secondary={<ViewSwitcher />}>
  <CalendarGrid />
</MainCard>

// Event form
<MainCard title="Create Event">
  <EventForm />
</MainCard>
```

**Props Used**:
- `title`: Section headings
- `content`: Enable content padding
- `secondary`: Action buttons
- `sx`: Custom styling

#### 2. IconButton
**Location**: `components/@extended/IconButton.jsx`
**Usage**: Navigation, actions, filters

```javascript
import IconButton from 'components/@extended/IconButton';

// Navigation
<IconButton onClick={previousMonth}>
  <LeftOutlined />
</IconButton>

// Delete action
<IconButton color="error" onClick={deleteEvent}>
  <DeleteOutlined />
</IconButton>
```

**Props Used**:
- `color`: primary, secondary, success, error
- `size`: small, medium, large
- `onClick`: Event handlers

#### 3. AnimateButton
**Location**: `components/@extended/AnimateButton.jsx`
**Usage**: Animated buttons for primary actions

```javascript
import AnimateButton from 'components/@extended/AnimateButton';

// Create event button
<AnimateButton>
  <Button variant="contained" startIcon={<PlusOutlined />}>
    Create Event
  </Button>
</AnimateButton>
```

#### 4. Avatar
**Location**: `components/@extended/Avatar.jsx`
**Usage**: User profile pictures, event creators

```javascript
import Avatar from 'components/@extended/Avatar';

// Event creator
<Avatar
  alt={creator.name}
  src={creator.avatar_url}
  size="sm"
  color="primary"
/>

// User profile
<Avatar
  alt={user.name}
  src={user.avatar_url}
  size="md"
/>
```

**Props Used**:
- `size`: xs, sm, md, lg, xl
- `color`: primary, secondary, success, error
- `alt`, `src`: Standard avatar props

#### 5. Breadcrumbs
**Location**: `components/@extended/Breadcrumbs.jsx`
**Usage**: Navigation hierarchy

```javascript
import Breadcrumbs from 'components/@extended/Breadcrumbs';

// Auto-generated breadcrumbs
<Breadcrumbs navigation={navigation} title />
```

#### 6. Dot
**Location**: `components/@extended/Dot.jsx`
**Usage**: Status indicators, category colors

```javascript
import Dot from 'components/@extended/Dot';

// Category indicator
<Dot color={category.color} />

// Event status
<Dot color="success" />
```

**Props Used**:
- `color`: Theme colors or custom
- `size`: Size variant

#### 7. Transitions
**Location**: `components/@extended/Transitions.jsx`
**Usage**: Animated transitions for modals, drawers

```javascript
import Transitions from 'components/@extended/Transitions';

// Modal transition
<Modal open={open}>
  <Transitions in={open} type="fade">
    <EventDetail />
  </Transitions>
</Modal>
```

#### 8. AnalyticEcommerce
**Location**: `components/cards/statistics/AnalyticEcommerce.jsx`
**Usage**: Admin dashboard statistics

```javascript
import AnalyticEcommerce from 'components/cards/statistics/AnalyticEcommerce';

// User count
<AnalyticEcommerce
  title="Total Users"
  count="42"
  extra="8"
/>

// Event count
<AnalyticEcommerce
  title="Active Events"
  count="156"
  percentage={12.5}
  isLoss={false}
  color="success"
/>
```

## New Calendar Components

### Location: `src/components/calendar/`

### 1. CalendarGrid

**File**: `components/calendar/CalendarGrid.jsx`

**Purpose**: Main calendar grid displaying month/week/day views

**Props**:
```typescript
interface CalendarGridProps {
  view: 'month' | 'week' | 'day';
  currentDate: Date;
  events: Event[];
  onEventClick: (event: Event) => void;
  onDateClick?: (date: Date) => void;
  onEventDrop?: (event: Event, newDate: Date) => void;
  loading?: boolean;
}
```

**Features**:
- Renders calendar grid based on view type
- Displays events on appropriate dates
- Handles click interactions
- Responsive design
- Loading skeleton
- Drag-and-drop (future enhancement)

**Usage**:
```javascript
import CalendarGrid from 'components/calendar/CalendarGrid';

<CalendarGrid
  view="month"
  currentDate={currentDate}
  events={events}
  onEventClick={handleEventClick}
  onDateClick={handleDateClick}
  loading={loading}
/>
```

**Internal Structure**:
- MonthGrid sub-component
- WeekGrid sub-component
- DayGrid sub-component
- EventCell sub-component
- DateCell sub-component

### 2. EventCard

**File**: `components/calendar/EventCard.jsx`

**Purpose**: Display event in calendar grid or list

**Props**:
```typescript
interface EventCardProps {
  event: Event;
  variant?: 'compact' | 'detailed';
  onClick?: () => void;
  showCreator?: boolean;
  showCategory?: boolean;
}
```

**Variants**:
- **compact**: Small card for calendar grid
- **detailed**: Full card for list view

**Usage**:
```javascript
import EventCard from 'components/calendar/EventCard';

// Compact (in grid)
<EventCard
  event={event}
  variant="compact"
  onClick={handleClick}
/>

// Detailed (in list)
<EventCard
  event={event}
  variant="detailed"
  showCreator
  showCategory
  onClick={handleClick}
/>
```

**Features**:
- Color-coded by category
- Truncated text with tooltip
- Time display
- Category indicator (Dot)
- Creator avatar (if shown)
- Responsive sizing

### 3. EventEditor

**File**: `components/calendar/EventEditor.jsx`

**Purpose**: Form for creating and editing events

**Props**:
```typescript
interface EventEditorProps {
  event?: Event; // For editing
  onSave: (event: Event) => Promise<void>;
  onCancel: () => void;
  mode: 'create' | 'edit';
}
```

**Features**:
- Controlled form inputs
- Real-time validation
- Date/time pickers
- Category selector
- Visibility radio group
- Recurrence configuration
- Loading state on submit
- Error handling

**Usage**:
```javascript
import EventEditor from 'components/calendar/EventEditor';

<EventEditor
  mode="create"
  onSave={handleSave}
  onCancel={handleCancel}
/>

<EventEditor
  mode="edit"
  event={selectedEvent}
  onSave={handleSave}
  onCancel={handleCancel}
/>
```

**Form Fields**:
- Title (TextField)
- Start Date/Time (DateTimePicker)
- End Date/Time (DateTimePicker)
- All-day toggle (Switch)
- Location (TextField)
- Category (Select)
- Visibility (Radio Group)
- Description (TextField multiline)
- Recurrence (RecurrenceEditor)

### 4. EventDetail

**File**: `components/calendar/EventDetail.jsx`

**Purpose**: Display full event details with actions

**Props**:
```typescript
interface EventDetailProps {
  event: Event;
  onEdit?: () => void;
  onDelete?: () => void;
  onClose: () => void;
  canEdit: boolean;
  canDelete: boolean;
}
```

**Features**:
- Full event information
- Formatted date/time
- Category with color
- Creator info (if visible)
- Recurrence info
- Action buttons (conditional)
- Share link (future)

**Usage**:
```javascript
import EventDetail from 'components/calendar/EventDetail';

<Dialog open={open} onClose={onClose}>
  <EventDetail
    event={selectedEvent}
    onEdit={handleEdit}
    onDelete={handleDelete}
    onClose={onClose}
    canEdit={canUserEdit(selectedEvent)}
    canDelete={canUserDelete(selectedEvent)}
  />
</Dialog>
```

### 5. CalendarHeader

**File**: `components/calendar/CalendarHeader.jsx`

**Purpose**: Navigation and view controls

**Props**:
```typescript
interface CalendarHeaderProps {
  view: CalendarView;
  currentDate: Date;
  onViewChange: (view: CalendarView) => void;
  onDateChange: (date: Date) => void;
  onCreateEvent?: () => void;
  canCreateEvent: boolean;
}
```

**Features**:
- View switcher (Month/Week/Day/List)
- Previous/Next navigation
- Today button
- Date display
- Create event button (conditional)

**Usage**:
```javascript
import CalendarHeader from 'components/calendar/CalendarHeader';

<CalendarHeader
  view={view}
  currentDate={currentDate}
  onViewChange={setView}
  onDateChange={setCurrentDate}
  onCreateEvent={handleCreateEvent}
  canCreateEvent={isManager || isAdmin}
/>
```

### 6. EventFilters

**File**: `components/calendar/EventFilters.jsx`

**Purpose**: Filter sidebar/drawer for events

**Props**:
```typescript
interface EventFiltersProps {
  categories: Category[];
  selectedCategories: string[];
  onCategoriesChange: (categories: string[]) => void;
  visibilityOptions?: VisibilityOption[];
  selectedVisibility: string[];
  onVisibilityChange: (visibility: string[]) => void;
  showMyEventsOnly: boolean;
  onMyEventsChange: (value: boolean) => void;
  onClearAll: () => void;
  variant?: 'sidebar' | 'drawer';
}
```

**Features**:
- Category checkboxes
- Visibility filter
- My events toggle
- Clear all button
- Active filter count
- Responsive (sidebar/drawer)

**Usage**:
```javascript
import EventFilters from 'components/calendar/EventFilters';

// Desktop sidebar
<EventFilters
  categories={categories}
  selectedCategories={selectedCategories}
  onCategoriesChange={setSelectedCategories}
  onClearAll={clearFilters}
  variant="sidebar"
/>

// Mobile drawer
<Drawer open={open} onClose={onClose}>
  <EventFilters
    categories={categories}
    selectedCategories={selectedCategories}
    onCategoriesChange={setSelectedCategories}
    onClearAll={clearFilters}
    variant="drawer"
  />
</Drawer>
```

### 7. RecurrenceEditor

**File**: `components/calendar/RecurrenceEditor.jsx`

**Purpose**: Configure recurring event rules

**Props**:
```typescript
interface RecurrenceEditorProps {
  recurrence?: RecurrenceRule;
  onChange: (recurrence: RecurrenceRule | null) => void;
  startDate: Date;
}
```

**Features**:
- Recurrence type selector (daily/weekly/monthly)
- Interval input
- End date picker
- Days of week selector (weekly)
- Preview of upcoming instances

**Usage**:
```javascript
import RecurrenceEditor from 'components/calendar/RecurrenceEditor';

<RecurrenceEditor
  recurrence={recurrence}
  onChange={setRecurrence}
  startDate={eventStartDate}
/>
```

**Fields**:
- Repeat type (Select)
- Every N (days/weeks/months) (TextField)
- Days of week (CheckboxGroup) - for weekly
- Ends (Radio): Never / On date / After N occurrences
- Preview list

### 8. EventList

**File**: `components/calendar/EventList.jsx`

**Purpose**: List view of events

**Props**:
```typescript
interface EventListProps {
  events: Event[];
  onEventClick: (event: Event) => void;
  groupBy?: 'date' | 'category' | 'none';
  loading?: boolean;
  emptyMessage?: string;
}
```

**Features**:
- Grouped by date or category
- Infinite scroll or pagination
- Loading skeleton
- Empty state
- Search highlighting

**Usage**:
```javascript
import EventList from 'components/calendar/EventList';

<EventList
  events={filteredEvents}
  onEventClick={handleEventClick}
  groupBy="date"
  loading={loading}
  emptyMessage="No events found"
/>
```

### 9. CategoryManager

**File**: `components/calendar/CategoryManager.jsx`

**Purpose**: Admin interface for managing categories

**Props**:
```typescript
interface CategoryManagerProps {
  categories: Category[];
  onAdd: (category: Partial<Category>) => Promise<void>;
  onEdit: (id: string, updates: Partial<Category>) => Promise<void>;
  onDelete: (id: string) => Promise<void>;
  onReorder: (categoryIds: string[]) => Promise<void>;
}
```

**Features**:
- List of categories
- Add category dialog
- Edit inline or dialog
- Delete with confirmation
- Drag-to-reorder
- Color picker
- Icon selector

**Usage**:
```javascript
import CategoryManager from 'components/calendar/CategoryManager';

<CategoryManager
  categories={categories}
  onAdd={handleAddCategory}
  onEdit={handleEditCategory}
  onDelete={handleDeleteCategory}
  onReorder={handleReorderCategories}
/>
```

### 10. UserManager

**File**: `components/calendar/UserManager.jsx`

**Purpose**: Admin interface for user management

**Props**:
```typescript
interface UserManagerProps {
  users: User[];
  onUpdateRole: (userId: string, role: UserRole) => Promise<void>;
  currentUserId: string;
}
```

**Features**:
- User table
- Role dropdown per user
- Search users
- User info (name, email, created date)
- Cannot change own role
- Confirmation on role change

**Usage**:
```javascript
import UserManager from 'components/calendar/UserManager';

<UserManager
  users={users}
  onUpdateRole={handleUpdateRole}
  currentUserId={currentUser.id}
/>
```

## Shared Utility Components

### Location: `src/components/calendar/shared/`

### 1. ConfirmDialog

**File**: `components/calendar/shared/ConfirmDialog.jsx`

**Purpose**: Reusable confirmation dialog

**Props**:
```typescript
interface ConfirmDialogProps {
  open: boolean;
  title: string;
  message: string;
  confirmText?: string;
  cancelText?: string;
  onConfirm: () => void;
  onCancel: () => void;
  severity?: 'info' | 'warning' | 'error';
}
```

**Usage**:
```javascript
import ConfirmDialog from 'components/calendar/shared/ConfirmDialog';

<ConfirmDialog
  open={deleteDialogOpen}
  title="Delete Event"
  message="Are you sure you want to delete this event?"
  confirmText="Delete"
  cancelText="Cancel"
  severity="error"
  onConfirm={confirmDelete}
  onCancel={cancelDelete}
/>
```

### 2. LoadingSpinner

**File**: `components/calendar/shared/LoadingSpinner.jsx`

**Purpose**: Consistent loading indicator

**Props**:
```typescript
interface LoadingSpinnerProps {
  size?: 'small' | 'medium' | 'large';
  fullPage?: boolean;
  message?: string;
}
```

**Usage**:
```javascript
import LoadingSpinner from 'components/calendar/shared/LoadingSpinner';

// Inline
<LoadingSpinner size="small" />

// Full page
<LoadingSpinner fullPage message="Loading events..." />
```

### 3. EmptyState

**File**: `components/calendar/shared/EmptyState.jsx`

**Purpose**: Empty state display

**Props**:
```typescript
interface EmptyStateProps {
  icon?: ReactNode;
  title: string;
  message?: string;
  action?: ReactNode;
}
```

**Usage**:
```javascript
import EmptyState from 'components/calendar/shared/EmptyState';

<EmptyState
  icon={<CalendarOutlined style={{ fontSize: 48 }} />}
  title="No events yet"
  message="Create your first event to get started"
  action={
    <Button onClick={handleCreate}>Create Event</Button>
  }
/>
```

### 4. ErrorBoundary

**File**: `components/calendar/shared/ErrorBoundary.jsx`

**Purpose**: Error boundary for error handling

**Props**:
```typescript
interface ErrorBoundaryProps {
  children: ReactNode;
  fallback?: ReactNode;
}
```

**Usage**:
```javascript
import ErrorBoundary from 'components/calendar/shared/ErrorBoundary';

<ErrorBoundary>
  <CalendarPage />
</ErrorBoundary>
```

## Component Dependency Tree

```
App
├── Layout (Mantis)
│   └── Calendar Pages
│       ├── CalendarPage
│       │   ├── CalendarHeader
│       │   │   ├── IconButton (Mantis)
│       │   │   └── AnimateButton (Mantis)
│       │   ├── EventFilters
│       │   │   └── Dot (Mantis)
│       │   └── CalendarGrid
│       │       ├── EventCard
│       │       │   ├── Dot (Mantis)
│       │       │   └── Avatar (Mantis)
│       │       └── EventDetail (in Dialog)
│       │           ├── IconButton (Mantis)
│       │           └── Avatar (Mantis)
│       ├── PublicCalendarPage
│       │   ├── CalendarGrid
│       │   └── EventDetail
│       └── AdminPage
│           ├── AnalyticEcommerce (Mantis)
│           ├── UserManager
│           └── CategoryManager
└── Shared
    ├── ConfirmDialog
    ├── LoadingSpinner
    ├── EmptyState
    └── ErrorBoundary
```

## Component Implementation Priority

### Phase 1: Core Components
1. LoadingSpinner
2. EmptyState
3. ErrorBoundary
4. ConfirmDialog

### Phase 2: Basic Calendar
5. EventCard
6. EventList
7. CalendarHeader
8. CalendarGrid (Month view only)

### Phase 3: Event Management
9. EventDetail
10. EventEditor
11. RecurrenceEditor

### Phase 4: Filtering & Admin
12. EventFilters
13. CategoryManager
14. UserManager

### Phase 5: Advanced Views
15. CalendarGrid (Week/Day views)
16. Advanced features

## Styling Guidelines

### Theme Integration
```javascript
import { useTheme } from '@mui/material/styles';

function MyComponent() {
  const theme = useTheme();

  return (
    <Box
      sx={{
        p: 2,
        bgcolor: theme.palette.background.paper,
        borderRadius: 1,
      }}
    >
      {/* Content */}
    </Box>
  );
}
```

### Consistent Spacing
```javascript
// Use theme spacing
sx={{
  p: 2,          // 16px padding
  mt: 1.5,       // 12px margin-top
  gap: 1,        // 8px gap
}}
```

### Responsive Design
```javascript
sx={{
  flexDirection: { xs: 'column', md: 'row' },
  width: { xs: '100%', md: '50%' },
}}
```

## Testing Components

### Unit Tests
```javascript
// EventCard.test.jsx
import { render, screen } from '@testing-library/react';
import EventCard from './EventCard';

test('renders event title', () => {
  const event = { title: 'Test Event', start_date: new Date() };
  render(<EventCard event={event} variant="compact" />);
  expect(screen.getByText('Test Event')).toBeInTheDocument();
});
```

### Integration Tests
```javascript
// CalendarGrid.test.jsx
import { render, screen, fireEvent } from '@testing-library/react';
import CalendarGrid from './CalendarGrid';

test('calls onEventClick when event is clicked', () => {
  const handleClick = jest.fn();
  const events = [{ id: '1', title: 'Event 1', start_date: new Date() }];

  render(
    <CalendarGrid
      view="month"
      currentDate={new Date()}
      events={events}
      onEventClick={handleClick}
    />
  );

  fireEvent.click(screen.getByText('Event 1'));
  expect(handleClick).toHaveBeenCalledWith(events[0]);
});
```

## Summary

✅ **11 new custom components** for calendar functionality
✅ **8 existing Mantis components** reused
✅ **4 shared utility components**
✅ Clear component hierarchy
✅ TypeScript prop interfaces
✅ Responsive and accessible
✅ Testable architecture
✅ Phased implementation plan
