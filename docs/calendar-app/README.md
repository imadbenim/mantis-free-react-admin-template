# NPO Calendar Sharing Application

## Project Overview

A comprehensive calendar sharing application designed for Non-Profit Organizations (NPOs) that enables teams to coordinate events with multi-level access control. Built with React, Material-UI, and Supabase, this application provides a mobile-first experience for managing and viewing organizational events.

## Purpose

Enable NPO staff, volunteers, and the public to coordinate through a shared calendar system with appropriate access levels for different user types.

## Key Features

### Core Functionality
- **Multi-View Calendar**: Month, Week, Day, and List views
- **Event Management**: Create, Read, Update, Delete events with rich details
- **Recurring Events**: Support for daily, weekly, and monthly recurring events
- **Four-Tier Access Control**: Public Viewer, Member, Manager, Administrator
- **Configurable Categories**: Admin-defined event categories
- **Mobile-First Design**: Responsive UI optimized for mobile devices
- **Public Event Display**: Separate public-facing calendar view
- **Advanced Filtering**: Filter by category, date range, creator, visibility

### User Roles

1. **Public Viewer** (No Authentication)
   - View public events only
   - See event details (title, date, time, location, description)
   - No creation or editing capabilities

2. **Member** (Registered User)
   - View public and internal events
   - View complete event details
   - Cannot create or modify events

3. **Manager** (Event Creator)
   - All Member permissions
   - Create new events
   - Edit and delete own events
   - Set event visibility (public/internal/private)
   - Assign event categories

4. **Administrator** (Full Access)
   - All Manager permissions
   - CRUD operations on any event
   - User management and role assignment
   - Configure event categories
   - System settings and configuration
   - View analytics and reports

## Technical Stack

### Frontend
- **Framework**: React 19.2.0
- **UI Library**: Material-UI v7.3.4
- **Template**: Mantis Free React Admin Template v4.0.0
- **Icons**: Ant Design Icons v6.1.0
- **Build Tool**: Vite
- **Routing**: React Router v7
- **State Management**: React Context / Zustand (TBD)
- **Date Handling**: date-fns or Day.js
- **Calendar Library**: React Big Calendar or custom implementation

### Backend
- **Platform**: Supabase
- **Database**: PostgreSQL (via Supabase)
- **Authentication**: Supabase Auth
- **Storage**: Supabase Storage (for attachments, if needed)
- **Real-time**: Supabase Realtime subscriptions
- **Security**: Row Level Security (RLS) policies

### Development Tools
- **TypeScript**: For type safety
- **ESLint**: Code quality
- **Prettier**: Code formatting

## Project Structure

```
src/
├── pages/
│   ├── calendar/
│   │   ├── index.jsx              # Main calendar view (protected)
│   │   ├── public.jsx             # Public calendar view
│   │   ├── admin.jsx              # Admin dashboard
│   │   └── event/
│   │       └── [id].jsx           # Event detail view
│   └── ...
├── components/
│   ├── calendar/
│   │   ├── CalendarGrid.jsx      # Calendar grid component
│   │   ├── EventCard.jsx          # Event display card
│   │   ├── EventEditor.jsx        # Create/Edit event form
│   │   ├── EventFilters.jsx       # Filter sidebar
│   │   ├── CalendarHeader.jsx     # View switcher & navigation
│   │   ├── EventDetail.jsx        # Event detail modal/view
│   │   └── RecurrenceEditor.jsx   # Recurring event config
│   └── ...
├── services/
│   ├── supabase/
│   │   ├── client.js              # Supabase client config
│   │   ├── events.js              # Event CRUD operations
│   │   ├── categories.js          # Category operations
│   │   ├── auth.js                # Authentication helpers
│   │   └── users.js               # User management
│   └── ...
├── hooks/
│   ├── useEvents.js               # Event data hooks
│   ├── useAuth.js                 # Authentication hooks
│   ├── useCategories.js           # Category hooks
│   └── useCalendar.js             # Calendar state management
├── utils/
│   ├── dateHelpers.js             # Date manipulation utilities
│   ├── recurrence.js              # Recurring event logic
│   ├── permissions.js             # Permission checking
│   └── validation.js              # Form validation
└── types/
    ├── event.ts                   # Event type definitions
    ├── user.ts                    # User type definitions
    └── category.ts                # Category type definitions
```

## Documentation Structure

This documentation is organized into the following sections:

1. **[Requirements](./requirements.md)** - Detailed functional and non-functional requirements
2. **[User Roles](./user-roles.md)** - Complete permissions matrix and role definitions
3. **[Data Model](./data-model.md)** - Supabase database schema and relationships
4. **[API Design](./api-design.md)** - Supabase queries, RPC functions, and RLS policies
5. **[UI/UX Specifications](./ui-ux-specs.md)** - Interface designs and user flows
6. **[Components](./components.md)** - Component breakdown and implementation details
7. **[Pages](./pages.md)** - Page structure, routing, and navigation
8. **[Implementation Plan](./implementation-plan.md)** - Phased development roadmap
9. **[Testing Plan](./testing-plan.md)** - Testing strategy and test cases

## Quick Start Guide

### For Developers

1. **Review Requirements**: Start with [requirements.md](./requirements.md) to understand the full scope
2. **Understand Data Model**: Review [data-model.md](./data-model.md) for database schema
3. **Set Up Supabase**: Follow backend setup in [api-design.md](./api-design.md)
4. **Review Components**: Check [components.md](./components.md) for component architecture
5. **Follow Implementation Plan**: Use [implementation-plan.md](./implementation-plan.md) for phased development

### For Product Owners

1. **Feature Overview**: Review this README and [requirements.md](./requirements.md)
2. **User Experience**: Check [ui-ux-specs.md](./ui-ux-specs.md) for interface designs
3. **Permissions**: Review [user-roles.md](./user-roles.md) for access control details
4. **Timeline**: See [implementation-plan.md](./implementation-plan.md) for development phases

## Key Design Decisions

### Mobile-First Approach
- All interfaces designed for mobile screens first
- Touch-friendly interactions (larger tap targets)
- Responsive breakpoints for tablet and desktop
- Optimized data loading for mobile networks

### Supabase Backend
- Leverages Row Level Security for data protection
- Real-time subscriptions for live calendar updates
- Simplified backend architecture
- Built-in authentication system

### No Attendance Tracking
- Focus on event visibility and management
- Simplified data model
- Reduced complexity for initial release
- Can be added in future iteration

### Configurable Categories
- Admin-defined event types
- Flexible categorization system
- Color-coded categories
- Easy to add/modify without code changes

### Recurring Events
- Support for common patterns (daily, weekly, monthly)
- Individual instance editing capability
- Automatic event generation
- Configurable end dates

## Success Metrics

### User Adoption
- Number of registered users (Members, Managers, Admins)
- Public calendar views
- Events created per month
- Active users (weekly, monthly)

### System Performance
- Page load times < 2 seconds
- Calendar view switch < 500ms
- Event creation < 1 second
- Mobile performance score > 90

### User Satisfaction
- Easy event creation (< 2 minutes average)
- Intuitive navigation
- Clear access control
- Reliable recurring events

## Future Enhancements

Potential features for future iterations:

1. **Attendance Tracking**: RSVP system and attendee lists
2. **Notifications**: Email reminders and event updates
3. **Calendar Export**: iCal/Google Calendar integration
4. **Resource Booking**: Room/equipment scheduling
5. **Approval Workflow**: Event approval process
6. **Department Filtering**: Multi-department support
7. **Public Widget**: Embeddable calendar for NPO website
8. **Event Templates**: Quick event creation from templates
9. **Bulk Operations**: Multi-event management
10. **Analytics Dashboard**: Event statistics and trends

## Support & Contribution

### Getting Help
- Review documentation in this folder
- Check component documentation in `/docs/components/`
- Refer to Mantis documentation: https://codedthemes.gitbook.io/mantis
- Supabase documentation: https://supabase.com/docs

### Contributing
- Follow existing code style and patterns
- Use Mantis design system components
- Write tests for new features
- Update documentation with changes

## Version History

- **v1.0.0** (Planned) - Initial release with core features
  - Four-tier access control
  - Multi-view calendar (Month, Week, Day, List)
  - Event CRUD operations
  - Recurring events
  - Configurable categories
  - Mobile-responsive design

---

**Documentation Version**: 1.0.0
**Last Updated**: November 2025
**Project Status**: Planning Phase
**Target Template Version**: Mantis v4.0.0
