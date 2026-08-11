# Enterprise Project Management SaaS — User Flows

This document defines the primary user journeys and workflows of the Enterprise Project Management SaaS.

The purpose is to describe **what users do and how the product behaves**, without defining technical implementation details such as React components, APIs, database schemas, or infrastructure.

---

## 1. Registration & Login

### 1.1 New User Registration

```text
User opens application
        ↓
Clicks Register
        ↓
Enters name, email, password
        ↓
Validation
        ↓
Create user account
        ↓
Authenticate user
        ↓
Continue to application
```

The system should validate required fields, email format, password requirements, and duplicate accounts.

### 1.2 User Login

```text
User opens application
        ↓
Login
        ↓
Enter email + password
        ↓
Validate credentials
        ↓
Authenticate
        ↓
Open dashboard
```

Invalid credentials, disabled accounts, and validation failures should be handled gracefully.

---

## 2. Organization Creation

A newly registered user can create an organization.

```text
New User
    ↓
Create Organization
    ↓
Enter Organization Name
    ↓
Create
    ↓
Organization Created
    ↓
User becomes Organization Owner
    ↓
Organization Dashboard
```

Example:

```text
Organization: Acme Technologies
Owner: Srikanth
```

---

## 3. Invite Members

Authorized organization users can invite members.

```text
Organization
      ↓
Members
      ↓
Invite Member
      ↓
Enter Email
      ↓
Select Role
      ↓
Send Invitation
      ↓
User Receives Invitation
      ↓
Accept Invitation
      ↓
User Joins Organization
```

Invitation states:

```text
PENDING → ACCEPTED
PENDING → EXPIRED
```

The system should prevent duplicate memberships and handle revoked or expired invitations.

---

## 4. Accept Organization Invitation

```text
User receives invitation
        ↓
Opens invitation
        ↓
Accept
        ↓
Login/Register if required
        ↓
Invitation validated
        ↓
User added to organization
```

Possible errors:

- Invitation expired.
- Invitation already accepted.
- Invitation revoked.
- User is already a member.

---

## 5. Team Creation

Authorized users can create teams inside an organization.

```text
Organization
      ↓
Teams
      ↓
Create Team
      ↓
Enter Name + Description
      ↓
Create
      ↓
Team Created
      ↓
Add Members
```

Example:

```text
Engineering
├── Srikanth
├── Rahul
└── Priya
```

A user can belong to multiple teams, and a team can participate in multiple projects.

---

## 6. Workspace Creation

A workspace groups related projects and work.

```text
Organization
      ↓
Workspaces
      ↓
Create Workspace
      ↓
Enter Name + Description
      ↓
Create
      ↓
Workspace Created
```

Example:

```text
Acme Technologies
├── Product Development
├── Marketing
└── Internal Operations
```

---

## 7. Project Creation

A project represents a specific product, initiative, or goal.

```text
Workspace
      ↓
Create Project
      ↓
Project Name
      ↓
Description
      ↓
Project Key
      ↓
Select Team
      ↓
Add Members
      ↓
Create Project
      ↓
Project Dashboard
```

Example:

```text
Project: E-Commerce Platform
Key: ECOM
Team: Engineering
```

Project areas:

```text
E-Commerce Platform
├── Board
├── Issues
├── Activity
├── Members
└── Settings
```

Project lifecycle:

```text
PLANNING → ACTIVE → COMPLETED → ARCHIVED
```

---

## 8. Add Project Members

```text
Project
    ↓
Members
    ↓
Add Member
    ↓
Select User
    ↓
Add
    ↓
User becomes Project Member
```

A user does not need access to every project in an organization.

---

## 9. Issue Creation

Issues are the primary units of work.

```text
Project
      ↓
Create Issue
      ↓
Title
      ↓
Description
      ↓
Issue Type
      ↓
Priority
      ↓
Assignee
      ↓
Labels
      ↓
Due Date
      ↓
Create
      ↓
Issue Created
      ↓
Default Status = TODO
```

Example:

```text
ECOM-42

Title: Implement Google OAuth
Type: Feature
Priority: HIGH
Assignee: Srikanth
Status: TODO
Labels: authentication, backend
```

---

## 10. Issue Lifecycle

The basic workflow is:

```text
TODO
  ↓
IN PROGRESS
  ↓
IN REVIEW
  ↓
DONE
```

Detailed flow:

```text
Project Manager creates issue
        ↓
Issue = TODO
        ↓
Developer starts work
        ↓
IN PROGRESS
        ↓
Developer completes implementation
        ↓
IN REVIEW
        ↓
Reviewer/QA checks work
        ↓
       ┌───────────────┐
       │               │
     Passed          Failed
       │               │
       ↓               ↓
      DONE       IN PROGRESS
```

Every significant status change should create an activity entry.

---

## 11. Assign / Reassign Issue

```text
Issue
  ↓
Assignee
  ↓
Select User
  ↓
Save
  ↓
Issue Assigned
  ↓
Assignee receives notification
```

When an issue is reassigned, the change should be recorded in activity history and the new assignee notified.

---

## 12. Update Issue

Authorized users can update:

- Title.
- Description.
- Type.
- Priority.
- Status.
- Assignee.
- Labels.
- Due date.

Example:

```text
Priority:
MEDIUM → HIGH
```

Important changes should be recorded in activity history.

---

## 13. Parent Issue & Subtask Flow

Large work can be broken into smaller subtasks.

```text
ECOM-50
Implement Checkout
        │
        ├── ECOM-51 Create checkout UI
        ├── ECOM-52 Create checkout API
        ├── ECOM-53 Integrate payment
        └── ECOM-54 Write checkout tests
```

User flow:

```text
Open Parent Issue
        ↓
Create Subtask
        ↓
Enter Details
        ↓
Create
        ↓
Subtask appears under Parent Issue
```

---

## 14. Kanban Board Flow

Issues are visually grouped by status.

```text
┌──────────────┐
│ TODO         │
├──────────────┤
│ ECOM-1       │
│ ECOM-2       │
└──────────────┘

┌──────────────┐
│ IN PROGRESS  │
├──────────────┤
│ ECOM-3       │
│ ECOM-4       │
└──────────────┘

┌──────────────┐
│ IN REVIEW    │
├──────────────┤
│ ECOM-5       │
└──────────────┘

┌──────────────┐
│ DONE         │
├──────────────┤
│ ECOM-6       │
└──────────────┘
```

### Drag & Drop

```text
User drags issue
        ↓
Issue moves to another column
        ↓
New status determined
        ↓
Issue updated
        ↓
Activity recorded
        ↓
Other users receive real-time update
```

---

## 15. Custom Workflow Flow

Projects may eventually define custom statuses.

Example:

```text
BACKLOG
   ↓
TODO
   ↓
DEVELOPMENT
   ↓
CODE REVIEW
   ↓
QA
   ↓
DONE
```

Authorized users may eventually:

- Create statuses.
- Rename statuses.
- Reorder statuses.
- Configure allowed transitions.
- Configure transition permissions.

Custom workflows are an advanced feature.

---

## 16. Comment Flow

Users can discuss an issue through comments.

```text
Open Issue
    ↓
Write Comment
    ↓
Submit
    ↓
Comment Appears
```

Example:

```text
ECOM-42

Rahul:
Can you also handle users who don't
have an existing account?

Srikanth:
Yes, I'll create the account automatically.
```

Comments should show author, content, and timestamp.

---

## 17. Mention Flow

Users can mention teammates inside comments.

```text
Write Comment
      ↓
Type @
      ↓
Select Teammate
      ↓
Submit
      ↓
Mention Stored
      ↓
Mentioned User Receives Notification
```

Example:

```text
@Srikanth please check the payment API.
```

---

## 18. Attachment Flow

Users can attach files to an issue.

```text
Open Issue
    ↓
Attach File
    ↓
Select File
    ↓
Upload
    ↓
File Stored
    ↓
Attachment Associated With Issue
    ↓
Attachment Appears On Issue
```

Examples:

```text
oauth-flow.png
error-screenshot.png
requirements.pdf
api-response.json
```

Users with appropriate permissions can remove attachments.

---

## 19. Notification Flow

Notifications are generated by relevant events.

Example:

```text
Project Manager assigns issue
        ↓
System records assignment
        ↓
Notification generated
        ↓
Assignee sees notification
```

Example notification:

```text
You were assigned "Implement Google OAuth"
```

Other notification events:

- Issue assignment.
- Mention.
- Comment.
- Status change.
- Project membership.
- Team membership.
- Important project updates.

---

## 20. Notification Center Flow

```text
User opens notification center
        ↓
View notifications
        ↓
Select notification
        ↓
Open related issue/project
        ↓
Notification marked as read
```

Users should be able to:

- Mark one notification as read.
- Mark all notifications as read.

---

## 21. Search Flow

```text
User enters search term
        ↓
System searches accessible issues
        ↓
Matching results returned
        ↓
User selects result
        ↓
Issue opens
```

Example:

```text
Search: payment
```

Results might include:

```text
ECOM-5  Payment gateway integration
ECOM-12 Fix payment failure
ECOM-18 Payment webhook
ECOM-23 Payment UI
```

---

## 22. Filtering Flow

```text
Open Project Issues
        ↓
Select Filters
        ↓
Status = IN PROGRESS
Priority = HIGH
Assignee = Srikanth
Label = backend
        ↓
Apply
        ↓
Filtered Issues
```

Filters can be combined.

---

## 23. Pagination Flow

When there are many issues:

```text
Request issue list
        ↓
Return limited number of issues
        ↓
User moves to next page
        ↓
Load next set
```

Example:

```text
Page 1 → Issues 1-20
Page 2 → Issues 21-40
Page 3 → Issues 41-60
```

---

## 24. Activity History Flow

Activity history records important actions.

```text
User performs action
        ↓
Action succeeds
        ↓
Activity event recorded
        ↓
Activity appears in timeline
```

Example:

```text
Rahul created this issue.

Srikanth was assigned.

Priority changed:
MEDIUM → HIGH

Status changed:
TODO → IN PROGRESS

Srikanth added attachment.

Status changed:
IN PROGRESS → DONE
```

Activity should include:

- Actor.
- Action.
- Target.
- Timestamp.
- Relevant old/new values where applicable.

---

## 25. Real-Time Update Flow

Multiple users can work on the same project simultaneously.

```text
User A
  ↓
Moves ECOM-42
TODO → IN PROGRESS
  ↓
Backend processes update
  ↓
Real-time event generated
  ↓
Connected users receive event
  ↓
User B's board updates automatically
```

Users should not need to refresh the browser.

Real-time events can eventually cover:

- Issue updates.
- Status changes.
- Assignments.
- Comments.
- Notifications.
- Project updates.

---

## 26. Project Completion Flow

```text
Project ACTIVE
      ↓
Issues being completed
      ↓
Remaining work reviewed
      ↓
Work completed
      ↓
Project Manager marks project COMPLETED
      ↓
Project status = COMPLETED
      ↓
Project can eventually be ARCHIVED
```

---

## 27. Complete End-to-End Example

```text
Srikanth registers
        ↓
Creates "Acme Technologies"
        ↓
Becomes Organization Owner
        ↓
Invites Rahul, Priya and Kiran
        ↓
Creates Engineering Team
        ↓
Creates "Product Development" Workspace
        ↓
Creates "E-Commerce Platform" Project
        ↓
Adds Engineering members
        ↓
Creates ECOM-42
"Implement Google OAuth"
        ↓
Assigns issue to Srikanth
        ↓
Srikanth receives notification
        ↓
Issue = TODO
        ↓
Srikanth starts work
        ↓
Issue = IN PROGRESS
        ↓
Adds comment
        ↓
Uploads OAuth flow image
        ↓
Completes implementation
        ↓
Issue = IN REVIEW
        ↓
QA reviews
        ↓
QA finds an issue
        ↓
Issue = IN PROGRESS
        ↓
Srikanth fixes it
        ↓
Issue = IN REVIEW
        ↓
QA approves
        ↓
Issue = DONE
        ↓
Activity history contains the complete lifecycle
        ↓
Project dashboard updates
```

---

## 28. Permission Check in User Flows

Every protected action should conceptually follow:

```text
User Action
     ↓
Is User Authenticated?
     ↓
    YES
     ↓
Does User Have Access?
     ↓
    YES
     ↓
Does User Have Permission?
     ↓
 ┌───┴────┐
 YES      NO
 ↓         ↓
Execute   403 Forbidden
Action
```

Frontend restrictions improve UX, but the backend must enforce authorization.

---

## 29. Primary User Journey

The complete product journey is:

```text
REGISTER / LOGIN
       ↓
ORGANIZATION
       ↓
INVITE MEMBERS
       ↓
CREATE TEAMS
       ↓
CREATE WORKSPACE
       ↓
CREATE PROJECT
       ↓
ADD PROJECT MEMBERS
       ↓
CREATE ISSUES
       ↓
ASSIGN WORK
       ↓
KANBAN BOARD
       ↓
TODO
       ↓
IN PROGRESS
       ↓
IN REVIEW
       ↓
DONE
       ↓
PROJECT ANALYTICS
       ↓
PROJECT COMPLETED
       ↓
ARCHIVE
```

Supporting flows run throughout the lifecycle:

```text
Comments
Mentions
Attachments
Notifications
Activity History
Search
Filtering
Permissions
Real-Time Updates
```

---

## 30. Flow Design Principle

These flows describe intended **product behavior**, not implementation.

The next design stages translate them into:

```text
User Flows
    ↓
Domain Entities
    ↓
Database Relationships
    ↓
API Endpoints
    ↓
System Architecture
    ↓
Frontend Screens
    ↓
Implementation
```

The user flow document should be updated whenever a major product behavior changes.
