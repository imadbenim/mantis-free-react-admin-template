# Testing Plan and Strategy

## Overview

This document outlines the testing strategy for the NPO Calendar application, including unit tests, integration tests, E2E tests, and manual testing procedures.

## Testing Goals

- **Coverage**: Achieve minimum 70% code coverage
- **Quality**: Catch bugs before production
- **Confidence**: Enable safe refactoring
- **Documentation**: Tests serve as living documentation
- **Regression**: Prevent bugs from reoccurring

## Testing Pyramid

```
      /\
     /E2E\
    /──────\
   /  Integ \
  /──────────\
 /    Unit    \
/──────────────\
```

- **Unit Tests**: 70% of tests (fast, isolated)
- **Integration Tests**: 20% of tests (component integration)
- **E2E Tests**: 10% of tests (critical user paths)

## Testing Tools

### Unit & Component Tests
- **Framework**: Vitest (or Jest)
- **React Testing**: @testing-library/react
- **User Events**: @testing-library/user-event
- **Mocking**: vi.mock (Vitest) or jest.mock

### Integration Tests
- **Framework**: Vitest with React Testing Library
- **API Mocking**: MSW (Mock Service Worker)

### E2E Tests
- **Framework**: Playwright or Cypress
- **Browser**: Chromium, Firefox, Safari (Playwright)

### Coverage
- **Tool**: Vitest coverage (c8) or Istanbul
- **Threshold**: 70% minimum

## Testing Setup

### Install Dependencies

```bash
npm install --save-dev \
  vitest \
  @testing-library/react \
  @testing-library/user-event \
  @testing-library/jest-dom \
  @vitest/ui \
  @vitest/coverage-c8 \
  msw \
  playwright
```

### Vitest Configuration

**File**: `vitest.config.js`

```javascript
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  plugins: [react()],
  test: {
    globals: true,
    environment: 'jsdom',
    setupFiles: './src/test/setup.js',
    coverage: {
      provider: 'c8',
      reporter: ['text', 'json', 'html'],
      exclude: [
        'node_modules/',
        'src/test/',
        '**/*.test.{js,jsx,ts,tsx}',
        '**/__mocks__/**',
      ],
      threshold: {
        global: {
          statements: 70,
          branches: 70,
          functions: 70,
          lines: 70
        }
      }
    }
  },
  resolve: {
    alias: {
      components: path.resolve(__dirname, './src/components'),
      hooks: path.resolve(__dirname, './src/hooks'),
      services: path.resolve(__dirname, './src/services'),
      utils: path.resolve(__dirname, './src/utils'),
    }
  }
});
```

### Test Setup File

**File**: `src/test/setup.js`

```javascript
import '@testing-library/jest-dom';
import { cleanup } from '@testing-library/react';
import { afterEach } from 'vitest';

// Clean up after each test
afterEach(() => {
  cleanup();
});

// Mock window.matchMedia
Object.defineProperty(window, 'matchMedia', {
  writable: true,
  value: vi.fn().mockImplementation(query => ({
    matches: false,
    media: query,
    onchange: null,
    addListener: vi.fn(),
    removeListener: vi.fn(),
    addEventListener: vi.fn(),
    removeEventListener: vi.fn(),
    dispatchEvent: vi.fn(),
  })),
});

// Mock IntersectionObserver
global.IntersectionObserver = class IntersectionObserver {
  constructor() {}
  disconnect() {}
  observe() {}
  takeRecords() { return []; }
  unobserve() {}
};
```

## Unit Tests

### Utility Functions

**Example**: `src/utils/dateHelpers.test.js`

```javascript
import { describe, it, expect } from 'vitest';
import {
  formatEventDate,
  isEventInDateRange,
  getMonthDates,
  formatTimeRange
} from './dateHelpers';

describe('dateHelpers', () => {
  describe('formatEventDate', () => {
    it('formats date correctly', () => {
      const date = new Date('2025-01-15T10:00:00');
      expect(formatEventDate(date)).toBe('January 15, 2025');
    });

    it('handles all-day events', () => {
      const date = new Date('2025-01-15');
      expect(formatEventDate(date, true)).toBe('January 15, 2025 (All Day)');
    });
  });

  describe('isEventInDateRange', () => {
    it('returns true when event is in range', () => {
      const event = {
        start_date: new Date('2025-01-15'),
        end_date: new Date('2025-01-15')
      };
      const start = new Date('2025-01-01');
      const end = new Date('2025-01-31');

      expect(isEventInDateRange(event, start, end)).toBe(true);
    });

    it('returns false when event is outside range', () => {
      const event = {
        start_date: new Date('2025-02-15'),
        end_date: new Date('2025-02-15')
      };
      const start = new Date('2025-01-01');
      const end = new Date('2025-01-31');

      expect(isEventInDateRange(event, start, end)).toBe(false);
    });
  });
});
```

### Permission Utils

**Example**: `src/utils/permissions.test.js`

```javascript
import { describe, it, expect } from 'vitest';
import { canEditEvent, canDeleteEvent, canViewEvent } from './permissions';

describe('permissions', () => {
  const user = { id: 'user-1', role: 'manager' };
  const admin = { id: 'admin-1', role: 'admin' };
  const event = { id: 'event-1', created_by: 'user-1', visibility: 'internal' };

  describe('canEditEvent', () => {
    it('allows creator to edit own event', () => {
      expect(canEditEvent(event, user)).toBe(true);
    });

    it('allows admin to edit any event', () => {
      expect(canEditEvent(event, admin)).toBe(true);
    });

    it('prevents non-creator from editing', () => {
      const otherUser = { id: 'user-2', role: 'manager' };
      expect(canEditEvent(event, otherUser)).toBe(false);
    });
  });

  describe('canViewEvent', () => {
    it('allows viewing public events without auth', () => {
      const publicEvent = { ...event, visibility: 'public' };
      expect(canViewEvent(publicEvent, null)).toBe(true);
    });

    it('prevents viewing internal events without auth', () => {
      expect(canViewEvent(event, null)).toBe(false);
    });

    it('allows members to view internal events', () => {
      const member = { id: 'member-1', role: 'member' };
      expect(canViewEvent(event, member)).toBe(true);
    });
  });
});
```

### React Hooks

**Example**: `src/hooks/useAuth.test.js`

```javascript
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { renderHook, waitFor } from '@testing-library/react';
import { useAuth } from './useAuth';
import { authService } from 'services/supabase/auth';

vi.mock('services/supabase/auth');

describe('useAuth', () => {
  beforeEach(() => {
    vi.clearAllMocks();
  });

  it('loads user on mount', async () => {
    const mockUser = { id: '1', email: 'test@example.com' };
    const mockProfile = { id: '1', name: 'Test User', role: 'member' };

    authService.getSession.mockResolvedValue({ user: mockUser });
    authService.getCurrentProfile.mockResolvedValue(mockProfile);

    const { result } = renderHook(() => useAuth());

    expect(result.current.loading).toBe(true);

    await waitFor(() => {
      expect(result.current.loading).toBe(false);
    });

    expect(result.current.user).toEqual(mockUser);
    expect(result.current.profile).toEqual(mockProfile);
    expect(result.current.isAuthenticated).toBe(true);
  });

  it('handles sign in', async () => {
    const { result } = renderHook(() => useAuth());

    await result.current.signIn('test@example.com', 'password');

    expect(authService.signIn).toHaveBeenCalledWith('test@example.com', 'password');
  });
});
```

## Component Tests

### EventCard

**Example**: `src/components/calendar/EventCard.test.jsx`

```javascript
import { describe, it, expect, vi } from 'vitest';
import { render, screen, fireEvent } from '@testing-library/react';
import { ThemeProvider } from '@mui/material/styles';
import EventCard from './EventCard';
import { theme } from 'themes';

describe('EventCard', () => {
  const mockEvent = {
    id: '1',
    title: 'Test Event',
    start_date: new Date('2025-01-15T10:00:00'),
    end_date: new Date('2025-01-15T11:00:00'),
    location: 'Conference Room',
    category: { name: 'Meeting', color: '#1677FF' }
  };

  const renderWithTheme = (component) => {
    return render(
      <ThemeProvider theme={theme}>
        {component}
      </ThemeProvider>
    );
  };

  it('renders event title', () => {
    renderWithTheme(<EventCard event={mockEvent} variant="compact" />);
    expect(screen.getByText('Test Event')).toBeInTheDocument();
  });

  it('displays time for compact variant', () => {
    renderWithTheme(<EventCard event={mockEvent} variant="compact" />);
    expect(screen.getByText('10:00 AM')).toBeInTheDocument();
  });

  it('shows location in detailed variant', () => {
    renderWithTheme(<EventCard event={mockEvent} variant="detailed" showCategory />);
    expect(screen.getByText('Conference Room')).toBeInTheDocument();
  });

  it('calls onClick when clicked', () => {
    const handleClick = vi.fn();
    renderWithTheme(<EventCard event={mockEvent} variant="compact" onClick={handleClick} />);

    fireEvent.click(screen.getByText('Test Event'));
    expect(handleClick).toHaveBeenCalledTimes(1);
  });

  it('applies category color', () => {
    const { container } = renderWithTheme(<EventCard event={mockEvent} variant="compact" />);
    const card = container.firstChild;

    // Check for category color in styles
    expect(card).toHaveStyle({ borderLeftColor: '#1677FF' });
  });
});
```

### EventEditor

**Example**: `src/components/calendar/EventEditor.test.jsx`

```javascript
import { describe, it, expect, vi } from 'vitest';
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import EventEditor from './EventEditor';

describe('EventEditor', () => {
  const mockOnSave = vi.fn();
  const mockOnCancel = vi.fn();

  it('renders create mode form', () => {
    render(<EventEditor mode="create" onSave={mockOnSave} onCancel={mockOnCancel} />);

    expect(screen.getByLabelText(/title/i)).toBeInTheDocument();
    expect(screen.getByLabelText(/start date/i)).toBeInTheDocument();
    expect(screen.getByLabelText(/end date/i)).toBeInTheDocument();
    expect(screen.getByRole('button', { name: /save/i })).toBeInTheDocument();
  });

  it('validates required fields', async () => {
    render(<EventEditor mode="create" onSave={mockOnSave} onCancel={mockOnCancel} />);

    const saveButton = screen.getByRole('button', { name: /save/i });
    fireEvent.click(saveButton);

    await waitFor(() => {
      expect(screen.getByText(/title is required/i)).toBeInTheDocument();
    });

    expect(mockOnSave).not.toHaveBeenCalled();
  });

  it('submits valid form', async () => {
    const user = userEvent.setup();
    render(<EventEditor mode="create" onSave={mockOnSave} onCancel={mockOnCancel} />);

    await user.type(screen.getByLabelText(/title/i), 'New Event');
    await user.type(screen.getByLabelText(/start date/i), '2025-01-15');
    await user.type(screen.getByLabelText(/start time/i), '10:00');
    await user.type(screen.getByLabelText(/end date/i), '2025-01-15');
    await user.type(screen.getByLabelText(/end time/i), '11:00');

    fireEvent.click(screen.getByRole('button', { name: /save/i }));

    await waitFor(() => {
      expect(mockOnSave).toHaveBeenCalledWith(
        expect.objectContaining({
          title: 'New Event',
        })
      );
    });
  });

  it('populates form in edit mode', () => {
    const event = {
      title: 'Existing Event',
      start_date: new Date('2025-01-15T10:00:00'),
      end_date: new Date('2025-01-15T11:00:00'),
      location: 'Room 1'
    };

    render(<EventEditor mode="edit" event={event} onSave={mockOnSave} onCancel={mockOnCancel} />);

    expect(screen.getByDisplayValue('Existing Event')).toBeInTheDocument();
    expect(screen.getByDisplayValue('Room 1')).toBeInTheDocument();
  });
});
```

## Integration Tests

### Calendar Page

**Example**: `src/pages/calendar/index.test.jsx`

```javascript
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import { BrowserRouter } from 'react-router-dom';
import CalendarPage from './index';
import { useAuth } from 'hooks/useAuth';
import { useEvents } from 'hooks/useEvents';

vi.mock('hooks/useAuth');
vi.mock('hooks/useEvents');

describe('CalendarPage Integration', () => {
  beforeEach(() => {
    useAuth.mockReturnValue({
      profile: { id: '1', role: 'manager' },
      isManager: true,
      isAdmin: false
    });

    useEvents.mockReturnValue({
      events: [
        {
          id: '1',
          title: 'Event 1',
          start_date: new Date('2025-01-15T10:00:00'),
          end_date: new Date('2025-01-15T11:00:00'),
          visibility: 'public'
        }
      ],
      loading: false,
      error: null
    });
  });

  it('displays events on calendar', async () => {
    render(
      <BrowserRouter>
        <CalendarPage />
      </BrowserRouter>
    );

    await waitFor(() => {
      expect(screen.getByText('Event 1')).toBeInTheDocument();
    });
  });

  it('opens event detail when event clicked', async () => {
    render(
      <BrowserRouter>
        <CalendarPage />
      </BrowserRouter>
    );

    const event = await screen.findByText('Event 1');
    fireEvent.click(event);

    await waitFor(() => {
      expect(screen.getByRole('dialog')).toBeInTheDocument();
    });
  });

  it('shows create button for managers', () => {
    render(
      <BrowserRouter>
        <CalendarPage />
      </BrowserRouter>
    );

    expect(screen.getByRole('button', { name: /create event/i })).toBeInTheDocument();
  });

  it('hides create button for members', () => {
    useAuth.mockReturnValue({
      profile: { id: '1', role: 'member' },
      isManager: false,
      isAdmin: false
    });

    render(
      <BrowserRouter>
        <CalendarPage />
      </BrowserRouter>
    );

    expect(screen.queryByRole('button', { name: /create event/i })).not.toBeInTheDocument();
  });
});
```

### Event CRUD Flow

**Example**: `src/test/integration/eventCrud.test.jsx`

```javascript
import { describe, it, expect, vi, beforeEach } from 'vitest';
import { render, screen, fireEvent, waitFor } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import { setupServer } from 'msw/node';
import { rest } from 'msw';
import App from 'App';

const server = setupServer(
  rest.post('/api/events', (req, res, ctx) => {
    return res(ctx.json({ id: 'new-event', ...req.body }));
  }),
  rest.put('/api/events/:id', (req, res, ctx) => {
    return res(ctx.json({ id: req.params.id, ...req.body }));
  }),
  rest.delete('/api/events/:id', (req, res, ctx) => {
    return res(ctx.status(204));
  })
);

beforeAll(() => server.listen());
afterEach(() => server.resetHandlers());
afterAll(() => server.close());

describe('Event CRUD Integration', () => {
  it('completes create event flow', async () => {
    const user = userEvent.setup();
    render(<App />);

    // Navigate to calendar
    fireEvent.click(screen.getByText(/calendar/i));

    // Click create button
    fireEvent.click(screen.getByRole('button', { name: /create event/i }));

    // Fill form
    await user.type(screen.getByLabelText(/title/i), 'New Meeting');
    await user.type(screen.getByLabelText(/location/i), 'Room 1');

    // Save
    fireEvent.click(screen.getByRole('button', { name: /save/i }));

    // Verify success
    await waitFor(() => {
      expect(screen.getByText('Event created successfully')).toBeInTheDocument();
    });

    // Verify event appears
    expect(screen.getByText('New Meeting')).toBeInTheDocument();
  });
});
```

## E2E Tests

### Critical User Paths

**Example**: `tests/e2e/calendar.spec.js` (Playwright)

```javascript
import { test, expect } from '@playwright/test';

test.describe('Calendar Application', () => {
  test.beforeEach(async ({ page }) => {
    await page.goto('http://localhost:3000');

    // Login
    await page.fill('input[name="email"]', 'manager@example.com');
    await page.fill('input[name="password"]', 'password123');
    await page.click('button:has-text("Login")');

    await expect(page).toHaveURL(/.*calendar/);
  });

  test('user can create an event', async ({ page }) => {
    // Click create event
    await page.click('button:has-text("Create Event")');

    // Fill form
    await page.fill('input[name="title"]', 'Test Event');
    await page.fill('input[name="location"]', 'Conference Room');
    await page.selectOption('select[name="category"]', 'Meeting');

    // Save
    await page.click('button:has-text("Save Event")');

    // Verify success
    await expect(page.locator('text=Event created successfully')).toBeVisible();

    // Verify event appears on calendar
    await expect(page.locator('text=Test Event')).toBeVisible();
  });

  test('user can filter events by category', async ({ page }) => {
    // Open filters
    await page.click('button:has-text("Filters")');

    // Uncheck all categories
    await page.uncheck('input[value="all-categories"]');

    // Check only "Meeting"
    await page.check('input[value="meeting"]');

    // Apply filters (mobile)
    if (await page.locator('button:has-text("Apply")').isVisible()) {
      await page.click('button:has-text("Apply")');
    }

    // Verify only meeting events visible
    const events = page.locator('[data-category="meeting"]');
    await expect(events.first()).toBeVisible();
  });

  test('admin can change user roles', async ({ page }) => {
    // Navigate to admin
    await page.click('text=Administration');

    // Find user
    const userRow = page.locator('tr:has-text("test@example.com")');

    // Change role
    await userRow.locator('select[name="role"]').selectOption('manager');

    // Verify confirmation
    await expect(page.locator('text=Role updated successfully')).toBeVisible();
  });
});
```

### Mobile Testing

```javascript
test.describe('Mobile Calendar', () => {
  test.use({
    viewport: { width: 375, height: 667 } // iPhone SE
  });

  test('mobile navigation works', async ({ page }) => {
    await page.goto('http://localhost:3000/calendar');

    // Test swipe navigation (simulate)
    const calendar = page.locator('[data-testid="calendar-grid"]');
    await calendar.swipe('left');

    // Verify next month
    await expect(page.locator('text=February')).toBeVisible();
  });
});
```

## Performance Testing

### Lighthouse CI

**File**: `.lighthouserc.json`

```json
{
  "ci": {
    "collect": {
      "url": ["http://localhost:3000/calendar"],
      "numberOfRuns": 3
    },
    "assert": {
      "assertions": {
        "categories:performance": ["error", { "minScore": 0.9 }],
        "categories:accessibility": ["error", { "minScore": 0.9 }],
        "categories:best-practices": ["error", { "minScore": 0.9 }],
        "categories:seo": ["error", { "minScore": 0.9 }]
      }
    }
  }
}
```

## Manual Testing Checklist

### Functional Testing

#### Authentication
- [ ] User can register
- [ ] Email verification works
- [ ] User can login
- [ ] User can logout
- [ ] Password reset works
- [ ] Session persists across reloads

#### Calendar Views
- [ ] Month view displays correctly
- [ ] Week view displays correctly
- [ ] Day view displays correctly
- [ ] List view displays correctly
- [ ] View switching works
- [ ] Navigation (prev/next/today) works

#### Event Management
- [ ] Manager can create event
- [ ] Event appears on calendar
- [ ] User can view event details
- [ ] Manager can edit own event
- [ ] Admin can edit any event
- [ ] Manager can delete own event
- [ ] Admin can delete any event
- [ ] Recurring events create correctly
- [ ] Edit recurring instance works
- [ ] Delete recurring instance works

#### Filtering
- [ ] Category filter works
- [ ] Visibility filter works (authenticated)
- [ ] My events filter works (managers)
- [ ] Multiple filters combine correctly
- [ ] Clear filters works

#### Search
- [ ] Text search finds events
- [ ] Search highlights results
- [ ] Clear search works

#### Admin
- [ ] Admin dashboard accessible (admin only)
- [ ] User list displays
- [ ] Role changes save
- [ ] Cannot change own role
- [ ] Category list displays
- [ ] Can add category
- [ ] Can edit category
- [ ] Can delete category (soft delete)
- [ ] Can reorder categories

#### Public Calendar
- [ ] Public calendar accessible without auth
- [ ] Only public events shown
- [ ] Event details work
- [ ] No edit/delete buttons shown

### Cross-Browser Testing
- [ ] Chrome (latest)
- [ ] Firefox (latest)
- [ ] Safari (latest)
- [ ] Edge (latest)

### Mobile Testing
- [ ] iOS Safari
- [ ] Chrome Mobile (Android)
- [ ] Touch interactions work
- [ ] Swipe gestures work
- [ ] Bottom sheet modals work
- [ ] Forms usable

### Accessibility Testing
- [ ] Keyboard navigation works
- [ ] Screen reader announces correctly
- [ ] Color contrast sufficient
- [ ] Focus indicators visible
- [ ] ARIA labels present

### Performance Testing
- [ ] Initial load < 2s (desktop)
- [ ] Initial load < 3s (mobile 4G)
- [ ] Calendar view switch < 500ms
- [ ] Event creation < 1s
- [ ] No janky animations
- [ ] Lighthouse score > 90

### Security Testing
- [ ] RLS policies enforce permissions
- [ ] Cannot access protected routes without auth
- [ ] Cannot access admin routes as non-admin
- [ ] Cannot edit others' events
- [ ] Cannot delete others' events
- [ ] XSS protection works
- [ ] SQL injection protected (via Supabase)

## Regression Testing

After each release, run:

1. **Smoke Tests**: Critical paths work
2. **Regression Suite**: All automated tests pass
3. **Manual Checklist**: Key features verified
4. **Performance Check**: Lighthouse scores maintained
5. **Accessibility Audit**: No regressions

## Bug Tracking

### Bug Report Template

```markdown
**Title**: [Brief description]

**Environment**:
- Browser: [Chrome/Firefox/Safari/Edge]
- Version: [Version number]
- Device: [Desktop/Mobile]
- OS: [Windows/Mac/iOS/Android]

**Steps to Reproduce**:
1. [Step 1]
2. [Step 2]
3. [Step 3]

**Expected Behavior**:
[What should happen]

**Actual Behavior**:
[What actually happens]

**Screenshots/Video**:
[If applicable]

**Console Errors**:
[Any errors in console]

**Severity**:
[Critical/High/Medium/Low]
```

## Test Coverage Goals

### Overall Coverage: 70%+

**By Module**:
- Utils: 90%+
- Hooks: 80%+
- Services: 80%+
- Components: 70%+
- Pages: 60%+

**Coverage Types**:
- Statements: 70%+
- Branches: 70%+
- Functions: 70%+
- Lines: 70%+

## Continuous Integration

### GitHub Actions Workflow

**File**: `.github/workflows/test.yml`

```yaml
name: Test

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'

      - name: Install dependencies
        run: npm ci

      - name: Run unit tests
        run: npm test -- --coverage

      - name: Upload coverage
        uses: codecov/codecov-action@v3

      - name: Run E2E tests
        run: npm run test:e2e

      - name: Run Lighthouse
        run: npm run lighthouse
```

## Summary

This comprehensive testing strategy ensures:

✅ **High quality** through multi-level testing
✅ **Confidence** in deployments
✅ **Fast feedback** with unit tests
✅ **Real-world validation** with E2E tests
✅ **Performance** monitoring
✅ **Accessibility** compliance
✅ **Security** verification
✅ **Regression** prevention

**Key Success Factors**:
- Write tests alongside features
- Run tests before commits
- Monitor coverage trends
- Fix flaky tests immediately
- Review test failures carefully
- Keep tests maintainable
