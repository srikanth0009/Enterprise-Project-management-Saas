# Enterprise Project Management SaaS — Frontend Architecture

## 1. Purpose

This document defines the architecture of the React frontend.

Goals:

- Scalable
- Maintainable
- Type-safe
- Reusable
- Testable
- Easy to extend

## 2. Technology Stack

```text
React
TypeScript
Vite
React Router
Tailwind CSS
ShadCN UI
TanStack Query
Zustand
React Hook Form
Zod
Axios / Fetch
WebSocket
```

## 3. Architecture Overview

```text
React Application
        │
 ┌──────┼──────────┐
 │      │          │
Router  UI       State
 │      │          │
Pages  Features  React Query + Zustand
 │
API Client
 │
REST API
 │
Backend
 │
WebSocket ↔ Real-time events
```

The frontend is organized around features/domains rather than one large collection of unrelated files.

## 4. Folder Structure

```text
src/
├── app/
│   ├── App.tsx
│   ├── providers/
│   ├── router/
│   └── layouts/
├── components/
│   ├── ui/
│   ├── common/
│   ├── feedback/
│   └── navigation/
├── features/
│   ├── auth/
│   ├── organizations/
│   ├── teams/
│   ├── workspaces/
│   ├── projects/
│   ├── issues/
│   ├── comments/
│   ├── labels/
│   ├── attachments/
│   ├── notifications/
│   └── dashboard/
├── hooks/
├── lib/
│   ├── api/
│   ├── websocket/
│   ├── utils/
│   └── constants/
├── stores/
├── types/
├── config/
├── styles/
└── main.tsx
```

## 5. Why Feature-Based Architecture?

For example:

```text
features/issues/
├── api/
├── components/
├── hooks/
├── pages/
├── schemas/
└── types/
```

All issue-related code stays together, making the application easier to navigate as it grows.

## 6. Application Layers

```text
UI
 ↓
Feature Logic
 ↓
State / Data Fetching
 ↓
API Client
 ↓
Backend
```

Example:

```text
IssuePage
 ↓
useIssues()
 ↓
React Query
 ↓
issueApi.getIssues()
 ↓
API Client
 ↓
REST API
```

## 7. Routing

Suggested structure:

```text
/
├── /login
├── /register
├── /forgot-password
└── /app
    ├── /select-organization
    └── /:organizationSlug
        ├── /dashboard
        ├── /teams
        ├── /workspaces
        └── /projects/:projectId
            ├── /overview
            ├── /issues
            ├── /board
            ├── /activity
            └── /settings
```

## 8. Layout Architecture

```text
AuthLayout
 └── Login / Register

AppLayout
 ├── Sidebar
 ├── Topbar
 └── Outlet

OrganizationLayout
 └── Organization pages

ProjectLayout
 ├── Project header
 ├── Project navigation
 └── Outlet
```

## 9. Authentication

Frontend tracks:

```text
loading
authenticated
unauthenticated
```

Flow:

```text
Login
 ↓
Backend
 ↓
Access token + refresh mechanism
 ↓
Authentication state
 ↓
Protected routes
```

Backend authorization remains the actual security boundary.

## 10. Protected Routes

```text
Visit /app
 ↓
ProtectedRoute
 ↓
Loading?
 ├── YES → Loading UI
 └── NO
      ↓
Authenticated?
 ├── YES → App
 └── NO  → Login
```

## 11. React Query

React Query owns server state:

```text
useCurrentUser()
useOrganizations()
useProjects()
useProject()
useIssues()
useComments()
useNotifications()
```

Mutations:

```text
useCreateProject()
useUpdateProject()
useDeleteProject()
useCreateIssue()
useUpdateIssue()
useDeleteIssue()
useCreateComment()
```

## 12. Query Key Strategy

Examples:

```text
["current-user"]
["organizations"]
["organization", organizationId]
["projects", workspaceId]
["project", projectId]
["issues", projectId, filters]
["issue", issueId]
["comments", issueId]
["notifications"]
```

Predictable query keys make cache invalidation easier.

## 13. React Query Mutations

```text
User action
 ↓
Mutation
 ↓
API request
 ↓
Backend
 ↓
Success
 ↓
Update / invalidate query
 ↓
UI updates
```

For Kanban movement, optimistic updates can make the interface feel immediate. Roll back if the server rejects the change.

## 14. Zustand

Zustand owns client/UI state.

Good examples:

```text
Sidebar
Modal state
Selected workspace
Selected project
Command palette
UI preferences
```

Do not use Zustand as a replacement for React Query.

## 15. State Decision Rule

```text
Does backend own this data?

YES → React Query
NO  → Local React state / Zustand
```

Examples:

```text
Project list       → React Query
Issue list         → React Query
Current user       → React Query
Sidebar open       → Zustand
Modal open         → Zustand
Input value        → Local state / React Hook Form
```

## 16. Forms

Use:

```text
React Hook Form
+
Zod
```

Flow:

```text
Form
 ↓
React Hook Form
 ↓
Zod validation
 ↓
React Query mutation
 ↓
API
```

Validate on both frontend and backend.

Frontend validation improves UX; backend validation provides security and correctness.

## 17. API Client

Do not call `fetch()`/Axios independently from every component.

Use:

```text
Component
 ↓
Feature API function
 ↓
Central API Client
 ↓
HTTP
```

Example:

```text
issueApi.getIssues()
issueApi.createIssue()
issueApi.updateIssue()
issueApi.deleteIssue()
```

The client handles common concerns such as:

```text
Base URL
Headers
Authentication
Response parsing
Errors
Refresh-token handling
```

## 18. Feature API Structure

```text
features/issues/
├── api/
│   ├── issue.api.ts
│   └── issue.queries.ts
├── components/
├── hooks/
├── pages/
├── schemas/
└── types/
```

`issue.api.ts` handles HTTP functions.

`issue.queries.ts` contains React Query hooks/options.

## 19. WebSocket Architecture

```text
WebSocket Client
 ↓
Connection
 ↓
Authentication
 ↓
Subscriptions
 ↓
Events
 ↓
React Query cache update/invalidation
```

Possible events:

```text
issue.created
issue.updated
issue.deleted
comment.created
notification.created
project.updated
member.added
member.removed
```

## 20. WebSocket Lifecycle

```text
Login
 ↓
Connect
 ↓
Authenticate
 ↓
Subscribe
 ↓
Receive events
 ↓
Update server-state cache
```

Logout:

```text
Logout
 ↓
Disconnect
 ↓
Clear user-specific state
```

Reconnect after temporary connection loss using controlled backoff.

## 21. Kanban Architecture

```text
KanbanBoard
 ├── KanbanColumn
 │    ├── IssueCard
 │    └── IssueCard
 ├── KanbanColumn
 │    └── IssueCard
 └── KanbanColumn
      └── IssueCard
```

Use a drag-and-drop library such as `dnd-kit`.

Flow:

```text
Drag
 ↓
Drop
 ↓
Calculate new status/position
 ↓
Optimistic UI update
 ↓
PATCH issue
 ↓
Success → keep
Failure → rollback
```

## 22. Issue Detail

```text
IssueDetail
├── IssueHeader
├── Description
├── Metadata
│   ├── Status
│   ├── Priority
│   ├── Assignee
│   └── Labels
├── Attachments
├── Comments
└── ActivityTimeline
```

## 23. Reusable Components

Base components:

```text
Button
Input
Dialog
Dropdown
Avatar
Badge
Tooltip
DataTable
Pagination
SearchInput
ConfirmDialog
EmptyState
LoadingState
ErrorState
```

Feature-specific components remain inside their feature:

```text
IssueCard
ProjectCard
OrganizationSwitcher
```

## 24. Component Responsibility

Avoid giant components.

Bad:

```text
ProjectPage.tsx
 ├── data fetching
 ├── forms
 ├── business logic
 ├── modals
 ├── tables
 └── WebSockets
```

Better:

```text
ProjectPage
├── ProjectHeader
├── ProjectStats
├── ProjectTabs
├── ProjectMembers
└── ProjectActivity
```

Logic belongs in hooks/services where appropriate.

## 25. Loading, Error and Empty States

Every asynchronous feature should handle:

```text
Initial loading
Background fetching
Mutation loading
Network errors
401
403
404
Validation errors
500 errors
Empty data
```

Examples:

```text
Skeleton
Spinner
Disabled submit button
ErrorState
EmptyState
```

## 26. Search

Search should be debounced.

```text
User types
 ↓
~300ms debounce
 ↓
API request
 ↓
Results
```

Do not request on every keystroke.

## 27. Filters

Important filters can live in the URL:

```text
/projects/123/issues?status=todo&priority=high
```

This makes filters shareable and restorable after refresh.

## 28. Pagination

Use server-side pagination for large lists:

```text
GET /issues?page=2&limit=20
```

For large feeds, cursor pagination can be introduced later.

## 29. Permissions

Frontend permissions are for UI behavior:

```text
Can user edit?
 ↓
Show/hide Edit button
```

Backend authorization must still enforce:

```text
PATCH /projects/:id
```

The frontend is never the security boundary.

## 30. Notifications

Initial data:

```text
GET /notifications
```

Real-time updates:

```text
WebSocket
 ↓
notification.created
 ↓
React Query cache update
 ↓
Badge updates
```

## 31. URL State

Good candidates:

```text
Search
Filters
Sort
Page
Selected issue
```

Example:

```text
/projects/123/issues?status=in_progress&priority=high&search=login
```

## 32. TypeScript Strategy

Use explicit types for:

```text
API responses
Request payloads
Entities
Forms
Component props
WebSocket messages
```

Example:

```typescript
type IssueStatus =
  | "TODO"
  | "IN_PROGRESS"
  | "IN_REVIEW"
  | "DONE";

type IssuePriority =
  | "LOW"
  | "MEDIUM"
  | "HIGH"
  | "URGENT";
```

Avoid `any` unless there is a strong reason.

## 33. API Response Types

Example:

```typescript
type ApiResponse<T> = {
  data: T;
  message?: string;
};

type ApiError = {
  code: string;
  message: string;
  details?: unknown;
};
```

These must match the API contract.

## 34. Authentication Error Handling

For `401`:

```text
API request
 ↓
401
 ↓
Attempt refresh
 ↓
Success → retry request
Failure → clear auth → disconnect WebSocket → login
```

Avoid infinite refresh loops.

For `403`:

```text
Show permission error
```

## 35. Performance

Use where appropriate:

```text
Route code splitting
Lazy loading
React Query caching
Debounced search
Server pagination
Optimistic updates
Virtualized large lists
Image optimization
Avoid unnecessary global state
```

Do not optimize prematurely; measure when possible.

## 36. Accessibility

Support:

```text
Keyboard navigation
Focus management
Semantic HTML
Accessible labels
ARIA where needed
Visible focus states
Color-independent status indicators
```

Pay special attention to:

```text
Dialogs
Dropdowns
Kanban
Forms
Tables
Navigation
```

## 37. Testing

Recommended:

```text
Vitest
React Testing Library
Playwright
```

Testing levels:

```text
Unit
 → utility functions

Component
 → IssueCard

Integration
 → issue form + mocked API

E2E
 → Login → Create Project → Create Issue → Move Issue
```

## 38. Environment Variables

Safe browser-visible values may include:

```text
VITE_API_URL
VITE_WS_URL
VITE_CLOUDINARY_CLOUD_NAME
```

Never expose:

```text
Database passwords
JWT signing secrets
Redis passwords
Private API keys
```

Secrets belong on the backend.

## 39. Frontend Security Rules

Never trust the frontend for:

```text
Authorization
Permissions
User identity
Organization ownership
Project access
```

The backend must verify all of these.

Avoid storing sensitive credentials in localStorage unnecessarily.

## 40. Architecture Rules

```text
1. Components focus on UI.
2. Business logic belongs in feature hooks/services.
3. API calls belong in feature API modules.
4. React Query owns server state.
5. Zustand owns selected client/UI state.
6. Keep local state local when possible.
7. Forms use React Hook Form + Zod.
8. API access goes through the centralized client.
9. WebSockets synchronize server state.
10. Backend remains the security boundary.
11. Avoid unnecessary global state.
12. Reuse components for repeated UI.
13. Keep feature modules independent where possible.
14. Keep TypeScript types explicit.
15. Handle loading, error and empty states.
```

## 41. Complete Frontend Flow

```text
React Application
      ↓
Router
      ↓
Layout
      ↓
Feature Page
      ↓
UI Components + Feature Hooks
      ↓
┌───────────────┬───────────────┐
│               │               │
React Query   Zustand       Local State
│
API Client
 ├── REST
 └── WebSocket
      ↓
Backend API
```

## 42. Design Progress

```text
1. Product Requirements       ✅
2. User Flows                 ✅
3. Domain Model               ✅
4. Database Design            ✅
5. API Design                 ✅
6. System Architecture        ✅
7. Frontend Architecture      ✅

8. UI Design                  ← NEXT
9. Implementation
```

The next step is **UI/UX Design**. We will define the application's screens, navigation, dashboard, project pages, issue pages, Kanban board, forms, dialogs, and reusable UI patterns before implementation.
