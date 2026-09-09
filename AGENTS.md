# Agent Instructions

## Project shape

This is a dependency-free vanilla ES6 single-page prototype for Northstar Learning. The browser opens `index.html` directly; there is no package manager, build step, test runner, or backend.

- `index.html` is the document shell and loads the stylesheet and script.
- `app.js` contains seed data, application state, hash routing, template rendering, learner validation, admin views, local persistence, and CSV export.
- `styles.css` contains the visual system and responsive layouts.
- `README.md` documents the current scope and the planned production architecture.

## Working conventions

- Preserve the plain ES6 style: `const`, arrow functions, template literals, semicolons, and no module system unless the project is deliberately migrated.
- Keep navigation hash-based and state centralized in `app.js` unless a larger architecture change is requested.
- Use the existing `esc()` helper for dynamic values inserted into HTML templates.
- Preserve the existing CSS custom properties, grid/flex layout approach, and 900px/640px responsive breakpoints.
- Keep learner workflows mobile-friendly and minimise enrolment steps.
- Do not add dependencies or a build configuration for small prototype changes.

## Validation

Open `index.html` in a modern browser after changes. Exercise the relevant flow: public catalogue, course details, required-field validation, successful enrolment, duplicate registration, admin login, dashboard, enrolment search, and CSV export. Clear browser `localStorage` when a clean enrolment run is needed.

There is no automated test command currently. Report browser checks performed and any limitations.

## Security boundary

This is prototype-only code. The demo admin credentials, `sessionStorage` admin flag, enrolment records in `localStorage`, client-side validation, and client-side capacity checks are not secure. Never describe them as production authentication or data protection, and never expand them into a production claim.

For production work, follow `README.md`: move persistence and authorization to Supabase/PostgreSQL, use Supabase Auth and role-based access, encrypt NRIC/FIN server-side, add row-level security, server-side validation, CSRF/rate limiting, audit logs, retention controls, transactional capacity checks, and real email delivery. Keep secrets out of frontend code.

## Scope discipline

Make focused edits and preserve unrelated user changes. Do not commit or introduce production infrastructure unless explicitly requested. When adding admin data operations, ensure the public learner flow and masked NRIC display continue to work.
