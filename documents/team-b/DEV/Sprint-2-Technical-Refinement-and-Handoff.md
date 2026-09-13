# Sprint 2 Technical Refinement and Development Handoff

## Purpose

This document finalises the Team B technical baseline following Sprint 1 Playback and prepares the development handoff for Sprint 2 Core MVP implementation.

The existing database structure, final Team B requirements, shared IMS dependencies, and frontend/backend/database development path were rechecked and refined based on the confirmed client and PM decisions.

---

## 1. Confirmed Team B Core MVP Requirements

The following requirements are confirmed for Sprint 2 implementation.

### Task Workflow

The confirmed task lifecycle is:

`Assigned → In Progress → Submitted → Completed`

- Interns progress tasks from Assigned to In Progress.
- An Intern submission moves the task to Submitted.
- A Mentor reviews the submission and confirms Submitted to Completed.
- Tasks include Priority.
- Tasks retain the Assigned By/Mentor relationship.

### Task Submission

Task Submission remains separate from Weekly Progress.

Minimum Task Submission content:

- Description
- File attachment

### Weekly Progress

Weekly Progress is one overall report per Intern per reporting week.

Minimum fields:

- Accomplishments
- Blockers
- Next Steps

Weekly Progress remains separate from individual Task Submissions.

---

## 2. Database Refinement

The Team B database structure was rechecked against the confirmed Core MVP requirements.

### tasks

The `tasks` table supports:

- Task title
- Description
- Due date
- Status
- Priority
- Assigned Intern
- Assigned By/Mentor
- Created and updated timestamps

The current status field can support the confirmed lifecycle:

`Assigned → In Progress → Submitted → Completed`

Application-level validation and Mentor confirmation logic will be implemented during Sprint 2.

### task_submissions

The Task Submission structure was refined to support the confirmed minimum content.

Current relevant fields include:

- `task_id`
- `submitted_by_intern_id`
- `description`
- `file_url`
- `submitted_at`
- `created_at`
- `updated_at`

The previous `submission_text` field was renamed to `description`, and `file_url` was added to support file attachment references.

Actual file storage and upload handling remains a Sprint 2 implementation dependency.

### weekly_progress

The Weekly Progress structure supports:

- Intern
- Reporting week
- Accomplishments
- Blockers
- Next Steps
- Created and updated timestamps

A database UNIQUE constraint was added for:

`(intern_id, reporting_week)`

This enforces a maximum of one Weekly Progress record for the same Intern and reporting week.

### weekly_progress_tasks

The `weekly_progress_tasks` junction table provides the ability to associate Weekly Progress reports with relevant tasks.

This supports the client recommendation to link Weekly Progress with assigned tasks while keeping the Core MVP Weekly Progress report separate from individual Task Submissions.

---

## 3. Shared Team A / Team B Integration

Team B continues to use the existing shared IMS account and profile foundation.

Team B entities reference `shared_accounts` rather than creating a separate Team B user/account structure.

The intended cross-team flow remains:

`Register → Applicant → Recruitment/Application → Accepted → Intern → Team B Intern Workspace`

The same shared account and profile are retained through the Applicant-to-Intern transition.

### Ownership Boundary

**Team A**

- Recruitment and application workflow
- Applicant information
- Accepted decision and handoff

**Team B**

- Intern task management
- Task Submission
- Weekly Progress
- Mentor review workflow

Application status remains separate from account role.

No duplicate Team B account is required after an Applicant becomes an Intern.

---

## 4. Development Path Revalidation

The Sprint 2 development path was revalidated after the database refinements.

### Backend to Database

The NestJS backend successfully starts with the existing TypeORM/PostgreSQL configuration.

Verification confirmed:

- NestJS application starts successfully
- TypeORM dependencies initialise successfully
- PostgreSQL/Supabase connection initialises without errors
- Backend development server is available on `localhost:3000`

### Frontend to Backend

A temporary development connectivity test was performed between the React Router frontend and NestJS backend.

Verification result:

- Frontend available on `localhost:5173`
- Backend available on `localhost:3000`
- Backend response received: `Hello World!`
- Connection status: `SUCCESS`

The temporary connectivity-test code was restored after verification and was not retained as production implementation.

---

## 5. Remaining Sprint 2 Technical Dependencies

The technical foundation is ready for Core MVP feature implementation. The following items remain Sprint 2 implementation dependencies:

- Detailed backend RBAC rules
- Supabase Row Level Security policies for Team B tables
- Version-controlled migration for the refined Team B database schema
- Permanent frontend/backend integration and CORS configuration
- Task status transition validation and Mentor completion confirmation
- Task Submission file storage and upload handling
- API DTO and validation implementation

These items do not block the technical handoff but must be addressed during Sprint 2 development.

---

## 6. Sprint 2 Development Handoff

The Team B technical baseline has been refined against the final Core MVP requirements.

Sprint 2 development can proceed using:

- Existing shared IMS authentication/account/profile foundation
- Refined Team B database structure
- Confirmed Task workflow
- Separate Task Submission and Weekly Progress models
- One Weekly Progress report per Intern per reporting week
- Existing NestJS to PostgreSQL/Supabase connectivity
- Verified frontend to backend development path

The recommended implementation priority is to complete the Core MVP Task, Task Submission, Weekly Progress, Mentor review, RBAC, and API integration before optional enhancements.

---

## Status

**Sprint 2 Technical Baseline: Ready for Implementation**

Prepared by: **Chan Hoang Truong - s3878262**  
Role: **DEV - Team B**
