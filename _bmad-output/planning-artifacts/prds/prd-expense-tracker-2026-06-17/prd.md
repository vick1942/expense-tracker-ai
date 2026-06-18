---
title: Spendly — Personal Expense Tracker
status: draft
created: 2026-06-17
updated: 2026-06-17
project: expense-tracker
owner: VK
---

# PRD: Spendly — Personal Expense Tracker

## 0. Document Purpose

This PRD is the authoritative requirements reference for the Spendly web application — written for the product owner (VK), any downstream developers, and future BMAD workflow owners (architecture, epics, stories). It is structured with Glossary-anchored vocabulary (§3), features grouped with FRs nested (§4), and all inferred decisions tagged `[ASSUMPTION]` and indexed in §9. The UX spec and architecture ADRs do not yet exist; this PRD feeds them.

---

## 1. Vision

Spendly is a personal expense-tracking web application that helps individual users in India log, categorize, and review their daily spending. The product answers a simple but chronic problem: people know they're overspending, but they can't see where. By giving users a frictionless way to record every transaction and a clear monthly breakdown by category, Spendly transforms a vague financial anxiety into specific, actionable awareness.

The product targets everyday Indian consumers — students, salaried professionals, and young families — who manage expenses in Indian Rupees and want a lightweight tool they can use from any browser without installing an app. Spendly is not a budgeting tool or a bank integration; it is a ledger with a lens: you record what you spend, and Spendly shows you the pattern.

The v1 scope deliberately stays narrow. Authentication, CRUD for expense entries, and category-level summaries are the complete surface. That simplicity is a feature: it makes the product something a user finishes setting up in under two minutes and returns to daily.

---

## 2. Target User

### 2.1 Jobs To Be Done

- **Functional:** Log an expense immediately after spending, before I forget the amount or category.
- **Functional:** See how much I've spent this month, broken down by category (Bills, Food, Health, Transport, etc.).
- **Functional:** Find and correct a mistakenly entered expense without losing the rest of my data.
- **Functional:** Filter my expense history to a specific time period (last week, last month, custom range).
- **Emotional:** Feel in control of my money rather than anxious about where it went.
- **Social:** Not need to share or expose financial data to any third party or bank API.
- **Contextual:** Use from my phone browser or laptop without installing anything.

### 2.2 Non-Users (v1)

- Teams or households tracking shared expenses — Spendly has no multi-user or shared ledger capability in v1.
- Users who need bank/card import or automated transaction sync — v1 is manual-entry only.
- Users who need budget goal-setting or savings projections — out of scope for v1.
- Businesses tracking business expenses — Spendly is personal finance only.

### 2.3 Key User Journeys

- **UJ-1. Rahul logs a grocery expense right after checkout.**
  - **Persona + context:** Rahul, 24, software intern in Pune, tracks spending on a tight monthly stipend.
  - **Entry state:** Authenticated (session active from this morning). On mobile browser.
  - **Path:** Opens Spendly → taps "Add Expense" → enters amount ₹640, selects category "Food", picks today's date, optionally writes "BigBazaar weekly groceries" → submits.
  - **Climax:** The expense appears at the top of his list. He can see it is recorded. The running total for "Food" this month has updated.
  - **Resolution:** He closes the browser and goes home. His record is saved.
  - **Edge case:** He accidentally entered ₹6400. He opens the expense, taps Edit, corrects to ₹640, saves. The list reflects the correction immediately.

- **UJ-2. Priya checks her spending before her next salary credit.**
  - **Persona + context:** Priya, 29, marketing executive in Delhi, wants to know if she's overspent on eating out this month.
  - **Entry state:** Not authenticated. Opens Spendly on laptop at end of month.
  - **Path:** Lands on login page → enters credentials → redirected to dashboard → sees current month's category breakdown — "Food" is ₹3,200 — scans expense list filtered to current month.
  - **Climax:** She can see the total and individual entries. She spots two duplicate restaurant entries from the same day and deletes the duplicate.
  - **Resolution:** Corrected total is shown. She logs out.

- **UJ-3. Kiran starts fresh with a new Spendly account.**
  - **Persona + context:** Kiran, 32, freelancer in Bengaluru, heard about Spendly and wants to try it.
  - **Entry state:** Not authenticated, no account.
  - **Path:** Lands on landing page → clicks "Start tracking free" → fills registration form (name, email, password) → account created → redirected to dashboard (empty state).
  - **Climax:** Kiran sees an empty expense list with a prompt to add the first expense. No confusion about what to do next.
  - **Resolution:** Kiran adds their first expense and the list shows it.

---

## 3. Glossary

- **User** — A registered individual with a unique account (email + password). All data is scoped to one User.
- **Expense** — A single financial transaction recorded by a User. Has: amount (₹), category, date, and optional description.
- **Category** — A predefined label classifying an Expense type. v1 set: Bills, Food, Health, Transport. [ASSUMPTION: categories are fixed in v1; user-defined categories are deferred.]
- **Dashboard** — The authenticated landing screen showing the Expense list and monthly summary for the current User.
- **Monthly Summary** — An aggregated view of total spend per Category for a given calendar month.
- **Session** — A server-side authenticated state for a User, started at login and ended at logout or expiry.
- **Date Range Filter** — A UI control allowing the User to restrict the Expense list to a start date and end date.
- **Profile** — A User's account settings page (name, email, password change). [ASSUMPTION: only basic fields in v1.]

---

## 4. Features

### 4.1 User Authentication

**Description:** Spendly uses email + password authentication. New users register with name, email, and password; returning users sign in with email and password. Sessions persist until the user logs out or the server session expires. [ASSUMPTION: session expiry is Flask's default server-side session; no "Remember me" or JWT in v1.] All expense data is strictly isolated per User — no cross-user data access is possible. Passwords are stored hashed, never in plaintext. Realizes UJ-1, UJ-2, UJ-3.

**Functional Requirements:**

#### FR-1: User Registration

A Guest can create an account by submitting name, email, and password. Realizes UJ-3.

**Consequences (testable):**
- System creates a new User record with hashed password; returns HTTP 302 redirect to Dashboard on success.
- System rejects registration if email is already registered; displays an inline error without clearing the form.
- System rejects a password shorter than 8 characters; displays an inline error.
- System rejects a blank name or invalid email format; displays an inline error.
- Password is stored as a bcrypt (or Werkzeug-equivalent) hash — plaintext is never written to the database.

#### FR-2: User Login

A registered User can sign in with email and password. Realizes UJ-2.

**Consequences (testable):**
- System creates a Session and redirects to Dashboard on correct credentials.
- System displays an error message "Invalid email or password" on incorrect credentials; does not reveal which field is wrong.
- System rejects blank email or password with inline validation.

#### FR-3: User Logout

An authenticated User can end their Session. Realizes UJ-2.

**Consequences (testable):**
- System destroys the Session and redirects to the landing page.
- After logout, attempting to access `/expenses/add` or Dashboard redirects to login (not a 403 error page).

#### FR-4: Route Protection

All expense and profile routes require an active Session.

**Consequences (testable):**
- Unauthenticated request to any protected route redirects to `/login` with the original URL preserved as a `next` parameter. [ASSUMPTION: `next` redirect is implemented to avoid UX friction post-login.]
- Protected routes never return data for a different User's records, even with a valid session.

---

### 4.2 Expense Management (CRUD)

**Description:** The core of Spendly. An authenticated User can add, view, edit, and delete their own Expenses. Each Expense captures: amount (₹, required), category (required, from fixed list), date (required, defaults to today), and description (optional free text). The Expense list is the Dashboard's primary surface. Realizes UJ-1, UJ-2.

**Functional Requirements:**

#### FR-5: Add Expense

An authenticated User can submit a new Expense via a form. Realizes UJ-1.

**Consequences (testable):**
- System saves the Expense linked to the current User and redirects to Dashboard; new entry appears at the top of the list.
- Amount field accepts only positive numeric values in ₹; rejects zero, negative, or non-numeric input with an inline error.
- Category field is a dropdown of the fixed Category list; submission with no category selected is rejected.
- Date field defaults to today's date; User may change it. Future dates are accepted. [ASSUMPTION: no business rule blocking future-dated expenses in v1.]
- Description field is optional; max 255 characters. [ASSUMPTION: 255 char limit is reasonable for a short note.]
- Expense is associated with the authenticated User's ID; no other User can see or modify it.

#### FR-6: View Expense List

An authenticated User can see their full Expense list on the Dashboard.

**Consequences (testable):**
- Dashboard shows all Expenses for the current User, ordered by date descending (most recent first). [ASSUMPTION: default sort is date descending.]
- Each list row shows: date, category, amount (₹), description (truncated if long).
- Empty state: when no Expenses exist, Dashboard shows a prompt to add the first one — not a blank page.
- Another User's Expenses never appear in the list, even with a crafted URL.

#### FR-7: Edit Expense

An authenticated User can modify any of their own Expenses. Realizes UJ-2 (duplicate correction).

**Consequences (testable):**
- Edit form pre-populates with existing Expense values.
- System saves changes and redirects to Dashboard; updated values are reflected.
- User cannot edit another User's Expense; attempting to do so returns an error or redirects.
- Validation rules from FR-5 apply on edit submission.

#### FR-8: Delete Expense

An authenticated User can permanently remove one of their own Expenses. Realizes UJ-2.

**Consequences (testable):**
- Deletion removes the Expense record from the database.
- [ASSUMPTION: no soft-delete or undo in v1 — deletion is permanent.]
- User cannot delete another User's Expense via a crafted URL; attempting returns a 403 or redirect.
- After deletion, Dashboard reflects the updated list and totals.

---

### 4.3 Spending Summary & Filtering

**Description:** The Dashboard presents a monthly Category breakdown (totals per Category for the current calendar month) alongside the Expense list. A Date Range Filter lets the User narrow the list to any period. This is the "pattern lens" that differentiates Spendly from a plain list. Realizes UJ-1 (running total), UJ-2 (month review). [ASSUMPTION: the monthly summary always defaults to the current calendar month; navigating to past months is deferred to v2.]

**Functional Requirements:**

#### FR-9: Monthly Category Summary

The Dashboard displays the total amount spent per Category for the current calendar month.

**Consequences (testable):**
- Each Category in the fixed list appears with its sum for the current month (₹0 if no expenses in that Category).
- Summary updates immediately when an Expense is added, edited, or deleted — no stale cache.
- Summary reflects only the authenticated User's data.

#### FR-10: Date Range Filter

An authenticated User can filter the Expense list by a start date and end date.

**Consequences (testable):**
- Applying a valid date range shows only Expenses whose date falls within [start, end] inclusive.
- Monthly summary updates to reflect the filtered range when a filter is active. [ASSUMPTION: summary recalculates for the filtered period, not always the full calendar month.]
- Clearing the filter restores the full unfiltered list and current-month summary.
- Invalid date range (end before start) shows an inline error; does not submit.

---

### 4.4 User Profile

**Description:** An authenticated User can view and update their account details (name, email, password). [ASSUMPTION: profile is a simple settings form; no avatar, notification preferences, or account deletion in v1.] Realizes a utility need — users need to fix a typo in their name or change a compromised password.

**Functional Requirements:**

#### FR-11: View and Update Profile

An authenticated User can update their name, email, and password.

**Consequences (testable):**
- Profile form pre-populates with current name and email.
- Name update: saved immediately; reflected in the navbar if name is displayed there. [ASSUMPTION: navbar shows user's name after login.]
- Email update: system checks for uniqueness before saving; rejects if already used by another User.
- Password change: requires current password confirmation before accepting a new password. New password must be ≥ 8 characters.
- All updated fields are validated with the same rules as registration (FR-1).

---

## 5. Non-Goals (Explicit)

- **No bank or card import.** Spendly is manual-entry only. No third-party financial API integration in v1 or v2 planning.
- **No budget goals or savings targets.** Spendly tracks actuals; it does not project, warn, or set limits.
- **No multi-user or shared ledger.** Each account is strictly personal. No family/household mode.
- **No user-defined categories.** The Category list is fixed in v1. [NON-GOAL for MVP]
- **No recurring expense automation.** Users enter each expense manually; no repeat/template feature.
- **No data export (CSV/PDF).** [NON-GOAL for MVP — high user value but deferred to v2.]
- **No mobile native app.** Web-only; responsive design is expected but no React Native or Flutter.
- **No email verification on registration.** [NON-GOAL for MVP — reduces signup friction for a student project.]
- **No soft-delete or undo.** Deletion is permanent in v1.
- **No past-month navigation in summary.** Monthly summary defaults to current month only in v1.
- **No notifications or reminders.**

---

## 6. MVP Scope

### 6.1 In Scope

- Landing / marketing page (public)
- User registration, login, logout (session-based auth)
- Profile page (view + update name, email, password)
- Add, view, edit, delete expenses (full CRUD)
- Fixed category list: Bills, Food, Health, Transport
- Dashboard: expense list (date-descending) + current-month category summary
- Date range filter on expense list
- Input validation with inline error messages
- Per-user data isolation (no cross-user access)
- Responsive web UI (mobile browser + desktop)
- SQLite database (single-file, no external DB server)
- Python/Flask backend with Jinja2 templates
- Pytest test suite (unit + integration)

### 6.2 Out of Scope for MVP

- Email verification on signup — reduces friction; revisit if spam becomes an issue.
- "Remember me" / persistent login token — deferred; session-based auth is sufficient.
- Past-month navigation in summary — [NOTE FOR PM] users will almost certainly ask for this first; good v2 candidate.
- User-defined categories — [NOTE FOR PM] emotionally important to power users; high implementation cost for v1.
- CSV/PDF export — high utility, low complexity; strong v2 candidate.
- Budget goals / spending alerts — requires a different product concept; deferred indefinitely.
- Soft-delete / undo — deferred to v2 if user complaints arise.
- Bank/card import — out of scope permanently for this product direction.
- Multi-user / household mode — separate product surface; not planned.

---

## 7. Success Metrics

**Primary**

- **SM-1: Daily Active Use** — User logs at least one expense per week after account creation. Target: 60% of registered users active at 30 days. Validates FR-5, FR-6.
- **SM-2: Retention** — User returns to view their summary at least once per month. Target: 50% monthly retention at 60 days. Validates FR-9.

**Secondary**

- **SM-3: Registration Completion** — Users who land on the registration page complete signup. Target: ≥ 80% completion rate. Validates FR-1.
- **SM-4: Edit/Delete Usage** — Users who correct an expense (edit or delete) within 24 hours of logging it. Target: < 15% of entries (high rate indicates poor UX on the add form). Validates FR-7, FR-8.
- **SM-5: Filter Usage** — Users who apply the date range filter at least once. Target: ≥ 30% of DAU use the filter in month 1. Validates FR-10.

**Counter-metrics (do not optimize)**

- **SM-C1: Entry Volume per Session** — Do not optimize for number of expenses logged per session. High volume may indicate bulk back-entry (low engagement signal) rather than healthy daily logging.
- **SM-C2: Time-on-page** — Do not use session duration as a positive signal. Spendly should be fast to use; long sessions likely indicate confusion, not engagement.

*Note: This is an educational/student project (v1 step-by-step implementation). Success metrics above represent the product vision; actual measurement tooling is out of scope for the learning exercise and would be instrumented in a production deployment.*

---

## 8. Open Questions

1. **Password hashing library** — Should the implementation use Werkzeug's `generate_password_hash` / `check_password_hash` (already a dependency) or add `bcrypt` explicitly? Werkzeug's default is acceptable for educational use; bcrypt is stronger for production.
2. **Session secret key management** — How should `app.secret_key` be stored? Hardcoded for the student exercise or loaded from an environment variable? [Recommend: env var even in v1 to teach good habits.]
3. **Empty-state dashboard design** — What should the Dashboard show for a brand-new user with zero expenses? A call-to-action prompt? A sample expense? [ASSUMPTION: prompt only, no seed data in production accounts.]
4. **Category extensibility** — Is the fixed 4-category list (Bills, Food, Health, Transport) the final v1 set, or should it also include Others/Miscellaneous as a catch-all?
5. **Date range filter scope** — When a filter is active, should the category summary reflect the filtered period or always show the current calendar month? (This PRD assumes the summary follows the filter — confirm.)
6. **Profile page — account deletion** — Should users be able to delete their account and all data in v1? (Currently marked non-goal; confirm this is intentional.)
7. **Navbar authenticated state** — Should the navbar change after login (show user name + logout link vs. sign in / get started)? [ASSUMPTION: yes — the base template needs an authenticated/unauthenticated conditional.]

---

## 9. Assumptions Index

- **§1** — v1 is manual-entry only; no bank/card sync is planned.
- **§3 / FR-5** — Category list is fixed in v1: Bills, Food, Health, Transport.
- **§4.1** — Session uses Flask's default server-side session; no JWT or "Remember me".
- **§4.1 / FR-4** — Login redirects preserve the original `next` URL.
- **§4.2 / FR-5** — Future-dated expenses are accepted (no business rule blocks them).
- **§4.2 / FR-5** — Description field has a 255-character max.
- **§4.2 / FR-6** — Default sort for expense list is date descending.
- **§4.2 / FR-8** — Deletion is permanent (no soft-delete or undo).
- **§4.3** — Monthly summary defaults to the current calendar month.
- **§4.3 / FR-10** — When a date range filter is active, the category summary recalculates for that range (not always the full calendar month).
- **§4.4 / FR-11** — Navbar displays the authenticated user's name.
- **§4.4** — Profile has no avatar, notification settings, or account deletion in v1.
- **§8 Q3** — Empty dashboard shows a prompt to add the first expense; no seeded sample data in production accounts.

---

## Cross-Cutting NFRs

**Security**
- Passwords never stored in plaintext; use Werkzeug password hashing at minimum.
- All expense routes enforce ownership checks — User A cannot read, edit, or delete User B's data regardless of URL manipulation.
- Flask's CSRF protection should be enabled for all POST/DELETE form actions. [ASSUMPTION: Flask-WTF or equivalent CSRF token in all forms.]
- No sensitive data (password, session token) logged to console or application logs.

**Performance**
- Page load time for Dashboard with up to 500 expenses: ≤ 1 second on a local SQLite instance. [ASSUMPTION: no pagination required in v1 at this scale.]
- SQLite is acceptable for single-user/dev load; not designed for multi-tenant production scale.

**Accessibility**
- All form inputs have associated `<label>` elements.
- Error messages are associated with their input fields (not just styled red text).
- Color is not the sole means of conveying information (category bars use labels + color).

**Platform**
- Web only. Responsive layout tested on mobile browser (≥ 375 px viewport) and desktop (≥ 1024 px).
- No native app in v1.

**Testing**
- Pytest + pytest-flask test suite covers: registration, login, logout, add/edit/delete expense, route protection (unauthenticated redirect), and cross-user isolation.

---

*Workspace: `_bmad-output/planning-artifacts/prds/prd-expense-tracker-2026-06-17/`*
*Common next steps: `bmad-ux` for UI design spec, `bmad-create-architecture` for ADRs, `bmad-create-epics-and-stories` to break into implementation units.*
