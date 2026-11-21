# Phase 2 – Public website

## Delivered
- Rebuilt the homepage with responsive navigation, opportunity categories, recent listings and safety messaging.
- Added directories for jobs, consultancies, scholarships and grants, with keyword, location and category/study-level filters.
- Added dynamic detail pages with metadata, official application links, deadlines and safety reminders.
- Added About, Contact, Privacy, Terms and Safety/Job Posting Guidelines pages.
- Added shared public site header/footer, listing cards, directory and detail components.
- Added title/description metadata, Open Graph/Twitter defaults, canonical detail URLs, sitemap and robots routes.
- Public data uses the existing Supabase schema and anon client; queries rely on the existing RLS policies.

## Notes and limitations
- Public listings depend on published/active records in Supabase. Empty directories are expected until approved records exist.
- Consultancies are represented by `jobs.listing_type = 'consultancy'`, as established in migration `00004_security_and_content.sql`.
- Contact page currently uses an email link rather than a database-backed contact form. A form requires a server-side service-role insert or mail delivery implementation and rate limiting.
- Privacy and terms pages are starter drafts, not legal advice; obtain local legal review and align them with actual data practices.
- Set `NEXT_PUBLIC_APP_URL` to the production domain to ensure canonical metadata, sitemap and robots URLs are correct.
- Phase 2 has not been run through a full Next.js build, lint or browser test in this environment because dependency installation/network access is unavailable. Re-run `npm install`, `npm run typecheck`, `npm run lint`, `npm test` and `npm run build` before deployment.

## Phase 3 – Candidate authentication and dashboard

- Added candidate registration/login using Supabase Auth and a trusted auth.users trigger that creates candidate-only public user rows.
- Added candidate profile editing, private CV upload/download/delete flows, job applications and application status tracking.
- Added saved jobs, keyword/location alerts, and an in-app notification centre.
- Added database RLS policies and triggers for candidate-owned data, application updates and matching active jobs.
- Email delivery/digests, scheduled alert dispatch and malware scanning are not configured in this phase; job matches and application events are delivered in-app.
