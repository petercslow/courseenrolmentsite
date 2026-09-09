# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project shape

Northstar Learning is a dependency-free vanilla ES6 single-page prototype for a Singapore training course enrolment and administration website. There is no package manager, build step, test runner, or backend — the browser opens `index.html` directly.

- `index.html` — document shell; loads Google Fonts, `styles.css`, and `app.js`. Contains only `#app` (render target) and `#toast` (notifications).
- `app.js` — everything: seed data, app state, hash routing, HTML template rendering, learner validation, admin views, `localStorage` persistence, and CSV export.
- `styles.css` — visual system (CSS custom properties in `:root`), grid/flex layouts, responsive breakpoints at 900px/640px.
- `README.md` — current scope and planned production architecture (Supabase/PostgreSQL migration plan).

## Running and testing

Open `index.html` in a browser. No build/install commands exist. There is no automated test suite or lint command — validate changes manually in-browser.

When validating changes, exercise the relevant flow: public catalogue → course details → enrolment form validation → successful enrolment → duplicate registration → admin login → dashboard → enrolment search → CSV export. Clear `localStorage` (`northstar-enrolments` key) for a clean enrolment run, and `sessionStorage` (`northstar-admin` key) to reset admin login state. Since there's no test command, report which browser flows you manually checked and any limitations.

Demo admin login: `admin@northstar.sg` / `northstar2026` (development-only, never to be treated as real auth).

## Architecture

Everything lives in `app.js` as one file with no modules/imports. Key pieces:

- **`state`** — single mutable object holding `courses`, `enrolments`, current `view`, `selectedCourse`, `selectedSession`, `adminTab`. `seed.enrolments` is hydrated once from `localStorage` at load.
- **Hash routing** — `location.hash` (`#home`, `#courses`, `#admin`) combined with `state.view` (set programmatically for `details`/`enrol`/`success` since those aren't hash-addressable) drives `render()`, which fully re-renders `app.innerHTML` from scratch on every state change. There is no virtual DOM/diffing.
- **Templates are template-literal functions** (`home()`, `courses()`, `details()`, `enrol()`, `success()`, `admin()`, `adminContent()`) that return HTML strings, composed via a shared `shell()` wrapper for the public site. Event handlers are wired via inline `onclick`/`oninput`/`onsubmit` attributes calling globally-scoped functions (e.g. `showDetails()`, `startEnrol()`, `submitEnrolment()`), since there's no event delegation layer.
- **`esc()`** escapes all dynamic values before HTML interpolation — always use it for any user-supplied or stored data inserted into a template string.
- **Validation** is client-side only: `validNric()` checks Singapore NRIC/FIN format (`[STFG]\d{7}[A-Z]`), plus inline regex checks for 8-digit SG mobile numbers and email format in `submitEnrolment()`. Duplicate enrolment protection checks session + email/mobile/NRIC match against existing `state.enrolments`.
- **NRIC masking** — full NRIC is stored in `state.enrolments` but only `nricMasked` (first char + `****` + last 4) is shown in admin views via `enrolmentRows()`.
- **Persistence** — `save()` writes `state.enrolments` to `localStorage` after every enrolment; nothing else is persisted (courses/seed data are hardcoded, admin login is `sessionStorage`-only).
- **Capacity** — `getSessionCount()` derives seats-taken by filtering non-cancelled enrolments per session; there's no transactional/server-side capacity lock, so this is a prototype-only check.

## Working conventions

- Preserve the plain ES6 style: `const`, arrow functions, template literals, semicolons, no module system — unless a larger architecture migration is explicitly requested.
- Keep navigation hash-based and state centralized in `app.js` unless a bigger refactor is asked for.
- Always use `esc()` when inserting dynamic values into HTML templates.
- Preserve the existing CSS custom properties (`:root` in `styles.css`), grid/flex layout approach, and 900px/640px responsive breakpoints.
- Keep learner-facing workflows mobile-friendly and minimize enrolment steps.
- Do not add dependencies or a build configuration for small prototype changes.
- Make focused edits; preserve unrelated user changes. Do not commit or introduce production infrastructure (real backend, auth, DB) unless explicitly requested.
- When adding admin data operations, make sure the public learner flow and masked NRIC display keep working.

## Security boundary — prototype only

This code is not production-safe. Never describe the following as production-grade: the demo admin credentials, the `sessionStorage` admin flag, enrolment records in `localStorage`, client-side field validation, or client-side capacity checks.

Production work should follow the plan in `README.md`: move state into Supabase/PostgreSQL (tables: `courses`, `training_sessions`, `trainers`, `venues`, `learners`, `enrolments`, `admin_users`, `audit_logs`), use Supabase Auth with role-based access, encrypt NRIC/FIN server-side, add row-level security, server-side validation, CSRF/rate limiting, audit logs, retention controls, transactional capacity checks, and real email delivery. Secrets must stay out of frontend code — never hardcode them in `app.js`.
