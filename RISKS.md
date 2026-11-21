# mwaiHub security risk register – Phase 1

| # | Risk / finding | Severity | Phase 1 disposition |
|---|---|---:|---|
| 1 | Candidate could manipulate application identity fields through broad application policies/trigger gaps | Critical | Hardened in migration 00005; live RLS test still required |
| 2 | Candidate could submit an application with another user's CV | Critical | API, RLS and database trigger enforce CV ownership |
| 3 | Employer could alter candidate/job/CV data on an application | Critical | Employer updates limited to status and employer notes |
| 4 | Employers could delete their company/job and cascade-delete applications | High | Employer policies no longer grant delete |
| 5 | Unverified employer's active job could appear publicly | High | Public job policy now requires verified employer |
| 6 | Employers could see CV metadata but not download attached CV objects | High | Exact-application storage authorization and signed URL endpoint added |
| 7 | Moderator was treated as administrator for privileged-column changes | Critical | Separate `is_admin()` and `is_staff()` checks |
| 8 | Moderators could view payment and audit information | High | Payment and audit reads restricted to administrators |
| 9 | Any user could update their own role column | Critical | Column-level update grants plus trigger guard |
| 10 | Public settings policy exposed every future settings row | High | Public reads limited to `contact`, `hours`, `about` |
| 11 | Missing updated-at triggers on content/settings tables | Low | Added in migration 00005 |
| 12 | Client-only admin checks could be bypassed by direct requests | High | Server-side staff guard added to admin pages; future actions must use guard |
| 13 | API request payloads lacked schema validation | High | Zod schemas and validated application endpoint added |
| 14 | CV upload signature/malware validation | High | Still outstanding; upload pipeline not present in supplied project |
| 15 | API rate limiting and abuse controls | Medium | Still outstanding; configure Supabase/provider limits and add app-level throttling |
| 16 | No automated regression suite | High | Vitest tests added; live RLS tests remain required |
| 17 | Dependency lockfile absent | Medium | Still outstanding because package installation could not complete in this environment |
| 18 | ESLint command/configuration mismatch | Medium | Flat ESLint config and `eslint .` script added |
| 19 | Employer capabilities could be used by ordinary candidate accounts with a company row | High | Employer ownership helpers and employer policies now require the controlled `employer` role |
| 20 | Candidate could delete/rename a CV object after applying, breaking employer access | Medium | Storage update/delete now blocked while the CV is referenced by an application |

## Verification status

Static source edits have been prepared. A successful install, typecheck, lint, build, SQL migration run and live RLS test have **not** been completed in this environment. Do not mark the project production-ready until all those checks pass.

## Phase 4 additions
| # | Risk / finding | Severity | Phase 4 disposition |
|---|---|---:|---|
| 21 | Employer role could be self-assigned during signup | Critical | Auth trigger still hard-codes candidate; staff-only review RPC performs promotion |
| 22 | Employer registration requests could be edited directly to approved | High | Authenticated UPDATE grant revoked; guarded staff RPC owns review decision |
| 23 | Employer could self-verify or publish jobs without moderation | High | Existing role/verification triggers and draft-only job API remain authoritative |
| 24 | Employer could access other employers' applicant CVs | Critical | Existing application CV RPC and storage policy scope downloads to owned jobs |
| 25 | Employer company profile changes could retain stale verification | High | Database trigger resets verification to pending when company fields change |
| 26 | Employer analytics could leak cross-company totals | High | Security-definer stats RPC filters by auth.uid() and employer role |

## Phase 6 operational risks
- Payment packages have no preset prices; administrators must enter business-approved prices and durations before checkout is available.
- PayChangu and Resend credentials, webhook configuration, sender-domain verification and the hourly cron must be configured before production use.
- Paid jobs deliberately enter `pending_review`; a moderator/admin must approve publication. Payment alone does not publish a job.
- The admin revenue metric totals MWK only; USD transactions are listed but are not converted.
- Email outbox delivery is retried up to five times; failed records require operational review.
- The Supabase migration and external-provider flows have not been executed against live or sandbox services in this build environment.
