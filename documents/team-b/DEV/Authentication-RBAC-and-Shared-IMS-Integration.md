# Authentication, RBAC and Shared IMS Integration

## 1. Purpose

This document defines the shared authentication, user/profile, role-based access control (RBAC), and cross-team integration architecture for the Intern Management System (IMS).

The purpose is to allow Team A and Team B to operate as modules within one IMS while maintaining a single user identity and profile across the Applicant-to-Intern lifecycle.

This document builds on the existing shared IMS foundation and the Team B Core Data and API Architecture.

---

## 2. Shared Authentication Approach

The IMS should use one shared authentication approach across Team A and Team B.

A user should not require separate authentication or a second account when moving between the Recruitment workflow and the Intern workflow.

### Authentication Principles

- One authenticated identity is used across the IMS.
- Team A and Team B should not maintain separate login systems.
- Authentication should identify the user before role-based access is evaluated.
- The existing shared account/profile foundation should be reused.
- Authentication logic should remain a shared concern rather than being duplicated inside individual Team A or Team B modules.

### Authentication Flow

```text
User
  |
  v
Shared Authentication
  |
  v
Shared Account
  |
  v
Shared Person Profile
  |
  v
Role / Access Check
  |
  +--------------------+
  |                    |
  v                    v
Team A Modules      Team B Modules
```

Authentication determines who the user is, while RBAC determines which IMS functionality the authenticated user is allowed to access.

---

## 3. Shared User and Profile Conventions

The existing shared IMS foundation contains the shared account and person profile concepts.

The shared account should represent the user's system identity, while the person profile should contain information about the person associated with that account.

Conceptually:

```text
Authentication Identity
        |
        v
  Shared Account
        |
        v
  Person Profile
```

Team A and Team B should reference this shared identity rather than creating separate user records for the same person.

### Shared Identity Principles

1. A person should have one IMS account.
2. The same account should continue across the Applicant-to-Intern transition.
3. Shared profile information should not be duplicated in Team B-specific entities.
4. Team B entities should reference the shared account/profile identity where a user relationship is required.
5. Application status and system role should remain separate concepts.

---

## 4. Role Conventions

The current IMS architecture requires role-based access to distinguish the functionality available to different types of users.

The current role model is based on the following conceptual roles:

- Applicant
- Intern
- Mentor
- Admin

These roles represent system access responsibilities and should not be confused with application status.

For example:

```text
Account Role: Applicant
Application Status: Pending
```

After an accepted Applicant transitions into the Intern workflow:

```text
Account Role: Intern
Application Status: Accepted
```

The application status remains part of the recruitment/application domain, while the account role controls access to IMS functionality.

Any additional roles or changes to these role names should be treated as an open decision until confirmed.

---

## 5. Applicant-to-Intern Role Transition

The Applicant-to-Intern transition should preserve the existing user identity.

The user should not create a second account after being accepted.

Conceptually:

```text
Register
   |
   v
Applicant Account
   |
   v
Application / Recruitment
   |
   v
Accepted
   |
   v
Role Transition
Applicant -> Intern
   |
   v
Same Account and Profile
   |
   v
Intern Workspace
```

### Transition Principles

- Team A owns the Recruitment workflow through the Accepted decision.
- Acceptance acts as the handoff point between the Recruitment workflow and the active Intern workflow.
- The existing account and profile should be retained.
- The role may transition from Applicant to Intern.
- Application status remains separate from account role.
- Team B should consume the shared identity rather than creating a new Intern account.
- A separate onboarding account or duplicate user record should not be introduced.

The exact mechanism and authorization for changing a user's role should be confirmed as part of the shared implementation and RBAC design.

---

## 6. Role-Based Access Control

RBAC should control access to modules and actions after authentication.

The backend should enforce authorization. Frontend visibility can improve the user experience, but hiding a page or button in the frontend should not be treated as sufficient security.

### Conceptual Authorization Flow

```text
Request
   |
   v
Authentication Check
   |
   v
Identify Shared Account
   |
   v
Read Account Role
   |
   v
RBAC Permission Check
   |
   +---------- Allowed ----------> Controller / Service
   |
   +---------- Denied -----------> Access Denied
```

### Applicant

An Applicant should primarily access Team A Recruitment functionality.

Conceptual access:

- Applicant profile
- Application/recruitment functionality
- Relevant application status information

An Applicant should not automatically receive access to active Intern task/progress functionality before the required transition occurs.

### Intern

An Intern should access Team B functionality relevant to their own internship workflow.

Conceptual access:

- Intern Workspace
- View assigned tasks
- View task details
- Update permitted task status
- Submit individual Task Submission information
- Create/view relevant Weekly Progress information

### Mentor

A Mentor requires access to functionality associated with the Interns for whom they are responsible.

Conceptual access:

- Assign tasks where permitted
- View relevant Intern tasks
- View relevant Task Submissions
- View relevant Weekly Progress
- Perform Mentor review actions where defined

The exact Mentor-to-Intern relationship and detailed Mentor permissions remain subject to confirmation.

### Admin

Admin is expected to provide system-level management capabilities where required.

Potential responsibilities include:

- Manage appropriate user access
- Support role transitions
- Access administrative functionality
- Resolve account/access issues where permitted

Detailed Admin permissions should be finalized as part of the complete RBAC definition.

---

## 7. Role-Based Module Boundaries

At a high level, access boundaries should follow the ownership of the two IMS modules.

```text
                    Shared IMS
                       |
          +------------+------------+
          |                         |
          v                         v
     Team A Module              Team B Module
 Recruitment/Application       Task & Progress
          |                         |
       Applicant             Intern / Mentor
          \                         /
           \                       /
            +------ Admin --------+
```

### Team A

Team A owns Recruitment functionality up to the Accepted decision.

Team A functionality should primarily operate on:

- Shared account/profile information where required
- Application/recruitment information
- Application status
- Applicant workflow

### Team B

Team B owns active Intern task and progress functionality.

Team B functionality should primarily operate on:

- Shared user identity references
- Tasks
- Task Submissions
- Weekly Progress
- Mentor/Intern interactions related to task and progress management

Shared authentication and user/profile infrastructure should not be duplicated by either module.

---

## 8. Team A / Team B Integration

Team A and Team B are expected to operate within the same IMS and therefore require a shared integration boundary.

The primary integration dependency is the Applicant-to-Intern handoff.

### Integration Flow

```text
Team A
Recruitment
   |
   v
Application Accepted
   |
   v
Applicant-to-Intern Handoff
   |
   +------ Shared Account
   |
   +------ Shared Person Profile
   |
   +------ Role / Access Transition
   |
   v
Team B
Intern Task & Progress Workflow
```

Team B should rely on shared identity information supplied by the common IMS foundation rather than copying Recruitment data into Team B-specific user records.

Only information required by Team B should be referenced or consumed across the module boundary.

---

## 9. Shared Data Dependencies

Team B depends on shared IMS data for user identification and authorization.

Conceptually:

```text
Shared Authentication
        |
        v
Shared Account
        |
        +------ Role
        |
        v
Person Profile
        |
        +-----------------------------+
        |                             |
        v                             v
Team A Recruitment              Team B Task/Progress
```

Team B entities should reference shared identities where required.

Examples include:

```text
Task
  -> assigned Intern
  -> assigning Mentor

Task Submission
  -> Task
  -> submitting Intern

Weekly Progress
  -> Intern
```

This allows Team B to maintain its own domain data without creating a second identity model.

---

## 10. Authentication and Authorization Boundaries

Authentication and authorization should be treated as separate responsibilities.

### Authentication

Authentication answers:

> Who is making this request?

Authentication should be handled through the shared IMS authentication mechanism.

### Authorization

Authorization answers:

> Is this authenticated user allowed to perform this action?

Authorization should evaluate the authenticated user's role and, where necessary, their relationship to the requested resource.

For example, being an Intern does not necessarily mean an Intern should be able to update another Intern's task.

Therefore, some authorization rules may require both:

```text
Role Check
+
Resource Ownership / Relationship Check
```

Detailed ownership rules should be implemented when the corresponding requirements are confirmed.

---

## 11. Backend Integration Approach

Authentication and RBAC should be implemented as shared backend concerns rather than separately inside each feature module.

Conceptually, the NestJS backend may use shared components such as:

```text
auth/
  auth.module
  authentication guard / middleware

authorization/
  role definitions
  role guard
  access policies

shared/
  account/profile integration

tasks/
task-submissions/
weekly-progress/
```

Feature modules should rely on the authenticated user context and shared authorization rules.

For example:

```text
Authenticated Request
        |
        v
Authentication Guard
        |
        v
RBAC / Access Check
        |
        v
Task / Submission / Progress Module
```

The exact NestJS file names, guards, decorators and DTO structures should be determined during implementation and should remain consistent with the final shared authentication approach.

---

## 12. Frontend Integration Approach

The frontend should use the authenticated user's role to provide the appropriate IMS experience.

Conceptually:

```text
Login
  |
  v
Authenticated User
  |
  v
Load Shared Account / Role
  |
  +------ Applicant -> Recruitment
  |
  +------ Intern ----> Intern Workspace
  |
  +------ Mentor ----> Mentor functionality
  |
  +------ Admin -----> Administrative functionality
```

Frontend routing and interface visibility should reflect RBAC decisions, but backend authorization must remain the source of enforcement.

---

## 13. Confirmed Integration Decisions and Working Assumptions

The following decisions and working assumptions form the current technical baseline:

1. Team A and Team B operate as modules within one IMS.
2. Authentication should be shared across both modules.
3. A user should maintain one account and profile across the lifecycle.
4. An accepted Applicant should transition into the Intern workflow without creating another account.
5. Application status and account role are separate concepts.
6. Team A owns Recruitment through the Accepted decision.
7. Team B owns the active Intern task/progress workflow.
8. Team B should reuse the shared IMS account/profile foundation.
9. Task Submission and Weekly Progress remain separate Team B concepts.
10. Authorization should be enforced by the backend.
11. Frontend role-based visibility should reflect, but not replace, backend authorization.
12. A separate duplicate onboarding account/system should not be introduced.

---

## 14. Pending Technical Decisions

The following items should remain open until confirmed:

- Exact authentication implementation and session/token handling
- Final list and naming of system roles
- Detailed Applicant permissions
- Detailed Intern permissions
- Detailed Mentor permissions
- Detailed Admin permissions
- Exact Mentor-to-Intern responsibility relationship
- Exact authorization rules for individual resources
- Exact mechanism for Applicant-to-Intern role transition
- Who is authorized to perform the role transition
- Whether additional lifecycle states or roles are required
- Final frontend route protection approach
- Exact backend guards, decorators and access-policy implementation
- Any changes resulting from pending client or cross-team feedback

These items should not be treated as final implementation requirements until confirmed.

---

## 15. Technical Impact of Pending Feedback

Pending feedback may affect authentication, role transitions, access boundaries or cross-team workflow details.

The architecture therefore separates stable shared principles from implementation details that remain open.

Stable principles include:

- One shared IMS identity
- Shared authentication
- No duplicate Applicant/Intern account
- Separation of role and application status
- Team A/Team B ownership boundary
- Backend authorization
- Reuse of shared profile/account information

Implementation-specific details should be updated when confirmed without requiring the core shared architecture to be redesigned.

---

## 16. Architecture Summary

The shared IMS authentication and RBAC architecture should provide one continuous identity across the Applicant and Intern lifecycle.

Team A manages the Recruitment workflow through acceptance, while Team B manages the active Intern task and progress workflow. Both modules should rely on shared authentication and shared account/profile information.

The Applicant-to-Intern transition should preserve the same account and profile while changing the user's access as required. Application status should remain separate from the user's system role.

RBAC should enforce module and resource access on the backend, with frontend role-based routing and visibility reflecting the same access model.

Detailed permissions, transition authorization, Mentor relationships and implementation-specific authentication mechanisms remain open until confirmed.
