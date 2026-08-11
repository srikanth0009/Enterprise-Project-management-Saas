# Enterprise Project Management SaaS — Domain Model

This document defines the core business entities of the Enterprise Project Management SaaS, what each entity represents, the information it owns, and how the entities relate to one another.

This is a **domain model**, not the final MongoDB schema.

The purpose is to understand the business structure before designing collections, indexes, APIs, and implementation details.

---

# 1. High-Level Domain

The application is organized around this hierarchy:

```text
User
  ↓
Organization
  ├── Organization Members
  ├── Teams
  └── Workspaces
          ↓
       Projects
          ↓
        Issues
          ├── Comments
          ├── Attachments
          ├── Labels
          ├── Activity
          └── Notifications
```

A simpler view:

```text
Organization
    │
    ├── Users / Members
    │
    ├── Teams
    │
    └── Workspaces
            │
            └── Projects
                    │
                    └── Issues
```

---

# 2. Core Entities

The initial domain contains these entities:

```text
1. User
2. Organization
3. OrganizationMember
4. Team
5. TeamMember
6. Workspace
7. WorkspaceMember
8. Project
9. ProjectMember
10. Issue
11. Label
12. Comment
13. Attachment
14. Notification
15. Activity
16. Workflow
17. Invitation
```

Some entities represent actual business objects, while others represent relationships between objects.

---

# 3. User

## 3.1 What is a User?

A User represents a person who has an account in the application.

Example:

```text
Srikanth
Email: srikanth@example.com
```

A user can participate in multiple organizations, teams, workspaces, and projects depending on permissions.

## 3.2 Important Information

A User conceptually contains:

```text
User
├── id
├── name
├── email
├── password
├── avatar
├── authentication information
├── account status
├── createdAt
└── updatedAt
```

The exact authentication fields will be decided during authentication design.

## 3.3 User Relationships

```text
User
 ├── belongs to Organizations
 ├── belongs to Teams
 ├── belongs to Workspaces
 ├── belongs to Projects
 ├── creates Issues
 ├── gets assigned Issues
 ├── writes Comments
 ├── uploads Attachments
 ├── generates Activities
 └── receives Notifications
```

---

# 4. Organization

## 4.1 What is an Organization?

An Organization represents a company or large group using the platform.

Example:

```text
Acme Technologies
```

It is the highest-level business container.

## 4.2 Organization Contains

```text
Organization
├── Members
├── Teams
├── Workspaces
└── Organization settings
```

## 4.3 Important Information

```text
Organization
├── id
├── name
├── owner
├── logo
├── settings
├── createdAt
└── updatedAt
```

## 4.4 Relationship

```text
Organization
    ├── has many OrganizationMembers
    ├── has many Teams
    └── has many Workspaces
```

---

# 5. OrganizationMember

## 5.1 Why Do We Need This Entity?

A User can belong to an organization, and an organization can contain many users.

This is a many-to-many relationship.

Instead of storing everything directly inside User or Organization, we need a membership concept.

```text
User ←── OrganizationMember ──→ Organization
```

## 5.2 Example

```text
Organization:
Acme Technologies

Members:

Srikanth → Owner
Rahul    → Admin
Priya    → Developer
Kiran    → QA
```

## 5.3 Important Information

```text
OrganizationMember
├── id
├── user
├── organization
├── role
├── status
├── joinedAt
└── createdAt
```

## 5.4 Role

Initial roles:

```text
OWNER
ADMIN
MEMBER
```

The more detailed project roles are handled separately.

---

# 6. Team

## 6.1 What is a Team?

A Team is a group of people who work together around a function or area.

Examples:

```text
Engineering
Design
QA
Marketing
```

## 6.2 Team Information

```text
Team
├── id
├── organization
├── name
├── description
├── createdBy
├── createdAt
└── updatedAt
```

## 6.3 Relationship

```text
Organization
     ↓
   Teams
     ↓
 Team Members
```

A team belongs to one organization.

---

# 7. TeamMember

## 7.1 Why Do We Need TeamMember?

A user can belong to multiple teams.

A team can contain multiple users.

Therefore:

```text
User ←── TeamMember ──→ Team
```

is a many-to-many relationship.

## 7.2 Example

```text
Srikanth
├── Engineering
└── AI Team
```

## 7.3 Important Information

```text
TeamMember
├── id
├── user
├── team
├── role
├── joinedAt
└── createdAt
```

For the first version, team roles can remain simple.

---

# 8. Workspace

## 8.1 What is a Workspace?

A Workspace is a container for related projects.

Example:

```text
Acme Technologies

Product Development
├── E-Commerce Platform
├── Mobile App
└── Admin Dashboard
```

## 8.2 Workspace Information

```text
Workspace
├── id
├── organization
├── name
├── description
├── createdBy
├── createdAt
└── updatedAt
```

## 8.3 Relationship

```text
Organization
      ↓
Workspace
      ↓
Projects
```

A workspace belongs to one organization.

A workspace can contain multiple projects.

---

# 9. WorkspaceMember

## 9.1 Why Do We Need This?

Not every organization member necessarily needs access to every workspace.

Therefore workspace membership is separate.

```text
User ←── WorkspaceMember ──→ Workspace
```

## 9.2 Important Information

```text
WorkspaceMember
├── id
├── user
├── workspace
├── role
├── joinedAt
└── createdAt
```

---

# 10. Project

## 10.1 What is a Project?

A Project represents a specific product, initiative, or goal.

Examples:

```text
E-Commerce Platform
Mobile Application
CRM System
Company Website
```

## 10.2 Project Information

```text
Project
├── id
├── workspace
├── organization
├── team
├── name
├── key
├── description
├── status
├── startDate
├── dueDate
├── createdBy
├── createdAt
└── updatedAt
```

## 10.3 Project Key

Every project should have a short unique key.

Example:

```text
Project:
E-Commerce Platform

Key:
ECOM
```

Issues then receive identifiers such as:

```text
ECOM-1
ECOM-2
ECOM-42
```

## 10.4 Project Status

Initial project statuses:

```text
PLANNING
ACTIVE
COMPLETED
ARCHIVED
```

## 10.5 Relationships

```text
Workspace
    ↓
Project
    ├── Team
    ├── Members
    └── Issues
```

---

# 11. ProjectMember

## 11.1 Why Do We Need This?

A project may have only a subset of workspace or organization members.

```text
User ←── ProjectMember ──→ Project
```

This is another many-to-many relationship.

## 11.2 Example

```text
Organization Members:
Srikanth
Rahul
Priya
Kiran
Anjali
Ravi

E-Commerce Project:
Srikanth
Rahul
Priya
Kiran
```

## 11.3 Important Information

```text
ProjectMember
├── id
├── user
├── project
├── role
├── joinedAt
└── createdAt
```

## 11.4 Project Roles

Initial project roles:

```text
PROJECT_MANAGER
DEVELOPER
DESIGNER
QA
VIEWER
```

These roles determine what users can do within a project.

---

# 12. Issue

## 12.1 What is an Issue?

An Issue is the primary unit of work.

Examples:

```text
Implement Google OAuth
Fix payment failure
Create product page
Write checkout tests
```

## 12.2 Issue Information

Conceptually:

```text
Issue
├── id
├── project
├── team
├── issueNumber
├── title
├── description
├── type
├── status
├── priority
├── reporter
├── assignee
├── labels
├── parentIssue
├── dueDate
├── createdAt
└── updatedAt
```

## 12.3 Issue Identifier

A project has a key.

Example:

```text
Project Key: ECOM
```

An issue has a number:

```text
42
```

Together:

```text
ECOM-42
```

## 12.4 Issue Types

Initial types:

```text
TASK
BUG
FEATURE
IMPROVEMENT
```

## 12.5 Issue Status

Initial statuses:

```text
TODO
IN_PROGRESS
IN_REVIEW
DONE
```

## 12.6 Issue Priority

```text
URGENT
HIGH
MEDIUM
LOW
```

## 12.7 Issue Relationships

An issue can have:

```text
Issue
 ├── Project
 ├── Reporter
 ├── Assignee
 ├── Labels
 ├── Comments
 ├── Attachments
 ├── Activities
 └── Parent Issue
```

---

# 13. Parent Issue / Subtask Relationship

Issues can have hierarchical relationships.

Example:

```text
ECOM-50
Implement Checkout
       │
       ├── ECOM-51 Create checkout UI
       ├── ECOM-52 Create checkout API
       ├── ECOM-53 Integrate payment
       └── ECOM-54 Write tests
```

Conceptually:

```text
Issue
  │
  ├── parentIssue
  │
  └── subtasks
```

This allows the application to represent larger work broken into smaller units.

---

# 14. Label

## 14.1 What is a Label?

A Label categorizes an issue.

Examples:

```text
frontend
backend
authentication
payment
security
performance
bug
```

## 14.2 Relationship

An issue can have multiple labels.

A label can be applied to multiple issues.

Therefore:

```text
Issue ←── Label Relationship ──→ Label
```

is many-to-many.

## 14.3 Important Information

```text
Label
├── id
├── project
├── name
├── description
├── createdBy
└── createdAt
```

Labels may be project-specific in the initial design.

---

# 15. Comment

## 15.1 What is a Comment?

A Comment is a message written by a user on an issue.

Example:

```text
Issue: ECOM-42

Rahul:
Please also handle existing users.

Srikanth:
Yes, I'll handle that.
```

## 15.2 Important Information

```text
Comment
├── id
├── issue
├── author
├── content
├── mentions
├── createdAt
└── updatedAt
```

## 15.3 Relationship

```text
Issue
  ↓
Comments
  ↓
User
```

One issue can have many comments.

A user can write many comments.

---

# 16. Attachment

## 16.1 What is an Attachment?

An Attachment represents a file associated with an issue.

Examples:

```text
oauth-flow.png
requirements.pdf
error-screenshot.png
api-response.json
```

## 16.2 Important Information

```text
Attachment
├── id
├── issue
├── uploadedBy
├── fileName
├── fileUrl
├── fileType
├── fileSize
├── createdAt
└── updatedAt
```

The actual file can be stored externally, while the application stores its metadata and URL.

---

# 17. Notification

## 17.1 What is a Notification?

A Notification informs a user about an event relevant to them.

Examples:

```text
You were assigned ECOM-42.

Rahul mentioned you.

ECOM-42 moved to DONE.
```

## 17.2 Important Information

```text
Notification
├── id
├── recipient
├── type
├── message
├── relatedEntity
├── isRead
├── createdAt
└── readAt
```

## 17.3 Relationship

```text
User
  ↓
Notifications
```

A user can have many notifications.

---

# 18. Activity

## 18.1 What is Activity?

Activity represents an important action performed in the system.

Comments describe communication.

Activities describe changes/actions.

Example:

```text
Srikanth was assigned to ECOM-42.

Priority changed:
MEDIUM → HIGH

Status changed:
TODO → IN_PROGRESS

Rahul added a label.
```

## 18.2 Important Information

```text
Activity
├── id
├── actor
├── action
├── entityType
├── entityId
├── metadata
└── createdAt
```

## 18.3 Example

```text
actor:
Srikanth

action:
STATUS_CHANGED

entity:
ECOM-42

metadata:
oldStatus = TODO
newStatus = IN_PROGRESS
```

This approach allows one activity model to represent many different actions.

---

# 19. Workflow

## 19.1 What is a Workflow?

A Workflow defines the statuses and transitions an issue can move through.

Basic workflow:

```text
TODO
  ↓
IN_PROGRESS
  ↓
IN_REVIEW
  ↓
DONE
```

A future custom workflow could be:

```text
BACKLOG
   ↓
TODO
   ↓
DEVELOPMENT
   ↓
CODE_REVIEW
   ↓
QA
   ↓
DONE
```

## 19.2 Workflow Information

Conceptually:

```text
Workflow
├── id
├── project
├── name
├── statuses
├── transitions
├── createdBy
├── createdAt
└── updatedAt
```

For the MVP, we can use fixed issue statuses and introduce a full Workflow entity later.

---

# 20. Invitation

## 20.1 What is an Invitation?

An Invitation represents an invitation sent to a user to join an organization.

Example:

```text
Organization:
Acme Technologies

Email:
rahul@example.com

Role:
Developer

Status:
PENDING
```

## 20.2 Important Information

```text
Invitation
├── id
├── organization
├── email
├── invitedBy
├── role
├── token/reference
├── status
├── expiresAt
└── createdAt
```

---

# 21. Entity Relationship Overview

The main relationships are:

```text
User
 │
 ├────────────── OrganizationMember ────────────── Organization
 │                                                    │
 │                                                    ├── Teams
 │                                                    │      │
 │                                                    │      └── TeamMember ─── User
 │                                                    │
 │                                                    └── Workspaces
 │                                                           │
 │                                                           └── WorkspaceMember ─── User
 │
 └────────────── ProjectMember ───────────────────── Project
                                                        │
                                                        ├── Team
                                                        │
                                                        └── Issues
                                                             │
                                                             ├── Assignee ─── User
                                                             ├── Reporter ─── User
                                                             ├── Labels
                                                             ├── Comments ─── User
                                                             ├── Attachments ─── User
                                                             ├── Activities ─── User
                                                             └── Parent Issue
```

---

# 22. Simplified Domain Graph

The most important business hierarchy is:

```text
ORGANIZATION
     │
     ├────────────── USERS
     │
     ├────────────── TEAMS
     │
     └────────────── WORKSPACES
                         │
                         └──────── PROJECTS
                                      │
                                      └──────── ISSUES
                                                  │
                                                  ├── Comments
                                                  ├── Attachments
                                                  ├── Labels
                                                  ├── Activities
                                                  └── Subtasks
```

---

# 23. Relationship Types

Understanding relationship types is important before designing MongoDB.

## One-to-One

Example:

```text
Organization
    ↓
Owner
```

One organization has one primary owner.

---

## One-to-Many

Example:

```text
Organization
    ↓
Many Teams
```

One organization can contain many teams.

Another example:

```text
Project
    ↓
Many Issues
```

One project contains many issues.

---

## Many-to-Many

Example:

```text
Users ←→ Teams
```

A user can belong to many teams.

A team can have many users.

The relationship is represented conceptually by:

```text
TeamMember
```

Another example:

```text
Users ←→ Projects
```

represented by:

```text
ProjectMember
```

---

# 24. Ownership vs Membership

Ownership and membership are different concepts.

Example:

```text
Organization
Owner:
Srikanth

Members:
Srikanth
Rahul
Priya
Kiran
```

The owner is a member, but has higher privileges.

Similarly:

```text
Project
Created By:
Rahul

Members:
Rahul
Srikanth
Priya
```

Creating something does not necessarily mean the creator permanently owns it.

This distinction will matter when designing permissions.

---

# 25. Access Hierarchy

Access can be understood as:

```text
Organization
      ↓
Workspace
      ↓
Project
      ↓
Issue
```

A user's access should be evaluated according to the relevant membership and permissions.

Example:

```text
User
 ↓
Organization Member?
 ↓
Workspace Member?
 ↓
Project Member?
 ↓
Can access Issue?
```

The exact authorization rules will be defined separately.

---

# 26. Example Real-World Data

Consider:

```text
Organization:
Acme Technologies
```

It contains:

```text
Teams:
Engineering
Design
QA
```

Workspace:

```text
Product Development
```

Projects:

```text
E-Commerce Platform
Mobile Application
```

Inside E-Commerce:

```text
ECOM-1  Create product API
ECOM-2  Build product page
ECOM-3  Implement checkout
ECOM-4  Integrate payment
```

ECOM-3 could contain:

```text
ECOM-31 Create checkout UI
ECOM-32 Create checkout API
ECOM-33 Add payment gateway
ECOM-34 Add checkout tests
```

Comments, attachments, labels, activities, and notifications are connected to these work items.

---

# 27. MVP Domain

For the first implementation, we should not build every entity immediately.

The MVP domain should focus on:

```text
User
Organization
OrganizationMember
Team
TeamMember
Workspace
WorkspaceMember
Project
ProjectMember
Issue
Label
Comment
Activity
```

After the core system works, add:

```text
Invitation
Attachment
Notification
Workflow
```

Then advanced features can follow.

---

# 28. Domain Implementation Order

The domain should be implemented in roughly this order:

```text
1. User
       ↓
2. Organization
       ↓
3. OrganizationMember
       ↓
4. Team
       ↓
5. TeamMember
       ↓
6. Workspace
       ↓
7. WorkspaceMember
       ↓
8. Project
       ↓
9. ProjectMember
       ↓
10. Issue
       ↓
11. Label
       ↓
12. Comment
       ↓
13. Activity
       ↓
14. Attachment
       ↓
15. Notification
       ↓
16. Workflow
```

---

# 29. Important Design Decision

The domain model intentionally separates:

```text
Entity
```

from:

```text
Relationship
```

For example:

```text
User
Team
```

are entities.

But:

```text
TeamMember
```

represents the relationship between them.

This gives us room to store relationship-specific information such as:

```text
role
joinedAt
status
permissions
```

This distinction will become important when we design the MongoDB collections.

---

# 30. What Comes Next

The current design process is:

```text
Product Requirements
        ↓
User Flows
        ↓
Domain Model       ← CURRENT
        ↓
Database Design
        ↓
API Design
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
database-design.md
```

That document will translate this domain model into actual MongoDB collections and define:

- Collection structure.
- Document fields.
- References.
- Embedded documents.
- Indexes.
- Unique constraints.
- Query patterns.
- Pagination strategy.
- Data ownership.
- Deletion behavior.
- Soft delete where appropriate.
