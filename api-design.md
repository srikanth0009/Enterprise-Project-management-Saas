# Enterprise Project Management SaaS — API Design

This document defines the REST API contract between the React frontend and Node.js/Express backend.

It explains not only the endpoints, but also the reasoning behind authentication, authorization, validation, pagination, filtering, errors, and real-time communication.

## 1. API Architecture

```text
React Frontend
      ↓
HTTP Request
      ↓
Express Router
      ↓
Authentication Middleware
      ↓
Authorization Middleware
      ↓
Validation Middleware
      ↓
Controller
      ↓
Service
      ↓
Repository / Database Layer
      ↓
MongoDB
```

### Why these layers?

- **Router** — maps URLs to handlers.
- **Middleware** — authentication, authorization, validation.
- **Controller** — handles HTTP request/response.
- **Service** — contains business logic.
- **Repository/data layer** — handles database operations.
- **Database** — persists data.

Keeping these responsibilities separate prevents business logic from becoming tightly coupled to HTTP handlers.

---

## 2. API Versioning

Base path:

```text
/api/v1
```

Example:

```text
/api/v1/projects
```

### Why?

A version allows us to introduce future breaking API changes without immediately breaking existing clients.

---

## 3. HTTP Methods

| Method | Purpose |
|---|---|
| GET | Retrieve resources |
| POST | Create a resource or perform an action |
| PATCH | Partially update a resource |
| PUT | Replace a resource when necessary |
| DELETE | Delete/archive a resource |

We will mainly use `PATCH` for updates because most UI actions change only a few fields.

---

# 4. Authentication

The application uses:

```text
Short-lived Access Token
+
Refresh Token
```

Flow:

```text
Login
  ↓
Access Token
  ↓
API Requests

Refresh Token
  ↓
New Access Token
```

### Why?

Access tokens can be short-lived while refresh tokens maintain the user's session.

The refresh token should preferably be stored in a secure, HTTP-only cookie.

---

# 5. Authentication Endpoints

## Register

```http
POST /api/v1/auth/register
```

Request:

```json
{
  "name": "Srikanth",
  "email": "srikanth@example.com",
  "password": "StrongPassword123!"
}
```

Response:

```json
{
  "data": {
    "user": {
      "id": "...",
      "name": "Srikanth",
      "email": "srikanth@example.com"
    },
    "accessToken": "..."
  }
}
```

Passwords are hashed before storage.

---

## Login

```http
POST /api/v1/auth/login
```

Request:

```json
{
  "email": "srikanth@example.com",
  "password": "StrongPassword123!"
}
```

Response:

```json
{
  "data": {
    "user": {
      "id": "...",
      "name": "Srikanth",
      "email": "srikanth@example.com"
    },
    "accessToken": "..."
  }
}
```

---

## Refresh Token

```http
POST /api/v1/auth/refresh
```

The refresh token is read from the secure cookie.

Response:

```json
{
  "data": {
    "accessToken": "..."
  }
}
```

---

## Logout

```http
POST /api/v1/auth/logout
```

Response:

```json
{
  "data": {
    "message": "Logged out successfully"
  }
}
```

### Why POST?

Logout changes authentication state, so it is modeled as an action rather than a normal resource deletion.

---

## Current User

```http
GET /api/v1/auth/me
```

Response:

```json
{
  "data": {
    "id": "...",
    "name": "Srikanth",
    "email": "srikanth@example.com",
    "avatarUrl": "..."
  }
}
```

### Why?

The frontend needs a reliable way to restore the current user after a page refresh.

---

# 6. Google OAuth

```http
GET /api/v1/auth/google
GET /api/v1/auth/google/callback
```

Flow:

```text
Frontend
   ↓
Google
   ↓
User authenticates
   ↓
Google callback
   ↓
Backend
   ↓
Find/Create User
   ↓
Create session
   ↓
Frontend
```

### Why backend-managed OAuth?

OAuth credentials and authorization-code handling should stay on the server.

---

# 7. Authentication Middleware

Protected requests use:

```http
Authorization: Bearer <accessToken>
```

Flow:

```text
Extract token
     ↓
Verify token
     ↓
Extract user ID
     ↓
Attach authenticated user
     ↓
Continue
```

Example:

```javascript
req.user = {
  id: "...",
  organizationId: "..."
};
```

### Why?

Token verification belongs in shared middleware rather than being duplicated across controllers.

---

# 8. Organizations API

## Create

```http
POST /api/v1/organizations
```

Request:

```json
{
  "name": "Acme Technologies"
}
```

## List My Organizations

```http
GET /api/v1/organizations
```

## Get

```http
GET /api/v1/organizations/:organizationId
```

## Update

```http
PATCH /api/v1/organizations/:organizationId
```

Request:

```json
{
  "name": "Acme Technologies India"
}
```

## Archive

```http
DELETE /api/v1/organizations/:organizationId
```

For the MVP, this should preferably archive/soft-delete rather than immediately destroy all data.

---

# 9. Organization Members API

## List

```http
GET /api/v1/organizations/:organizationId/members
```

Query:

```text
?page=1
&limit=20
&search=srikanth
&role=MEMBER
```

## Invite

```http
POST /api/v1/organizations/:organizationId/invitations
```

## Update Role

```http
PATCH /api/v1/organizations/:organizationId/members/:userId
```

Request:

```json
{
  "role": "ADMIN"
}
```

## Remove

```http
DELETE /api/v1/organizations/:organizationId/members/:userId
```

Only authorized organization roles may perform administrative actions.

---

# 10. Teams API

## Create

```http
POST /api/v1/organizations/:organizationId/teams
```

Request:

```json
{
  "name": "Engineering",
  "description": "Product engineering team"
}
```

## List

```http
GET /api/v1/organizations/:organizationId/teams
```

## Get

```http
GET /api/v1/teams/:teamId
```

## Update

```http
PATCH /api/v1/teams/:teamId
```

## Delete

```http
DELETE /api/v1/teams/:teamId
```

---

# 11. Team Members API

```http
GET    /api/v1/teams/:teamId/members
POST   /api/v1/teams/:teamId/members
DELETE /api/v1/teams/:teamId/members/:userId
```

Add member request:

```json
{
  "userId": "..."
}
```

---

# 12. Workspaces API

## Create

```http
POST /api/v1/organizations/:organizationId/workspaces
```

Request:

```json
{
  "name": "Product Development"
}
```

## List

```http
GET /api/v1/organizations/:organizationId/workspaces
```

## Get

```http
GET /api/v1/workspaces/:workspaceId
```

## Update

```http
PATCH /api/v1/workspaces/:workspaceId
```

## Archive

```http
DELETE /api/v1/workspaces/:workspaceId
```

---

# 13. Workspace Members API

```http
GET    /api/v1/workspaces/:workspaceId/members
POST   /api/v1/workspaces/:workspaceId/members
DELETE /api/v1/workspaces/:workspaceId/members/:userId
```

Add member:

```json
{
  "userId": "...",
  "role": "MEMBER"
}
```

### Why separate workspace membership?

An organization member does not necessarily need access to every workspace.

---

# 14. Projects API

## Create

```http
POST /api/v1/workspaces/:workspaceId/projects
```

Request:

```json
{
  "name": "E-Commerce Platform",
  "key": "ECOM",
  "description": "Build the new e-commerce platform",
  "teamId": "..."
}
```

Response:

```json
{
  "data": {
    "id": "...",
    "name": "E-Commerce Platform",
    "key": "ECOM",
    "slug": "e-commerce-platform"
  }
}
```

## List

```http
GET /api/v1/workspaces/:workspaceId/projects
```

Query:

```text
?page=1
&limit=20
&status=ACTIVE
```

## Get

```http
GET /api/v1/projects/:projectId
```

## Update

```http
PATCH /api/v1/projects/:projectId
```

## Archive

```http
DELETE /api/v1/projects/:projectId
```

---

# 15. Project Members API

```http
GET    /api/v1/projects/:projectId/members
POST   /api/v1/projects/:projectId/members
PATCH  /api/v1/projects/:projectId/members/:userId
DELETE /api/v1/projects/:projectId/members/:userId
```

Add member:

```json
{
  "userId": "...",
  "role": "DEVELOPER"
}
```

---

# 16. Issues API

Issues are the central resource.

## Create Issue

```http
POST /api/v1/projects/:projectId/issues
```

Request:

```json
{
  "title": "Implement Google OAuth",
  "description": "Add Google authentication flow",
  "type": "FEATURE",
  "priority": "HIGH",
  "assigneeId": "...",
  "labelIds": ["..."]
}
```

Response:

```json
{
  "data": {
    "id": "...",
    "key": "ECOM-42",
    "title": "Implement Google OAuth",
    "status": "TODO",
    "priority": "HIGH",
    "assigneeId": "..."
  }
}
```

Backend flow:

```text
Validate
   ↓
Verify project access
   ↓
Generate issue number
   ↓
Create issue
   ↓
Create activity
   ↓
Create notification if required
   ↓
Return issue
```

---

# 17. List Issues

```http
GET /api/v1/projects/:projectId/issues
```

Query parameters:

```text
?page=1
&limit=20
&status=TODO
&priority=HIGH
&assigneeId=...
&type=BUG
&labelId=...
&search=oauth
&sortBy=createdAt
&sortOrder=desc
```

### Why query parameters?

Filtering and sorting are characteristics of collection retrieval. We do not need separate endpoints for every combination.

---

# 18. Get Issue

```http
GET /api/v1/issues/:issueId
```

---

# 19. Update Issue

```http
PATCH /api/v1/issues/:issueId
```

Request:

```json
{
  "status": "IN_PROGRESS"
}
```

Or:

```json
{
  "assigneeId": "...",
  "priority": "URGENT"
}
```

### Why PATCH?

Only changed fields need to be transmitted.

---

# 20. Delete Issue

```http
DELETE /api/v1/issues/:issueId
```

Initially this should preferably soft-delete the issue.

---

# 21. Kanban Drag and Drop

Moving an issue from one column to another can simply use:

```http
PATCH /api/v1/issues/:issueId
```

Request:

```json
{
  "status": "IN_PROGRESS"
}
```

If explicit ordering is later implemented:

```json
{
  "status": "IN_PROGRESS",
  "position": 1200
}
```

### Why?

Dragging an issue changes the issue's state. A separate endpoint is unnecessary unless the ordering rules become more complex.

---

# 22. Comments API

```http
GET    /api/v1/issues/:issueId/comments
POST   /api/v1/issues/:issueId/comments
PATCH  /api/v1/comments/:commentId
DELETE /api/v1/comments/:commentId
```

Create comment:

```json
{
  "content": "Please check the payment API.",
  "mentionUserIds": ["..."]
}
```

For comments, cursor pagination is preferred because comment histories can become large.

---

# 23. Labels API

```http
POST   /api/v1/projects/:projectId/labels
GET    /api/v1/projects/:projectId/labels
PATCH  /api/v1/labels/:labelId
DELETE /api/v1/labels/:labelId
```

Create:

```json
{
  "name": "backend",
  "description": "Backend-related work"
}
```

---

# 24. Attachments API

## Upload

```http
POST /api/v1/issues/:issueId/attachments
```

Request:

```text
multipart/form-data
file = oauth-flow.png
```

Flow:

```text
Frontend
   ↓
Backend
   ↓
Validate file
   ↓
Upload to Cloudinary
   ↓
Store metadata in MongoDB
   ↓
Return attachment
```

### Why?

MongoDB stores metadata while Cloudinary handles the actual file.

---

# 25. Notifications API

```http
GET   /api/v1/notifications
PATCH /api/v1/notifications/:notificationId
POST  /api/v1/notifications/read-all
```

Query:

```text
?page=1
&limit=20
&isRead=false
```

Mark read:

```json
{
  "isRead": true
}
```

`read-all` is an action affecting multiple resources, so POST is appropriate.

---

# 26. Activities API

```http
GET /api/v1/issues/:issueId/activities
GET /api/v1/projects/:projectId/activities
```

Query:

```text
?limit=20
&cursor=...
```

### Why separate endpoints?

The same activity data can be viewed at different scopes:

```text
Issue timeline
Project activity feed
```

---

# 27. Invitations API

```http
POST /api/v1/organizations/:organizationId/invitations
GET  /api/v1/organizations/:organizationId/invitations
POST /api/v1/invitations/:token/accept
```

Accepting an invitation requires validating:

```text
Token
Expiration
Status
User identity
```

---

# 28. Dashboard API

```http
GET /api/v1/projects/:projectId/dashboard
```

Example response:

```json
{
  "data": {
    "summary": {
      "totalIssues": 120,
      "completedIssues": 72,
      "overdueIssues": 8
    },
    "byStatus": {
      "TODO": 20,
      "IN_PROGRESS": 18,
      "IN_REVIEW": 10,
      "DONE": 72
    },
    "byPriority": {
      "LOW": 30,
      "MEDIUM": 50,
      "HIGH": 32,
      "URGENT": 8
    }
  }
}
```

### Why?

Without a dashboard endpoint, the frontend may need many separate API calls. A dedicated endpoint can aggregate the information on the server.

---

# 29. Search API

```http
GET /api/v1/search
```

Query:

```text
?q=oauth
&projectId=...
&type=issue
```

Example:

```json
{
  "data": [
    {
      "type": "issue",
      "id": "...",
      "title": "Implement Google OAuth",
      "key": "ECOM-42"
    }
  ]
}
```

Initially search can be limited to issues. A dedicated search service can later search projects, users, comments, and other resources.

---

# 30. Response Format

Single resource:

```json
{
  "data": {
    "id": "...",
    "name": "..."
  }
}
```

Collection:

```json
{
  "data": [],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 100,
    "totalPages": 5
  }
}
```

### Why?

A consistent response shape makes frontend data handling predictable.

---

# 31. Cursor Pagination

For large feeds:

```json
{
  "data": [],
  "pagination": {
    "nextCursor": "...",
    "hasNextPage": true
  }
}
```

Use cursor pagination for:

```text
Activities
Notifications
Comments
```

and potentially very large issue lists.

Use page/limit where simple pagination is sufficient.

---

# 32. Error Format

All API errors should follow a consistent structure:

```json
{
  "error": {
    "code": "ISSUE_NOT_FOUND",
    "message": "Issue not found",
    "details": null
  }
}
```

Validation example:

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid request",
    "details": {
      "title": "Title is required"
    }
  }
}
```

### Why?

The frontend can reliably use:

```text
error.code
error.message
error.details
```

---

# 33. HTTP Status Codes

| Status | Meaning |
|---|---|
| 200 | Success |
| 201 | Created |
| 204 | Success with no response body |
| 400 | Invalid request |
| 401 | Not authenticated |
| 403 | Authenticated but not authorized |
| 404 | Resource not found |
| 409 | Conflict |
| 422 | Validation/business-rule error |
| 429 | Rate limit exceeded |
| 500 | Unexpected server error |

Important distinction:

```text
401 → Who are you?
403 → I know who you are, but you cannot do this.
```

---

# 34. Validation

Every write endpoint must validate input.

Example:

```text
title       → required
title       → maximum length
type        → allowed enum
priority    → allowed enum
assigneeId  → valid ObjectId
labelIds    → valid ObjectIds
```

### Why backend validation?

Frontend validation is for UX.

Backend validation is required for correctness and security because clients can bypass the UI and call the API directly.

---

# 35. Authorization

Authentication answers:

> Who are you?

Authorization answers:

> Are you allowed to perform this action?

Example:

```text
User
 ↓
Authenticated
 ↓
Organization member
 ↓
Project member
 ↓
Authorized to update issue
```

The backend must enforce permissions even if the frontend hides the action.

---

# 36. Role-Based Authorization

Example organization roles:

```text
OWNER
ADMIN
MEMBER
```

Example project roles:

```text
MANAGER
DEVELOPER
VIEWER
```

Example policy:

```text
OWNER
→ manage organization

ADMIN
→ manage members

MANAGER
→ manage project

DEVELOPER
→ create/update issues

VIEWER
→ read-only
```

The exact policy can evolve as the product is implemented.

---

# 37. Filtering

Example:

```http
GET /api/v1/projects/:projectId/issues?status=TODO&priority=HIGH
```

Allowed filters should be explicitly defined by the API.

### Why?

This keeps the API predictable and prevents arbitrary database queries from being exposed.

---

# 38. Sorting

Example:

```text
GET /api/v1/projects/:projectId/issues
    ?sortBy=createdAt
    &sortOrder=desc
```

Allowed fields should be whitelisted:

```javascript
[
  "createdAt",
  "updatedAt",
  "priority",
  "dueDate",
  "issueNumber"
]
```

### Why?

Never directly trust arbitrary client-provided database field names.

---

# 39. Pagination Limits

Default:

```text
limit = 20
```

Maximum:

```text
limit = 100
```

A request such as:

```text
?limit=10000
```

should be rejected or capped.

### Why?

It protects the database and server from accidental or malicious unbounded queries.

---

# 40. Rate Limiting

Stricter rate limits should apply to:

```text
Login
Register
Refresh token
Password reset
Invitation
```

Redis can later be used for distributed rate limiting.

### Why?

Authentication and account-management endpoints are common abuse targets.

---

# 41. Idempotency

Some operations may need an idempotency key:

```http
Idempotency-Key: abc123
```

This is especially useful for operations where retries could create duplicates.

### Why?

A network timeout can cause a client to retry even though the server already processed the first request.

For the MVP, implement idempotency only where it provides real value.

---

# 42. Controller vs Service

Example:

```text
PATCH /api/v1/issues/:issueId
```

Controller:

```text
Read params
Read body
Call service
Return HTTP response
```

Service:

```text
Check issue
Check permissions
Apply business rules
Update issue
Create activity
Create notification
```

### Why?

Controllers should stay thin.

Business logic belongs in services so it can be reused and tested independently.

---

# 43. Real-Time Boundary

WebSockets should **not replace REST**.

REST handles:

```text
Create issue
Update issue
Delete issue
Get issues
Get comments
```

WebSockets handle:

```text
Issue changed
Comment added
Notification created
Presence
```

Flow:

```text
User A
   ↓
PATCH /issues/123
   ↓
Backend updates MongoDB
   ↓
Publish issue.updated
   ↓
WebSocket
   ↓
User B's browser
   ↓
React Query cache update
```

### Why?

MongoDB remains the source of truth.

WebSocket messages are synchronization events, not permanent storage.

---

# 44. Suggested WebSocket Events

```text
issue.created
issue.updated
issue.deleted

comment.created
comment.updated
comment.deleted

notification.created

project.updated

member.added
member.removed
```

Example:

```json
{
  "event": "issue.updated",
  "data": {
    "issueId": "...",
    "projectId": "...",
    "changes": {
      "status": "IN_PROGRESS"
    }
  }
}
```

---

# 45. Backend Route Organization

A feature-based backend can eventually look like:

```text
src/
├── modules/
│   ├── auth/
│   │   ├── auth.routes.ts
│   │   ├── auth.controller.ts
│   │   ├── auth.service.ts
│   │   └── auth.validation.ts
│   │
│   ├── organizations/
│   ├── teams/
│   ├── workspaces/
│   ├── projects/
│   ├── issues/
│   ├── comments/
│   ├── labels/
│   ├── notifications/
│   └── activities/
│
├── middleware/
├── config/
├── database/
├── utils/
└── app.ts
```

### Why feature-based modules?

Keeping each business domain together scales better than having hundreds of unrelated files in global `controllers`, `services`, and `routes` folders.

---

# 46. Complete API Summary

```text
AUTH
POST   /api/v1/auth/register
POST   /api/v1/auth/login
POST   /api/v1/auth/refresh
POST   /api/v1/auth/logout
GET    /api/v1/auth/me

ORGANIZATIONS
POST   /api/v1/organizations
GET    /api/v1/organizations
GET    /api/v1/organizations/:id
PATCH  /api/v1/organizations/:id
DELETE /api/v1/organizations/:id

TEAMS
POST   /api/v1/organizations/:id/teams
GET    /api/v1/organizations/:id/teams
GET    /api/v1/teams/:id
PATCH  /api/v1/teams/:id
DELETE /api/v1/teams/:id

WORKSPACES
POST   /api/v1/organizations/:id/workspaces
GET    /api/v1/organizations/:id/workspaces
GET    /api/v1/workspaces/:id
PATCH  /api/v1/workspaces/:id
DELETE /api/v1/workspaces/:id

PROJECTS
POST   /api/v1/workspaces/:id/projects
GET    /api/v1/workspaces/:id/projects
GET    /api/v1/projects/:id
PATCH  /api/v1/projects/:id
DELETE /api/v1/projects/:id

ISSUES
POST   /api/v1/projects/:id/issues
GET    /api/v1/projects/:id/issues
GET    /api/v1/issues/:id
PATCH  /api/v1/issues/:id
DELETE /api/v1/issues/:id

COMMENTS
GET    /api/v1/issues/:id/comments
POST   /api/v1/issues/:id/comments
PATCH  /api/v1/comments/:id
DELETE /api/v1/comments/:id

LABELS
POST   /api/v1/projects/:id/labels
GET    /api/v1/projects/:id/labels
PATCH  /api/v1/labels/:id
DELETE /api/v1/labels/:id

ATTACHMENTS
POST   /api/v1/issues/:id/attachments
GET    /api/v1/issues/:id/attachments

NOTIFICATIONS
GET    /api/v1/notifications
PATCH  /api/v1/notifications/:id
POST   /api/v1/notifications/read-all

ACTIVITIES
GET    /api/v1/issues/:id/activities
GET    /api/v1/projects/:id/activities

SEARCH
GET    /api/v1/search

DASHBOARD
GET    /api/v1/projects/:id/dashboard
```

---

# 47. Example: Complete Issue Update Flow

User drags:

```text
ECOM-42
```

from:

```text
TODO
```

to:

```text
IN_PROGRESS
```

Frontend:

```http
PATCH /api/v1/issues/abc123
Authorization: Bearer <token>
Content-Type: application/json
```

Body:

```json
{
  "status": "IN_PROGRESS"
}
```

Backend:

```text
1. Authenticate
        ↓
2. Validate issue ID
        ↓
3. Find issue
        ↓
4. Check project membership
        ↓
5. Validate status
        ↓
6. Update issue
        ↓
7. Create activity
        ↓
8. Publish real-time event
        ↓
9. Return updated issue
```

Response:

```json
{
  "data": {
    "id": "abc123",
    "key": "ECOM-42",
    "status": "IN_PROGRESS"
  }
}
```

This demonstrates the relationship between frontend, API, business logic, database, activity history, and real-time updates.

---

# 48. API Design Principles

The API should be:

```text
Consistent
Predictable
Secure
Versioned
Validated
Resource-oriented
Permission-aware
Pagination-friendly
```

Avoid:

```text
Random endpoint naming
Business logic inside controllers
Trusting frontend permissions
Inconsistent responses
Unbounded queries
```

---

# 49. Next Step

The project design process is now:

```text
Product Requirements
        ↓
User Flows
        ↓
Domain Model
        ↓
Database Design
        ↓
API Design              ← CURRENT
        ↓
System Architecture     ← NEXT
        ↓
Frontend Architecture
        ↓
UI Design
        ↓
Implementation
```

The next document should be:

```text
system-architecture.md
```

It will connect:

```text
React
   ↓
API
   ↓
Node + Express
   ↓
Services
   ↓
MongoDB
Redis
Cloudinary
WebSockets
```

and explain where each component runs, how authentication flows through the system, how caching and real-time updates work, how files are uploaded, and how Docker/deployment fit into the architecture.
