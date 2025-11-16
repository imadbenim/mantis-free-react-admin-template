# API Design and Supabase Integration

## Overview

This document details the API design for the NPO Calendar application using Supabase as the backend. It covers client configuration, Row Level Security (RLS) policies, database functions, and API patterns.

## Supabase Project Setup

### 1. Create Supabase Project

1. Go to [https://supabase.com](https://supabase.com)
2. Click "New Project"
3. Configure project:
   - **Name**: `npo-calendar` (or your organization name)
   - **Database Password**: Generate strong password (save securely)
   - **Region**: Select closest to your users
   - **Pricing Plan**: Free tier (upgrade as needed)

### 2. Environment Variables

Create `.env.local` file in project root:

```bash
# Supabase Configuration
VITE_SUPABASE_URL=https://your-project.supabase.co
VITE_SUPABASE_ANON_KEY=your-anon-key-here

# Optional: Enable Supabase logging in development
VITE_SUPABASE_DEBUG=true
```

**Security Notes**:
- Never commit `.env.local` to version control
- Anon key is safe for client-side use (protected by RLS)
- Service role key should NEVER be exposed to client

### 3. Supabase Client Setup

**File**: `src/services/supabase/client.js`

```javascript
import { createClient } from '@supabase/supabase-js';

const supabaseUrl = import.meta.env.VITE_SUPABASE_URL;
const supabaseAnonKey = import.meta.env.VITE_SUPABASE_ANON_KEY;

if (!supabaseUrl || !supabaseAnonKey) {
  throw new Error('Missing Supabase environment variables');
}

export const supabase = createClient(supabaseUrl, supabaseAnonKey, {
  auth: {
    autoRefreshToken: true,
    persistSession: true,
    detectSessionInUrl: true,
    storage: window.localStorage,
  },
  db: {
    schema: 'public',
  },
  global: {
    headers: {
      'x-application-name': 'npo-calendar',
    },
  },
  realtime: {
    enabled: true,
  },
});

// Helper to get current user
export const getCurrentUser = async () => {
  const { data: { user }, error } = await supabase.auth.getUser();
  if (error) throw error;
  return user;
};

// Helper to get current user profile
export const getCurrentProfile = async () => {
  const user = await getCurrentUser();
  if (!user) return null;

  const { data, error } = await supabase
    .from('profiles')
    .select('*')
    .eq('id', user.id)
    .single();

  if (error) throw error;
  return data;
};
```

## Row Level Security (RLS) Policies

### Enable RLS on All Tables

```sql
-- Enable RLS on all tables
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE categories ENABLE ROW LEVEL SECURITY;
ALTER TABLE recurrences ENABLE ROW LEVEL SECURITY;
ALTER TABLE events ENABLE ROW LEVEL SECURITY;
ALTER TABLE event_exceptions ENABLE ROW LEVEL SECURITY;
```

### Profiles Table Policies

```sql
-- Users can view all profiles (for creator info)
CREATE POLICY "Profiles are viewable by authenticated users"
ON profiles FOR SELECT
TO authenticated
USING (true);

-- Users can update their own profile
CREATE POLICY "Users can update own profile"
ON profiles FOR UPDATE
TO authenticated
USING (auth.uid() = id)
WITH CHECK (auth.uid() = id);

-- Only admins can view all profile details
CREATE POLICY "Admins can view all profiles"
ON profiles FOR SELECT
TO authenticated
USING (
  EXISTS (
    SELECT 1 FROM profiles
    WHERE id = auth.uid() AND role = 'admin'
  )
);

-- Only admins can update user roles
CREATE POLICY "Admins can update any profile"
ON profiles FOR UPDATE
TO authenticated
USING (
  EXISTS (
    SELECT 1 FROM profiles
    WHERE id = auth.uid() AND role = 'admin'
  )
);

-- Prevent users from changing own role
CREATE POLICY "Users cannot change own role"
ON profiles FOR UPDATE
TO authenticated
USING (auth.uid() = id)
WITH CHECK (
  role = (SELECT role FROM profiles WHERE id = auth.uid())
);
```

### Categories Table Policies

```sql
-- Everyone can view active categories
CREATE POLICY "Active categories are viewable by all"
ON categories FOR SELECT
TO anon, authenticated
USING (is_active = true);

-- Admins can view all categories
CREATE POLICY "Admins can view all categories"
ON categories FOR SELECT
TO authenticated
USING (
  EXISTS (
    SELECT 1 FROM profiles
    WHERE id = auth.uid() AND role = 'admin'
  )
);

-- Only admins can insert categories
CREATE POLICY "Admins can insert categories"
ON categories FOR INSERT
TO authenticated
WITH CHECK (
  EXISTS (
    SELECT 1 FROM profiles
    WHERE id = auth.uid() AND role = 'admin'
  )
);

-- Only admins can update categories
CREATE POLICY "Admins can update categories"
ON categories FOR UPDATE
TO authenticated
USING (
  EXISTS (
    SELECT 1 FROM profiles
    WHERE id = auth.uid() AND role = 'admin'
  )
);

-- Only admins can delete categories
CREATE POLICY "Admins can delete categories"
ON categories FOR DELETE
TO authenticated
USING (
  EXISTS (
    SELECT 1 FROM profiles
    WHERE id = auth.uid() AND role = 'admin'
  )
);
```

### Events Table Policies

```sql
-- Public events are viewable by everyone
CREATE POLICY "Public events are viewable by all"
ON events FOR SELECT
TO anon, authenticated
USING (visibility = 'public');

-- Internal events are viewable by authenticated users
CREATE POLICY "Internal events are viewable by authenticated users"
ON events FOR SELECT
TO authenticated
USING (
  visibility = 'internal'
  OR (
    visibility = 'private' AND created_by = auth.uid()
  )
);

-- Admins can view all events
CREATE POLICY "Admins can view all events"
ON events FOR SELECT
TO authenticated
USING (
  EXISTS (
    SELECT 1 FROM profiles
    WHERE id = auth.uid() AND role = 'admin'
  )
);

-- Managers and admins can create events
CREATE POLICY "Managers and admins can create events"
ON events FOR INSERT
TO authenticated
WITH CHECK (
  EXISTS (
    SELECT 1 FROM profiles
    WHERE id = auth.uid()
      AND role IN ('manager', 'admin')
  )
  AND created_by = auth.uid()
);

-- Users can update their own events
CREATE POLICY "Users can update own events"
ON events FOR UPDATE
TO authenticated
USING (created_by = auth.uid())
WITH CHECK (created_by = auth.uid());

-- Admins can update any event
CREATE POLICY "Admins can update any event"
ON events FOR UPDATE
TO authenticated
USING (
  EXISTS (
    SELECT 1 FROM profiles
    WHERE id = auth.uid() AND role = 'admin'
  )
);

-- Users can delete their own events
CREATE POLICY "Users can delete own events"
ON events FOR DELETE
TO authenticated
USING (created_by = auth.uid());

-- Admins can delete any event
CREATE POLICY "Admins can delete any event"
ON events FOR DELETE
TO authenticated
USING (
  EXISTS (
    SELECT 1 FROM profiles
    WHERE id = auth.uid() AND role = 'admin'
  )
);
```

### Recurrences Table Policies

```sql
-- Users can view recurrences for events they can see
CREATE POLICY "Users can view recurrences for visible events"
ON recurrences FOR SELECT
TO authenticated
USING (
  EXISTS (
    SELECT 1 FROM events
    WHERE events.recurrence_id = recurrences.id
      AND (
        events.visibility IN ('public', 'internal')
        OR events.created_by = auth.uid()
        OR EXISTS (
          SELECT 1 FROM profiles
          WHERE id = auth.uid() AND role = 'admin'
        )
      )
  )
);

-- Managers and admins can create recurrences
CREATE POLICY "Managers and admins can create recurrences"
ON recurrences FOR INSERT
TO authenticated
WITH CHECK (
  EXISTS (
    SELECT 1 FROM profiles
    WHERE id = auth.uid() AND role IN ('manager', 'admin')
  )
);

-- Users can update recurrences for their own events
CREATE POLICY "Users can update own recurrences"
ON recurrences FOR UPDATE
TO authenticated
USING (
  EXISTS (
    SELECT 1 FROM events
    WHERE events.recurrence_id = recurrences.id
      AND events.created_by = auth.uid()
  )
);

-- Admins can update any recurrence
CREATE POLICY "Admins can update any recurrence"
ON recurrences FOR UPDATE
TO authenticated
USING (
  EXISTS (
    SELECT 1 FROM profiles
    WHERE id = auth.uid() AND role = 'admin'
  )
);

-- Similar policies for DELETE
CREATE POLICY "Users can delete own recurrences"
ON recurrences FOR DELETE
TO authenticated
USING (
  EXISTS (
    SELECT 1 FROM events
    WHERE events.recurrence_id = recurrences.id
      AND events.created_by = auth.uid()
  )
);

CREATE POLICY "Admins can delete any recurrence"
ON recurrences FOR DELETE
TO authenticated
USING (
  EXISTS (
    SELECT 1 FROM profiles
    WHERE id = auth.uid() AND role = 'admin'
  )
);
```

### Event Exceptions Table Policies

```sql
-- Users can view exceptions for events they can see
CREATE POLICY "Users can view exceptions for visible events"
ON event_exceptions FOR SELECT
TO authenticated
USING (
  EXISTS (
    SELECT 1 FROM events
    WHERE events.id = event_exceptions.event_id
      AND (
        events.visibility IN ('public', 'internal')
        OR events.created_by = auth.uid()
        OR EXISTS (
          SELECT 1 FROM profiles
          WHERE id = auth.uid() AND role = 'admin'
        )
      )
  )
);

-- Users can create exceptions for their own events
CREATE POLICY "Users can create exceptions for own events"
ON event_exceptions FOR INSERT
TO authenticated
WITH CHECK (
  EXISTS (
    SELECT 1 FROM events
    WHERE events.id = event_exceptions.event_id
      AND events.created_by = auth.uid()
  )
);

-- Admins can create exceptions for any event
CREATE POLICY "Admins can create exceptions for any event"
ON event_exceptions FOR INSERT
TO authenticated
WITH CHECK (
  EXISTS (
    SELECT 1 FROM profiles
    WHERE id = auth.uid() AND role = 'admin'
  )
);

-- Similar policies for UPDATE and DELETE
```

## API Service Layer

### Authentication Service

**File**: `src/services/supabase/auth.js`

```javascript
import { supabase } from './client';

export const authService = {
  // Sign up
  async signUp(email, password, name) {
    const { data, error } = await supabase.auth.signUp({
      email,
      password,
      options: {
        data: {
          name,
        },
      },
    });

    if (error) throw error;
    return data;
  },

  // Sign in
  async signIn(email, password) {
    const { data, error } = await supabase.auth.signInWithPassword({
      email,
      password,
    });

    if (error) throw error;
    return data;
  },

  // Sign out
  async signOut() {
    const { error } = await supabase.auth.signOut();
    if (error) throw error;
  },

  // Get current session
  async getSession() {
    const { data: { session }, error } = await supabase.auth.getSession();
    if (error) throw error;
    return session;
  },

  // Reset password
  async resetPassword(email) {
    const { data, error } = await supabase.auth.resetPasswordForEmail(email, {
      redirectTo: `${window.location.origin}/reset-password`,
    });

    if (error) throw error;
    return data;
  },

  // Update password
  async updatePassword(newPassword) {
    const { data, error } = await supabase.auth.updateUser({
      password: newPassword,
    });

    if (error) throw error;
    return data;
  },

  // Subscribe to auth changes
  onAuthStateChange(callback) {
    return supabase.auth.onAuthStateChange(callback);
  },
};
```

### Events Service

**File**: `src/services/supabase/events.js`

```javascript
import { supabase } from './client';

export const eventsService = {
  // Get events for date range
  async getEventsForDateRange(startDate, endDate) {
    const { data, error } = await supabase.rpc('get_events_for_date_range', {
      start_range: startDate.toISOString(),
      end_range: endDate.toISOString(),
      user_id: (await supabase.auth.getUser()).data.user?.id || null,
    });

    if (error) throw error;
    return data;
  },

  // Get single event
  async getEvent(id) {
    const { data, error } = await supabase
      .from('events')
      .select(`
        *,
        category:categories(*),
        creator:profiles!created_by(*),
        recurrence:recurrences(*)
      `)
      .eq('id', id)
      .single();

    if (error) throw error;
    return data;
  },

  // Create event
  async createEvent(event) {
    let recurrenceId = null;

    // Create recurrence rule if needed
    if (event.recurrence) {
      const { data: recurrence, error: recError } = await supabase
        .from('recurrences')
        .insert(event.recurrence)
        .select()
        .single();

      if (recError) throw recError;
      recurrenceId = recurrence.id;
    }

    // Create event
    const { data, error } = await supabase
      .from('events')
      .insert({
        ...event,
        recurrence_id: recurrenceId,
        created_by: (await supabase.auth.getUser()).data.user.id,
      })
      .select()
      .single();

    if (error) throw error;
    return data;
  },

  // Update event
  async updateEvent(id, updates) {
    const { data, error } = await supabase
      .from('events')
      .update(updates)
      .eq('id', id)
      .select()
      .single();

    if (error) throw error;
    return data;
  },

  // Delete event
  async deleteEvent(id) {
    const { error } = await supabase
      .from('events')
      .delete()
      .eq('id', id);

    if (error) throw error;
  },

  // Delete single instance of recurring event
  async deleteRecurringInstance(eventId, date) {
    const { data, error } = await supabase
      .from('event_exceptions')
      .insert({
        event_id: eventId,
        exception_date: date,
        reason: 'Deleted instance',
      });

    if (error) throw error;
    return data;
  },

  // Search events
  async searchEvents(query, filters = {}) {
    let queryBuilder = supabase
      .from('events')
      .select(`
        *,
        category:categories(*),
        creator:profiles!created_by(*)
      `)
      .or(`title.ilike.%${query}%,description.ilike.%${query}%`);

    // Apply filters
    if (filters.categoryId) {
      queryBuilder = queryBuilder.eq('category_id', filters.categoryId);
    }

    if (filters.visibility) {
      queryBuilder = queryBuilder.in('visibility', filters.visibility);
    }

    if (filters.createdBy) {
      queryBuilder = queryBuilder.eq('created_by', filters.createdBy);
    }

    if (filters.startDate) {
      queryBuilder = queryBuilder.gte('start_date', filters.startDate);
    }

    if (filters.endDate) {
      queryBuilder = queryBuilder.lte('end_date', filters.endDate);
    }

    queryBuilder = queryBuilder.order('start_date', { ascending: true });

    const { data, error } = await queryBuilder;

    if (error) throw error;
    return data;
  },

  // Get user's events
  async getMyEvents() {
    const user = (await supabase.auth.getUser()).data.user;

    const { data, error } = await supabase
      .from('events')
      .select(`
        *,
        category:categories(*),
        recurrence:recurrences(*)
      `)
      .eq('created_by', user.id)
      .order('start_date', { ascending: false });

    if (error) throw error;
    return data;
  },

  // Subscribe to event changes (real-time)
  subscribeToEvents(callback) {
    return supabase
      .channel('events_channel')
      .on(
        'postgres_changes',
        { event: '*', schema: 'public', table: 'events' },
        callback
      )
      .subscribe();
  },
};
```

### Categories Service

**File**: `src/services/supabase/categories.js`

```javascript
import { supabase } from './client';

export const categoriesService = {
  // Get all active categories
  async getCategories() {
    const { data, error } = await supabase
      .from('categories')
      .select('*')
      .eq('is_active', true)
      .order('display_order', { ascending: true });

    if (error) throw error;
    return data;
  },

  // Get all categories (admin only)
  async getAllCategories() {
    const { data, error } = await supabase
      .from('categories')
      .select('*')
      .order('display_order', { ascending: true });

    if (error) throw error;
    return data;
  },

  // Create category (admin only)
  async createCategory(category) {
    const { data, error } = await supabase
      .from('categories')
      .insert(category)
      .select()
      .single();

    if (error) throw error;
    return data;
  },

  // Update category (admin only)
  async updateCategory(id, updates) {
    const { data, error } = await supabase
      .from('categories')
      .update(updates)
      .eq('id', id)
      .select()
      .single();

    if (error) throw error;
    return data;
  },

  // Delete category (admin only)
  async deleteCategory(id) {
    // Soft delete by marking inactive
    const { data, error } = await supabase
      .from('categories')
      .update({ is_active: false })
      .eq('id', id)
      .select()
      .single();

    if (error) throw error;
    return data;
  },

  // Reorder categories (admin only)
  async reorderCategories(categoryIds) {
    const updates = categoryIds.map((id, index) => ({
      id,
      display_order: index,
    }));

    const { data, error } = await supabase
      .from('categories')
      .upsert(updates);

    if (error) throw error;
    return data;
  },
};
```

### Users Service

**File**: `src/services/supabase/users.js`

```javascript
import { supabase } from './client';

export const usersService = {
  // Get current user profile
  async getCurrentProfile() {
    const user = (await supabase.auth.getUser()).data.user;
    if (!user) return null;

    const { data, error } = await supabase
      .from('profiles')
      .select('*')
      .eq('id', user.id)
      .single();

    if (error) throw error;
    return data;
  },

  // Update own profile
  async updateProfile(updates) {
    const user = (await supabase.auth.getUser()).data.user;

    const { data, error } = await supabase
      .from('profiles')
      .update(updates)
      .eq('id', user.id)
      .select()
      .single();

    if (error) throw error;
    return data;
  },

  // Get all users (admin only)
  async getAllUsers() {
    const { data, error } = await supabase
      .from('profiles')
      .select('*')
      .order('created_at', { ascending: false });

    if (error) throw error;
    return data;
  },

  // Update user role (admin only)
  async updateUserRole(userId, role) {
    const { data, error } = await supabase
      .from('profiles')
      .update({ role })
      .eq('id', userId)
      .select()
      .single();

    if (error) throw error;
    return data;
  },

  // Upload avatar
  async uploadAvatar(file) {
    const user = (await supabase.auth.getUser()).data.user;
    const fileExt = file.name.split('.').pop();
    const fileName = `${user.id}-${Date.now()}.${fileExt}`;
    const filePath = `avatars/${fileName}`;

    // Upload file
    const { error: uploadError } = await supabase.storage
      .from('avatars')
      .upload(filePath, file);

    if (uploadError) throw uploadError;

    // Get public URL
    const { data: { publicUrl } } = supabase.storage
      .from('avatars')
      .getPublicUrl(filePath);

    // Update profile
    const { data, error } = await supabase
      .from('profiles')
      .update({ avatar_url: publicUrl })
      .eq('id', user.id)
      .select()
      .single();

    if (error) throw error;
    return data;
  },
};
```

## React Hooks for Data Fetching

### useAuth Hook

**File**: `src/hooks/useAuth.js`

```javascript
import { useState, useEffect } from 'react';
import { authService } from '../services/supabase/auth';
import { usersService } from '../services/supabase/users';

export function useAuth() {
  const [user, setUser] = useState(null);
  const [profile, setProfile] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    // Get initial session
    authService.getSession().then(session => {
      setUser(session?.user ?? null);
      if (session?.user) {
        loadProfile();
      } else {
        setLoading(false);
      }
    });

    // Subscribe to auth changes
    const { data: { subscription } } = authService.onAuthStateChange(
      async (event, session) => {
        setUser(session?.user ?? null);
        if (session?.user) {
          await loadProfile();
        } else {
          setProfile(null);
        }
        setLoading(false);
      }
    );

    return () => {
      subscription?.unsubscribe();
    };
  }, []);

  async function loadProfile() {
    try {
      const profileData = await usersService.getCurrentProfile();
      setProfile(profileData);
    } catch (error) {
      console.error('Error loading profile:', error);
    } finally {
      setLoading(false);
    }
  }

  return {
    user,
    profile,
    loading,
    isAuthenticated: !!user,
    isAdmin: profile?.role === 'admin',
    isManager: profile?.role === 'manager' || profile?.role === 'admin',
    signIn: authService.signIn,
    signUp: authService.signUp,
    signOut: authService.signOut,
    resetPassword: authService.resetPassword,
    updatePassword: authService.updatePassword,
  };
}
```

### useEvents Hook

**File**: `src/hooks/useEvents.js`

```javascript
import { useState, useEffect } from 'react';
import { eventsService } from '../services/supabase/events';

export function useEvents(startDate, endDate) {
  const [events, setEvents] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    loadEvents();

    // Subscribe to real-time updates
    const subscription = eventsService.subscribeToEvents((payload) => {
      // Refresh events on change
      loadEvents();
    });

    return () => {
      subscription.unsubscribe();
    };
  }, [startDate, endDate]);

  async function loadEvents() {
    try {
      setLoading(true);
      const data = await eventsService.getEventsForDateRange(startDate, endDate);
      setEvents(data);
      setError(null);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }

  return {
    events,
    loading,
    error,
    refresh: loadEvents,
  };
}
```

### useCategories Hook

**File**: `src/hooks/useCategories.js`

```javascript
import { useState, useEffect } from 'react';
import { categoriesService } from '../services/supabase/categories';

export function useCategories() {
  const [categories, setCategories] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    loadCategories();
  }, []);

  async function loadCategories() {
    try {
      setLoading(true);
      const data = await categoriesService.getCategories();
      setCategories(data);
      setError(null);
    } catch (err) {
      setError(err.message);
    } finally {
      setLoading(false);
    }
  }

  return {
    categories,
    loading,
    error,
    refresh: loadCategories,
  };
}
```

## Error Handling

### API Error Handler

**File**: `src/utils/errorHandler.js`

```javascript
export class APIError extends Error {
  constructor(message, code, details) {
    super(message);
    this.name = 'APIError';
    this.code = code;
    this.details = details;
  }
}

export function handleSupabaseError(error) {
  // Auth errors
  if (error.message.includes('Invalid login credentials')) {
    return new APIError('Invalid email or password', 'AUTH_INVALID_CREDENTIALS');
  }

  if (error.message.includes('Email not confirmed')) {
    return new APIError('Please verify your email address', 'AUTH_EMAIL_NOT_CONFIRMED');
  }

  // Permission errors
  if (error.code === '42501') {
    return new APIError('You do not have permission to perform this action', 'PERMISSION_DENIED');
  }

  // Unique constraint violations
  if (error.code === '23505') {
    return new APIError('This record already exists', 'DUPLICATE_RECORD');
  }

  // Foreign key violations
  if (error.code === '23503') {
    return new APIError('Cannot delete: related records exist', 'FOREIGN_KEY_VIOLATION');
  }

  // Default
  return new APIError(error.message || 'An unexpected error occurred', 'UNKNOWN_ERROR', error);
}
```

## Performance Optimization

### Caching Strategy

```javascript
// Simple in-memory cache
const cache = new Map();

export function getCachedData(key, fetchFn, ttl = 60000) {
  const cached = cache.get(key);

  if (cached && Date.now() - cached.timestamp < ttl) {
    return Promise.resolve(cached.data);
  }

  return fetchFn().then(data => {
    cache.set(key, { data, timestamp: Date.now() });
    return data;
  });
}

// Usage
const categories = await getCachedData(
  'categories',
  () => categoriesService.getCategories(),
  300000 // 5 minutes
);
```

### Pagination

```javascript
export async function getPaginatedEvents(page = 1, pageSize = 20) {
  const from = (page - 1) * pageSize;
  const to = from + pageSize - 1;

  const { data, error, count } = await supabase
    .from('events')
    .select('*', { count: 'exact' })
    .range(from, to)
    .order('start_date', { ascending: false });

  if (error) throw error;

  return {
    events: data,
    totalCount: count,
    page,
    pageSize,
    totalPages: Math.ceil(count / pageSize),
  };
}
```

## Real-time Subscriptions

### Calendar Updates

```javascript
export function subscribeToCalendarUpdates(callback) {
  const channel = supabase
    .channel('calendar_updates')
    .on(
      'postgres_changes',
      { event: '*', schema: 'public', table: 'events' },
      (payload) => {
        callback({ type: 'event', payload });
      }
    )
    .on(
      'postgres_changes',
      { event: '*', schema: 'public', table: 'categories' },
      (payload) => {
        callback({ type: 'category', payload });
      }
    )
    .subscribe();

  return () => {
    channel.unsubscribe();
  };
}
```

## Testing

### Mock Supabase Client

**File**: `src/services/supabase/__mocks__/client.js`

```javascript
export const supabase = {
  from: jest.fn(() => ({
    select: jest.fn().mockReturnThis(),
    insert: jest.fn().mockReturnThis(),
    update: jest.fn().mockReturnThis(),
    delete: jest.fn().mockReturnThis(),
    eq: jest.fn().mockReturnThis(),
    single: jest.fn(),
  })),
  auth: {
    signIn: jest.fn(),
    signUp: jest.fn(),
    signOut: jest.fn(),
    getUser: jest.fn(),
    getSession: jest.fn(),
  },
  rpc: jest.fn(),
};
```

## Summary

This API design provides:

✅ **Secure Authentication**: Email/password with Supabase Auth
✅ **Row Level Security**: Database-level permission enforcement
✅ **Type-Safe Services**: Organized service layer
✅ **React Hooks**: Convenient data fetching
✅ **Real-time Updates**: Live calendar synchronization
✅ **Error Handling**: Comprehensive error management
✅ **Performance**: Caching and pagination
✅ **Testing**: Mockable architecture
✅ **Scalability**: Optimized queries and indexes

All security is enforced at the database level through RLS policies, ensuring that even if frontend code is bypassed, data remains protected.
