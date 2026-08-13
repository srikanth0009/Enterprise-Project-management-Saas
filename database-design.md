# Enterprise Project Management SaaS — Database Design

This document translates the domain model into a practical MongoDB database design.

The goal is not only to define what data is stored, but also to explain **why each database decision is made**.

---

# 1. Database Technology

## Database: MongoDB

MongoDB is selected because this application contains document-oriented entities with flexible fields, while still requiring strong relationships and indexing.

### Why MongoDB?

- Flexible schema as the product evolves.
- Natural fit for entities such as issues, activities, notifications, and settings.
- Good integration with Node.js.
- Supports indexing, aggregation, transactions, and horizontal scaling.
- Lets us iterate quickly during development.

---

# 2. Collections

The planned collections are:

```text
users
organizations
organization_members
teams
team_members
workspaces
workspace_members
projects
project_members
issues
labels
comments
attachments
activities
notifications
invitations
workflows
```

We will not implement all of them immediately. The MVP will use only the collections needed for the core workflow.

---

# 3. General Document Conventions

Most documents should contain:

```javascript
{
  _id: ObjectId("..."),
  createdAt: ISODate("..."),
  updatedAt: ISODate("...")
}
```

### Why?

These fields give every document:

- A stable identity.
- Creation tracking.
- Modification tracking.
- Sorting capability.
- Debugging/audit information.

---

# 4. References vs Embedded Data

MongoDB allows us to either embed data or reference another document.

## Use References When

- The related data can grow significantly.
- The data has its own lifecycle.
- The data is independently queried.
- The same entity is shared by multiple records.

Examples:

```text
Project → Issues
Issue → Comments
Issue → Attachments
User → Organizations
User → Teams
User → Projects
```

## Use Embedding When

- Data is small.
- Data belongs strongly to its parent.
- Data is usually read together.
- Data has limited growth.

Example:

```javascript
organization.settings
```

### Why?

Embedding everything creates very large documents and difficult updates.

Referencing absolutely everything can create excessive queries.

Therefore we use a balanced design.

---

# 5. Users

Collection:

```text
users
```

Example:

```javascript
{
  _id: ObjectId("..."),
  name: "Srikanth",
  email: "srikanth@example.com",
  passwordHash: "...",
  avatarUrl: "...",

  authProviders: [
    {
      provider: "google",
      providerUserId: "..."
    }
  ],

  status: "ACTIVE",

  createdAt: ISODate("..."),
  updatedAt: ISODate("...")
}
```

### Why these fields?

- `name` — display name.
- `email` — login and contact identity.
- `passwordHash` — secure local authentication.
- `avatarUrl` — profile image.
- `authProviders` — supports OAuth identities.
- `status` — allows account deactivation without deleting the user.

### Important Security Decision

Never store plain-text passwords.

Only secure password hashes should be stored.

### Indexes

```javascript
{ email: 1 } // unique
```

### Why?

Email is frequently used for login, registration, invitations, and user lookup.

---

# 6. Organizations

Collection:

```text
organizations
```

Example:

```javascript
{
  _id: ObjectId("..."),

  name: "Acme Technologies",
  slug: "acme-technologies",

  ownerId: ObjectId("..."),

  logoUrl: "...",

  settings: {
    timezone: "Asia/Kolkata",
    defaultIssuePriority: "MEDIUM"
  },

  createdAt: ISODate("..."),
  updatedAt: ISODate("...")
}
```

### Why `ownerId`?

The organization needs an explicit primary owner.

Membership still determines whether the user belongs to the organization, but `ownerId` gives us a direct ownership reference.

### Indexes

```javascript
{ slug: 1 } // unique
{ ownerId: 1 }
```

### Why?

`slug` supports human-readable URLs and fast organization lookup.

---

# 7. Organization Members

Collection:

```text
organization_members
```

Example:

```javascript
{
  _id: ObjectId("..."),

  organizationId: ObjectId("..."),
  userId: ObjectId("..."),

  role: "OWNER",
  status: "ACTIVE",

  joinedAt: ISODate("..."),

  createdAt: ISODate("..."),
  updatedAt: ISODate("...")
}
```

### Why a separate collection?

This is a many-to-many relationship:

```text
User ←→ Organization
```

The relationship itself has data:

```text
role
status
joinedAt
```

Therefore it deserves its own document.

### Indexes

```javascript
{
  organizationId: 1,
  userId: 1
} // unique

{ userId: 1 }
```

### Why?

The compound unique index prevents duplicate memberships.

The `userId` index helps find all organizations belonging to a user.

---

# 8. Teams

Collection:

```text
teams
```

Example:

```javascript
{
  _id: ObjectId("..."),

  organizationId: ObjectId("..."),

  name: "Engineering",
  description: "Product engineering team",

  createdBy: ObjectId("..."),

  createdAt: ISODate("..."),
  updatedAt: ISODate("...")
}
```

### Why `organizationId`?

A team belongs to exactly one organization.

It also makes tenant-level queries simple.

### Index

```javascript
{ organizationId: 1 }
```

---

# 9. Team Members

Collection:

```text
team_members
```

Example:

```javascript
{
  _id: ObjectId("..."),

  teamId: ObjectId("..."),
  userId: ObjectId("..."),

  role: "MEMBER",

  joinedAt: ISODate("..."),

  createdAt: ISODate("..."),
  updatedAt: ISODate("...")
}
```

### Why?

Users and teams have a many-to-many relationship.

```text
User ←→ Team
```

### Indexes

```javascript
{
  teamId: 1,
  userId: 1
} // unique

{ userId: 1 }
```

---

# 10. Workspaces

Collection:

```text
workspaces
```

Example:

```javascript
{
  _id: ObjectId("..."),

  organizationId: ObjectId("..."),

  name: "Product Development",
  slug: "product-development",
  description: "Product engineering workspace",

  createdBy: ObjectId("..."),

  createdAt: ISODate("..."),
  updatedAt: ISODate("...")
}
```

### Why?

A workspace groups related projects without putting every project directly under an organization.

### Indexes

```javascript
{ organizationId: 1 }

{
  organizationId: 1,
  slug: 1
} // unique
```

### Why compound uniqueness?

Different organizations may both have a workspace called `Product Development`, but the same organization should not have duplicate workspace slugs.

---

# 11. Workspace Members

Collection:

```text
workspace_members
```

Example:

```javascript
{
  _id: ObjectId("..."),

  workspaceId: ObjectId("..."),
  userId: ObjectId("..."),

  role: "MEMBER",

  joinedAt: ISODate("..."),

  createdAt: ISODate("..."),
  updatedAt: ISODate("...")
}
```

### Why?

Not every organization member should automatically have access to every workspace.

This gives us workspace-level access control.

### Index

```javascript
{
  workspaceId: 1,
  userId: 1
} // unique
```

---

# 12. Projects

Collection:

```text
projects
```

Example:

```javascript
{
  _id: ObjectId("..."),

  organizationId: ObjectId("..."),
  workspaceId: ObjectId("..."),
  teamId: ObjectId("..."),

  name: "E-Commerce Platform",
  key: "ECOM",
  slug: "e-commerce-platform",

  description: "Build the new e-commerce platform",

  status: "ACTIVE",

  startDate: ISODate("..."),
  dueDate: ISODate("..."),

  createdBy: ObjectId("..."),

  createdAt: ISODate("..."),
  updatedAt: ISODate("...")
}
```

### Why store both `organizationId` and `workspaceId`?

The organization can technically be derived through the workspace.

However, storing `organizationId` directly helps with:

- Tenant filtering.
- Authorization.
- Organization-level queries.
- Indexing.

This is intentional denormalization.

The backend must ensure the project and workspace belong to the same organization.

### Project Key

Example:

```text
ECOM
```

Issues can then be displayed as:

```text
ECOM-1
ECOM-42
```

### Unique Index

```javascript
{
  organizationId: 1,
  key: 1
} // unique
```

### Why?

Two organizations can both use `ECOM`, but one organization should not have two projects with the same key.

### Other Indexes

```javascript
{ organizationId: 1 }
{ workspaceId: 1 }
{ teamId: 1 }
{ createdBy: 1 }
```

---

# 13. Project Members

Collection:

```text
project_members
```

Example:

```javascript
{
  _id: ObjectId("..."),

  projectId: ObjectId("..."),
  userId: ObjectId("..."),

  role: "DEVELOPER",

  joinedAt: ISODate("..."),

  createdAt: ISODate("..."),
  updatedAt: ISODate("...")
}
```

### Why?

A project can contain only a subset of workspace members.

The relationship also stores the project-specific role.

### Index

```javascript
{
  projectId: 1,
  userId: 1
} // unique
```

---

# 14. Issues

Collection:

```text
issues
```

This is one of the most important collections.

Example:

```javascript
{
  _id: ObjectId("..."),

  organizationId: ObjectId("..."),
  projectId: ObjectId("..."),
  teamId: ObjectId("..."),

  issueNumber: 42,

  title: "Implement Google OAuth",
  description: "Add Google authentication flow",

  type: "FEATURE",
  status: "TODO",
  priority: "HIGH",

  reporterId: ObjectId("..."),
  assigneeId: ObjectId("..."),

  labelIds: [
    ObjectId("..."),
    ObjectId("...")
  ],

  parentIssueId: null,

  dueDate: ISODate("..."),

  createdAt: ISODate("..."),
  updatedAt: ISODate("...")
}
```

### Why separate issues from projects?

We should **not embed all issues inside a project document**.

A project could eventually contain:

```text
10,000+
50,000+
100,000+
```

issues.

Embedding them would create:

- Very large documents.
- Expensive updates.
- Difficult pagination.
- Poor scalability.
- Potential MongoDB document-size problems.

Therefore:

```text
Project
   ↓
Issues collection
```

is the better design.

---

# 15. Issue Number

A project has a key:

```text
ECOM
```

An issue has a number:

```text
42
```

Together:

```text
ECOM-42
```

Store:

```javascript
issueNumber: 42
```

and generate the display identifier from:

```text
project.key + "-" + issueNumber
```

### Why?

Separating the project key and number makes indexing and querying easier.

### Unique Index

```javascript
{
  projectId: 1,
  issueNumber: 1
} // unique
```

### Why?

`42` only needs to be unique inside a project.

This is valid:

```text
ECOM-42
CRM-42
MOB-42
```

---

# 16. Issue Indexes

Recommended indexes:

```javascript
{ projectId: 1, status: 1 }
{ projectId: 1, assigneeId: 1 }
{ projectId: 1, priority: 1 }
{ projectId: 1, type: 1 }
{ projectId: 1, createdAt: -1 }
{ projectId: 1, issueNumber: 1 }
{ organizationId: 1 }
{ assigneeId: 1 }
```

### Why?

These support common queries:

```text
Get project issues
Get TODO issues
Get issues assigned to a user
Get HIGH priority issues
Get bugs
Get newest issues
Get issue by project + number
```

Do not create every possible index blindly. Indexes improve reads but consume storage and make writes more expensive. We should keep indexes based on real query patterns.

---

# 17. Issue Search

Initially, search can use MongoDB text search over:

```text
title
description
```

Conceptually:

```javascript
{
  title: "text",
  description: "text"
}
```

### Why?

It provides a simple MVP search without introducing another infrastructure component.

For a much larger production system, a dedicated search engine such as OpenSearch/Elasticsearch could be introduced later.

We should **not add that complexity to the MVP**.

---

# 18. Labels

Collection:

```text
labels
```

Example:

```javascript
{
  _id: ObjectId("..."),

  organizationId: ObjectId("..."),
  projectId: ObjectId("..."),

  name: "backend",
  description: "Backend-related work",

  createdBy: ObjectId("..."),

  createdAt: ISODate("..."),
  updatedAt: ISODate("...")
}
```

### Why separate labels?

Labels are reusable.

Instead of storing arbitrary strings on every issue, label documents provide:

- Consistent naming.
- Reuse.
- Project-level management.
- Future metadata.

Issues reference labels using:

```javascript
labelIds: [
  ObjectId("...")
]
```

### Index

```javascript
{
  projectId: 1,
  name: 1
} // unique
```

### Why?

A project should not have two labels named `backend`.

---

# 19. Comments

Collection:

```text
comments
```

Example:

```javascript
{
  _id: ObjectId("..."),

  issueId: ObjectId("..."),
  authorId: ObjectId("..."),

  content: "Please check the payment API.",

  mentionUserIds: [
    ObjectId("...")
  ],

  createdAt: ISODate("..."),
  updatedAt: ISODate("...")
}
```

### Why separate comments?

An issue could eventually contain hundreds or thousands of comments.

Separate documents provide:

- Pagination.
- Independent updates.
- Efficient loading.
- Better scalability.

### Index

```javascript
{
  issueId: 1,
  createdAt: 1
}
```

### Why?

The common query is:

```text
Get comments for issue X ordered by time.
```

---

# 20. Attachments

Collection:

```text
attachments
```

Example:

```javascript
{
  _id: ObjectId("..."),

  issueId: ObjectId("..."),
  uploadedBy: ObjectId("..."),

  fileName: "oauth-flow.png",
  fileUrl: "https://...",
  storageProvider: "cloudinary",

  fileType: "image/png",
  fileSize: 245760,

  createdAt: ISODate("...")
}
```

### Why separate attachments?

Files can be numerous and large.

The database should store **metadata**, not the actual binary file.

The actual file is stored in external object/file storage.

### Index

```javascript
{
  issueId: 1,
  createdAt: -1
}
```

---

# 21. Activities

Collection:

```text
activities
```

Example:

```javascript
{
  _id: ObjectId("..."),

  organizationId: ObjectId("..."),
  projectId: ObjectId("..."),
  issueId: ObjectId("..."),

  actorId: ObjectId("..."),

  action: "STATUS_CHANGED",

  metadata: {
    oldStatus: "TODO",
    newStatus: "IN_PROGRESS"
  },

  createdAt: ISODate("...")
}
```

### Why separate activities?

Activity history can grow continuously.

An issue could eventually contain thousands of activity events.

Separate documents provide:

- Pagination.
- Efficient timelines.
- Independent storage.
- Better scalability.

### Indexes

```javascript
{ issueId: 1, createdAt: -1 }
{ projectId: 1, createdAt: -1 }
```

---

# 22. Notifications

Collection:

```text
notifications
```

Example:

```javascript
{
  _id: ObjectId("..."),

  recipientId: ObjectId("..."),

  type: "ISSUE_ASSIGNED",

  message: "You were assigned ECOM-42",

  entityType: "ISSUE",
  entityId: ObjectId("..."),

  isRead: false,
  readAt: null,

  createdAt: ISODate("...")
}
```

### Why separate notifications?

Notifications belong to users and have their own lifecycle:

```text
UNREAD
   ↓
READ
```

### Index

```javascript
{
  recipientId: 1,
  isRead: 1,
  createdAt: -1
}
```

### Why?

The notification center frequently needs:

```text
Get unread notifications for current user
Get newest notifications
```

---

# 23. Invitations

Collection:

```text
invitations
```

Example:

```javascript
{
  _id: ObjectId("..."),

  organizationId: ObjectId("..."),

  email: "rahul@example.com",

  invitedBy: ObjectId("..."),

  role: "MEMBER",

  tokenHash: "...",

  status: "PENDING",

  expiresAt: ISODate("..."),

  createdAt: ISODate("...")
}
```

### Why separate invitations?

An invitation is temporary business data with its own lifecycle:

```text
PENDING
 ↓
ACCEPTED

or

PENDING
 ↓
EXPIRED

or

PENDING
 ↓
REVOKED
```

It should not be embedded into an organization or user document.

### Indexes

```javascript
{ organizationId: 1, email: 1 }
{ expiresAt: 1 }
{ organizationId: 1, email: 1, status: 1 }
```

---

# 24. Workflows

Collection:

```text
workflows
```

This is an advanced feature.

Example:

```javascript
{
  _id: ObjectId("..."),

  projectId: ObjectId("..."),

  name: "Engineering Workflow",

  statuses: [
    {
      key: "TODO",
      name: "Todo",
      position: 1
    },
    {
      key: "IN_PROGRESS",
      name: "In Progress",
      position: 2
    },
    {
      key: "IN_REVIEW",
      name: "In Review",
      position: 3
    },
    {
      key: "DONE",
      name: "Done",
      position: 4
    }
  ],

  createdAt: ISODate("..."),
  updatedAt: ISODate("...")
}
```

### Why postpone this?

The MVP can use fixed statuses:

```text
TODO
IN_PROGRESS
IN_REVIEW
DONE
```

Custom workflows add significant complexity, so they should come after the core issue system works.

---

# 25. Multi-Tenant Data Isolation

This is a multi-tenant SaaS.

Example:

```text
Organization A
├── Projects
└── Issues

Organization B
├── Projects
└── Issues
```

Users from Organization A must never access Organization B's data.

Important tenant-owned collections should carry:

```text
organizationId
```

where useful.

Examples:

```text
projects.organizationId
issues.organizationId
labels.organizationId
activities.organizationId
```

### Why?

It makes tenant filtering explicit and supports authorization and indexing.

Every backend query involving tenant-owned data must be scoped to the authenticated user's organization.

---

# 26. Soft Delete

For important business entities, we should consider:

```javascript
deletedAt: null
```

Example:

```javascript
{
  _id: ObjectId("..."),
  name: "Old Project",
  deletedAt: ISODate("...")
}
```

### Why?

Soft deletion allows:

- Recovery.
- Auditability.
- Safer accidental deletion.
- Preservation of historical references.

For the MVP, this can be used for important entities such as projects and issues.

We do not need soft-delete behavior on every collection immediately.

---

# 27. Pagination

Large collections should never be returned completely.

## Initial Pagination

Use:

```text
page
limit
```

Example:

```text
?page=2&limit=20
```

## Cursor Pagination

For continuously growing feeds, cursor pagination is preferable:

```text
?cursor=<lastSeenId>&limit=20
```

### Why?

Offset pagination becomes less efficient at very large offsets.

Cursor pagination works better for:

```text
Activities
Notifications
Comments
```

and potentially very large issue lists.

### MVP Decision

Use page/limit initially where simple pagination is sufficient.

Introduce cursor pagination for large feeds when needed.

---

# 28. Kanban Board Query

The board needs issues grouped by status.

Common query:

```text
Find issues for project ECOM
filtered by status
```

Recommended index:

```javascript
{
  projectId: 1,
  status: 1,
  createdAt: -1
}
```

### Why?

The Kanban board repeatedly queries issues by project and status.

Example:

```text
TODO
├── ECOM-1
└── ECOM-4

IN_PROGRESS
├── ECOM-2
└── ECOM-5

DONE
└── ECOM-3
```

---

# 29. Assignee Query

A common dashboard query is:

```text
Show all issues assigned to Srikanth.
```

Recommended index:

```javascript
{
  assigneeId: 1,
  status: 1
}
```

### Why?

This supports workload and personal task views.

---

# 30. Dashboard Analytics

The dashboard may need:

```text
Total issues
Issues by status
Issues by priority
Issues by assignee
Overdue issues
Recently completed issues
```

Initially, calculate these using MongoDB aggregation.

### Why not store counters immediately?

If we store:

```text
project.completedIssues = 138
```

we must update that counter every time an issue changes.

That creates consistency problems.

Aggregation keeps the issue collection as the source of truth.

Caching can be introduced later if performance requires it.

---

# 31. Controlled Denormalization

Some duplication is intentional.

For example:

```text
issues.organizationId
```

can exist even though:

```text
Issue → Project → Workspace → Organization
```

already provides a path to the organization.

### Why?

`organizationId` is frequently needed for:

- Authorization.
- Tenant filtering.
- Indexing.
- Reporting.

The application must keep duplicated ownership fields consistent.

---

# 32. Multi-Document Operations

Some user actions affect multiple collections.

Example:

```text
Assign Issue
    ↓
Update issue.assigneeId
    ↓
Create activity
    ↓
Create notification
```

### Why transactions may be needed

We do not want:

```text
Issue says Srikanth is assigned
BUT
notification/activity was not created
```

MongoDB transactions can be used for important multi-document operations where atomicity matters.

Not every operation requires a transaction.

---

# 33. Deletion Strategy

Project-management systems depend heavily on historical information.

Recommended approach:

### User

Prefer:

```text
status = INACTIVE
```

instead of immediate deletion.

### Project

Prefer archive/soft delete.

### Issue

Prefer soft delete for important historical records.

### Comment

Depending on product policy:

```text
"This comment was deleted."
```

can remain instead of removing the record completely.

### Why?

Aggressive physical deletion can destroy audit history and break historical references.

---

# 34. Database Security

The backend must:

- Validate all input.
- Authenticate protected requests.
- Authorize every protected operation.
- Scope tenant-owned queries by organization.
- Never expose password hashes.
- Never trust IDs supplied by the frontend.
- Validate ObjectIds.
- Protect file uploads.
- Store credentials in environment variables.
- Restrict database access to the backend.

---

# 35. MVP Collections

We should not implement every collection on day one.

## Phase 1 — Core

```text
users
organizations
organization_members
teams
team_members
workspaces
workspace_members
projects
project_members
issues
```

## Phase 2 — Collaboration

```text
labels
comments
activities
```

## Phase 3 — Supporting Features

```text
attachments
notifications
invitations
```

## Phase 4 — Advanced Workflow

```text
workflows
```

### Why?

This keeps the implementation manageable and lets us get the core project-management workflow working first.

---

# 36. Final Database Relationship

```text
                        ┌─────────────┐
                        │    USERS    │
                        └──────┬──────┘
                               │
              ┌────────────────┼─────────────────┐
              │                │                 │
              ▼                ▼                 ▼
     organization_members  team_members   project_members
              │                │                 │
              ▼                ▼                 ▼
       ORGANIZATIONS          TEAMS            PROJECTS
              │                                  │
              │                                  │
              ▼                                  ▼
         WORKSPACES ──────────────────────────► ISSUES
                                                   │
                          ┌────────────────────────┼─────────────────────┐
                          │                        │                     │
                          ▼                        ▼                     ▼
                       COMMENTS              ATTACHMENTS            ACTIVITIES

USERS ───────────────────────────────────────────────────────────────► NOTIFICATIONS
```

---

# 37. Key Database Decisions

| Decision | Choice | Reason |
|---|---|---|
| Database | MongoDB | Flexible document model and Node.js integration |
| Memberships | Separate collections | Many-to-many relationships with role/status |
| Issues | Separate collection | Can grow very large |
| Comments | Separate collection | Independent growth and pagination |
| Activities | Separate collection | Potentially large history |
| Attachments | Metadata in DB, files externally | Avoid storing large binaries |
| Labels | Separate reusable documents | Consistency and management |
| Project key | Compound unique index | Unique within organization |
| Issue number | Compound unique index | Unique within project |
| Tenant isolation | `organizationId` | Explicit multi-tenant filtering |
| Pagination | Page/limit initially | Simple MVP |
| Large feeds | Cursor pagination | Better scalability |
| Deletes | Archive/soft delete | Preserve history |
| Dashboard metrics | Aggregation initially | Avoid inconsistent counters |
| Real-time state | Database is source of truth | WebSocket synchronizes clients |

---

# 38. Most Important Design Principle

The database should follow:

```text
Normalize where consistency matters
+
Denormalize strategically where common queries benefit
```

Avoid both extremes:

```text
Everything embedded
        ❌

Everything referenced
        ❌

Balanced document design
        ✅
```

---

# 39. What Comes Next

The project design process is now:

```text
Product Requirements
        ↓
User Flows
        ↓
Domain Model
        ↓
Database Design          ← CURRENT
        ↓
API Design               ← NEXT
        ↓
System Architecture
        ↓
Frontend Architecture
        ↓
UI Design
        ↓
Implementation
```

The next document should be:

```text
api-design.md
```

It will translate this domain/database model into REST APIs and define:

- Authentication endpoints.
- Organization endpoints.
- Team endpoints.
- Workspace endpoints.
- Project endpoints.
- Issue endpoints.
- Comment endpoints.
- Label endpoints.
- Notification endpoints.
- Activity endpoints.
- HTTP methods.
- Request/response structures.
- Status codes.
- Authentication requirements.
- Authorization rules.
- Pagination.
- Filtering.
- Sorting.
- Error handling.
- API versioning.
