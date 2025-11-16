# UI/UX Specifications

## Design Principles

### Mobile-First
- Design for 320px width minimum (iPhone SE)
- Touch targets minimum 44x44px
- Swipe gestures for navigation
- Bottom navigation for key actions

### Consistency
- Use Mantis design system throughout
- Consistent spacing (8px grid)
- Standard color palette
- Unified typography

### Accessibility
- WCAG 2.1 Level AA compliance
- Keyboard navigation
- Screen reader support
- Sufficient color contrast

## Color Palette (from Mantis)

```javascript
Primary:   #1677FF (Blue) - Main actions, links
Secondary: Grey variants - Text, borders
Success:   #52C41A (Green) - Success states, positive actions
Error:     #FF4D4F (Red) - Errors, destructive actions
Warning:   #FAAD14 (Orange) - Warnings, important info
Info:      #13C2C2 (Cyan) - Information, neutral highlights

Category Colors (Defaults):
Meeting:   #1677FF (Blue)
Event:     #52C41A (Green)
Deadline:  #FF4D4F (Red)
Holiday:   #722ED1 (Purple)
Other:     #8C8C8C (Grey)
```

## Typography

```
Headings:
h1: 38px/600 - Page titles
h2: 30px/600 - Section headers
h3: 24px/600 - Subsections
h4: 20px/600 - Card titles
h5: 16px/600 - Component headers
h6: 14px/400 - Labels

Body:
body1: 14px/400 - Primary text
body2: 12px/400 - Secondary text
caption: 12px/400 - Helper text
```

## Spacing System

Based on 8px grid:
- 4px (0.5): Tight spacing
- 8px (1): Close elements
- 12px (1.5): Related elements
- 16px (2): Component padding
- 24px (3): Section spacing
- 32px (4): Major sections

## Page Layouts

### 1. Main Calendar Page (Protected)

**Route**: `/calendar`
**Access**: Member, Manager, Administrator

**Layout Structure**:
```
┌─────────────────────────────────────────┐
│ Header (Mantis App Bar)                 │
│ [Logo] Calendar     [Avatar] [Logout]   │
├─────────────────────────────────────────┤
│ Calendar Header                          │
│ [Month/Week/Day/List] [Today] [<] [>]   │
│ [+ Create Event] (Manager/Admin only)    │
├─────────────────────────────────────────┤
│┌────────┬─────────────────────────────┐│
││Filters │ Calendar View               ││
││        │                             ││
││☐ Cat 1 │  [Calendar Grid/List]       ││
││☐ Cat 2 │                             ││
││☐ Cat 3 │                             ││
││        │                             ││
││[Clear] │                             ││
│└────────┴─────────────────────────────┘│
└─────────────────────────────────────────┘

Mobile (<768px):
┌──────────────────┐
│ Header           │
├──────────────────┤
│ View Switcher    │
│ [M][W][D][List]  │
├──────────────────┤
│ Month: Jan 2025  │
│ [<]  [Today]  [>]│
├──────────────────┤
│ Calendar Grid    │
│                  │
│  S M T W T F S   │
│     1  2  3  4 5 │
│  6  7  8  9 10..│
│                  │
└──────────────────┘
│ [Filter] [+]     │ <- Bottom Actions
└──────────────────┘
```

**Components Used**:
- MainCard for calendar container
- IconButton for navigation
- AnimateButton for create button
- Breadcrumbs for navigation
- Custom CalendarGrid component

**Interactions**:
- Click date cell → Create event (if Manager/Admin)
- Click event → View event details
- Swipe left/right → Next/previous period (mobile)
- Pull down → Refresh (mobile)

### 2. Event Detail Modal/Drawer

**Triggered by**: Clicking an event

**Layout**:
```
Desktop (Modal):
┌─────────────────────────────────┐
│ Event Title             [X]     │
├─────────────────────────────────┤
│                                 │
│ 📅 Jan 15, 2025                 │
│ ⏰ 10:00 AM - 11:00 AM          │
│ 📍 Conference Room A            │
│ 🏷️ Meeting                      │
│                                 │
│ Description:                    │
│ Lorem ipsum dolor sit amet...   │
│                                 │
│ Created by: John Doe            │
│                                 │
├─────────────────────────────────┤
│     [Edit] [Delete] [Close]     │
└─────────────────────────────────┘

Mobile (Bottom Sheet):
┌──────────────────┐
│ ═══ (handle)     │
│ Event Title      │
├──────────────────┤
│ 📅 Date          │
│ ⏰ Time          │
│ 📍 Location      │
│ 🏷️ Category     │
│                  │
│ Description...   │
│                  │
│ [Edit] [Delete]  │
└──────────────────┘
```

**Components Used**:
- Dialog/Drawer (MUI)
- MainCard
- IconButton
- Dot (for category indicator)
- Avatar (for creator)

**Conditional Display**:
- Edit/Delete buttons only if user has permission
- Creator info hidden for public viewers
- Recurrence info shown if recurring event

### 3. Event Editor (Create/Edit)

**Route**: `/calendar/event/new` or `/calendar/event/:id/edit`
**Access**: Manager, Administrator

**Layout**:
```
┌─────────────────────────────────┐
│ Create Event / Edit Event       │
├─────────────────────────────────┤
│                                 │
│ Title *                         │
│ [_____________________________] │
│                                 │
│ Start Date/Time *               │
│ [____Date____] [___Time____]    │
│                                 │
│ End Date/Time *                 │
│ [____Date____] [___Time____]    │
│                                 │
│ ☐ All-day event                 │
│                                 │
│ Location                        │
│ [_____________________________] │
│                                 │
│ Category                        │
│ [Select category ▼]            │
│                                 │
│ Visibility *                    │
│ ○ Public ○ Internal ○ Private   │
│                                 │
│ Description                     │
│ [                             ] │
│ [                             ] │
│ [                             ] │
│                                 │
│ ☐ Recurring Event               │
│   [Recurrence options...]       │
│                                 │
├─────────────────────────────────┤
│     [Cancel]     [Save Event]   │
└─────────────────────────────────┘
```

**Validation**:
- Title required (max 200 chars)
- Start date required
- End date required & must be after start
- Real-time validation feedback
- Disable save until valid

**Components Used**:
- MainCard
- TextField (MUI)
- DateTimePicker
- Select (MUI)
- Radio Group (MUI)
- Checkbox (MUI)
- AnimateButton

### 4. Filters Sidebar/Panel

**Layout**:
```
Desktop Sidebar:
┌─────────────────┐
│ Filters         │
├─────────────────┤
│ Categories      │
│ ☑ Meeting       │
│ ☑ Event         │
│ ☐ Deadline      │
│ ☑ Holiday       │
│                 │
│ Visibility      │
│ ☑ Public        │
│ ☑ Internal      │
│ ☐ Private       │
│                 │
│ Date Range      │
│ [This Month ▼]  │
│                 │
│ Creator         │
│ ○ All Events    │
│ ○ My Events     │
│                 │
│ [Clear Filters] │
└─────────────────┘

Mobile (Drawer from bottom):
┌──────────────────┐
│ Filters   [Apply]│
├──────────────────┤
│ Categories       │
│ [☑Meeting][☑Event]
│ [☐Deadline]      │
│                  │
│ Show:            │
│ [☑Public][☑Int.] │
│                  │
│ [Clear All]      │
└──────────────────┘
```

**Features**:
- Real-time filtering (desktop)
- Apply button (mobile)
- Clear all filters
- Show active filter count
- Persist filters in session

### 5. Admin Dashboard

**Route**: `/calendar/admin`
**Access**: Administrator only

**Layout**:
```
┌─────────────────────────────────────────┐
│ Admin Dashboard                         │
├─────────────────────────────────────────┤
│ ┌─────────┐ ┌─────────┐ ┌──────────┐   │
│ │ Total   │ │ Active  │ │ Upcoming │   │
│ │ Users   │ │ Events  │ │ Events   │   │
│ │  42     │ │  156    │ │   28     │   │
│ └─────────┘ └─────────┘ └──────────┘   │
├─────────────────────────────────────────┤
│ ┌─────────────────┬──────────────────┐  │
│ │ User Management │ Categories       │  │
│ ├─────────────────┼──────────────────┤  │
│ │ - John Doe      │ - Meeting (edit) │  │
│ │   Admin         │ - Event (edit)   │  │
│ │ - Jane Smith    │ - Deadline       │  │
│ │   Manager       │ + New Category   │  │
│ │ - ...           │                  │  │
│ │ + Invite User   │                  │  │
│ └─────────────────┴──────────────────┘  │
└─────────────────────────────────────────┘
```

**Components Used**:
- AnalyticEcommerce for stats cards
- MainCard for sections
- Table (MUI) for user list
- IconButton for actions
- Chip (MUI) for role badges

### 6. Public Calendar Page

**Route**: `/calendar/public`
**Access**: Everyone (no auth required)

**Layout**:
```
┌─────────────────────────────────────────┐
│ [NPO Logo] Public Events Calendar      │
├─────────────────────────────────────────┤
│ January 2025      [<]  [Today]  [>]     │
├─────────────────────────────────────────┤
│                                         │
│  Sun  Mon  Tue  Wed  Thu  Fri  Sat      │
│             1    2    3    4    5       │
│   6    7    8    9   10   11   12       │
│  13   14   15   16   17   18   19       │
│        • Gala                           │
│  20   21   22   23   24   25   26       │
│                                         │
└─────────────────────────────────────────┘

Mobile:
┌──────────────────┐
│ [Logo] Events    │
├──────────────────┤
│ Jan 2025         │
│ [<] [Today] [>]  │
├──────────────────┤
│ Upcoming Events: │
│                  │
│ ┌──────────────┐ │
│ │ Fundraising  │ │
│ │ Gala         │ │
│ │ Feb 1, 6 PM  │ │
│ └──────────────┘ │
│                  │
│ ┌──────────────┐ │
│ │ Volunteer    │ │
│ │ Orientation  │ │
│ │ Feb 5, 2 PM  │ │
│ └──────────────┘ │
└──────────────────┘
```

**Features**:
- Simplified interface
- Month view and list view only
- No login required
- Mobile-optimized
- Click event for details (modal)

## User Flows

### Flow 1: Create Event (Manager)

```
1. User logged in as Manager
2. Navigate to /calendar
3. Click "Create Event" button
4. Fill event form:
   - Enter title
   - Select start date/time
   - Select end date/time
   - Add location (optional)
   - Select category
   - Choose visibility
   - Add description
   - Configure recurrence (optional)
5. Click "Save Event"
6. Validation passes
7. Event created in database
8. Success message shown
9. Calendar refreshes with new event
10. User sees event on calendar
```

### Flow 2: Edit Event (Manager editing own event)

```
1. User views calendar
2. Click on own event
3. Event detail modal opens
4. Click "Edit" button
5. Event editor opens with populated fields
6. Modify fields (e.g., change time)
7. Click "Save Event"
8. Validation passes
9. Event updated in database
10. Success message shown
11. Modal closes
12. Calendar refreshes
13. User sees updated event
```

### Flow 3: Delete Recurring Event Instance

```
1. User clicks recurring event instance
2. Event detail shows "Part of recurring series"
3. Click "Delete" button
4. Confirmation dialog appears:
   "Delete this instance or all future?"
   [This Instance] [All Future] [Cancel]
5. User selects "This Instance"
6. Exception created in database
7. Calendar refreshes
8. Instance removed from view
9. Other instances remain
```

### Flow 4: Filter Events by Category

```
1. User on calendar page
2. Sidebar shows category filters
3. Click "Meeting" checkbox
4. Calendar immediately filters
5. Only Meeting events shown
6. Click "Event" checkbox
7. Now Meeting + Event shown
8. Click "Clear Filters"
9. All categories shown again
```

### Flow 5: Public User Views Event

```
1. Public user visits /calendar/public
2. Month view shows public events
3. Click on event
4. Bottom sheet opens with details:
   - Title
   - Date/time
   - Location
   - Description
   - Category
5. No creator info shown
6. No edit/delete buttons
7. Click close or swipe down
8. Back to calendar
```

### Flow 6: Admin Assigns Role

```
1. Admin navigates to /calendar/admin
2. User Management section visible
3. Find user "Jane Smith" (role: Member)
4. Click role dropdown
5. Select "Manager"
6. Confirmation: "Assign Manager role to Jane Smith?"
7. Click "Confirm"
8. Database updated
9. Jane's role changed to Manager
10. Jane can now create events
```

## Responsive Breakpoints

### Mobile (< 768px)
- Single column layout
- Bottom navigation
- Full-width cards
- Simplified calendar grid
- Drawer filters
- Bottom sheet modals

### Tablet (768px - 1024px)
- Two column layout where appropriate
- Sidebar filters visible
- Full calendar features
- Modal dialogs

### Desktop (> 1024px)
- Multi-column layout
- Persistent sidebar
- Expanded calendar grid
- Modal dialogs
- Hover states

## Accessibility Features

### Keyboard Navigation
- Tab through interactive elements
- Enter to activate buttons
- Escape to close modals
- Arrow keys for date navigation

### Screen Reader Support
- ARIA labels on all controls
- Role attributes
- Live regions for dynamic updates
- Form labels associated with inputs

### Color Contrast
- All text meets WCAG AA (4.5:1)
- Large text meets AAA (3:1)
- Focus indicators visible
- Don't rely on color alone

## Loading States

### Skeleton Screens
```
Calendar loading:
┌──────────────────┐
│ ▓▓▓▓ ▓▓▓▓ ▓▓▓▓  │ <- Header skeleton
├──────────────────┤
│ ░░ ░░ ░░ ░░ ░░  │ <- Day headers
│ ▓▓ ▓▓ ▓▓ ▓▓ ▓▓  │ <- Date cells
│ ▓▓ ▓▓ ▓▓ ▓▓ ▓▓  │
└──────────────────┘
```

### Spinners
- Page load: Full-screen spinner
- Button action: Button spinner
- Inline load: Small spinner

### Progress Indicators
- Form submission: Linear progress
- File upload: Circular with percentage

## Empty States

### No Events
```
┌──────────────────┐
│  📅              │
│  No events yet   │
│                  │
│  [Create Event]  │
└──────────────────┘
```

### No Search Results
```
┌──────────────────┐
│  🔍              │
│  No events found │
│  Try different   │
│  filters         │
│                  │
│  [Clear Filters] │
└──────────────────┘
```

## Error States

### Form Errors
```
Title *
[___________________________]
❌ Title is required
```

### API Errors
```
┌──────────────────┐
│  ⚠️              │
│  Failed to load  │
│  events          │
│                  │
│  [Retry]         │
└──────────────────┘
```

## Animations

### Transitions
- Page transitions: 200ms ease
- Modal open/close: 300ms ease-out
- Filter apply: 150ms ease
- Calendar view change: 250ms ease-in-out

### Micro-interactions
- Button hover: Scale 1.02
- Card hover: Lift (shadow)
- Event click: Ripple effect
- Success: Checkmark animation

## Summary

✅ Mobile-first responsive design
✅ Mantis design system compliance
✅ Accessible (WCAG 2.1 AA)
✅ Clear user flows
✅ Comprehensive states (loading, empty, error)
✅ Smooth animations
✅ Intuitive interactions
