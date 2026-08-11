1. Product Overview

2. Users & Roles

3. Organization

4. Teams

5. Workspaces

6. Projects

7. Issues / Tasks

8. Kanban Board

9. Comments & Mentions

10. Attachments

11. Notifications

12. Search & Filtering

13. Activity History

14. Permissions

15. Dashboard & Analytics



# Section 1

## Overview 
    Our Application is a Saas platform where companies create organizations, add team members, create projects, break projects, into issues/tasks, assign those tasks to people, track their progress through workflows, and collaborate through comments, attachments and notifications.

# Section 2 

## Users & Roles  

 | Role                   | Purpose                                         |
| ---------------------- | ----------------------------------------------- |
| **Organization Owner** | Owns the organization and has full control      |
| **Admin**              | Manages organization/team members and settings  |
| **Project Manager**    | Creates/manages projects and assigns work       |
| **Developer**          | Works on assigned issues/tasks                  |
| **Designer**           | Works on design-related tasks                   |
| **QA**                 | Tests and verifies completed work               |
| **Viewer**             | Can view projects/issues but cannot modify them |


# Section 3

## Organization, Workspace & Teams

### Organisation 
    - Create organization
    - Organization name
    - Organization members
    - Organization owner
    - Organization settings
    - Invite members

### Teams
    - Create team
    - Team name
    - Add/remove members
    - Team members
    - Team projects

### Workspaces
    - Create workspace
    - Workspace name
    - Workspace members
    - Workspace projects
    - Workspace settings

One person can work on multiple project. 

One team can work on multiple projects.



# Section 4

## Projects

    A project represents a specific product, initiative, or goal that a team is working toward.


#### Project Key

        This is a small but important feature.

        if the project key is ECOM

        then issues can have identifiers like 

        ECOM-1
        ECOM-2
        ECOM-3
        ECOM-4

    So instead of saying : "The Google OAuth issue" someone can say: "ECOM-42"
    This is how Jira-style systems identify issues.


#### Project Members

    Not everyone on the organization necessarily works on every project.

#### Project Status

    A project itself cna have a lifecycle.

    Example: Planning -> Active -> Completed -> Archived

#### Project Settings

    A project manager may need to configure.
    
    Project Settings

    ├── Members
    ├── Issue statuses
    ├── Issue priorities
    ├── Labels
    ├── Workflow
    └── Permissions



# Section 5

## Issues/Tasks

    Project = the thing we're building.
    Issue = one piece of work needed to build it.

    Example:

    E-Commerce Platform
    │
    ├── ECOM-1  Create database schema
    ├── ECOM-2  Build login API
    ├── ECOM-3  Create product page
    ├── ECOM-4  Implement shopping cart
    ├── ECOM-5  Integrate payment
    └── ECOM-6  Write checkout tests

### What Information does an issue contain ?


    Issue
    ├── ID          - Each issue needs a unique identifier
    ├── Title       - short desc of work
    ├── Description - What exactly needs to be done
    ├── Issue type
    ├── Status      - Where the work currently is 
    ├── Priority`   - How important/ urgent is this issue ? 
    ├── Assignee    - Who is reponsible for doing the work ?
    ├── Reporter    - Who created/reported the issue ?
    ├── Labels      - Categorizes issue
    ├── Project     
    ├── Team
    ├── Due date
    ├── Comments
    ├── Attachments
    └── Activity history

    
