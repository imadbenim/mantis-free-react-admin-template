# User Roles and Permissions

## Overview

The NPO Calendar application implements a four-tier role-based access control (RBAC) system. Each role has specific permissions for viewing and managing events, users, and system settings.

## Role Hierarchy

```
Public Viewer (No Auth)
    ↓
Member (Authenticated)
    ↓
Manager (Event Creator)
    ↓
Administrator (Full Access)
```

Higher roles inherit all permissions from lower roles, plus additional capabilities.

## Role Definitions

### 1. Public Viewer

**Authentication**: None required
**Target Users**: General public, website visitors, community members

**Description**:
Anonymous users who can view public events without creating an account. This role enables NPOs to share their public event calendar with the community.

**Use Cases**:
- Community members checking upcoming public events
- Website visitors browsing NPO activities
- People looking for volunteer opportunities
- Public discovering NPO programs

### 2. Member

**Authentication**: Required (email/password)
**Target Users**: NPO volunteers, supporters, team members without event creation needs

**Description**:
Registered users who need to see internal NPO events but don't create events themselves. They have read-only access to public and internal events.

**Use Cases**:
- Volunteers checking internal schedules
- Team members viewing all organizational events
- Supporters staying informed about activities
- Staff who coordinate but don't create events

**Account Features**:
- User profile with name and avatar
- Personalized dashboard
- Saved filter preferences
- Event search history

### 3. Manager

**Authentication**: Required (email/password)
**Target Users**: Program coordinators, team leads, event organizers

**Description**:
Users who can create and manage their own events. They control visibility and categorization of events they create but cannot modify other users' events.

**Use Cases**:
- Program coordinators creating program events
- Team leads scheduling team meetings
- Event organizers planning fundraisers
- Department heads managing departmental calendars

**Event Ownership**:
- Events created by a Manager belong to that Manager
- Only the creator or an Admin can edit/delete the event
- Managers cannot transfer ownership (Admin feature)

### 4. Administrator

**Authentication**: Required (email/password)
**Target Users**: NPO directors, IT staff, system managers

**Description**:
Users with full system access. They can manage all events, configure system settings, manage users and roles, and configure event categories.

**Use Cases**:
- Executive director overseeing all events
- IT staff managing system configuration
- Operations manager coordinating across programs
- NPO director handling special events

**System Management**:
- Full CRUD on all events regardless of creator
- User management and role assignment
- Event category configuration
- System settings and preferences
- Analytics and reporting access

## Permissions Matrix

### Event Permissions

| Permission | Public Viewer | Member | Manager | Administrator |
|-----------|---------------|--------|---------|---------------|
| **View Public Events** | ✅ | ✅ | ✅ | ✅ |
| **View Internal Events** | ❌ | ✅ | ✅ | ✅ |
| **View Private Events** | ❌ | ❌ | Own only | ✅ All |
| **View Event Creator** | ❌ | ✅ | ✅ | ✅ |
| **View Event Metadata** | ❌ | ❌ | ❌ | ✅ |
| **Create Events** | ❌ | ❌ | ✅ | ✅ |
| **Edit Own Events** | ❌ | ❌ | ✅ | ✅ |
| **Edit Others' Events** | ❌ | ❌ | ❌ | ✅ |
| **Delete Own Events** | ❌ | ❌ | ✅ | ✅ |
| **Delete Others' Events** | ❌ | ❌ | ❌ | ✅ |
| **Set Event Visibility** | ❌ | ❌ | ✅ | ✅ |
| **Transfer Event Ownership** | ❌ | ❌ | ❌ | ✅ |

### Calendar View Permissions

| Permission | Public Viewer | Member | Manager | Administrator |
|-----------|---------------|--------|---------|---------------|
| **Access Calendar** | Public only | ✅ | ✅ | ✅ |
| **Month View** | ✅ | ✅ | ✅ | ✅ |
| **Week View** | ✅ | ✅ | ✅ | ✅ |
| **Day View** | ✅ | ✅ | ✅ | ✅ |
| **List View** | ✅ | ✅ | ✅ | ✅ |
| **Filter Events** | Limited | ✅ | ✅ | ✅ |
| **Search Events** | Limited | ✅ | ✅ | ✅ |
| **Save Preferences** | ❌ | ✅ | ✅ | ✅ |

### User Management Permissions

| Permission | Public Viewer | Member | Manager | Administrator |
|-----------|---------------|--------|---------|---------------|
| **View Own Profile** | ❌ | ✅ | ✅ | ✅ |
| **Edit Own Profile** | ❌ | ✅ | ✅ | ✅ |
| **View User List** | ❌ | ❌ | ❌ | ✅ |
| **View User Details** | ❌ | ❌ | ❌ | ✅ |
| **Assign Roles** | ❌ | ❌ | ❌ | ✅ |
| **Deactivate Users** | ❌ | ❌ | ❌ | ✅ |
| **Reset User Passwords** | ❌ | ❌ | ❌ | ✅ |

### Category Management Permissions

| Permission | Public Viewer | Member | Manager | Administrator |
|-----------|---------------|--------|---------|---------------|
| **View Categories** | ✅ | ✅ | ✅ | ✅ |
| **Create Categories** | ❌ | ❌ | ❌ | ✅ |
| **Edit Categories** | ❌ | ❌ | ❌ | ✅ |
| **Delete Categories** | ❌ | ❌ | ❌ | ✅ |
| **Reorder Categories** | ❌ | ❌ | ❌ | ✅ |

### System Permissions

| Permission | Public Viewer | Member | Manager | Administrator |
|-----------|---------------|--------|---------|---------------|
| **Access Dashboard** | ❌ | ✅ | ✅ | ✅ |
| **View Analytics** | ❌ | ❌ | ❌ | ✅ |
| **Export Data** | ❌ | ❌ | ❌ | ✅ |
| **System Settings** | ❌ | ❌ | ❌ | ✅ |
| **View Logs** | ❌ | ❌ | ❌ | ✅ |

## Event Visibility Rules

### Public Events
- **Visible to**: Everyone (including Public Viewers)
- **Purpose**: Community-facing events, public programs, open events
- **Examples**: Fundraising gala, volunteer orientation, public workshops
- **Display**: Shown on public calendar, main calendar

### Internal Events
- **Visible to**: Members, Managers, Administrators
- **Purpose**: NPO-internal events, staff meetings, planning sessions
- **Examples**: Staff meetings, planning sessions, internal deadlines
- **Display**: Shown on authenticated calendar only

### Private Events
- **Visible to**: Event creator and Administrators
- **Purpose**: Personal reminders, draft events, sensitive planning
- **Examples**: Personal reminders, one-on-one meetings, draft proposals
- **Display**: Shown only to creator and admins

## Role Assignment

### Default Assignment
- New registrations → **Member** role
- First user in system → **Administrator** role (auto-assigned)
- Subsequent users → **Member** role (can be upgraded by Admin)

### Role Changes
- **Who can assign**: Administrators only
- **How**: Through admin user management interface
- **Effect**: Immediate (takes effect on next page load/action)
- **Restrictions**:
  - Cannot demote last Administrator
  - Cannot change own role (requires another Admin)

### Role Recommendations

**Assign Member to**:
- General volunteers
- Supporters who need visibility
- Team members without event responsibility
- Newly registered users (default)

**Assign Manager to**:
- Program coordinators
- Team leads
- Department heads
- Event planners
- Anyone who needs to create events

**Assign Administrator to**:
- Executive director
- IT/operations staff
- System managers
- Minimum 2 people (redundancy)
- Trusted, tech-savvy individuals

## Permission Implementation

### Frontend Guards

```javascript
// Route protection
<ProtectedRoute roles={['member', 'manager', 'admin']}>
  <CalendarPage />
</ProtectedRoute>

// Component-level permissions
{hasPermission('event.create') && (
  <Button onClick={createEvent}>Create Event</Button>
)}

// Event action permissions
const canEdit = isEventOwner || isAdmin;
const canDelete = isEventOwner || isAdmin;
```

### Backend Security (Supabase RLS)

All permissions enforced at database level using Row Level Security policies:

```sql
-- Events table: View permissions
CREATE POLICY "Public events visible to all"
ON events FOR SELECT
USING (visibility = 'public');

CREATE POLICY "Internal events visible to authenticated users"
ON events FOR SELECT
USING (
  visibility = 'internal'
  AND auth.role() IN ('member', 'manager', 'admin')
);

CREATE POLICY "Private events visible to creator and admins"
ON events FOR SELECT
USING (
  visibility = 'private'
  AND (created_by = auth.uid() OR auth.role() = 'admin')
);

-- Events table: Create permissions
CREATE POLICY "Managers and admins can create events"
ON events FOR INSERT
WITH CHECK (
  auth.role() IN ('manager', 'admin')
  AND created_by = auth.uid()
);

-- Events table: Update permissions
CREATE POLICY "Users can update own events"
ON events FOR UPDATE
USING (created_by = auth.uid())
WITH CHECK (created_by = auth.uid());

CREATE POLICY "Admins can update any event"
ON events FOR UPDATE
USING (auth.role() = 'admin');

-- Events table: Delete permissions
CREATE POLICY "Users can delete own events"
ON events FOR DELETE
USING (created_by = auth.uid());

CREATE POLICY "Admins can delete any event"
ON events FOR DELETE
USING (auth.role() = 'admin');
```

## User Interface Adaptations

### Public Viewer UI
- Simple calendar view (month view default)
- No navigation menu
- No user profile
- No create/edit buttons
- Minimal header with NPO branding
- Event details in read-only modal

### Member UI
- Full calendar views (month/week/day/list)
- Navigation menu
- User profile access
- Filter and search capabilities
- No create/edit buttons
- Event details with creator info

### Manager UI
- Full calendar views
- Navigation menu
- User profile access
- **"Create Event" button**
- Edit/delete buttons on own events
- Event ownership indicators

### Administrator UI
- Full calendar views
- **Admin navigation menu**
- User profile access
- "Create Event" button
- Edit/delete buttons on all events
- **"Admin Dashboard" link**
- **User management access**
- **Category management access**
- **System settings access**

## Permission Checking Utilities

```javascript
// utils/permissions.js

export const ROLES = {
  PUBLIC: 'public',
  MEMBER: 'member',
  MANAGER: 'manager',
  ADMIN: 'admin'
};

export const PERMISSIONS = {
  // Event permissions
  'event.view.public': ['public', 'member', 'manager', 'admin'],
  'event.view.internal': ['member', 'manager', 'admin'],
  'event.view.private': ['manager', 'admin'], // + ownership check
  'event.create': ['manager', 'admin'],
  'event.edit.own': ['manager', 'admin'],
  'event.edit.any': ['admin'],
  'event.delete.own': ['manager', 'admin'],
  'event.delete.any': ['admin'],

  // User permissions
  'user.view.list': ['admin'],
  'user.edit.own': ['member', 'manager', 'admin'],
  'user.edit.any': ['admin'],
  'user.assign.roles': ['admin'],

  // Category permissions
  'category.view': ['public', 'member', 'manager', 'admin'],
  'category.manage': ['admin'],

  // System permissions
  'system.settings': ['admin'],
  'system.analytics': ['admin'],
};

export function hasPermission(permission, userRole) {
  const allowedRoles = PERMISSIONS[permission];
  return allowedRoles && allowedRoles.includes(userRole);
}

export function canEditEvent(event, user) {
  if (!user) return false;
  if (user.role === 'admin') return true;
  return event.created_by === user.id &&
         ['manager', 'admin'].includes(user.role);
}

export function canDeleteEvent(event, user) {
  return canEditEvent(event, user); // Same logic
}

export function canViewEvent(event, user) {
  if (event.visibility === 'public') return true;
  if (!user) return false;
  if (event.visibility === 'internal') {
    return ['member', 'manager', 'admin'].includes(user.role);
  }
  if (event.visibility === 'private') {
    return event.created_by === user.id || user.role === 'admin';
  }
  return false;
}
```

## Security Considerations

### Prevent Privilege Escalation
- Users cannot change their own role
- Frontend permissions checks are UX only
- All security enforced at database level (RLS)
- API validates user role on every request

### Audit Trail
- Track who created each event
- Track last modified by (future enhancement)
- Log role changes (admin actions)
- Maintain user activity history

### Session Management
- Secure JWT tokens
- 7-day expiration
- Refresh token rotation
- Logout invalidates tokens

### Data Exposure
- Public viewers see minimal event data
- Creator information hidden from public
- Email addresses never exposed in UI
- User list only visible to admins

## Testing Role Permissions

### Test Cases

**Public Viewer Tests**:
- [ ] Can access `/calendar/public`
- [ ] Can view public events
- [ ] Cannot view internal/private events
- [ ] Cannot access main calendar
- [ ] No create/edit/delete buttons visible
- [ ] Redirected from protected routes

**Member Tests**:
- [ ] Can access main calendar
- [ ] Can view public and internal events
- [ ] Cannot view others' private events
- [ ] No create/edit/delete buttons visible
- [ ] Can filter and search
- [ ] Cannot access admin areas

**Manager Tests**:
- [ ] Can create events
- [ ] Can edit own events
- [ ] Can delete own events
- [ ] Cannot edit others' events
- [ ] Cannot delete others' events
- [ ] Cannot access admin areas
- [ ] Can set event visibility

**Administrator Tests**:
- [ ] Can edit any event
- [ ] Can delete any event
- [ ] Can access user management
- [ ] Can assign roles
- [ ] Can manage categories
- [ ] Can access system settings
- [ ] Can view analytics

### Permission Enforcement Tests

- [ ] RLS policies prevent unauthorized data access
- [ ] API endpoints validate user permissions
- [ ] Frontend hides unauthorized actions
- [ ] Direct URL access blocked for protected pages
- [ ] Database queries respect role permissions
- [ ] Token expiration enforced

## FAQs

**Q: Can a Manager see other Managers' events?**
A: Yes, Managers can see all public and internal events, including those created by other Managers. They can only edit/delete their own events.

**Q: Can an Admin create events on behalf of others?**
A: Events are always created with the current user as the creator. Admins can transfer ownership after creation (future feature).

**Q: What happens to events if a Manager is demoted to Member?**
A: Their existing events remain intact and visible. They lose the ability to edit/delete them (unless promoted back or an Admin edits them).

**Q: Can there be zero Administrators?**
A: No, the system prevents demoting the last Administrator to ensure system management capability.

**Q: How does the system handle the first user?**
A: The first registered user is automatically assigned the Administrator role.

**Q: Can a user have multiple roles?**
A: No, each user has exactly one role. Roles are hierarchical, so higher roles include lower permissions.

**Q: How quickly do role changes take effect?**
A: Immediately. The next API call will use the new role. Users may need to refresh their page to see UI changes.
