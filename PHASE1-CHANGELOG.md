# Phase 1 – Security and foundation repair

## Added

- `00005_security_hardening.sql`: tightened application, employer, job, CV, storage, payment, audit-log and public-settings permissions.
- `POST /api/applications`: authenticated, Zod-validated application submission. Candidate identity comes from the verified Supabase session.
- `GET /api/applications/[applicationId]/cv`: authorised short-lived signed CV URL endpoint.
- Server-side `requireStaff()` and `requireAdmin()` guards.
- Server-side admin guards for the admin landing and security pages; staff-only login completion.
- Shared Zod schemas for applications, CV metadata, contact submissions, candidate profiles and jobs.
- Vitest tests for validation and utility regressions.
- Supabase live-security test checklist.
- ESLint flat config, typecheck/test scripts, safer environment parsing and baseline HTTP security headers.

## Important permission changes

- Normal candidates may read and submit applications but cannot update or delete them.
- Application candidate/job/CV identity fields are immutable after submission.
- Employers can only update application status and employer notes for jobs they own.
- Employer-owned job/company deletion is no longer allowed through employer RLS policies.
- Employer operations require `public.users.role = 'employer'`.
- Only verified employers' active, unexpired jobs are publicly visible and eligible for applications.
- Moderators can moderate job status, employer verification and application status/notes, but cannot change user roles, job ownership/content, employer ownership/company data, payment data or audit logs.
- Administrator role changes remain a trusted operational action; no browser-side role assignment is provided.
- CV object reads are limited to the candidate, the employer owning a job to which the CV was attached, and administrators. Moderators do not get CV document access by default.
- Candidate CV objects cannot be renamed/deleted while referenced by an application.

## Apply and verify

1. Back up the Supabase database before applying migrations.
2. Apply `00005_security_hardening.sql` after migrations `00001`–`00004` in a disposable/staging project first.
3. Ensure each employer's public.users row has the controlled `employer` role; ordinary users must not be able to set this role themselves.
4. Run `npm install` in a network-enabled development environment to generate `package-lock.json`.
5. Run `npm run typecheck`, `npm run lint`, `npm test`, and `npm run build`.
6. Run every scenario in `tests/security-checklist.md` against Supabase with separate candidate, employer, moderator and administrator identities.
7. Only then promote the migration to production.

## Not verified in this environment

- The npm install attempts timed out, so no lockfile was generated and package-based checks could not run.
- The TypeScript files passed compiler transpilation/syntax checks, but a full typecheck was not possible without installed dependencies.
- PostgreSQL/pgTAP and a Supabase project were unavailable, so migration execution and live RLS/storage tests remain unverified.
- The CV upload pipeline, file-signature validation, malware scanning and app-level rate limiting are not implemented in this phase.

## Phase 5 – Admin dashboard

- Added an administration overview with operational counts and role-aware navigation.
- Added administrator-only user role management backed by `admin_set_user_role`; moderators cannot assign roles.
- Added job moderation, consultancy management, scholarship and grant CRUD, editorial post management and public site settings editing.
- Added report/fraud review workflow and administrator-only audit log viewer.
- Added append-only database audit triggers for user, employer, employer onboarding, job, application, scholarship, grant, post, report and site-setting changes. Audit metadata records changed field names rather than sensitive row values.
- Added server-side role checks, same-origin API checks and Zod validation for admin operations.
- Added migration `00008_admin_dashboard.sql` and admin validation tests/checklist.

Phase 5 has not been run against a live Supabase database. In particular, execute migration `00008` on staging and verify RLS, RPC grants, audit trigger behavior and role boundaries before deployment.
