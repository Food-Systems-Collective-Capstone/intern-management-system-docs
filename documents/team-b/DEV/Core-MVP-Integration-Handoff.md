# Core MVP Workflow and Navigation Integration Handoff

## Purpose

This document provides the DEV handoff for BA acceptance testing following the final Sprint 2 Core MVP integration.

The integrated Team B workflow, navigation, frontend/backend data flow, Weekly Progress workflow, and applicable post-client UX v2 changes have been reviewed and tested.

The purpose of this handoff is to identify what is ready for BA acceptance testing, what integration issues were addressed by DEV, and what remains deferred to Sprint 3.

---

## 1. Integrated Core MVP Flow

The following Core MVP task lifecycle has been tested end-to-end:

`Assigned → In Progress → Submitted → Completed`

The verified flow is:

1. Mentor creates and assigns a task to an Intern.
2. The new task is created with Assigned status.
3. Intern views the task through My Tasks and Task Detail.
4. Intern starts the task, changing the status from Assigned to In Progress.
5. Intern submits task work with submission content and/or attachment.
6. The task changes from In Progress to Submitted.
7. Mentor views the submitted work through Review Progress.
8. Mentor marks the eligible submitted task as Completed.
9. The Completed status remains visible to the Intern and Mentor.
10. Submitted work remains visible after task completion.

Weekly Progress operates separately from individual Task Submissions.

The verified Weekly Progress flow is:

1. Intern opens Weekly Progress through the Intern navigation.
2. Intern submits Accomplishments, Blockers, and Next Steps.
3. The report is persisted through the backend to the `weekly_progress` table.
4. An existing submission for the same stored `reporting_week` is protected against duplicate submission.
5. The submitted state remains available after the data is retrieved again.
6. Mentor can view the Intern's Weekly Progress as read-only through Review Progress.

---

## 2. Integration Verification

| Integrated Function | Status | DEV Verification |
| --- | --- | --- |
| Mentor Task Assignment | PASS | Mentor can create a task and assign it to an Intern through the backend. New tasks begin with Assigned status. |
| Mentor Reference Attachment | PASS | Optional Mentor reference attachment is uploaded through the backend to Supabase Storage and associated with the task. The attachment can be retrieved again through a signed URL. |
| Intern My Tasks | PASS | Assigned tasks are retrieved from live backend task data and displayed to the Intern. |
| Intern Task Detail | PASS | Intern can view task information and Mentor reference attachment where available. |
| Assigned → In Progress | PASS | Start Task updates the task from Assigned to In Progress through the backend. |
| Task Submission | PASS | Intern can submit task work and an attachment. Successful submission changes the task to Submitted. |
| Mentor Review | PASS | Mentor can retrieve submitted task information and submission attachment information through Review Progress. |
| Submitted → Completed | PASS | Mentor can explicitly complete an eligible Submitted task. Completed state persists and submitted work remains visible. |
| Weekly Progress Submission | PASS | Accomplishments, Blockers, and Next Steps are persisted separately from Task Submission. |
| Weekly Progress Duplicate Protection | PASS | Backend/database persistence prevents another record for the same Intern and stored `reporting_week` value. |
| Mentor Weekly Progress View | PASS | Mentor Review retrieves and displays the selected Intern's Weekly Progress as read-only. |
| Task Summary Metrics | PASS | UX v2 summary values are calculated from retrieved task data rather than hardcoded summary values. |
| Core MVP Navigation | PASS | Mentor and Intern Core MVP screens are connected through application navigation, including Intern Weekly Progress. |
| Mentor–Intern Relationship Enforcement | DEFERRED | Final Mentor-to-Intern relationship enforcement depends on the shared Team A/B account and RBAC integration. |
| Final Authenticated User Identity | DEFERRED | Temporary Team 40 Mentor/Intern test identities are used for current integration testing. Final identity must come from the shared authentication/account flow. |
| Reporting Week Convention | ISSUE | The database protects uniqueness for `(intern_id, reporting_week)`, but the exact reporting-week date convention still requires confirmation before claiming calendar-week normalisation. |

---

## 3. Integration Issues Found and Addressed

### Mentor Review Submission Attachment Handling

Mentor Review can encounter a stored submission `file_url` whose corresponding Supabase Storage object is unavailable.

Signed URL generation is now handled per attachment so an unavailable Storage object does not cause the complete Mentor Review request to fail.

If the attachment cannot be resolved, the task and submission information can still be returned while the attachment URL is treated as unavailable.

A valid submission attachment was also tested successfully.

The exact previous stale `Object not found` scenario was not destructively reproduced during the final verification, so this handoff records the defensive fix and valid attachment verification rather than claiming exact reproduction of the historical Storage state.

### Core Navigation Merge Integration

The latest frontend `main` branch introduced additional shared Team A routes while the Team B branch introduced the Intern Weekly Progress route.

The resulting `app/routes.ts` merge conflict was resolved by retaining both the latest shared routes and the Team B Core MVP routes.

The merged frontend passed the local TypeScript and diff checks after resolution.

### Post-Client UX v2 Alignment

The affected Team B screens were updated against the approved post-client UX v2 flow, including:

- Intern Workspace
- Intern My Tasks
- Intern Task Detail
- Intern Weekly Progress
- Mentor Assign Task
- Mentor Review Progress
- Task Summary information
- Completed task/submission visibility
- Optional Mentor reference attachment

No additional optional Sprint 3 functionality was introduced as part of this integration task.

---

## 4. Relevant Implementation PRs

### Frontend

**Weekly Progress Flow and UX v2 Updates - PR #10**

https://github.com/Food-Systems-Collective-Capstone/intern-management-system-frontend/pull/10

Previous Team B Core MVP frontend implementation is also represented by the earlier Task Management and Mentor Review implementation PRs.

### Backend

**Weekly Progress and Task Reference Attachment Flow - PR #29**

https://github.com/Food-Systems-Collective-Capstone/intern-management-system-backend/pull/29

Previous Team B Core MVP backend implementation is also represented by the earlier Task Assignment, Task Management, Task Submission, and Mentor Review/Completion implementation PRs.

---

## 5. Known Sprint 3 Dependencies

The following items are intentionally not treated as completed Sprint 2 integration work.

### Final Team A / Team B Account Integration

The intended shared IMS flow remains:

`Register → Applicant → Recruitment/Application → Accepted → Intern → Team B Intern Workspace`

Team B should continue to reuse the shared account/profile foundation rather than create a separate permanent user model.

Temporary Team 40 Mentor/Intern accounts are currently used to support Team B integration testing.

### Mentor–Intern Relationship and RBAC

The current Mentor Weekly Progress flow validates the relevant shared accounts but does not yet enforce a final Mentor-to-Intern relationship.

The final relationship model and route/API authorisation should be implemented with the shared Team A/B authentication, account, and RBAC integration rather than introducing a separate Team B-only relationship model.

### Authenticated Identity

Current Team B integration testing uses temporary known test identities.

Final Mentor and Intern identity should be derived from the authenticated shared account rather than temporary test IDs.

### Reporting Week Convention

Weekly Progress currently persists a `reporting_week` date and enforces uniqueness using:

`(intern_id, reporting_week)`

The final convention for the reporting-week date should be confirmed before calendar-week normalisation is treated as final behaviour.

---

## 6. Note for Next Role - BA

The Core MVP Team B workflow is ready for BA acceptance testing against the current Sprint 2 scope.

BA can validate the integrated user flow:

`Mentor Assign Task → Intern My Tasks → Start Task → Submit Task → Mentor Review → Complete Task`

BA can also validate Weekly Progress separately through:

`Intern Weekly Progress → Submit Progress → Submitted/Locked State → Mentor Review Progress`

Acceptance testing should confirm the expected task information, navigation, status transitions, submission visibility, Weekly Progress behaviour, and approved UX v2 screen behaviour.

The Sprint 3 dependencies documented above are outside the current Sprint 2 acceptance baseline and should be recorded as known deferred dependencies rather than Sprint 2 implementation failures. In particular, final shared authenticated identity and Mentor–Intern relationship/RBAC integration remain dependent on the shared Team A/B account integration.

---

## Status

**Core MVP Integration: Ready for BA Acceptance Testing**

Prepared by: **Chan Hoang Truong - s3878262**
Role: **DEV - Team B**
