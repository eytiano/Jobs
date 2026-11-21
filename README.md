# mwaiHub

A Malawi-focused opportunities platform foundation built with Next.js App Router, TypeScript, Tailwind CSS and Supabase.

## Requirements

- Node.js 20 LTS or later
- npm 10 or later
- A Supabase project (or local Supabase CLI stack for security integration testing)

## Setup

1. Create a Supabase project.
2. Apply migrations in order: `00001_initial_schema.sql` through `00009_payments_automation.sql`.
3. Copy `.env.local.example` to `.env.local` and set the project URL, anon key and application URL.
4. Install dependencies with `npm install`.
5. Run `npm run dev`.

Never expose `SUPABASE_SERVICE_ROLE_KEY` to browser code or commit a real `.env` file. The Phase 1 application and CV endpoints use the signed-in user's Supabase session and database policies; they do not need a service-role key.

## Phase 1 security hardening

Migration `00005_security_hardening.sql`:

- Removes broad candidate application update/delete access.
- Restricts application inserts to the authenticated candidate's own CV and active jobs from verified employers.
- Freezes application identity fields and restricts employer updates to status/notes.
- Prevents employers from deleting company/job rows through broad `FOR ALL` policies.
- Requires verified employers for public job visibility.
- Separates administrator and moderator privileges for roles, CV metadata, payments and audit logs.
- Adds employer-authorised CV object access and a 60-second signed-download flow.
- Restricts public site settings to the approved public keys.
- Adds missing content-table `updated_at` triggers and supporting indexes.

The API endpoints added are:

- `POST /api/applications` – authenticated application submission; candidate identity is taken from the session, never the request body.
- `GET /api/applications/:applicationId/cv` – short-lived signed CV download for the candidate, owning employer or administrator.

The admin security page and admin landing page perform server-side staff checks. Middleware is an additional navigation gate, not the sole authorisation layer. Future admin server actions and API routes must call `requireStaff()` (or `requireAdmin()` for administrator-only operations) themselves. Employer-owned operations require `public.users.role = 'employer'`; role assignment is an administrative operation and must never be exposed as a self-service profile field.

## Quality checks

- `npm run typecheck`
- `npm run lint`
- `npm test`
- `npm run build`

The Vitest suite covers validation and utility regressions. `tests/security-checklist.md` contains the required live Supabase/RLS integration scenarios. Unit tests do not replace those database tests.

## Important remaining operational work

- Run the migration and security checklist against a disposable Supabase project before production. Grant the initial administrator/employer roles through a trusted SQL editor or secured server-side admin operation, never from browser input.
- Configure Supabase Auth redirect URLs and provider rate limits.
- Add rate limiting to public/authenticated API endpoints before launch.
- Validate uploaded CV file signatures and scan files for malware before accepting production uploads. The current Phase 1 change secures access/downloads; it does not introduce the full upload pipeline.
- Configure a production `NEXT_PUBLIC_APP_URL`, email delivery, backups and monitoring.
- Create and commit `package-lock.json` after a successful `npm install` in a network-enabled development environment.

## Phase 2 – Public website

The public website now includes the homepage, searchable directories for jobs, consultancies, scholarships and grants, listing detail pages, About, Contact, Privacy, Terms and Safety/Posting Guidelines pages. Public listings are read through the Supabase anon client and remain subject to the public RLS policies; jobs and consultancies are restricted to active listings from verified employers. Search filters are URL-based so results can be shared and indexed.

SEO includes route metadata, Open Graph/Twitter defaults, canonical URLs on listing details, and generated `/sitemap.xml` and `/robots.txt` routes. Set `NEXT_PUBLIC_APP_URL` to the production canonical origin before deployment. The legal pages are starter copy and require review by the platform operator and qualified local counsel before launch. Contact currently provides a direct email path; a persisted contact form should be added only with a server-side delivery/storage path and abuse controls.

## Candidate accounts (Phase 3)

- Candidate registration: `/register`; sign-in: `/login`; workspace: `/candidate`.
- Apply from a job detail page after signing in and uploading a CV. CV files are stored in the private `cvs` bucket and downloaded through short-lived signed URLs.
- Candidate profile, CV metadata, applications, saved jobs, alerts and notifications are protected by row-level security.
- Apply `supabase/migrations/00006_candidate_dashboard.sql` after migrations `00001`–`00005`.
- Application status changes and new matching active jobs create in-app notifications. Email notifications/digests, scheduled alert dispatch and malware scanning require separately configured services and are not represented as enabled.

## Phase 4 – Employer dashboard

Apply `supabase/migrations/00007_employer_dashboard.sql` after `00001`–`00006`.

Employer onboarding uses the normal candidate signup first: create an account or sign in, then visit `/employer/register` to submit organisation details. Staff review requests at `/admin/employers`. Approval creates the verified employer account and enables `/employer`. Employers can edit their company profile, create job drafts, review applications, update application status and download CVs attached to their own vacancies. Job drafts still require the existing moderation/payment activation workflow before becoming public.


## Phase 5 – Admin dashboard

- Administration overview: `/admin`; users: `/admin/users`; employer verification: `/admin/employers`; job moderation: `/admin/jobs`; consultancy management: `/admin/consultancies`; scholarships: `/admin/scholarships`; grants: `/admin/grants`; editorial content and public settings: `/admin/content`; reports/fraud: `/admin/reports`; audit history: `/admin/audit`.
- Moderators can review employers, moderate job status, manage scholarship/grant/editorial records under the existing staff policies, and resolve reports. Only administrators can manage user roles, create/edit/delete consultancy listings, delete content records, edit public settings and read audit logs.
- `00008_admin_dashboard.sql` adds the guarded role-change RPC, append-only audit triggers and staff report/contact visibility. Audit metadata stores changed field names, not old/new field values.
- Run `tests/admin-security-checklist.md` with separate candidate, employer, moderator and administrator accounts after applying the migration. The automated tests cover input validation only; live database permission tests remain required.

## Phase 6 – payments and automation

Apply `supabase/migrations/00009_payments_automation.sql` after migrations `00001`–`00008`.

### PayChangu setup

1. Create a PayChangu merchant account and copy the **secret API key** from Settings → API Keys & Webhooks into `PAYCHANGU_SECRET_KEY` (server-side only).
2. Set `PAYCHANGU_WEBHOOK_SECRET` to the webhook secret generated in the PayChangu dashboard.
3. Configure the PayChangu webhook URL as `https://YOUR_DOMAIN/api/payments/webhook` and enable payment notifications.
4. Set `NEXT_PUBLIC_APP_URL` to the public HTTPS origin of this deployment.
5. Start with PayChangu test credentials/mode, then switch to live credentials only after end-to-end verification.

Checkout uses PayChangu Standard Checkout. The return endpoint and signed webhook both call PayChangu's transaction verification endpoint; the app checks the exact transaction reference, success status, currency and amount before recording payment. A successful payment moves a job to `pending_review`; it does **not** bypass staff moderation. Once staff activates the paid job, its package duration starts and the scheduled expiry timestamp is set.

### Packages, receipts and email

Administrators configure prices and durations at `/admin/packages`. No package prices are seeded because mwaiHub's approved price list must be supplied by the platform operator. Employers pay from `/employer/billing`. Receipts are available from payment history and can be printed or saved as PDF from the browser. Configure `RESEND_API_KEY` and `RESEND_FROM_EMAIL` for application, employer and payment receipt emails.

### Scheduled automation

Configure a strong random `CRON_SECRET`. `vercel.json` schedules `/api/cron/phase6` hourly; the route requires `Authorization: Bearer $CRON_SECRET`. It expires paid listings, polls pending PayChangu transactions as a webhook fallback, sends daily/weekly job alert digests, and drains the email outbox. If hosting outside Vercel, configure an hourly scheduler to call the same endpoint with that header.

### Reporting

Administrators can view verified payment totals and recent transactions at `/admin/payments`. Employer dashboards retain application/job counts and link to payment history. Revenue totals shown in the admin report are MWK-only; USD transactions remain visible in the transaction ledger but are not converted.

### Production checklist

- Confirm package names, MWK/USD amounts and listing durations with the business owner before activating packages.
- Test PayChangu checkout, success, failure, duplicate webhook delivery, delayed webhook/polling, mismatched amount/currency/reference and job moderation in sandbox.
- Configure the Resend sender domain and verify delivery/retry behaviour.
- Set the cron secret in the hosting environment and confirm hourly invocation.
- Apply the migration to a staging database first and review the resulting RLS grants/policies.


## Phase 7 – testing and launch readiness

- Run `npm run check:launch` with the intended production environment variables loaded before deployment. The checker validates required configuration and rejects obvious placeholders; it does not connect to Supabase, PayChangu, Resend or the hosting provider.
- Run `npm run typecheck`, `npm run lint`, `npm test` and `npm run build` in CI and resolve all failures.
- Follow `docs/PHASE7-LAUNCH-PLAN.md`, `docs/SECURITY-REVIEW.md` and `docs/SEO-VALIDATION.md`. Live RLS, device/browser, Lighthouse, PayChangu sandbox and production smoke tests require the real staging/production services and must be recorded by the release owner.
- Production deployment is intentionally not performed from this source package. The operator must configure secrets, staging database, domain/DNS, hosting project, backups, monitoring and rollback ownership before pressing the production deploy button.
