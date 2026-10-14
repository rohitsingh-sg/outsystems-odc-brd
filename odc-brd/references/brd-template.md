# ODC BRD template

Copy this structure. Keep it text and tables only. Replace every bracket. Write "Not applicable (reason)" instead of deleting a section.

```
# [App name] — Build Requirements & Test Cases
[date] · @[author] · Version [x] · Status [draft/agreed]

## 1. Scope and conventions
- Summary: what v1 is (single-org? currency? languages? time zone?) and which modules.
- In scope (list). Out of scope v1 (list). Backlog / parked (list).
- Precedence: this doc and its seed data win over mock-up screens. Screen values are layout only. Anything not here is not in v1. Latest decisions section wins on conflict.
- Conventions: BR-xx rules, TC-xx tests, F-x flows, D-xx decisions. Entities singular PascalCase, (S) = static entity. Dates stored UTC, shown in [tz]. Money Decimal(18,2) [currency]. T = today when seed runs.
- Every rule is enforced in server actions; UI only calls them.
- Architecture: [one app / several] and why (atomicity, ownership, reuse). Names of apps and libraries.
- Labels used for uncertain statements: FACT / ASSUMPTION / ANALYSIS / ESTIMATE.
- ODC hints are marked "ODC:" and must be checked against the tenant.

## 2. Roles and access
- Role table: role, who, home screen.
- Visibility rule (own / team / all) and the single server action that decides it.
- Access matrix: objects x roles (None / Read / Own / Own+team / Full).
- Create-rights matrix: objects x roles, who becomes owner.
- BR-01 access enforcement (ACCESS_DENIED, writes nothing). BR-02 delete vs deactivate.
- How users, teams, managers are stored (UserProfile etc.) and how test personas work if no IdP.

## 3. Data model
- 3.1 Static entities: name, records in order, extra attributes.
- 3.2 People and settings: Team, UserProfile, AppSetting keys and defaults, Counter.
- 3.3+ Entities by module: table of Entity | Attribute | Type | Rules (required, unique, default, calculated, read-only).
- Shared entities: audit, history, notifications, notes/activities, parent-link rule (exactly one parent, INVALID_PARENT).
- Indexes list.
- Common attributes on all entities (CreatedOn, CreatedBy, UpdatedOn, UpdatedBy, IsActive).

## 4. Business rules
Grouped by module. One line each: BR-xx Name: rule, who, enforcing action, error code on refusal.
Include: validation (VALIDATION_ERROR with field+message list), auto-numbering, field history, ownership change, deactivation guards, duplicates (with ConfirmDuplicate override if allowed), state locks (RECORD_LOCKED), transitions (INVALID_TRANSITION), calculations (scores, totals, SLA due dates).
Error code table: code, HTTP status if API, meaning.

## 5. Flows
One block per flow: F1 Name (owning server action). Numbered steps: trigger, actions called, rules applied, transaction boundary ("any failure rolls back all steps"), what the UI shows after.
Include status models as transition tables: From | To | Who | Condition | Error.

## 6. Screens and features
- 6.1 Common patterns: app shell, list (views, search, filters, paging 50, sorting), detail (header, path bar, tabs), form (side panel, field errors), lookups, global search, responsive rules, empty/loading/error states.
- 6.2 Screens by module: Module | Screens | Default list views | Key actions | Roles.
- 6.3 Screen details that matter: layouts, widgets, rules per screen, what each role does NOT see.
- Navigation per role (one menu per role).
- Mock-up reference (artifact link / PDF name) and what is layout-only.

## 7. Dashboards and reports
For each KPI: ID, definition (numerator/denominator, date field, filters), who sees what. Filters (period, owner/team). Expected values are in section 9.

## 8. Notifications, scheduled jobs, API and AI
- 8.1 Notifications: event, recipients, channel, text, link target.
- 8.2 Scheduled jobs: name, purpose, schedule (+ fallback), idempotency.
- 8.3 REST API: auth (header, 401), endpoint table (method/path, body, success code, errors), batch result shape, row limit, idempotency key.
- 8.4 AI assist: feature, where, inputs, output, "draft only" rule, timeout/failure behaviour, data/region constraint.
- 8.5 Integrations: system, direction, auth, data, failure handling.

## 9. Seed data for testing
- Reset action (admin only): wipes non-static data, resets counters, loads exactly the rows below. Also a repair action if the reset is large.
- 9.1 Users/personas (name, role, team, manager, limits). 9.2+ records per object with key values and owners.
- Expected values: dashboard figures per viewer, rankings, calculated fields, assignment state after reset.

## 10. Test cases
Table per area: ID | Rule | User | Steps | Expected. Every test starts from the reset state unless Setup says otherwise. Areas: access, general rules, each module, dashboards, notifications/jobs, API, AI, UI end-to-end, responsive.
Count tests in the intro line.

## 11. Non-functional requirements
Table: Area | Requirement. Security, privacy/consent/retention, residency, auth, authorisation on every action, uploads, audit, performance, availability, accessibility, language, devices, observability. Mark ESTIMATE where unsourced.

## 12. Theme, branding and UI (if relevant)
Tokens, logo source, prototype banner, UI patterns (ODC OutSystems UI), status chip colours.

## 13. Build order
Table: Stage | Build | Tests to pass. Stage 1 = data + roles + reset action; last = API/AI/responsive; role-to-screen assignment last if screens start anonymous. Add the per-stage brief wording. A later stage never starts on failing tests.

## 14. Open questions
Checkbox list: tenant checks (timer interval, AI model/region), customer-only inputs (rates, APIs, brand assets).

## 15. Decisions log
Table: ID | Decision | Affects. Add sub-section "test cases for these decisions". Later audit rounds append a dated section; newest wins.
```

## Writing rules
- Tables for anything with more than two attributes; sentences for rules.
- Quote exact names in `code` style only when helpful; keep them identical everywhere.
- Prefer numbers over adjectives ("50 rows per page", "15 minutes", "within 2 s at p95").
- If a value is a design default (SLA hours, probabilities), say it is a default and which tests change if it changes.
