---
name: odc-brd
description: Write or review a Business Requirements Document (BRD) for an OutSystems Developer Cloud (ODC) app so it can be built reliably through Mentor / the outsystems MCP. Use when the user wants to build an ODC app from an idea, asks for an ODC BRD, requirements doc, build spec or test cases, or hands over a vague/partial requirement before building. Produces a text-and-tables Markdown BRD (rules, data model, flows, screens, seed data, test cases, staged build order). For building use `odc-build-playbook`; for design rules use `odc-best-practices`.
---

# ODC BRD skill

Derived from the ODC CRM "Build Requirements & Test Cases" doc, the GSIS Auto Shield BRD, and the problems hit while building them with Mentor. The goal is not a business-style essay. It is a **build spec an agent can execute and a human can test**: unambiguous names, rules with error codes, seed data with expected values, and tests that name the rule they prove.

## Workflow (follow in order)

1. **Intake (ask, don't assume).** Read anything the user supplied (docs, mock-up PDFs, artifact links, screenshots). Then ask only what is missing, in one batch, using `references/intake-questions.md`. Always settle: app boundary, roles, v1 scope vs out-of-scope, real vs fake users (IdP?), mock-up authority, integrations/AI/timers, and open tenant questions.
2. **Pick the shape.** Decide one app vs several (see ODC facts below) and say why in the BRD.
3. **Draft the BRD** from `references/brd-template.md`. Fill every section, or write `Not applicable (reason)`. No generic filler.
4. **Self-audit** with the checklist at the end. Fix gaps, conflicts and invented facts before showing the user.
5. **Show the user** a short summary: scope, roles, entity count, flows, stage plan, open questions, assumptions. Ask them to confirm or correct. Record each answer in the Decisions log (D-xx).
6. **Iterate by decision, not by rewrite.** When the user corrects something, add a D-xx entry, edit the affected sections to match, and add or adjust tests. State in the doc: "where anything disagrees, the latest decisions section wins."
7. **Hand off to the build.** Use the staged build order and the per-stage brief (below). Load `odc-build-playbook` for Mentor mechanics and `odc-best-practices` for design rules.

Output: one Markdown file (text and tables only; no images or diagrams that need rendering; flows as numbered steps). Save where the user chooses. Offer .docx/PDF only if asked.

## Principles (each one comes from a real problem)

- **Doc wins.** State the precedence rule in section 1: the BRD and its seed data beat mock-up screens; screen values are layout only; anything not in the BRD is not in v1. This ended every mock-up vs spec argument in the CRM build.
- **Label uncertain claims:** FACT (cited), ASSUMPTION (owner must confirm), ANALYSIS (recommendation), ESTIMATE (unsourced number). Never invent rates, SLAs, legal rules or product facts. Mark placeholders.
- **Stable IDs and exact names.** BR-xx rules, TC-xx tests, F-x flows, D-xx decisions. Entities singular PascalCase, static entities marked (S). Name every entity, attribute, server action, role and error code; the build agent must use them unchanged.
- **Rules are server-side.** Every rule names the enforcing server action and the error code it returns (ACCESS_DENIED, VALIDATION_ERROR, RECORD_LOCKED, ...). The UI hides; the server refuses and writes nothing.
- **One visibility function.** If access depends on owner/team, define a single server action (e.g. GetVisibleOwnerIds) used by every list, detail, roll-up, dashboard and API. State that related lists, KPIs, banners and activity feeds are filtered too.
- **Seed data with expected values.** Define an admin-only reset action and the exact rows it loads. Give expected numbers (dashboard values per viewer, rankings, health results) so tests are checkable.
- **Every test names its rule**, the user/persona, steps and expected result, starting from the reset state.
- **Staged build order with gates.** Stages in dependency order: data, access layer, simple modules, logic-heavy modules, dashboards, API/AI, responsive pass. A stage starts only when the previous stage's tests pass. Each stage is small enough for one or two Mentor prompts.
- **Decisions log, not silent edits.** Every conflict found gets a D-xx entry with what it affects.
- **Out of scope is explicit.** List v1 exclusions and parked backlog items so the agent does not build them.

## ODC and Mentor facts the BRD must account for

Confirm against the user's tenant; mark unverified ones as "ODC hint".
- **Architecture:** service actions called across apps run in their own transactions. Anything that must be atomic (lead conversion, quote sync) stays in one app's server actions. Use separate apps/libraries only where reuse or ownership justifies it (shared theme library; core service app plus UI apps). Admin is usually a role inside an app, not a separate app.
- **Data:** no polymorphic foreign keys (use nullable FKs plus an "exactly one parent" rule); built-in User has no manager/team (add a UserProfile entity); Static Entities for reference lists; audit attributes on every entity; indexes listed per filter; a Counter entity for auto-numbers incremented in the same transaction as the insert.
- **Clashing names** (ActivityStatus, Attachment collided in the CRM build): prefix app-specific names or check the tenant first.
- **Timers:** minimum interval is not guaranteed; write the desired schedule plus a fallback ("shortest allowed interval").
- **Identity:** if real IdP is not available during the build, specify a test persona mechanism (TestUser entity, persona picker screen, current-user variable) and say real roles are assigned to screens at the end.
- **Anonymous first:** during build, screens may be anonymous so each can be tested in a browser; role assignment is a final stage. Put this in the build order if the user wants it.
- **REST:** specify exact status codes (401 bad API key, 400 validation), per-row results for batch calls, row limits and idempotency by external id. The CRM build lost time on a 500 instead of 401 and on batch calls returning only the first row.
- **AI:** drafts only, user clicks Use; define failure/timeout behaviour; use trial model and seed data until a provider and region are named.
- **Mentor limits (observed):** attachments to prompts did not work, so the BRD must be pasteable inline; very large screens or actions (e.g. a full seed reset) stall Mentor, so specify them as several small actions (reset plus a repair action); Mentor may reply with a plan instead of building, so briefs say "implement now, no plan". Keep sections as text and tables so chunks cut cleanly.
- **Verification** happens in a browser against the published app and with curl for REST. Write tests that can be checked that way (visible text, row counts, status codes).

## Per-stage brief to the build agent

Give the BRD plus: "Build stage N exactly as specified. Use the entity, attribute, action and error-code names in the BRD unchanged. Do not add features outside the stage. When done, publish, run the listed tests and report passes and failures before moving on." Then follow `odc-build-playbook` (chunking, polling, publishing, recovery).

## Self-audit checklist

- [ ] Precedence rule, scope, out-of-scope and backlog stated
- [ ] Roles x objects access matrix and create-rights matrix exist; one visibility action named
- [ ] Every entity has attributes, types, required/unique/default rules; statics listed with values
- [ ] Every business rule has an ID, enforcing action and error code
- [ ] Each flow is numbered steps naming the owning server action and transaction boundary
- [ ] Every screen listed with role, data, actions, states (empty/error/loading) and responsive behaviour; mock-up authority stated
- [ ] Notifications, timers, REST (with status codes), AI each specified or marked not applicable
- [ ] Seed data and expected values exist; reset action defined
- [ ] Every rule has at least one test; every test names its rule
- [ ] NFRs (security, privacy, performance, accessibility, audit) concrete or marked ESTIMATE
- [ ] Staged build order with gating tests; open questions as checkboxes
- [ ] No invented facts; assumptions labelled; decisions logged
