# Core Data and API Architecture

## 1. Purpose

This document defines the implementation-ready data and API architecture for Team B's Core MVP within the shared Intern Management System.

The architecture supports:

- Mentor Task Assignment
- Intern Task Viewing and Status Updates
- Individual Task Submission
- Weekly Progress Submission
- Mentor Weekly Progress Review
- Integration with the shared Team A / Team B user and account foundation

## 2. Existing Shared IMS Foundation

The existing backend database foundation already provides shared identity through:

- Supabase `auth.users`
- `shared_accounts`
- `person_profile`

Team B must reuse this shared identity model rather than create separate Team B-only user accounts for Interns or Mentors.

The shared account/profile structure acts as the identity foundation for Team B functionality. Team B entities should reference the existing shared user identity where an Intern or Mentor relationship is required.

## 3. Core Team B Data Entities

Team B requires three main Core MVP data entities in addition to the existing shared IMS identity foundation:

### 3.1 Task

A Task represents work assigned by a Mentor to an Intern.

Proposed architectural fields:

- `id`
- `title`
- `description`
- `due_date`
- `status`
- `assigned_intern_id`
- `assigned_by_mentor_id`
- `created_at`
- `updated_at`

The `assigned_intern_id` and `assigned_by_mentor_id` references should use the existing shared IMS account/profile identity rather than introducing separate Team B-only user records.

### 3.2 Task Submission

A Task Submission represents an individual submission made by an Intern against a specific assigned Task.

Required relationships:

- A Task Submission belongs to one Task.
- A Task Submission belongs to the Intern making the submission.
- Task Submission must remain separate from Weekly Progress.

The exact submission content and mandatory fields are not yet finalised and should therefore not be hardcoded into the architecture as confirmed requirements.

At architecture level, a Task Submission requires enough information to identify:

- the submission itself;
- the related Task;
- the submitting Intern;
- submission content once confirmed;
- creation and update timestamps.

### 3.3 Weekly Progress

Weekly Progress represents the Intern's overall progress report for a reporting week.

Proposed architectural fields:

- `id`
- `intern_id`
- `reporting_week`
- `accomplishments`
- `blockers`
- `next_steps`
- `created_at`
- `updated_at`

Weekly Progress is separate from individual Task Submissions and is associated with the relevant Intern and reporting period.

### 3.4 Core Relationships

The current Team B data relationships are:

```text
shared_accounts / person_profile
        |
        +---- Mentor
        |       |
        |       +---- assigns ---- Task
        |
        +---- Intern
                |
                +---- assigned ---- Task
                |                     |
                |                     +---- TaskSubmission
                |
                +---- WeeklyProgress
```

This structure keeps Team B functionality connected to the shared IMS identity while separating task-specific work from weekly progress reporting.

## 4. Task Status Lifecycle and Task Submission Relationship

### 4.1 Task Status Lifecycle

The Core MVP uses the following task status lifecycle:

```text
Assigned -> In Progress -> Completed
```

A newly assigned task begins with the `Assigned` status.

When the Intern begins work, the task can move to `In Progress`.

When the work has been completed, the task can move to `Completed`.

Task status is stored as part of the Task entity and represents the current lifecycle state of the assigned work.

The architecture should avoid introducing additional task statuses until they are confirmed by the requirements.

### 4.2 Task Submission Relationship

Task Submission is associated with an individual Task and the Intern who submits it.

The relationship is:

```text
Intern -> Task -> TaskSubmission
```

Task Submission and task status are related but represent different concepts:

- Task status represents the lifecycle state of the assigned task.
- Task Submission represents the Intern's submitted work for that task.
- A Task Submission must reference the Task to which the submitted work belongs.
- A Task Submission must be attributable to the Intern who submitted it.
- Updating task status does not itself create a Weekly Progress record.
- Task Submission must remain separate from Weekly Progress reporting.

The exact Task Submission content and required fields remain an open implementation detail until the relevant requirements are finalised.

## 5. Weekly Progress Relationship

### 5.1 Separation from Task Submission

Weekly Progress must be represented as a separate data concept from Task Submission.

The two concepts serve different purposes:

```text
Task
  |
  +---- TaskSubmission
        Individual work submitted for a specific Task

Intern
  |
  +---- WeeklyProgress
        Overall progress reported for a reporting week
```

A Task Submission is task-specific, while Weekly Progress describes an Intern's broader progress for a reporting period.

Therefore:

- Weekly Progress does not belong to an individual Task.
- Weekly Progress belongs to an Intern and a reporting week.
- A Task Submission does not replace a Weekly Progress report.
- Completing or submitting an individual Task does not automatically create a Weekly Progress record.

### 5.2 Mentor Review Relationship

Mentors require access to Weekly Progress information for the Interns they are responsible for reviewing.

At the Core MVP level, the architecture must support:

```text
Intern -> submits -> WeeklyProgress
                         |
                         +---- reviewed by ---- Mentor
```

The exact review fields, such as review comments or review status, should not be treated as final until confirmed by the requirements.

The architecture should therefore support Mentor retrieval/review of Weekly Progress without prematurely fixing unconfirmed review fields.

## 6. Core API and Module Boundaries

The backend should organise Team B functionality around separate domain responsibilities.

### 6.1 Task Module

Responsible for Mentor task assignment and Intern task interaction.

Expected API responsibilities:

```text
POST   /tasks
GET    /tasks
GET    /tasks/:id
PATCH  /tasks/:id/status
```

Expected behaviour:

- Mentors can create and assign Tasks.
- Interns can retrieve Tasks assigned to them.
- Relevant users can retrieve Task details.
- Interns can update the status of their assigned Tasks.

Authorization must be applied according to the authenticated user's role and relationship to the requested resource.

### 6.2 Task Submission Module

Responsible for individual Task Submissions.

Expected API responsibilities:

```text
POST   /tasks/:taskId/submissions
GET    /tasks/:taskId/submissions
```

Expected behaviour:

- An Intern can submit work against an assigned Task.
- A submission is linked to both the Task and the submitting Intern.
- Authorized Mentors can retrieve submissions associated with relevant Tasks.

Exact submission fields should remain flexible until the requirements are finalised.

### 6.3 Weekly Progress Module

Responsible for weekly progress reporting and Mentor review access.

Expected API responsibilities:

```text
POST   /weekly-progress
GET    /weekly-progress
GET    /weekly-progress/:id
```

Expected behaviour:

- Interns can create Weekly Progress reports.
- Interns can retrieve their own progress information.
- Authorized Mentors can retrieve relevant Intern progress reports.
- Weekly Progress remains independent from Task Submission.

Additional review/update endpoints can be introduced when the Mentor review requirements are finalised.

### 6.4 Shared Identity and Authorization

Team B modules must use the existing shared IMS identity rather than implementing separate authentication.

The expected request flow is:

```text
Supabase Authentication
        |
        v
Authenticated User
        |
        v
shared_accounts / person_profile
        |
        v
Role and resource authorization
        |
        +---- Task APIs
        +---- Task Submission APIs
        +---- Weekly Progress APIs
```

Authentication determines who the user is.

Role-based access control and resource ownership determine what the user is allowed to do.

The exact RBAC implementation is addressed separately by the shared authentication/RBAC architecture work.

## 7. Team A / Team B Shared Data Dependencies

### 7.1 Shared User Identity

Team A and Team B operate as modules of the same IMS rather than separate products.

A user should retain the same account and profile when moving between the Recruitment/Application workflow and the Intern Workspace.

Team B therefore depends on the shared:

- Supabase authentication identity;
- `shared_accounts` record;
- `person_profile` record;
- account role/permission information.

Team B must not require an accepted Applicant to register a second account.

### 7.2 Applicant-to-Intern Transition

Team A owns the Recruitment/Application workflow up to the Applicant acceptance decision.

Once an Applicant is accepted and the appropriate transition is performed, the same shared user identity becomes eligible to access Team B's Intern functionality.

The high-level relationship is:

```text
Team A

Applicant
    |
    v
Application / Recruitment
    |
    v
Accepted
    |
    v
Applicant-to-Intern Transition
    |
    v

Shared IMS Identity
    |
    v

Team B

Intern Workspace
    |
    +---- Tasks
    +---- Task Submissions
    +---- Weekly Progress
```

Team B should consume the resulting shared Intern identity rather than reproduce Team A's Recruitment/Application data model.

### 7.3 Ownership Boundaries

The current ownership boundary is:

**Team A**

- Applicant workflow
- Recruitment/Application process
- Application status
- Applicant acceptance decision
- Applicant-to-Intern handoff requirements

**Shared IMS Foundation**

- Authentication
- Shared accounts
- Person profile
- Shared roles/permissions
- Common identity used across both modules

**Team B**

- Intern Workspace
- Mentor Task Assignment
- Intern Task Viewing
- Task Status Updates
- Individual Task Submission
- Weekly Progress
- Mentor Progress Review

Team B entities should reference shared identities but should not duplicate Team A-owned Recruitment/Application entities.

## 8. Week 2 Working Assumptions and Open Decisions

The architecture is based on the current available requirements, completed BA work, UX workflow, existing backend foundation and cross-team integration direction.

The following are treated as current working assumptions:

1. Team A and Team B are modules of one shared IMS.
2. Users maintain one account/profile across the system.
3. An accepted Applicant transitions to Intern access without creating a second account.
4. Team B reuses the existing shared account/profile identity.
5. The Core MVP task lifecycle is `Assigned -> In Progress -> Completed`.
6. Task Submission and Weekly Progress are separate concepts.
7. Weekly Progress is associated with an Intern and reporting period rather than an individual Task.
8. Mentors require access to relevant Intern Tasks, Task Submissions and Weekly Progress.
9. Team B does not implement a separate onboarding system.
10. Authentication and RBAC are shared IMS concerns and should not be duplicated inside individual Team B modules.

The following decisions remain open or should be confirmed before implementation is treated as final:

- Exact Task Submission fields and validation requirements.
- Exact Weekly Progress required fields if these change following review.
- Mentor Weekly Progress review fields and review state.
- Exact RBAC permissions for Applicant, Intern, Mentor and Admin.
- How Mentor-to-Intern responsibility/assignment is represented in the shared data model.
- Whether additional Task statuses are required beyond the current Core MVP lifecycle.
- Exact API response/request DTO structures.
- Final database naming conventions and migration structure.
- Any changes resulting from pending client or cross-team clarification.

These open decisions should not block the architecture bootstrap. Implementation can begin against the confirmed boundaries while avoiding hardcoded assumptions for unresolved fields.

## 9. Proposed Implementation Structure

Based on the current NestJS backend structure, Team B functionality can be implemented using separate modules:

```text
src/
|
+---- tasks/
|     +---- tasks.module.ts
|     +---- tasks.controller.ts
|     +---- tasks.service.ts
|     +---- entities/
|     +---- dto/
|
+---- task-submissions/
|     +---- task-submissions.module.ts
|     +---- task-submissions.controller.ts
|     +---- task-submissions.service.ts
|     +---- entities/
|     +---- dto/
|
+---- weekly-progress/
      +---- weekly-progress.module.ts
      +---- weekly-progress.controller.ts
      +---- weekly-progress.service.ts
      +---- entities/
      +---- dto/
```

This structure separates the three Core MVP responsibilities while allowing all modules to reuse the shared IMS authentication, account and profile foundation.

## 10. Architecture Summary

The Team B Core MVP architecture extends the existing shared IMS foundation rather than creating a separate user system.

The architecture introduces three primary Team B concepts:

```text
Task
TaskSubmission
WeeklyProgress
```

These concepts support the required Mentor and Intern workflow while maintaining clear separation between individual Task work and weekly progress reporting.

The architecture also preserves the Team A / Team B integration boundary:

```text
Team A Recruitment
        |
        v
Applicant Accepted
        |
        v
Shared Applicant-to-Intern Transition
        |
        v
Existing Shared IMS Identity
        |
        v
Team B Intern Workspace
```

This provides an implementation-ready foundation while leaving unresolved requirement details explicitly open for later confirmation.
