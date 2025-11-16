# Implementation Plan

## Overview

This document outlines a phased approach to implementing the NPO Calendar application. The plan is structured to deliver working features incrementally, allowing for testing and feedback at each stage.

## Timeline Estimate

**Total Duration**: 10-12 weeks (single developer)
- Phase 1: 2 weeks
- Phase 2: 3 weeks
- Phase 3: 2 weeks
- Phase 4: 2 weeks
- Phase 5: 1-2 weeks

## Prerequisites

Before starting development:

- [ ] Mantis React Admin Template installed and running
- [ ] Supabase account created
- [ ] Environment variables configured
- [ ] Git repository initialized
- [ ] Development tools installed (Node.js 18+, npm/yarn)
- [ ] IDE set up (VS Code recommended)

## Phase 1: Foundation (Week 1-2)

**Goal**: Set up backend, authentication, and basic infrastructure

### Week 1: Supabase Setup

#### Tasks

1. **Supabase Project Configuration**
   - [ ] Create Supabase project
   - [ ] Configure environment variables
   - [ ] Set up local development environment
   - [ ] Install Supabase client library

2. **Database Schema**
   - [ ] Create `profiles` table
   - [ ] Create `categories` table
   - [ ] Create `recurrences` table
   - [ ] Create `events` table
   - [ ] Create `event_exceptions` table
   - [ ] Add indexes for performance
   - [ ] Create database functions
   - [ ] Insert default categories

3. **Row Level Security (RLS)**
   - [ ] Enable RLS on all tables
   - [ ] Create policies for `profiles` table
   - [ ] Create policies for `categories` table
   - [ ] Create policies for `events` table
   - [ ] Create policies for `recurrences` table
   - [ ] Create policies for `event_exceptions` table
   - [ ] Test policies with different user roles

4. **Authentication Setup**
   - [ ] Configure Supabase Auth settings
   - [ ] Enable email/password authentication
   - [ ] Set up email templates (verification, reset)
   - [ ] Configure redirect URLs

**Deliverables**:
- ✅ Supabase database fully configured
- ✅ RLS policies active and tested
- ✅ Authentication working
- ✅ Seed data loaded

### Week 2: Frontend Foundation

#### Tasks

1. **Project Structure**
   - [ ] Create folder structure for calendar app
   - [ ] Set up TypeScript (optional but recommended)
   - [ ] Configure ESLint and Prettier
   - [ ] Create shared constants file

2. **Supabase Client Integration**
   - [ ] Create Supabase client (`services/supabase/client.js`)
   - [ ] Create auth service (`services/supabase/auth.js`)
   - [ ] Create events service (`services/supabase/events.js`)
   - [ ] Create categories service (`services/supabase/categories.js`)
   - [ ] Create users service (`services/supabase/users.js`)

3. **React Hooks**
   - [ ] Create `useAuth` hook
   - [ ] Create `useEvents` hook
   - [ ] Create `useCategories` hook
   - [ ] Create `useUsers` hook (admin)

4. **Route Protection**
   - [ ] Create `ProtectedRoute` component
   - [ ] Create `PublicRoute` component
   - [ ] Configure router with protected routes
   - [ ] Add role-based route guards

5. **Shared Components**
   - [ ] Create `LoadingSpinner` component
   - [ ] Create `EmptyState` component
   - [ ] Create `ErrorBoundary` component
   - [ ] Create `ConfirmDialog` component

6. **Authentication Pages**
   - [ ] Integrate Supabase with existing Mantis login page
   - [ ] Update registration page for Supabase
   - [ ] Add password reset functionality
   - [ ] Test authentication flow

**Deliverables**:
- ✅ Supabase integrated with React app
- ✅ Authentication working end-to-end
- ✅ Protected routes functional
- ✅ Shared components ready

**Testing**:
- User registration creates profile
- Login/logout works
- Protected routes redirect to login
- Role-based access enforced

---

## Phase 2: Core Calendar (Week 3-5)

**Goal**: Build main calendar interface with basic event viewing

### Week 3: Calendar Display

#### Tasks

1. **Calendar Components**
   - [ ] Create `CalendarHeader` component
     - View switcher (Month/Week/Day/List)
     - Date navigation (prev/next/today)
     - Create event button (conditional)
   - [ ] Create `EventCard` component
     - Compact variant for grid
     - Detailed variant for list
     - Color-coded by category
   - [ ] Create `EventList` component
     - List view implementation
     - Group by date
     - Loading and empty states

2. **Month View**
   - [ ] Create `CalendarGrid` component
   - [ ] Create `MonthGrid` sub-component
     - 7x6 grid (weeks)
     - Day headers
     - Date cells
     - Event display in cells
   - [ ] Handle overflow events (scrollable/truncated)
   - [ ] Current day highlighting
   - [ ] Mobile-responsive grid

3. **Main Calendar Page**
   - [ ] Create `/pages/calendar/index.jsx`
   - [ ] Integrate `CalendarHeader`
   - [ ] Integrate `CalendarGrid`
   - [ ] Load events for current month
   - [ ] Handle view switching
   - [ ] Handle date navigation

4. **Event Detail View**
   - [ ] Create `EventDetail` component
   - [ ] Display all event information
   - [ ] Show category with color
   - [ ] Show creator (if permitted)
   - [ ] Conditional action buttons
   - [ ] Implement as Dialog/Modal

**Deliverables**:
- ✅ Month view calendar working
- ✅ Events display on calendar
- ✅ Event detail view functional
- ✅ Navigation between months
- ✅ Mobile responsive

**Testing**:
- Month grid displays correctly
- Events appear on correct dates
- Clicking event shows details
- Navigation changes months
- Mobile layout works

### Week 4: Additional Views

#### Tasks

1. **Week View**
   - [ ] Create `WeekGrid` sub-component
   - [ ] Time slots (hourly grid)
   - [ ] All-day events section
   - [ ] Timed events in grid
   - [ ] Current time indicator
   - [ ] Mobile: scrollable day view

2. **Day View**
   - [ ] Create `DayGrid` sub-component
   - [ ] Detailed time grid
   - [ ] All-day events section
   - [ ] Current time indicator
   - [ ] Scroll to current time

3. **List View**
   - [ ] Enhance `EventList` component
   - [ ] Chronological listing
   - [ ] Group by date
   - [ ] Infinite scroll or pagination
   - [ ] Date range selector

4. **View State Management**
   - [ ] Persist view preference (localStorage)
   - [ ] Maintain date context when switching views
   - [ ] Smooth transitions between views

**Deliverables**:
- ✅ Week view functional
- ✅ Day view functional
- ✅ List view functional
- ✅ View switching seamless

**Testing**:
- All views display events correctly
- View preferences saved
- Date context maintained
- Mobile views optimized

### Week 5: Filtering & Search

#### Tasks

1. **Filter Sidebar**
   - [ ] Create `EventFilters` component
   - [ ] Category checkboxes
   - [ ] Visibility filter (authenticated users)
   - [ ] My events toggle (managers)
   - [ ] Clear all filters button
   - [ ] Desktop sidebar layout
   - [ ] Mobile drawer layout

2. **Search Functionality**
   - [ ] Add search bar to CalendarHeader
   - [ ] Implement text search (title, description)
   - [ ] Real-time search results
   - [ ] Highlight search terms
   - [ ] Clear search button

3. **Filter Logic**
   - [ ] Implement category filtering
   - [ ] Implement visibility filtering
   - [ ] Implement creator filtering
   - [ ] Combine multiple filters
   - [ ] Update calendar display

4. **Filter State**
   - [ ] Manage filter state
   - [ ] Persist filters in session
   - [ ] Display active filter count
   - [ ] Update URL query params (optional)

**Deliverables**:
- ✅ Filtering by category works
- ✅ Search functionality implemented
- ✅ Filters combine correctly
- ✅ Filter state persisted

**Testing**:
- Category filters show/hide events
- Search returns correct results
- Multiple filters work together
- Mobile filter drawer works

---

## Phase 3: Event Management (Week 6-7)

**Goal**: Enable creating, editing, and deleting events

### Week 6: Event Creation & Editing

#### Tasks

1. **Event Editor Component**
   - [ ] Create `EventEditor` component
   - [ ] Title field with validation
   - [ ] Start date/time picker
   - [ ] End date/time picker
   - [ ] All-day event toggle
   - [ ] Location field
   - [ ] Category selector
   - [ ] Visibility radio group
   - [ ] Description textarea
   - [ ] Form validation

2. **Date/Time Pickers**
   - [ ] Integrate date picker library (MUI DatePicker)
   - [ ] Combine date and time selection
   - [ ] Validate end > start
   - [ ] Handle all-day events
   - [ ] Mobile-friendly pickers

3. **Create Event Flow**
   - [ ] Add create button to CalendarHeader
   - [ ] Open EventEditor in dialog
   - [ ] Handle form submission
   - [ ] Create event via API
   - [ ] Show success/error messages
   - [ ] Refresh calendar
   - [ ] Close dialog

4. **Edit Event Flow**
   - [ ] Add edit button to EventDetail
   - [ ] Populate EventEditor with event data
   - [ ] Handle form submission
   - [ ] Update event via API
   - [ ] Show success/error messages
   - [ ] Refresh calendar
   - [ ] Close dialog

5. **Permissions**
   - [ ] Show create button only for Managers/Admins
   - [ ] Show edit button only if user can edit
   - [ ] Validate permissions server-side (RLS)
   - [ ] Handle permission errors gracefully

**Deliverables**:
- ✅ Managers can create events
- ✅ Users can edit own events
- ✅ Admins can edit any event
- ✅ Form validation working
- ✅ Success/error feedback

**Testing**:
- Create event saves to database
- Edit event updates database
- Permissions enforced
- Validation prevents invalid data
- Calendar refreshes after save

### Week 7: Event Deletion & Recurring Events

#### Tasks

1. **Event Deletion**
   - [ ] Add delete button to EventDetail
   - [ ] Create confirmation dialog
   - [ ] Handle delete via API
   - [ ] Show success message
   - [ ] Refresh calendar
   - [ ] Handle errors

2. **Recurring Event Setup**
   - [ ] Create `RecurrenceEditor` component
   - [ ] Recurrence type selector (daily/weekly/monthly)
   - [ ] Interval input
   - [ ] End date picker
   - [ ] Days of week selector (weekly)
   - [ ] Preview upcoming instances

3. **Recurring Event Creation**
   - [ ] Integrate RecurrenceEditor into EventEditor
   - [ ] Save recurrence rule to database
   - [ ] Link event to recurrence rule
   - [ ] Generate event instances (via function)
   - [ ] Display recurring events on calendar

4. **Recurring Event Editing**
   - [ ] "Edit This Instance" option
     - Create exception event
     - Modify single instance
   - [ ] "Edit All Future" option
     - Update recurrence rule
     - Create new rule from date
   - [ ] Dialog for user choice

5. **Recurring Event Deletion**
   - [ ] "Delete This Instance" option
     - Create event exception
   - [ ] "Delete All Instances" option
     - Delete event and recurrence rule
   - [ ] Dialog for user choice

**Deliverables**:
- ✅ Event deletion working
- ✅ Recurring events can be created
- ✅ Recurring instances display correctly
- ✅ Edit/delete single instances
- ✅ Edit/delete all instances

**Testing**:
- Delete removes event
- Recurring events generate correctly
- Edit instance creates exception
- Delete instance creates exception
- Edit all updates future instances

---

## Phase 4: Admin Features (Week 8-9)

**Goal**: Build admin dashboard and management interfaces

### Week 8: User Management

#### Tasks

1. **Admin Dashboard Page**
   - [ ] Create `/pages/calendar/admin/index.jsx`
   - [ ] Add route protection (admin only)
   - [ ] Create layout with statistics
   - [ ] Add navigation menu item

2. **Statistics Cards**
   - [ ] Total users count
   - [ ] Active events count
   - [ ] Upcoming events count
   - [ ] Use `AnalyticEcommerce` component

3. **User Manager Component**
   - [ ] Create `UserManager` component
   - [ ] Display users in table
   - [ ] Show user info (name, email, role, created date)
   - [ ] Role dropdown per user
   - [ ] Handle role changes
   - [ ] Prevent changing own role
   - [ ] Confirmation dialog for role changes

4. **Users Service**
   - [ ] Implement `getAllUsers` function
   - [ ] Implement `updateUserRole` function
   - [ ] Add error handling
   - [ ] Update `useUsers` hook

5. **Permissions**
   - [ ] Verify admin-only access
   - [ ] Test RLS policies
   - [ ] Handle permission errors

**Deliverables**:
- ✅ Admin dashboard page accessible
- ✅ User list displays
- ✅ Admins can change user roles
- ✅ Statistics display correctly

**Testing**:
- Only admins can access page
- User list loads correctly
- Role changes save to database
- Cannot change own role
- Statistics accurate

### Week 9: Category Management

#### Tasks

1. **Category Manager Component**
   - [ ] Create `CategoryManager` component
   - [ ] Display categories in list
   - [ ] Show category properties
   - [ ] Add category button
   - [ ] Edit category inline or dialog
   - [ ] Delete category with confirmation
   - [ ] Drag-to-reorder categories

2. **Add Category Flow**
   - [ ] Create category form dialog
   - [ ] Name input with validation (unique)
   - [ ] Color picker
   - [ ] Icon selector (optional)
   - [ ] Description textarea
   - [ ] Handle form submission
   - [ ] Create via API
   - [ ] Update category list

3. **Edit Category Flow**
   - [ ] Populate form with category data
   - [ ] Allow changing all properties
   - [ ] Handle form submission
   - [ ] Update via API
   - [ ] Update category list

4. **Delete Category Flow**
   - [ ] Confirmation dialog
   - [ ] Check for events using category
   - [ ] Soft delete (mark inactive)
   - [ ] Handle events with deleted category

5. **Reorder Categories**
   - [ ] Drag-and-drop functionality
   - [ ] Update display_order
   - [ ] Save new order to database

**Deliverables**:
- ✅ Category list displays
- ✅ Admins can add categories
- ✅ Admins can edit categories
- ✅ Admins can delete categories
- ✅ Categories can be reordered

**Testing**:
- Add category creates in database
- Edit updates database
- Delete marks inactive
- Reorder saves correctly
- Changes reflect in event editor

---

## Phase 5: Public Calendar & Polish (Week 10-12)

**Goal**: Public calendar page and final polish

### Week 10: Public Calendar

#### Tasks

1. **Public Calendar Page**
   - [ ] Create `/pages/calendar/public.jsx`
   - [ ] Create simplified layout (no auth)
   - [ ] Add NPO branding header
   - [ ] Month view calendar
   - [ ] List view (optional)

2. **Public Event Display**
   - [ ] Load only public events
   - [ ] No authentication required
   - [ ] No creator information shown
   - [ ] Read-only event details

3. **Public Event Detail**
   - [ ] Dialog or bottom sheet
   - [ ] Show title, date, time, location, description
   - [ ] No edit/delete buttons
   - [ ] Share button (future)

4. **Mobile Optimization**
   - [ ] Touch-friendly interface
   - [ ] Swipe gestures
   - [ ] Responsive layout
   - [ ] Bottom sheet for events

5. **SEO & Sharing**
   - [ ] Add meta tags
   - [ ] Open Graph tags
   - [ ] Structured data (schema.org)
   - [ ] Social sharing preview

**Deliverables**:
- ✅ Public calendar accessible
- ✅ Public events display
- ✅ No authentication required
- ✅ Mobile optimized

**Testing**:
- Public users can access
- Only public events shown
- Mobile experience smooth
- No permission errors

### Week 11: Testing & Bug Fixes

#### Tasks

1. **Unit Tests**
   - [ ] Write tests for utility functions
   - [ ] Write tests for hooks
   - [ ] Write tests for services
   - [ ] Achieve 60%+ code coverage

2. **Component Tests**
   - [ ] Test CalendarGrid rendering
   - [ ] Test EventCard display
   - [ ] Test EventEditor form
   - [ ] Test EventFilters logic

3. **Integration Tests**
   - [ ] Test complete user flows
   - [ ] Test create event flow
   - [ ] Test edit event flow
   - [ ] Test delete event flow
   - [ ] Test filter events flow

4. **E2E Tests (optional)**
   - [ ] Set up Cypress or Playwright
   - [ ] Write critical path tests
   - [ ] Test authentication flow
   - [ ] Test event CRUD flow

5. **Bug Fixes**
   - [ ] Fix reported bugs
   - [ ] Address edge cases
   - [ ] Handle error scenarios
   - [ ] Improve error messages

**Deliverables**:
- ✅ Test coverage > 60%
- ✅ Critical bugs fixed
- ✅ Error handling improved

### Week 12: Polish & Documentation

#### Tasks

1. **Performance Optimization**
   - [ ] Optimize database queries
   - [ ] Add pagination where needed
   - [ ] Implement caching
   - [ ] Lazy load components
   - [ ] Optimize images
   - [ ] Reduce bundle size

2. **Accessibility**
   - [ ] Keyboard navigation
   - [ ] Screen reader testing
   - [ ] ARIA labels
   - [ ] Color contrast check
   - [ ] Focus indicators

3. **UI Polish**
   - [ ] Consistent spacing
   - [ ] Animation refinements
   - [ ] Loading states
   - [ ] Empty states
   - [ ] Error states
   - [ ] Success messages

4. **User Documentation**
   - [ ] User guide for creating events
   - [ ] Admin guide for management
   - [ ] FAQ document
   - [ ] Help tooltips in UI

5. **Developer Documentation**
   - [ ] README updates
   - [ ] Setup instructions
   - [ ] Environment variables guide
   - [ ] Deployment guide
   - [ ] API documentation

6. **Final Testing**
   - [ ] Cross-browser testing
   - [ ] Mobile device testing
   - [ ] Performance testing
   - [ ] Security audit
   - [ ] Accessibility audit

**Deliverables**:
- ✅ Performance optimized
- ✅ Accessibility compliant
- ✅ UI polished
- ✅ Documentation complete
- ✅ Ready for production

---

## Deployment Checklist

Before deploying to production:

- [ ] Environment variables configured
- [ ] Supabase production project ready
- [ ] Database migrations applied
- [ ] RLS policies enabled
- [ ] Default categories seeded
- [ ] First admin user created
- [ ] SSL certificate configured
- [ ] Domain configured
- [ ] Analytics set up (optional)
- [ ] Error tracking set up (Sentry, etc.)
- [ ] Backup strategy configured
- [ ] Monitoring set up
- [ ] Performance tested
- [ ] Security audited
- [ ] User acceptance testing complete

## Post-Launch Tasks

After initial launch:

1. **Monitor & Support**
   - Monitor error logs
   - Track performance metrics
   - Respond to user feedback
   - Fix critical bugs quickly

2. **User Training**
   - Train NPO staff
   - Create tutorial videos
   - Hold Q&A sessions

3. **Iteration**
   - Gather user feedback
   - Prioritize improvements
   - Plan next features

## Future Enhancements

Features to consider after v1.0:

- [ ] Attendance tracking / RSVP system
- [ ] Email notifications
- [ ] Calendar export (iCal, Google Calendar)
- [ ] Event comments/discussion
- [ ] File attachments
- [ ] Event templates
- [ ] Approval workflows
- [ ] Resource booking
- [ ] Multi-organization support
- [ ] Advanced analytics
- [ ] Public embeddable widget
- [ ] Mobile app (React Native)

## Risk Mitigation

### Technical Risks

**Risk**: Supabase performance issues with many events
**Mitigation**: Implement pagination, caching, optimize queries

**Risk**: Complex recurring event logic
**Mitigation**: Thorough testing, limit max occurrences, clear error messages

**Risk**: Permission bypass
**Mitigation**: RLS policies enforce security, never rely on frontend only

### Project Risks

**Risk**: Timeline delays
**Mitigation**: Phased approach allows shipping MVP early

**Risk**: Scope creep
**Mitigation**: Clearly defined out-of-scope items, disciplined prioritization

**Risk**: Single developer
**Mitigation**: Good documentation, modular code, code reviews (if possible)

## Success Metrics

### Launch Criteria (v1.0)

- [ ] All Phase 1-4 tasks complete
- [ ] Test coverage > 60%
- [ ] Performance benchmarks met
- [ ] Security audit passed
- [ ] User acceptance testing passed
- [ ] Documentation complete

### Post-Launch Metrics

- User adoption (registrations per month)
- Event creation rate
- Active users (DAU, MAU)
- Page load performance
- Error rates
- User satisfaction scores

## Summary

This implementation plan provides a structured approach to building the NPO Calendar application over 10-12 weeks. Each phase delivers working features, allowing for iterative development and continuous testing. The phased approach also allows for adjustments based on feedback and changing requirements.

**Key Success Factors**:
- ✅ Start with solid foundation (Supabase, auth)
- ✅ Deliver incrementally (working features each phase)
- ✅ Test continuously (unit, integration, E2E)
- ✅ Focus on mobile experience
- ✅ Maintain good documentation
- ✅ Prioritize security (RLS policies)
- ✅ Plan for iteration (feedback loops)
