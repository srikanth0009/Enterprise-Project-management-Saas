# Enterprise Project Management SaaS — UI/UX Design

## 1. Purpose

This document defines the visual structure, navigation, screens, interactions, and reusable UI patterns for the Enterprise Project Management SaaS.

The product takes inspiration from Jira, Linear, and Notion while maintaining its own design.

Goals:

- Clear navigation
- Fast common actions
- Consistent UI
- Useful feedback
- Responsive design
- Accessibility

## 2. Application Navigation

Desktop:

```text
┌────────────────────────────────────────────────────────────┐
│ Logo / Product Name                                        │
├────────────────────────────────────────────────────────────┤
│ Organization Switcher                                     │
├────────────────────────────────────────────────────────────┤
│ 🏠 Dashboard                                               │
│ 📁 Projects                                                │
│ 👥 Teams                                                   │
│ 🗂 Workspaces                                              │
│                                                            │
│ Recent Projects                                            │
│   E-Commerce                                               │
│   Mobile App                                               │
│                                                            │
│ ⚙ Settings                                                 │
│ User Profile                                               │
└────────────────────────────────────────────────────────────┘
```

Topbar:

```text
┌────────────────────────────────────────────────────────────┐
│ Search...                         🔔 Notifications   Avatar │
└────────────────────────────────────────────────────────────┘
```

Responsive behavior:

```text
Desktop → Sidebar + Topbar
Tablet  → Collapsed Sidebar + Topbar
Mobile  → Topbar + Navigation Drawer
```

The Kanban board can remain horizontally scrollable on mobile.

## 3. Authentication Screens

### Login

```text
Product Logo

Welcome back
Sign in to continue

Email
[________________________]

Password
[________________________]

Forgot password?

[ Sign In ]

──────── OR ────────

[ Continue with Google ]

Don't have an account? Sign up
```

### Register

Fields:

```text
Name
Email
Password
Confirm Password
```

Actions:

```text
Create Account
Continue with Google
Login
```

### Forgot Password

```text
Email
 ↓
Send Reset Link
 ↓
Confirmation
```

## 4. Organization Selection

```text
Select an organization

┌──────────────────────────────┐
│ ACME Corp                    │
│ 12 members                   │
└──────────────────────────────┘

┌──────────────────────────────┐
│ Startup Inc                  │
│ 8 members                    │
└──────────────────────────────┘

+ Create organization
```

## 5. Create Organization

Fields:

```text
Organization name
Organization slug
Optional logo
```

After creation:

```text
Organization
 ↓
Initial workspace
 ↓
Organization dashboard
```

## 6. Main Dashboard

The dashboard provides a high-level overview.

```text
┌──────────────────────────────────────────────────────────────┐
│ Dashboard                                  + Create Project  │
├──────────────────────────────────────────────────────────────┤
│ Good morning                                                 │
│ Here's what's happening across your projects.               │
├───────────────┬───────────────┬───────────────┬──────────────┤
│ Projects      │ Open Issues   │ In Progress   │ Completed    │
│ 8             │ 42            │ 17            │ 126          │
├───────────────┴───────────────┴───────────────┴──────────────┤
│ My Work                       Recent Activity                │
│ ┌─────────────────────────┐   ┌───────────────────────────┐ │
│ │ Fix login bug           │   │ John created ISSUE-123    │ │
│ │ High · Today            │   │ Sarah moved ISSUE-89      │ │
│ └─────────────────────────┘   └───────────────────────────┘ │
├──────────────────────────────────────────────────────────────┤
│ Project Progress                                             │
│ E-Commerce       ███████████████░░░                          │
│ Mobile App       ███████████░░░░░░                          │
└──────────────────────────────────────────────────────────────┘
```

Widgets:

```text
Project count
Open issues
Issues in progress
Completed issues
My work
Recent activity
Project progress
```

## 7. Projects Page

```text
┌──────────────────────────────────────────────────────────────┐
│ Projects                                  + New Project      │
├──────────────────────────────────────────────────────────────┤
│ Search projects...        Filter       Sort                  │
├──────────────────────────────────────────────────────────────┤
│ E-Commerce                                                   │
│ Online commerce platform                                     │
│ 24 issues · 7 members                                       │
│ Progress ███████████░░                                      │
│                                                              │
│ Mobile Application                                           │
│ Customer mobile application                                  │
│ 18 issues · 5 members                                       │
└──────────────────────────────────────────────────────────────┘
```

Project cards can show:

```text
Name
Description
Status
Progress
Issue count
Member count
Last updated
```

## 8. Create Project

Fields:

```text
Project name
Project key
Description
Workspace
Project lead
Members
```

Example:

```text
Name: E-Commerce Platform
Key: ECOM
```

Issue identifiers then become:

```text
ECOM-1
ECOM-2
ECOM-3
```

## 9. Project Layout

```text
┌──────────────────────────────────────────────────────────────┐
│ E-Commerce                                  ⚙ Settings       │
│ Online commerce platform                                     │
├──────────────────────────────────────────────────────────────┤
│ Overview │ Issues │ Board │ Activity                         │
├──────────────────────────────────────────────────────────────┤
│                     PROJECT CONTENT                          │
└──────────────────────────────────────────────────────────────┘
```

Navigation:

```text
Overview
Issues
Board
Activity
Settings
```

## 10. Project Overview

```text
Project Overview

┌─────────────────────┬─────────────────────┐
│ Total Issues        │ Completed           │
│ 84                  │ 52                  │
└─────────────────────┴─────────────────────┘

Progress
██████████████░░░░

Recent Activity
John created ECOM-120
Sarah completed ECOM-115
Mike commented on ECOM-98
```

## 11. Issues Page

```text
┌──────────────────────────────────────────────────────────────┐
│ Issues                                    + Create Issue     │
├──────────────────────────────────────────────────────────────┤
│ Search issues...                                             │
│ Filter     Status     Priority     Assignee     Label       │
├──────────────────────────────────────────────────────────────┤
│ ECOM-124  Fix checkout bug       HIGH     John      Bug      │
│ ECOM-123  Update login UI        MEDIUM   Sarah     UI       │
│ ECOM-122  Add payment retry      URGENT   Mike      Backend  │
├──────────────────────────────────────────────────────────────┤
│                     1  2  3  4  5                            │
└──────────────────────────────────────────────────────────────┘
```

Columns:

```text
Issue ID
Title
Status
Priority
Assignee
Labels
Updated
```

## 12. Kanban Board

```text
┌──────────────────────────────────────────────────────────────┐
│ Board                                      + Create Issue     │
├──────────────────────────────────────────────────────────────┤
│ Search   Filter   Assignee   Priority                       │
├───────────────┬───────────────┬───────────────┬──────────────┤
│ TODO          │ IN PROGRESS   │ IN REVIEW     │ DONE         │
│ 4             │ 3             │ 2             │ 8            │
│               │               │               │              │
│ ECOM-124      │ ECOM-120      │ ECOM-116      │ ECOM-101     │
│ Checkout bug  │ Login bug     │ Review UI     │ Payment      │
│ HIGH          │ MEDIUM        │ LOW           │ DONE         │
└───────────────┴───────────────┴───────────────┴──────────────┘
```

Interaction:

```text
Drag issue
 ↓
Drop into column
 ↓
Change status
 ↓
Optimistic UI update
 ↓
Backend update
 ↓
Rollback on failure
```

## 13. Issue Card

```text
┌──────────────────────────────┐
│ ECOM-124                    │
│ Fix checkout payment bug    │
│                             │
│ HIGH       Bug              │
│                             │
│ John                        │
└──────────────────────────────┘
```

Keep cards focused on important information.

## 14. Create Issue

Accessible from:

```text
Issues page
Kanban board
Project overview
Global quick-create
```

Fields:

```text
Title
Description
Status
Priority
Assignee
Labels
Due date
Attachments
```

## 15. Create Issue Modal

```text
┌──────────────────────────────────────────────┐
│ Create Issue                            ✕    │
├──────────────────────────────────────────────┤
│ Title                                        │
│ [ Fix checkout payment                    ] │
│                                              │
│ Description                                  │
│ [                                          ] │
│                                              │
│ Status        Priority       Assignee        │
│ [Todo ▼]      [High ▼]       [John ▼]       │
│                                              │
│ Labels                                        │
│ [Bug] [+]                                     │
│                                              │
│                     Cancel   Create Issue     │
└──────────────────────────────────────────────┘
```

## 16. Issue Detail

```text
┌──────────────────────────────────────────────────────────────┐
│ ECOM-124                                      ⋮              │
│ Fix checkout payment bug                                     │
├──────────────────────────────────────────────────────────────┤
│ Status       In Progress                                     │
│ Priority     High                                            │
│ Assignee     John                                            │
│ Labels       Bug · Payment                                   │
├──────────────────────────────────────────────────────────────┤
│ Description                                                  │
│ Customers are unable to complete checkout...                │
├──────────────────────────────────────────────────────────────┤
│ Attachments                                                  │
│ checkout-error.png                                           │
├──────────────────────────────────────────────────────────────┤
│ Comments                                                     │
│ John: I found the issue in the payment callback.            │
│ Sarah: I'll review the fix.                                 │
├──────────────────────────────────────────────────────────────┤
│ Activity                                                     │
│ John changed status Todo → In Progress                       │
└──────────────────────────────────────────────────────────────┘
```

## 17. Comments

```text
┌──────────────────────────────────────────────────────────────┐
│ Write a comment...                                           │
│                                                              │
│ @mention   📎 attachment                      [Comment]      │
└──────────────────────────────────────────────────────────────┘
```

Support:

```text
Text
Mentions
Attachments
Editing
Deleting
```

## 18. Mentions

Typing:

```text
@jo
```

shows matching members.

Selecting one inserts the mention and allows the backend to generate a notification.

## 19. Attachments

Flow:

```text
Select file
 ↓
Validate type/size
 ↓
Upload
 ↓
Cloudinary
 ↓
Save metadata
 ↓
Display attachment
```

Example:

```text
📎 checkout-error.png
   1.2 MB
   [Open] [Remove]
```

## 20. Activity Timeline

```text
Today

09:45  John created ECOM-124
10:10  Sarah assigned ECOM-124 to Mike
10:30  Mike changed status Todo → In Progress
11:05  Sarah added label "Bug"

Yesterday

16:20  John commented on ECOM-124
```

## 21. Teams Page

```text
┌──────────────────────────────────────────────────────────────┐
│ Teams                                      + Create Team     │
├──────────────────────────────────────────────────────────────┤
│ Engineering                                                  │
│ 12 members · 4 projects                                     │
│                                                              │
│ Design                                                       │
│ 6 members · 3 projects                                      │
│                                                              │
│ QA                                                           │
│ 5 members · 4 projects                                      │
└──────────────────────────────────────────────────────────────┘
```

## 22. Team Detail

```text
Engineering

Members
John Smith
Sarah Johnson
Mike Thomas

Projects
E-Commerce
Mobile App

Team Settings
```

## 23. Workspaces

```text
Workspace
├── Overview
├── Projects
├── Members
└── Settings
```

Example:

```text
Product Development
 ├── E-Commerce
 ├── Mobile App
 └── Internal Platform
```

## 24. Organization Settings

```text
General
Members
Teams
Roles & Permissions
Security
Billing (future)
```

Only authorized users should see administrative settings.

## 25. Project Settings

```text
General
Members
Issue Settings
Labels
Statuses
Danger Zone
```

Destructive operations:

```text
Archive Project
Delete Project
```

must require confirmation.

## 26. Notifications

```text
┌──────────────────────────────────────────┐
│ Notifications                            │
├──────────────────────────────────────────┤
│ ● John mentioned you in ECOM-124         │
│   5 minutes ago                           │
│ ● Sarah assigned ECOM-120 to you         │
│   20 minutes ago                          │
│ ○ Project E-Commerce was updated         │
│   1 hour ago                              │
├──────────────────────────────────────────┤
│ Mark all as read                          │
└──────────────────────────────────────────┘
```

Unread notifications should be visually distinct.

## 27. Global Search

```text
┌──────────────────────────────────────────────┐
│ Search                                      │
├──────────────────────────────────────────────┤
│ 🔎 login payment                            │
├──────────────────────────────────────────────┤
│ Issues                                       │
│ ECOM-124 Fix login payment                   │
│ ECOM-98 Login callback                       │
│                                              │
│ Projects                                     │
│ E-Commerce Platform                          │
│                                              │
│ People                                       │
│ John Smith                                   │
└──────────────────────────────────────────────┘
```

Suggested shortcut:

```text
Ctrl/Cmd + K
```

## 28. User Profile

Profile menu:

```text
Profile
Preferences
Notifications
Theme
Logout
```

Profile page:

```text
Avatar
Name
Email
Password
Preferences
```

## 29. Modal Strategy

Use modals for focused actions:

```text
Create Issue
Create Project
Create Team
Invite Member
Confirm Delete
Edit Labels
```

Complex workflows should use dedicated pages or side panels instead of oversized modals.

## 30. Confirmation Dialogs

Destructive actions require confirmation.

```text
┌─────────────────────────────────────────────┐
│ Delete Project?                             │
│                                             │
│ This action cannot be undone.               │
│                                             │
│              Cancel    Delete Project       │
└─────────────────────────────────────────────┘
```

## 31. Toast Notifications

Use lightweight feedback:

```text
✓ Issue created
✓ Project updated
✓ Comment added
✕ Failed to update issue
```

Critical information should not depend only on toasts.

## 32. Loading UX

Use skeletons for content-heavy pages.

Buttons should show loading states:

```text
[ Creating... ]
```

to prevent duplicate submissions.

## 33. Empty States

Example:

```text
┌──────────────────────────────────────────┐
│              No projects yet             │
│                                          │
│ Create your first project to start      │
│ managing your work.                     │
│                                          │
│          [ Create Project ]              │
└──────────────────────────────────────────┘
```

## 34. Error States

```text
┌──────────────────────────────────────────┐
│          Something went wrong            │
│                                          │
│ Unable to load project data.             │
│                                          │
│             [ Try Again ]                │
└──────────────────────────────────────────┘
```

## 35. Design System

Core primitives:

```text
Button
Input
Textarea
Select
Checkbox
Radio
Switch
Badge
Avatar
Card
Dialog
Popover
Dropdown
Tabs
Tooltip
Table
Pagination
Toast
Skeleton
```

ShadCN UI can provide the base components.

## 36. Typography

Suggested hierarchy:

```text
Page title       24–32px
Section title    18–24px
Card title       14–18px
Body             14–16px
Secondary text   12–14px
```

Tune exact values during implementation.

## 37. Color System

Use semantic colors:

```text
Primary       → Main actions
Success       → Completed / success
Warning       → Attention
Destructive   → Delete / critical
Muted         → Secondary information
Background    → Application surface
Border        → Separation
```

Do not rely only on color to communicate status. Use text/icons as well.

## 38. Spacing

Use a consistent spacing scale:

```text
4px
8px
12px
16px
20px
24px
32px
48px
```

## 39. Responsive Board

Desktop:

```text
TODO | IN PROGRESS | IN REVIEW | DONE
```

Mobile:

```text
← TODO | IN PROGRESS →
```

Horizontal scrolling is acceptable for Kanban.

## 40. Accessibility

Support:

```text
Keyboard navigation
Visible focus
Proper labels
Semantic HTML
Accessible dialogs
Accessible dropdowns
Screen-reader-friendly controls
Keyboard-accessible Kanban interactions
```

Drag-and-drop should have a keyboard-accessible alternative.

## 41. Interaction Patterns

### Create

```text
Button
 ↓
Form
 ↓
Validation
 ↓
Submit
 ↓
Success
 ↓
Close / navigate / update UI
```

### Edit

```text
Edit
 ↓
Save
 ↓
Server update
 ↓
UI update
```

### Delete

```text
Delete
 ↓
Confirmation
 ↓
Delete request
 ↓
Success
 ↓
Remove from UI
```

### Drag

```text
Drag
 ↓
Drop
 ↓
Optimistic update
 ↓
Server update
 ↓
Rollback on failure
```

## 42. Quick Actions

Important actions:

```text
Create Issue
Create Project
Invite Member
Search
Switch Organization
Switch Project
```

A command palette can later expose these actions.

## 43. MVP Screen Scope

First implementation:

```text
Authentication
Organization creation/selection
Dashboard
Projects
Project overview
Issues list
Create issue
Issue detail
Kanban board
Comments
Basic notifications
Basic settings
```

Later:

```text
Advanced analytics
Advanced search
Advanced permissions
Rich text editing
Complex notification preferences
Advanced workspace features
Billing
```

## 44. Recommended Implementation Order

```text
1. Application shell
   ├── Sidebar
   ├── Topbar
   └── Routing

2. Authentication
   ├── Login
   ├── Register
   └── Protected routes

3. Organization
   ├── Create organization
   └── Organization selection

4. Projects
   ├── Project list
   ├── Create project
   └── Project layout

5. Issues
   ├── Issue list
   ├── Create issue
   ├── Issue detail
   └── Edit issue

6. Kanban
   ├── Columns
   ├── Issue cards
   └── Drag and drop

7. Collaboration
   ├── Comments
   ├── Attachments
   ├── Activity
   └── Mentions

8. Real-time
   ├── WebSocket
   ├── Issue events
   └── Notifications

9. Dashboard
   ├── Statistics
   ├── My work
   └── Activity

10. Polish
    ├── Loading states
    ├── Error states
    ├── Empty states
    ├── Accessibility
    └── Responsive behavior
```

## 45. Final UI Architecture

```text
Application
    │
    ├── Topbar
    ├── Sidebar
    │
    └── Router
          │
       Organization
          │
       Workspace
          │
        Project
          │
     ┌────┼─────────┐
     │    │         │
 Overview Issues   Board
          │         │
          ▼         ▼
     Issue Detail Kanban
          │
     ┌────┼────┐
     │    │    │
 Comments Attachments Activity
```

## 46. Design Progress

```text
1. Product Requirements       ✅
2. User Flows                 ✅
3. Domain Model               ✅
4. Database Design            ✅
5. API Design                 ✅
6. System Architecture        ✅
7. Frontend Architecture      ✅
8. UI/UX Design               ✅

9. Project Initialization     ← NEXT
10. Implementation
11. Testing
12. Docker / Deployment
13. Documentation / Resume
```

The design phase is now complete.

The next step is to initialize the actual React + TypeScript project, configure the development environment, install dependencies, establish the folder structure, and make the first Git commit.
