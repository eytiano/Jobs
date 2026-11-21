# mwaiHub Phase 6 – Payments and automation

## Included
- Admin-managed payment packages with MWK/USD amounts and configurable listing duration.
- PayChangu Standard Checkout session creation using server-side credentials.
- Signed PayChangu webhook validation and server-side transaction verification on both webhook and return flows.
- Idempotent payment reference/receipt records; exact reference, status, currency and amount checks.
- Paid jobs move into `pending_review`; moderator/admin activation is still required. Paid package duration begins on activation and sets listing expiry.
- Employer payment history and print/save-as-PDF receipt page.
- Email outbox with retry attempts for application and payment receipt notifications; employer application emails.
- Daily/weekly job alert email dispatch, hourly job expiry and pending-payment verification fallback.
- Admin payment/revenue report and hourly Vercel cron configuration.

## Configuration required
- `SUPABASE_SERVICE_ROLE_KEY`
- `PAYCHANGU_SECRET_KEY`
- `PAYCHANGU_WEBHOOK_SECRET`
- `NEXT_PUBLIC_APP_URL`
- `RESEND_API_KEY`
- `RESEND_FROM_EMAIL`
- `CRON_SECRET`
- Approved package pricing entered at `/admin/packages`

## Operational limitations
- No package prices are pre-seeded; the platform owner must enter approved amounts.
- Receipt delivery is email-queued; PayChangu may also send its own provider receipt depending on merchant settings.
- USD payments are recorded but admin revenue aggregation currently reports MWK only (no exchange-rate conversion).
- Email outbox retries are capped at five attempts; failed records need operational review.
- Full PayChangu sandbox, Supabase migration, RLS, cron-host, email-provider and production-build tests require deployment credentials/services and have not been run in this environment.
