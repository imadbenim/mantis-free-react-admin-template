# Data Model and Database Schema

## Overview

The calendar application uses Supabase (PostgreSQL) as its database. The data model is designed to support multi-user event management with role-based access control, recurring events, and configurable categories.

## Entity Relationship Diagram

```
┌──────────────┐
│   profiles   │
│              │
│ - id (PK)    │◄──────┐
│ - email      │       │
│ - role       │       │
│ - name       │       │
│ - avatar_url │       │
└──────────────┘       │
                       │ created_by
                       │
                       │
┌──────────────────┐   │
│   categories     │   │
│                  │   │
│ - id (PK)        │   │
│ - name           │   │
│ - color          │   │
│ - icon           │   │
│ - display_order  │   │
└──────────────────┘   │
         │             │
         │ category_id │
         ▼             │
┌──────────────────┐   │
│     events       │   │
│                  │   │
│ - id (PK)        │───┘
│ - title          │
│ - description    │
│ - start_date     │
│ - end_date       │
│ - all_day        │
│ - location       │
│ - visibility     │
│ - category_id    │───┘
│ - created_by     │
│ - recurrence_id  │──┐
│ - color          │  │
└──────────────────┘  │
         │            │
         │            │
         ▼            │
┌──────────────────┐  │
│event_exceptions  │  │
│                  │  │
│ - id (PK)        │  │
│ - event_id       │  │
│ - exception_date │  │
│ - reason         │  │
└──────────────────┘  │
                      │
┌──────────────────┐  │
│   recurrences    │  │
│                  │  │
│ - id (PK)        │◄─┘
│ - type           │
│ - interval       │
│ - end_date       │
│ - days_of_week   │
└──────────────────┘
```

## Database Tables

### 1. profiles

Extends Supabase auth.users with application-specific user data.

```sql
CREATE TABLE profiles (
  id UUID PRIMARY KEY REFERENCES auth.users(id) ON DELETE CASCADE,
  email TEXT NOT NULL UNIQUE,
  role TEXT NOT NULL DEFAULT 'member' CHECK (role IN ('member', 'manager', 'admin')),
  name TEXT NOT NULL,
  avatar_url TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_profiles_role ON profiles(role);
CREATE INDEX idx_profiles_email ON profiles(email);

-- Trigger for updated_at
CREATE OR REPLACE FUNCTION update_updated_at_column()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_profiles_updated_at
  BEFORE UPDATE ON profiles
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at_column();

-- Auto-create profile on user signup
CREATE OR REPLACE FUNCTION public.handle_new_user()
RETURNS TRIGGER AS $$
BEGIN
  INSERT INTO public.profiles (id, email, name, role)
  VALUES (
    NEW.id,
    NEW.email,
    COALESCE(NEW.raw_user_meta_data->>'name', split_part(NEW.email, '@', 1)),
    CASE
      WHEN (SELECT COUNT(*) FROM public.profiles) = 0 THEN 'admin'
      ELSE 'member'
    END
  );
  RETURN NEW;
END;
$$ LANGUAGE plpgsql SECURITY DEFINER;

CREATE TRIGGER on_auth_user_created
  AFTER INSERT ON auth.users
  FOR EACH ROW
  EXECUTE FUNCTION public.handle_new_user();
```

**Fields**:
- `id` (UUID, PK): Links to auth.users, unique identifier
- `email` (TEXT, NOT NULL, UNIQUE): User's email address
- `role` (TEXT, NOT NULL): User role (member/manager/admin)
- `name` (TEXT, NOT NULL): Display name
- `avatar_url` (TEXT, NULLABLE): Profile picture URL
- `created_at` (TIMESTAMPTZ): Account creation timestamp
- `updated_at` (TIMESTAMPTZ): Last update timestamp

**Business Rules**:
- First user auto-assigned 'admin' role
- Subsequent users default to 'member' role
- Email must be unique
- Name is required
- Role changes tracked via updated_at

### 2. categories

Event categories configured by administrators.

```sql
CREATE TABLE categories (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL UNIQUE,
  color TEXT NOT NULL DEFAULT '#808080',
  icon TEXT,
  description TEXT,
  display_order INTEGER NOT NULL DEFAULT 0,
  is_active BOOLEAN NOT NULL DEFAULT true,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_categories_active ON categories(is_active);
CREATE INDEX idx_categories_display_order ON categories(display_order);

-- Trigger for updated_at
CREATE TRIGGER update_categories_updated_at
  BEFORE UPDATE ON categories
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at_column();

-- Insert default categories
INSERT INTO categories (name, color, icon, description, display_order) VALUES
  ('Meeting', '#1677FF', 'TeamOutlined', 'Team meetings and discussions', 1),
  ('Event', '#52C41A', 'CalendarOutlined', 'General events and activities', 2),
  ('Deadline', '#FF4D4F', 'ClockCircleOutlined', 'Important deadlines', 3),
  ('Holiday', '#722ED1', 'SmileOutlined', 'Holidays and time off', 4),
  ('Other', '#8C8C8C', 'AppstoreOutlined', 'Miscellaneous events', 5);
```

**Fields**:
- `id` (UUID, PK): Unique identifier
- `name` (TEXT, NOT NULL, UNIQUE): Category name
- `color` (TEXT, NOT NULL): Hex color code
- `icon` (TEXT, NULLABLE): Ant Design icon name
- `description` (TEXT, NULLABLE): Category description
- `display_order` (INTEGER): Sort order for display
- `is_active` (BOOLEAN): Active status
- `created_at` (TIMESTAMPTZ): Creation timestamp
- `updated_at` (TIMESTAMPTZ): Last update timestamp

**Business Rules**:
- Category names must be unique
- Default color if not specified: #808080 (grey)
- Inactive categories hidden but events retain reference
- Cannot delete category with associated events (must reassign)

### 3. recurrences

Recurrence rules for repeating events.

```sql
CREATE TABLE recurrences (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  type TEXT NOT NULL CHECK (type IN ('daily', 'weekly', 'monthly')),
  interval INTEGER NOT NULL DEFAULT 1 CHECK (interval > 0),
  end_date DATE,
  max_occurrences INTEGER CHECK (max_occurrences > 0 AND max_occurrences <= 365),
  days_of_week INTEGER[], -- 0=Sunday, 1=Monday, ..., 6=Saturday
  day_of_month INTEGER CHECK (day_of_month >= 1 AND day_of_month <= 31),
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_recurrences_type ON recurrences(type);
```

**Fields**:
- `id` (UUID, PK): Unique identifier
- `type` (TEXT, NOT NULL): Recurrence type (daily/weekly/monthly)
- `interval` (INTEGER, NOT NULL): Every N days/weeks/months
- `end_date` (DATE, NULLABLE): Last occurrence date
- `max_occurrences` (INTEGER, NULLABLE): Maximum number of occurrences
- `days_of_week` (INTEGER[], NULLABLE): Days for weekly recurrence
- `day_of_month` (INTEGER, NULLABLE): Day for monthly recurrence
- `created_at` (TIMESTAMPTZ): Creation timestamp

**Business Rules**:
- Interval must be > 0
- Maximum 365 occurrences
- `days_of_week` used for weekly recurrence (array of 0-6)
- `day_of_month` used for monthly recurrence (1-31)
- Either `end_date` or `max_occurrences` should be set (not both)

### 4. events

Core event data.

```sql
CREATE TABLE events (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title TEXT NOT NULL CHECK (char_length(title) <= 200),
  description TEXT CHECK (char_length(description) <= 5000),
  start_date TIMESTAMPTZ NOT NULL,
  end_date TIMESTAMPTZ NOT NULL CHECK (end_date > start_date),
  all_day BOOLEAN NOT NULL DEFAULT false,
  location TEXT CHECK (char_length(location) <= 500),
  visibility TEXT NOT NULL DEFAULT 'internal' CHECK (visibility IN ('public', 'internal', 'private')),
  color TEXT,
  category_id UUID REFERENCES categories(id) ON DELETE SET NULL,
  created_by UUID NOT NULL REFERENCES profiles(id) ON DELETE CASCADE,
  recurrence_id UUID REFERENCES recurrences(id) ON DELETE CASCADE,
  original_start_date TIMESTAMPTZ, -- For recurring event instances
  is_recurring_exception BOOLEAN NOT NULL DEFAULT false,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Indexes
CREATE INDEX idx_events_start_date ON events(start_date);
CREATE INDEX idx_events_end_date ON events(end_date);
CREATE INDEX idx_events_visibility ON events(visibility);
CREATE INDEX idx_events_category ON events(category_id);
CREATE INDEX idx_events_created_by ON events(created_by);
CREATE INDEX idx_events_recurrence ON events(recurrence_id);
CREATE INDEX idx_events_date_range ON events USING GIST (
  tstzrange(start_date, end_date)
);

-- Trigger for updated_at
CREATE TRIGGER update_events_updated_at
  BEFORE UPDATE ON events
  FOR EACH ROW
  EXECUTE FUNCTION update_updated_at_column();
```

**Fields**:
- `id` (UUID, PK): Unique identifier
- `title` (TEXT, NOT NULL): Event title (max 200 chars)
- `description` (TEXT, NULLABLE): Event description (max 5000 chars)
- `start_date` (TIMESTAMPTZ, NOT NULL): Event start date/time
- `end_date` (TIMESTAMPTZ, NOT NULL): Event end date/time
- `all_day` (BOOLEAN, NOT NULL): All-day event flag
- `location` (TEXT, NULLABLE): Event location (max 500 chars)
- `visibility` (TEXT, NOT NULL): Visibility level (public/internal/private)
- `color` (TEXT, NULLABLE): Custom color override
- `category_id` (UUID, FK): Reference to categories
- `created_by` (UUID, NOT NULL, FK): Reference to profiles
- `recurrence_id` (UUID, FK): Reference to recurrences
- `original_start_date` (TIMESTAMPTZ): Original date for recurring instances
- `is_recurring_exception` (BOOLEAN): Marks exception to recurring series
- `created_at` (TIMESTAMPTZ): Creation timestamp
- `updated_at` (TIMESTAMPTZ): Last update timestamp

**Business Rules**:
- `end_date` must be after `start_date`
- Title limited to 200 characters
- Description limited to 5000 characters
- Location limited to 500 characters
- Default visibility: 'internal'
- `original_start_date` used for tracking edited recurring instances
- `is_recurring_exception` marks instances that differ from recurrence rule

### 5. event_exceptions

Tracks dates when recurring events should not occur.

```sql
CREATE TABLE event_exceptions (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  event_id UUID NOT NULL REFERENCES events(id) ON DELETE CASCADE,
  exception_date DATE NOT NULL,
  reason TEXT,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  UNIQUE(event_id, exception_date)
);

-- Indexes
CREATE INDEX idx_event_exceptions_event ON event_exceptions(event_id);
CREATE INDEX idx_event_exceptions_date ON event_exceptions(exception_date);
```

**Fields**:
- `id` (UUID, PK): Unique identifier
- `event_id` (UUID, NOT NULL, FK): Reference to events
- `exception_date` (DATE, NOT NULL): Date to exclude
- `reason` (TEXT, NULLABLE): Reason for exception
- `created_at` (TIMESTAMPTZ): Creation timestamp

**Business Rules**:
- Each event can have multiple exceptions
- Exception date must be unique per event
- When an instance is deleted from recurring series, exception is created
- Exceptions prevent instance generation for that date

## Views

### upcoming_events

Materialized view for frequently accessed upcoming events.

```sql
CREATE MATERIALIZED VIEW upcoming_events AS
SELECT
  e.id,
  e.title,
  e.start_date,
  e.end_date,
  e.all_day,
  e.location,
  e.visibility,
  e.color,
  c.name as category_name,
  c.color as category_color,
  p.name as creator_name,
  p.avatar_url as creator_avatar
FROM events e
LEFT JOIN categories c ON e.category_id = c.id
LEFT JOIN profiles p ON e.created_by = p.id
WHERE e.start_date >= NOW()
ORDER BY e.start_date
LIMIT 100;

-- Refresh periodically
CREATE INDEX idx_upcoming_events_start ON upcoming_events(start_date);
```

## Functions

### get_events_for_date_range

Fetch events within a date range, including recurring instances.

```sql
CREATE OR REPLACE FUNCTION get_events_for_date_range(
  start_range TIMESTAMPTZ,
  end_range TIMESTAMPTZ,
  user_id UUID DEFAULT NULL
)
RETURNS TABLE (
  id UUID,
  title TEXT,
  description TEXT,
  start_date TIMESTAMPTZ,
  end_date TIMESTAMPTZ,
  all_day BOOLEAN,
  location TEXT,
  visibility TEXT,
  color TEXT,
  category_id UUID,
  created_by UUID,
  is_recurring_instance BOOLEAN,
  original_event_id UUID
) AS $$
BEGIN
  RETURN QUERY
  -- Regular events (non-recurring)
  SELECT
    e.id,
    e.title,
    e.description,
    e.start_date,
    e.end_date,
    e.all_day,
    e.location,
    e.visibility,
    COALESCE(e.color, c.color) as color,
    e.category_id,
    e.created_by,
    false as is_recurring_instance,
    NULL::UUID as original_event_id
  FROM events e
  LEFT JOIN categories c ON e.category_id = c.id
  WHERE e.recurrence_id IS NULL
    AND e.start_date >= start_range
    AND e.start_date < end_range
    AND (
      e.visibility = 'public'
      OR (user_id IS NOT NULL AND e.visibility = 'internal')
      OR (user_id IS NOT NULL AND e.visibility = 'private' AND e.created_by = user_id)
      OR (user_id IS NOT NULL AND EXISTS (
        SELECT 1 FROM profiles WHERE id = user_id AND role = 'admin'
      ))
    )

  UNION ALL

  -- Recurring event instances
  SELECT
    gen_random_uuid() as id,
    e.title,
    e.description,
    generate_series_start as start_date,
    generate_series_start + (e.end_date - e.start_date) as end_date,
    e.all_day,
    e.location,
    e.visibility,
    COALESCE(e.color, c.color) as color,
    e.category_id,
    e.created_by,
    true as is_recurring_instance,
    e.id as original_event_id
  FROM events e
  LEFT JOIN categories c ON e.category_id = c.id
  CROSS JOIN LATERAL (
    SELECT generate_series(
      e.start_date,
      LEAST(COALESCE(r.end_date::TIMESTAMPTZ, end_range), end_range),
      CASE r.type
        WHEN 'daily' THEN (r.interval || ' days')::INTERVAL
        WHEN 'weekly' THEN (r.interval || ' weeks')::INTERVAL
        WHEN 'monthly' THEN (r.interval || ' months')::INTERVAL
      END
    ) as generate_series_start
    FROM recurrences r
    WHERE r.id = e.recurrence_id
  ) series
  WHERE e.recurrence_id IS NOT NULL
    AND series.generate_series_start >= start_range
    AND series.generate_series_start < end_range
    AND NOT EXISTS (
      SELECT 1 FROM event_exceptions ex
      WHERE ex.event_id = e.id
        AND ex.exception_date = series.generate_series_start::DATE
    )
    AND (
      e.visibility = 'public'
      OR (user_id IS NOT NULL AND e.visibility = 'internal')
      OR (user_id IS NOT NULL AND e.visibility = 'private' AND e.created_by = user_id)
      OR (user_id IS NOT NULL AND EXISTS (
        SELECT 1 FROM profiles WHERE id = user_id AND role = 'admin'
      ))
    );
END;
$$ LANGUAGE plpgsql STABLE;
```

### can_user_edit_event

Check if a user can edit an event.

```sql
CREATE OR REPLACE FUNCTION can_user_edit_event(
  event_id UUID,
  user_id UUID
)
RETURNS BOOLEAN AS $$
DECLARE
  user_role TEXT;
  event_creator UUID;
BEGIN
  SELECT role INTO user_role FROM profiles WHERE id = user_id;
  SELECT created_by INTO event_creator FROM events WHERE id = event_id;

  RETURN user_role = 'admin' OR event_creator = user_id;
END;
$$ LANGUAGE plpgsql STABLE SECURITY DEFINER;
```

## Data Access Patterns

### Query Patterns

**1. Get events for month view**
```sql
SELECT * FROM get_events_for_date_range(
  '2025-01-01 00:00:00'::TIMESTAMPTZ,
  '2025-02-01 00:00:00'::TIMESTAMPTZ,
  auth.uid()
);
```

**2. Get user's events**
```sql
SELECT * FROM events
WHERE created_by = auth.uid()
ORDER BY start_date DESC;
```

**3. Search events**
```sql
SELECT e.*, c.name as category_name
FROM events e
LEFT JOIN categories c ON e.category_id = c.id
WHERE (
  e.title ILIKE '%search%'
  OR e.description ILIKE '%search%'
)
AND visibility IN ('public', 'internal') -- based on user role
ORDER BY start_date;
```

**4. Get events by category**
```sql
SELECT * FROM events
WHERE category_id = 'category-uuid'
  AND start_date >= NOW()
ORDER BY start_date;
```

### Performance Optimization

**Indexes**:
- Date range queries: GiST index on tstzrange
- Category filtering: B-tree index on category_id
- User events: B-tree index on created_by
- Visibility filtering: B-tree index on visibility

**Caching Strategy**:
- Cache category list (rarely changes)
- Cache user profile (for session)
- Invalidate event cache on mutations
- Use SWR (stale-while-revalidate) for calendar views

**Pagination**:
```sql
-- List view pagination
SELECT * FROM events
WHERE start_date >= '2025-01-01'
ORDER BY start_date
LIMIT 20 OFFSET 0;
```

## Data Validation

### Application-Level Validation

```typescript
// Event validation schema (Zod)
const eventSchema = z.object({
  title: z.string().min(1).max(200),
  description: z.string().max(5000).optional(),
  start_date: z.date(),
  end_date: z.date(),
  all_day: z.boolean().default(false),
  location: z.string().max(500).optional(),
  visibility: z.enum(['public', 'internal', 'private']).default('internal'),
  category_id: z.string().uuid().optional(),
  color: z.string().regex(/^#[0-9A-F]{6}$/i).optional(),
  recurrence: z.object({
    type: z.enum(['daily', 'weekly', 'monthly']),
    interval: z.number().min(1).max(365),
    end_date: z.date().optional(),
    days_of_week: z.array(z.number().min(0).max(6)).optional(),
  }).optional(),
}).refine(
  (data) => data.end_date > data.start_date,
  { message: "End date must be after start date" }
);
```

### Database Constraints

- Check constraints on enums (role, visibility, recurrence type)
- Foreign key constraints for referential integrity
- Unique constraints (email, category name)
- NOT NULL constraints on required fields
- Check constraints on date logic (end > start)
- Check constraints on string lengths

## Migration Strategy

### Initial Setup

```sql
-- 001_create_profiles.sql
-- 002_create_categories.sql
-- 003_create_recurrences.sql
-- 004_create_events.sql
-- 005_create_event_exceptions.sql
-- 006_create_functions.sql
-- 007_create_views.sql
-- 008_insert_default_data.sql
```

### Seeding Development Data

```sql
-- Seed users (profiles created via trigger)
-- Seed categories (default 5 categories)
-- Seed sample events

INSERT INTO events (title, start_date, end_date, visibility, category_id, created_by)
VALUES
  ('Team Meeting', '2025-01-15 10:00:00', '2025-01-15 11:00:00', 'internal',
   (SELECT id FROM categories WHERE name = 'Meeting'),
   (SELECT id FROM profiles WHERE role = 'admin' LIMIT 1)),
  ('Fundraising Gala', '2025-02-01 18:00:00', '2025-02-01 23:00:00', 'public',
   (SELECT id FROM categories WHERE name = 'Event'),
   (SELECT id FROM profiles WHERE role = 'admin' LIMIT 1));
```

## Backup and Recovery

### Supabase Automatic Backups
- Daily automated backups (retained per Supabase plan)
- Point-in-time recovery available
- Manual backup via pg_dump

### Critical Data
- Events (events table)
- User profiles (profiles table)
- Custom categories (categories table)
- Recurrence rules (recurrences table)

### Export Capability
```sql
-- Export all events to JSON (admin only)
COPY (
  SELECT row_to_json(e)
  FROM events e
) TO '/tmp/events_backup.json';
```

## Data Retention

### Active Data
- All events (no automatic deletion)
- User profiles (until account deletion)
- Categories (soft delete via is_active)

### Soft Deletes
- Events: 7-day retention in deleted_events table (future)
- Categories: Marked inactive, not deleted
- Users: Deactivated, not deleted (admin action)

### Archival (Future)
- Events older than 2 years
- Inactive user accounts (1 year)
- Audit logs (5 years)

## Summary

The data model supports:
✅ Multi-user event management
✅ Role-based access control (4 tiers)
✅ Recurring events with exceptions
✅ Configurable event categories
✅ Efficient date range queries
✅ Public and internal calendars
✅ Mobile-optimized queries
✅ Scalable for 1000+ events
✅ Supabase RLS integration
✅ Real-time subscriptions ready
