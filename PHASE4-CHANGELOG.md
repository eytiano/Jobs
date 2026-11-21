# Phase 4 – Employer dashboard

## Delivered
- Employer onboarding request for existing candidate-authenticated accounts. The public signup trigger continues to assign the candidate role; no browser metadata can create an employer account.
- Employer registration details: organisation name, registration number, website, contact email and description.
- Staff verification queue at `/admin/employers`. Staff review uses a guarded database RPC; approval promotes the account and creates a verified employer record atomically. Rejections retain reviewer notes.
- Company profile editor. Changes to company profile fields automatically return verification to `pending` so updated identity details are reviewed again.
- Employer dashboard at `/employer` with company profile, job posting, job list, application list and summary counts, with database-computed totals rather than just the visible 200-application list.
- Job creation API validates all fields server-side and only creates `draft` jobs. Public activation remains controlled by moderation/payment workflows and existing Phase 1 protections.
- Application management with allowed status updates (`viewed`, `shortlisted`, `rejected`, `hired`) and employer notes validation.
- CV downloads reuse the Phase 1 application-specific authorization RPC and short-lived signed URLs. Employers can only download CVs attached to applications for their own jobs.
- Employer contact information is exposed through a narrowly scoped RPC, not broad candidate user-table access.
- Employer dashboard and verification queue routes are server-gated; APIs enforce role, same-origin and Zod validation.

## New migration
`supabase/migrations/00007_employer_dashboard.sql` – apply after migrations 00001 through 00006.

## Account flow
1. A new organisation contact first creates a normal mwaiHub account (the Auth trigger assigns `candidate`).
2. While signed in, they submit `/employer/register`.
3. Admin/moderator reviews the request at `/admin/employers`.
4. Approval changes the account role to `employer` and creates a verified employer record. The user can then sign in and use `/employer`.

## Deliberate restrictions
- Employers cannot self-verify, promote their own role, activate jobs, or bypass the job moderation/payment workflow.
- New job posts are saved as drafts; a staff/payment workflow must activate them.
- Editing company identity/profile details resets verification to pending.
- Employer analytics in this phase are basic counts (jobs, active jobs, applications and shortlisted applications), not traffic/conversion analytics.
- CV downloads are restricted to applications on the employer's own jobs and signed URLs expire after 60 seconds.

## Verification status
Static syntax and package/ZIP checks are performed during packaging. A full dependency install, typecheck, lint, automated test run, production build, migration execution and live Supabase RLS/storage tests require a configured staging environment and were not available in this environment.
