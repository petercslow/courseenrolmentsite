# Northstar Learning

A dependency-free prototype for a Singapore training course enrolment and administration website.

## Run locally

Open `index.html` in a modern browser. No build step or package installation is required.

## Included in this prototype

- Responsive public course catalogue with search and filters
- Course details with outlines, dates, trainers, venues and capacity
- Learner enrolment form with Singapore mobile, email and NRIC / FIN format validation
- Mandatory privacy consent and optional marketing consent
- Duplicate registration protection by session, email, mobile or NRIC / FIN
- Confirmation page with registration number and print action
- Administrator demo login
- Dashboard, course management, schedule view and enrolment view
- Masked NRIC display
- CSV export with authorised fields only
- Local browser persistence through `localStorage`

## Demo administrator account

Development-only credentials:

- Email: `admin@northstar.sg`
- Password: `northstar2026`

Do not reuse these credentials in production.

## Production architecture

For deployment, replace the local state layer in `app.js` with a Next.js or React frontend backed by Supabase Auth and PostgreSQL. Recommended tables are `courses`, `training_sessions`, `trainers`, `venues`, `learners`, `enrolments`, `admin_users` and `audit_logs`.

Production must add server-side validation, encrypted NRIC / FIN storage, row-level security, CSRF protection, secure cookies, rate limiting, audit logging, retention controls, transactional capacity checks and email delivery. Secrets belong in environment variables and must never be exposed in frontend code.

## Next implementation phases

1. Move seed data and CRUD operations into Supabase migrations and server actions.
2. Add role-based access for Super Administrator, Course Administrator, Registration Administrator and Trainer.
3. Add trainer and venue master lists, edit forms and audit history.
4. Add Excel export, charts, email confirmation and waitlist handling.
5. Add automated tests for registration, duplicate detection, capacity and permissions.
# courseenrolmentsite
