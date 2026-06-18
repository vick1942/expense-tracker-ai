---
stepsCompleted: [1, 2, 3]
inputDocuments:
  - '_bmad-output/planning-artifacts/prds/prd-expense-tracker-2026-06-17/prd.md'
workflowType: 'architecture'
project_name: 'expense-tracker'
user_name: 'VK'
date: '2026-06-18'
---

# Architecture Decision Document

_This document builds collaboratively through step-by-step discovery. Sections are appended as we work through each architectural decision together._

## Project Context Analysis

### Requirements Overview

**Functional Requirements:**
11 FRs across 4 feature areas:
- User Authentication (FR-1 Registration, FR-2 Login, FR-3 Logout, FR-4 Route Protection) — drives session/auth architecture and a route-protection cross-cutting concern.
- Expense Management / CRUD (FR-5 Add, FR-6 View List, FR-7 Edit, FR-8 Delete) — drives data model, ownership-scoped queries, and form validation patterns.
- Spending Summary & Filtering (FR-9 Monthly Category Summary, FR-10 Date Range Filter) — drives aggregation/query logic shared between the default dashboard view and filtered views.
- User Profile (FR-11 View/Update Profile) — reuses registration validation rules; touches auth (password change) and uniqueness checks (email).

**Non-Functional Requirements:**
- Security: hashed passwords (Werkzeug minimum), strict per-user ownership enforcement on every expense/profile route, CSRF protection on all POST/DELETE forms, no sensitive data in logs.
- Performance: Dashboard ≤1s load with up to 500 expenses on local SQLite; no pagination required at this scale.
- Accessibility: labeled form inputs, field-associated error messages, color not the sole indicator.
- Platform: responsive web only (mobile ≥375px, desktop ≥1024px); no native app.
- Testing: Pytest + pytest-flask suite is an explicit MVP deliverable, covering registration, login, logout, CRUD, route protection, and cross-user isolation.

**Scale & Complexity:**
- Primary domain: full-stack web (Flask backend + server-rendered Jinja2 frontend).
- Complexity level: low-to-medium — single service, single relational DB, no real-time features, no multi-tenancy beyond per-user row isolation.
- Estimated architectural components: ~5 (auth/session, data model/ORM, expense CRUD routes, summary/aggregation logic, profile management) plus shared validation and CSRF cross-cutting layers.

### Technical Constraints & Dependencies

- Stack is largely pre-decided by the PRD: Python/Flask backend, Jinja2 templates, SQLite (single-file, no external DB server).
- Educational/student project — bias toward simplicity and teaching value over production-scale robustness (per PRD §7).
- Fixed category list (Bills, Food, Health, Transport) — no user-defined categories in v1, simplifying the data model.
- No email verification, no "remember me"/JWT, no soft-delete — reduces auth and data-model complexity.

### Cross-Cutting Concerns Identified

- Session-based authentication and per-route protection (FR-4) applied uniformly across all expense and profile routes.
- Per-user authorization checks on every data access (ownership enforcement), required even against crafted URLs.
- Shared input validation rules reused across registration, profile update, and password change (FR-1, FR-11).
- CSRF protection required on all state-changing form submissions.
- Aggregation logic (monthly category summary) must recalculate consistently whether scoped to the current month or an active date-range filter (FR-9, FR-10).

## Starter Template Evaluation

### Primary Technology Domain

Full-stack web (Python/Flask backend + server-rendered Jinja2 frontend), pre-decided by the PRD.

### Starter Options Considered

- **cookiecutter-flask** (Bootstrap + webpack asset bundling + registration scaffold) — rejected. Brings a webpack build pipeline and asset bundling that add setup overhead disproportionate to this project's scope (4 fixed categories, no rich frontend tooling needed) and reduce teaching value for a student project.
- **microsoft/cookiecutter-python-flask-clean-architecture** — rejected. Onion/clean-architecture layering is over-engineering for an 11-FR, single-developer educational app; the PRD explicitly biases toward simplicity (§7).
- **Hand-built minimal Flask app (application factory + blueprints)** — selected. No generator; the architecture decisions below (extensions, structure) are applied directly to a fresh `flask` project. This keeps every file purposeful and matches the PRD's "simple enough to finish in under two minutes" ethos, while still teaching the standard application-factory pattern used in production Flask apps.

### Selected Starter: None (hand-built Flask application factory)

**Rationale for Selection:**
The PRD already fixes the stack (Flask, Jinja2, SQLite) and explicitly calls this a student/educational project favoring simplicity. A generated boilerplate would impose unused structure (asset bundlers, layered architecture) that obscures rather than teaches. Building the app factory and blueprint structure by hand, using current, individually-verified library versions, gives full control and a clear learning path.

**Initialization Command:**

```bash
python -m venv .venv
. .venv/Scripts/activate  # Windows
pip install Flask Flask-SQLAlchemy Flask-WTF Flask-Login pytest pytest-flask
```

**Architectural Decisions Provided by Starter:**

**Language & Runtime:**
Python 3.11+, Flask 3.1.3 (current stable as of Feb 2026; verified via PyPI/Flask changelog).

**Styling Solution:**
Server-rendered Jinja2 templates + plain CSS (no build step) — consistent with PRD's no-JS-framework, responsive-web-only scope.

**Build Tooling:**
None required — Flask's built-in dev server; no webpack/Vite needed for a template-rendered app.

**Testing Framework:**
pytest 9.1.0 + pytest-flask 1.3.0 (Flask-3-compatible, verified current).

**Code Organization:**
Flask application-factory pattern (`create_app()`) with blueprints per feature area (auth, expenses, profile) — isolates the cross-cutting concerns identified in step 2 (route protection, ownership checks) into reusable decorators/middleware.

**Development Experience:**
Flask's built-in debug/reload server; SQLite file-based DB requires no separate service to run locally.

**Note:** Project initialization using this command should be the first implementation story.
