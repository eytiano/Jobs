# Phase 3 – Authentication and candidate dashboard

## Delivered
- Supabase Auth candidate registration, login and email-confirmation callback.
- Trusted `auth.users` trigger creates a public user record with the candidate role; signup metadata cannot assign privileged roles.
- Candidate dashboard with editable profile and optional phone-sharing preference.
- Private CV upload (PDF/DOC/DOCX, maximum 5 MB), basic file-signature checks, metadata ownership, short-lived signed downloads and deletion restrictions for CVs attached to applications.
- Job detail application form using a candidate-owned CV and the existing server-side application endpoint.
- Candidate application tracking and saved-job management.
- Keyword/location job alerts and in-app notifications for application submissions, status changes and newly activated matching jobs.
- New RLS policies for candidate profiles, CV metadata, saved jobs, alerts and notifications; same-origin checks on mutation APIs.

## New migration
`supabase/migrations/00006_candidate_dashboard.sql` – apply after migrations 00001–00005.

## Not configured / operational follow-up
- Email notification delivery and scheduled daily/weekly digest processing are not configured. Matching jobs and application events currently create in-app notifications.
- No malware scanning or document-content parsing service is configured; file checks validate extension/MIME and basic signatures only.
- No production rate limiter is configured for authentication, CV upload or alert creation.
- Supabase email templates, SMTP/provider settings, redirect allow-list and production site URL must be configured in the Supabase project.
- Full dependency install, TypeScript check, ESLint, Vitest, Next.js build and live RLS/storage tests were not run because package installation timed out in this environment.
