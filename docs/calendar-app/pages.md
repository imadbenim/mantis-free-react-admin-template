# Pages Structure and Routing

## Overview

This document defines the page structure, routing configuration, and navigation for the NPO Calendar application built on Mantis React Admin Template.

## Route Structure

```
/                           → Dashboard (existing Mantis)
/calendar                   → Main Calendar (protected)
/calendar/public            → Public Calendar (public access)
/calendar/admin             → Admin Dashboard (admin only)
/login                      → Login Page
/register                   → Registration Page
/forgot-password            → Password Reset
/reset-password             → Set New Password
```

## Page Definitions

### 1. Main Calendar Page

**Route**: `/calendar`
**File**: `src/pages/calendar/index.jsx`
**Access**: Member, Manager, Administrator (protected)
**Layout**: MainLayout (Mantis)

**Purpose**: Primary calendar interface for authenticated users

**Features**:
- Multi-view calendar (Month/Week/Day/List)
- Event filtering and search
- Create event button (Manager/Admin)
- Event detail view
- Real-time updates

**Page Structure**:
```javascript
// src/pages/calendar/index.jsx
import { useState } from 'react';
import { Grid, Box } from '@mui/material';
import MainCard from 'components/MainCard';
import CalendarHeader from 'components/calendar/CalendarHeader';
import CalendarGrid from 'components/calendar/CalendarGrid';
import EventFilters from 'components/calendar/EventFilters';
import EventDetail from 'components/calendar/EventDetail';
import EventEditor from 'components/calendar/EventEditor';
import { useEvents } from 'hooks/useEvents';
import { useAuth } from 'hooks/useAuth';

export default function CalendarPage() {
  const { profile, isManager, isAdmin } = useAuth();
  const [view, setView] = useState('month');
  const [currentDate, setCurrentDate] = useState(new Date());
  const [selectedEvent, setSelectedEvent] = useState(null);
  const [isEditing, setIsEditing] = useState(false);

  const { events, loading } = useEvents(
    startOfMonth(currentDate),
    endOfMonth(currentDate)
  );

  const canCreateEvent = isManager || isAdmin;

  return (
    <Grid container spacing={3}>
      {/* Filters Sidebar - Desktop */}
      <Grid item xs={12} md={3} sx={{ display: { xs: 'none', md: 'block' } }}>
        <EventFilters
          categories={categories}
          selectedCategories={selectedCategories}
          onCategoriesChange={setSelectedCategories}
          variant="sidebar"
        />
      </Grid>

      {/* Main Calendar */}
      <Grid item xs={12} md={9}>
        <MainCard>
          <CalendarHeader
            view={view}
            currentDate={currentDate}
            onViewChange={setView}
            onDateChange={setCurrentDate}
            onCreateEvent={() => setIsEditing(true)}
            canCreateEvent={canCreateEvent}
          />

          <CalendarGrid
            view={view}
            currentDate={currentDate}
            events={filteredEvents}
            onEventClick={setSelectedEvent}
            loading={loading}
          />
        </MainCard>
      </Grid>

      {/* Event Detail Dialog */}
      {selectedEvent && (
        <Dialog open={!!selectedEvent} onClose={() => setSelectedEvent(null)}>
          <EventDetail
            event={selectedEvent}
            onEdit={() => setIsEditing(true)}
            onDelete={handleDelete}
            onClose={() => setSelectedEvent(null)}
            canEdit={canEditEvent(selectedEvent, profile)}
            canDelete={canDeleteEvent(selectedEvent, profile)}
          />
        </Dialog>
      )}

      {/* Event Editor Dialog */}
      {isEditing && (
        <Dialog open={isEditing} onClose={() => setIsEditing(false)} maxWidth="md" fullWidth>
          <EventEditor
            event={selectedEvent}
            mode={selectedEvent ? 'edit' : 'create'}
            onSave={handleSave}
            onCancel={() => setIsEditing(false)}
          />
        </Dialog>
      )}
    </Grid>
  );
}
```

**State Management**:
- `view`: Current calendar view
- `currentDate`: Current date for calendar
- `selectedEvent`: Event being viewed/edited
- `isEditing`: Editor dialog open state
- `selectedCategories`: Active category filters
- `selectedVisibility`: Active visibility filters

**Data Loading**:
- Events for current date range
- Categories for filters
- User profile for permissions

**Permissions**:
- View: Member+
- Create: Manager+
- Edit: Own events (Manager) or all (Admin)
- Delete: Own events (Manager) or all (Admin)

### 2. Public Calendar Page

**Route**: `/calendar/public`
**File**: `src/pages/calendar/public.jsx`
**Access**: Public (no authentication required)
**Layout**: MinimalLayout (custom)

**Purpose**: Public-facing calendar showing only public events

**Features**:
- Month view (primary)
- List view (optional)
- Public events only
- Read-only event details
- No authentication required
- NPO branding

**Page Structure**:
```javascript
// src/pages/calendar/public.jsx
import { useState } from 'react';
import { Box, Container, Typography } from '@mui/material';
import CalendarGrid from 'components/calendar/CalendarGrid';
import EventDetail from 'components/calendar/EventDetail';
import { useEvents } from 'hooks/useEvents';

export default function PublicCalendarPage() {
  const [currentDate, setCurrentDate] = useState(new Date());
  const [selectedEvent, setSelectedEvent] = useState(null);

  const { events, loading } = useEvents(
    startOfMonth(currentDate),
    endOfMonth(currentDate),
    { visibility: 'public', publicOnly: true }
  );

  return (
    <Box sx={{ bgcolor: 'background.default', minHeight: '100vh', py: 4 }}>
      <Container maxWidth="lg">
        {/* Header with NPO branding */}
        <Box sx={{ mb: 4, textAlign: 'center' }}>
          <Typography variant="h2" gutterBottom>
            {process.env.VITE_NPO_NAME || 'Public Events Calendar'}
          </Typography>
          <Typography variant="body1" color="text.secondary">
            View our upcoming public events
          </Typography>
        </Box>

        {/* Calendar */}
        <CalendarGrid
          view="month"
          currentDate={currentDate}
          events={events}
          onEventClick={setSelectedEvent}
          onDateChange={setCurrentDate}
          loading={loading}
          publicMode
        />

        {/* Event Detail (read-only) */}
        {selectedEvent && (
          <Dialog open={!!selectedEvent} onClose={() => setSelectedEvent(null)}>
            <EventDetail
              event={selectedEvent}
              onClose={() => setSelectedEvent(null)}
              canEdit={false}
              canDelete={false}
              publicView
            />
          </Dialog>
        )}
      </Container>
    </Box>
  );
}
```

**Differences from Main Calendar**:
- No authentication required
- Public events only
- Simplified interface
- No create/edit/delete actions
- Custom branding header
- Month view only (initially)

### 3. Admin Dashboard Page

**Route**: `/calendar/admin`
**File**: `src/pages/calendar/admin/index.jsx`
**Access**: Administrator only
**Layout**: MainLayout

**Purpose**: System administration and configuration

**Features**:
- User management
- Category configuration
- System statistics
- Event overview
- Settings

**Page Structure**:
```javascript
// src/pages/calendar/admin/index.jsx
import { Grid } from '@mui/material';
import MainCard from 'components/MainCard';
import AnalyticEcommerce from 'components/cards/statistics/AnalyticEcommerce';
import UserManager from 'components/calendar/UserManager';
import CategoryManager from 'components/calendar/CategoryManager';
import { useUsers } from 'hooks/useUsers';
import { useCategories } from 'hooks/useCategories';
import { useEventStats } from 'hooks/useEventStats';

export default function AdminDashboard() {
  const { users, updateUserRole } = useUsers();
  const { categories, addCategory, updateCategory, deleteCategory, reorderCategories } = useCategories();
  const { stats, loading } = useEventStats();

  return (
    <Grid container spacing={3}>
      {/* Statistics */}
      <Grid item xs={12} sm={6} md={4}>
        <AnalyticEcommerce
          title="Total Users"
          count={stats.totalUsers}
        />
      </Grid>

      <Grid item xs={12} sm={6} md={4}>
        <AnalyticEcommerce
          title="Active Events"
          count={stats.activeEvents}
          percentage={stats.eventGrowth}
          color="success"
        />
      </Grid>

      <Grid item xs={12} sm={6} md={4}>
        <AnalyticEcommerce
          title="Upcoming Events"
          count={stats.upcomingEvents}
        />
      </Grid>

      {/* User Management */}
      <Grid item xs={12} lg={6}>
        <MainCard title="User Management">
          <UserManager
            users={users}
            onUpdateRole={updateUserRole}
            currentUserId={currentUser.id}
          />
        </MainCard>
      </Grid>

      {/* Category Management */}
      <Grid item xs={12} lg={6}>
        <MainCard title="Event Categories">
          <CategoryManager
            categories={categories}
            onAdd={addCategory}
            onEdit={updateCategory}
            onDelete={deleteCategory}
            onReorder={reorderCategories}
          />
        </MainCard>
      </Grid>
    </Grid>
  );
}
```

**Permissions Check**:
```javascript
// Protect route in router config
{
  path: '/calendar/admin',
  element: (
    <ProtectedRoute roles={['admin']}>
      <AdminDashboard />
    </ProtectedRoute>
  )
}
```

### 4. Authentication Pages

#### Login Page
**Route**: `/login`
**File**: `src/pages/authentication/Login.jsx`
**Access**: Public
**Layout**: MinimalLayout (Mantis auth layout)

Uses existing Mantis authentication pages with Supabase integration.

#### Register Page
**Route**: `/register`
**File**: `src/pages/authentication/Register.jsx`
**Access**: Public
**Layout**: MinimalLayout

#### Forgot Password Page
**Route**: `/forgot-password`
**File**: `src/pages/authentication/ForgotPassword.jsx`
**Access**: Public

#### Reset Password Page
**Route**: `/reset-password`
**File**: `src/pages/authentication/ResetPassword.jsx`
**Access**: Public (with token)

## Routing Configuration

**File**: `src/routes/index.jsx`

```javascript
import { lazy } from 'react';
import { createBrowserRouter } from 'react-router-dom';
import Loadable from 'components/Loadable';
import MainLayout from 'layout/MainLayout';
import MinimalLayout from 'layout/MinimalLayout';
import ProtectedRoute from 'components/ProtectedRoute';
import PublicRoute from 'components/PublicRoute';

// Lazy load pages
const DashboardDefault = Loadable(lazy(() => import('pages/dashboard')));
const CalendarPage = Loadable(lazy(() => import('pages/calendar')));
const PublicCalendarPage = Loadable(lazy(() => import('pages/calendar/public')));
const AdminDashboard = Loadable(lazy(() => import('pages/calendar/admin')));
const Login = Loadable(lazy(() => import('pages/authentication/Login')));
const Register = Loadable(lazy(() => import('pages/authentication/Register')));

const router = createBrowserRouter([
  {
    path: '/',
    element: <MainLayout />,
    children: [
      {
        path: '/',
        element: (
          <ProtectedRoute>
            <DashboardDefault />
          </ProtectedRoute>
        )
      },
      {
        path: 'calendar',
        element: (
          <ProtectedRoute roles={['member', 'manager', 'admin']}>
            <CalendarPage />
          </ProtectedRoute>
        )
      },
      {
        path: 'calendar/admin',
        element: (
          <ProtectedRoute roles={['admin']}>
            <AdminDashboard />
          </ProtectedRoute>
        )
      }
    ]
  },
  {
    path: '/',
    element: <MinimalLayout />,
    children: [
      {
        path: 'calendar/public',
        element: <PublicCalendarPage />
      },
      {
        path: 'login',
        element: (
          <PublicRoute>
            <Login />
          </PublicRoute>
        )
      },
      {
        path: 'register',
        element: (
          <PublicRoute>
            <Register />
          </PublicRoute>
        )
      }
    ]
  }
]);

export default router;
```

## Route Guards

### ProtectedRoute Component

**File**: `src/components/ProtectedRoute.jsx`

```javascript
import { Navigate, useLocation } from 'react-router-dom';
import { useAuth } from 'hooks/useAuth';
import LoadingSpinner from 'components/calendar/shared/LoadingSpinner';

export default function ProtectedRoute({ children, roles = [] }) {
  const { isAuthenticated, profile, loading } = useAuth();
  const location = useLocation();

  if (loading) {
    return <LoadingSpinner fullPage />;
  }

  if (!isAuthenticated) {
    return <Navigate to="/login" state={{ from: location }} replace />;
  }

  if (roles.length > 0 && !roles.includes(profile?.role)) {
    return <Navigate to="/calendar" replace />;
  }

  return children;
}
```

### PublicRoute Component

**File**: `src/components/PublicRoute.jsx`

```javascript
import { Navigate } from 'react-router-dom';
import { useAuth } from 'hooks/useAuth';

export default function PublicRoute({ children }) {
  const { isAuthenticated } = useAuth();

  // Redirect authenticated users to calendar
  if (isAuthenticated) {
    return <Navigate to="/calendar" replace />;
  }

  return children;
}
```

## Navigation Menu Integration

### Update Mantis Menu Items

**File**: `src/menu-items/index.js`

```javascript
import { CalendarOutlined, TeamOutlined, SettingOutlined } from '@ant-design/icons';

const calendar = {
  id: 'calendar',
  title: 'Calendar',
  type: 'group',
  children: [
    {
      id: 'calendar-view',
      title: 'Calendar',
      type: 'item',
      url: '/calendar',
      icon: CalendarOutlined,
      breadcrumbs: true
    },
    {
      id: 'calendar-admin',
      title: 'Administration',
      type: 'item',
      url: '/calendar/admin',
      icon: SettingOutlined,
      breadcrumbs: true,
      roles: ['admin'] // Only show to admins
    }
  ]
};

// Add to menu items
const menuItems = {
  items: [dashboard, calendar, pages, utilities]
};

export default menuItems;
```

### Conditional Menu Items

**File**: `src/layout/MainLayout/Drawer/DrawerContent/Navigation/index.jsx`

```javascript
// Filter menu items based on user role
const filterMenuItems = (items, userRole) => {
  return items.filter(item => {
    if (item.roles && !item.roles.includes(userRole)) {
      return false;
    }
    if (item.children) {
      item.children = filterMenuItems(item.children, userRole);
    }
    return true;
  });
};

// In component
const { profile } = useAuth();
const filteredItems = filterMenuItems(menuItems.items, profile?.role);
```

## Breadcrumbs Configuration

**File**: `src/routes/breadcrumbs.js`

```javascript
export const breadcrumbConfig = {
  '/calendar': {
    title: 'Calendar',
    url: '/calendar'
  },
  '/calendar/admin': {
    title: 'Calendar',
    url: '/calendar',
    children: [
      {
        title: 'Administration',
        url: '/calendar/admin'
      }
    ]
  },
  '/calendar/public': {
    title: 'Public Calendar',
    url: '/calendar/public'
  }
};
```

## SEO and Meta Tags

### Page-specific Meta Tags

```javascript
// In each page component
import { Helmet } from 'react-helmet-async';

// Calendar Page
<Helmet>
  <title>Calendar | NPO Calendar App</title>
  <meta name="description" content="Manage and view organizational events" />
</Helmet>

// Public Calendar
<Helmet>
  <title>Public Events | {NPO_NAME}</title>
  <meta name="description" content="View upcoming public events" />
  <meta property="og:title" content={`Public Events | ${NPO_NAME}`} />
  <meta property="og:type" content="website" />
</Helmet>
```

## Error Pages

### 404 Not Found
**Route**: `*`
**File**: `src/pages/error/404.jsx`

### 403 Forbidden
**Route**: `/forbidden`
**File**: `src/pages/error/403.jsx`

### 500 Server Error
**Route**: `/error`
**File**: `src/pages/error/500.jsx`

## Mobile Navigation

### Bottom Navigation Bar (Mobile)

```javascript
// src/layout/MobileLayout/BottomNav.jsx
import { BottomNavigation, BottomNavigationAction } from '@mui/material';
import {
  CalendarOutlined,
  FilterOutlined,
  PlusOutlined,
  UserOutlined
} from '@ant-design/icons';

export default function BottomNav() {
  const [value, setValue] = useState(0);

  return (
    <BottomNavigation
      value={value}
      onChange={(event, newValue) => setValue(newValue)}
      sx={{
        position: 'fixed',
        bottom: 0,
        left: 0,
        right: 0,
        display: { xs: 'flex', md: 'none' }
      }}
    >
      <BottomNavigationAction label="Calendar" icon={<CalendarOutlined />} />
      <BottomNavigationAction label="Filter" icon={<FilterOutlined />} />
      {canCreateEvent && (
        <BottomNavigationAction label="Create" icon={<PlusOutlined />} />
      )}
      <BottomNavigationAction label="Profile" icon={<UserOutlined />} />
    </BottomNavigation>
  );
}
```

## Summary

✅ **4 main pages**: Calendar, Public Calendar, Admin, Authentication
✅ **Protected routes** with role-based access
✅ **Lazy loading** for performance
✅ **Breadcrumbs** for navigation context
✅ **Mobile-optimized** navigation
✅ **SEO-friendly** meta tags
✅ **Error handling** pages
✅ **Integrated** with Mantis layout system
