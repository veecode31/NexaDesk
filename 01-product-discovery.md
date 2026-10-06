# 01 - Product Discovery: AI Helpdesk

**Version: v1.1 - Requirements Baseline** (aligned with Requirements Freeze & Decisions 1-4) | **Status:** Frozen for implementation after review. No application code has been written.
**Precedence:** Where this document and `02-software-requirements-specification.md` (SRS v1.0) differ, the SRS wins. Sections that describe the reference product record what it does, not what NexaDesk AI adopts.
**Scope of this document:** Analysis of a public reference repository, plus our own proposed product definition.
**Rule:** The reference is used only to understand *capabilities*. Our implementation, naming, UI, and code will be original.

---

## 0. Evidence and limits of this analysis

**What was actually read** (from `github.com/mosh-hamedani/helpdesk`, branch `main`):

| File | Read? | Notes |
|---|---|---|
| `README.md` | Yes | Features, stack, setup, Railway deployment, env vars |
| `project-scope.md` | Yes | Problem, solution, statuses, categories, roles |
| `tech-stack.md` | Yes | Original planned stack |
| `implementation-plan.md` | Yes | 8 phases, all checkboxes unchecked |
| `CLAUDE.md` | Yes | Current conventions, queues, ticket lifecycle, auth, testing |

**What was NOT inspected** (GitHub blocked directory browsing):
`client/`, `server/`, `core/`, `e2e/` source code, the Prisma schema, `Dockerfile`, `railway.toml`, `.github/workflows/`, `.claude/`, `.agents/skills/`.

Anything below about those areas is **inferred from the docs, not verified in code**. Examples: exact database tables, API routes, whether the knowledge base was actually built, CI steps, and server-side test coverage. These are marked *(unverified)* where relevant. If you want those verified, upload the files or paste their contents.

**Documents disagree with each other.** `tech-stack.md` (Claude API, React Router, SendGrid *or* Mailgun) is older than `README.md` / `CLAUDE.md` (OpenAI via Vercel AI SDK, Bun, shadcn/ui, pg-boss, SendGrid). The implementation plan is a pre-build plan (every box unchecked), not a status report. This analysis treats `README.md` and `CLAUDE.md` as the most current.

---

## 1. Product overview

### 1.1 Purpose
An AI-assisted ticket system. Support emails become tickets automatically. AI classifies each ticket, writes a summary, drafts or sends a reply, and routes what it cannot resolve to human agents.

### 1.2 Business problem (as stated by the reference)
A support team receives hundreds of emails daily. Agents read, categorize, and answer each one by hand. This is slow and produces impersonal canned replies.

### 1.3 Target users
- **Reference:** a small internal support team (admins and agents). Customers never log in; they only send and receive email.
- **Ours (decided):** the same staff roles, plus a customer portal in the MVP (decision D-02) and email-to-ticket later.

### 1.4 Main workflows (reference)
1. **Inbound:** customer emails support address, email provider posts it to a webhook, a ticket is created with status `new`.
2. **AI processing (reference behaviour; auto-resolve is NOT adopted by NexaDesk AI):** background jobs classify the ticket and attempt an auto-resolve. Status moves `new` → `processing` → `open` (not resolved) or `resolved` (auto-resolved).
3. **Agent work:** agents see only `open`, `resolved`, and `closed` tickets. They filter and sort the list, open a ticket, read the AI summary and thread, use a suggested reply, and respond.
4. **Outbound:** an agent reply is emailed to the customer. Replies are meant to thread back onto the same ticket.
5. **Admin:** manages agents; views the dashboard.

### 1.5 Where our product intentionally differs
| Area | Reference | Ours (proposed) |
|---|---|---|
| Customer identity | Email sender only | Registered Customer role + optional email-in |
| Roles | Admin, Agent | Admin, Agent, Customer (Super Admin only if we ever become multi-tenant; Future) |
| AI autonomy | AI may auto-resolve tickets | Human-in-the-loop only: AI suggests; staff decide and send. AI never closes, resolves, replies, refunds, or changes accounts (decision OQ-12) |
| Internal notes, timeline, audit log | Not mentioned in docs | Included |
| Priority field | Not mentioned in docs | Included |

---

## 2. Feature inventory

**Source:** `Ref` = described in the reference docs. `New` = our addition.
**Priority:** `MVP` = Release 1. `Post-MVP` = Release 2. `Future` = later. This follows the v2 requirements split Decision D-01 is closed: core helpdesk first, AI and email-in after.

| ID | Feature | Description | Role | Priority | Dependencies | Acceptance criteria | Source |
|---|---|---|---|---|---|---|---|
| F-01 | Login / logout | Email + password login, database-backed sessions, logout | Admin, Agent | MVP | None | Valid login creates a session; logout invalidates it server-side; wrong credentials give a neutral error | Ref |
| F-02 | Seeded first admin | First admin created by seed script from environment values; public sign-up disabled for staff | Admin | MVP | F-01 | Fresh DB + seed yields one admin who can log in; no public staff sign-up route exists | Ref |
| F-03 | Role-based access control | Every endpoint checks auth, role, and object access on the server | All | MVP | F-01 | Test matrix of every role vs every endpoint passes in CI; UI hiding is never the only check | Ref (partial) |
| F-04 | User management | Admin creates, edits, deactivates agents; changes roles | Admin | MVP | F-03 | Admin can create and deactivate an agent; deactivated user's sessions end; last admin cannot be removed | Ref |
| F-05 | Customer accounts and portal | Customers register (verified email), sign in, land on the Customer Dashboard, submit tickets, list their tickets via "My Tickets", read the conversation and reply, and edit their profile (eight portal screens, SRS FR-PORTAL-001..008) | Customer | MVP | F-01, F-03 | Customer lands on Customer Dashboard; sees only own tickets; changing a ticket ID in the URL returns 404; sees the assigned agent's display name only; internal notes/audit info hidden | New |
| F-06 | Account basics | Profile, change password, password reset, email verification | All | MVP | F-01, F-18 | Reset link works once and expires; verification required before customer login | New |
| F-07 | Ticket creation (web) | Create a ticket with subject, description, category | Customer, Agent | MVP | F-03 | Valid input creates a ticket with status Open; invalid input returns field-level errors | Ref (CRUD) |
| F-08 | Ticket numbers | Customer-facing ticket number format `NX-` plus six-digit zero-padded sequence, e.g., NX-000001, NX-000042, NX-001042, unique per organization | All | MVP | F-07 | Formatted ticket number is unique and customer-facing; internal stable identifier is UUID (not DB primary key); appears in lists, detail, search, and emails | New |
| F-09 | Ticket list | Filter by status, category, assignee, date; sort; paginate | Agent, Admin | MVP | F-07 | Filters combine correctly; large lists paginate; empty state shown when no results | Ref |
| F-10 | Ticket detail and thread | View ticket with ordered public replies | All (scoped) | MVP | F-07 | Replies show author and time; customers cannot see internal items | Ref |
| F-11 | Status workflow | Statuses Open, In Progress, Pending, Resolved, Closed with defined transitions and who may perform them (SRS BR-LC-001) | Agent, Admin, System | MVP | F-10 | Illegal transitions are rejected by the API; every transition is logged with previous status, new status, actor, time, optional reason | Ref (statuses differ), New (rules) |
| F-12 | Assignment | Assign or reassign to an agent; unassigned queue | Agent, Admin | MVP | F-04, F-10 | Only active agents/admins assignable; deactivation returns tickets to queue | Ref |
| F-13 | Categories and priority | Admin-managed categories; agent-set priority | Admin, Agent | MVP | F-07 | Customer cannot set priority; deleting a used category is blocked or reassigns | Ref (3 fixed categories), New (priority, admin-managed) |
| F-14 | Internal notes | Agent-only notes on a ticket | Agent, Admin | MVP | F-10 | Notes absent from every customer-facing API response, email, and AI input | New |
| F-15 | Activity timeline | Record of status, priority, assignee, category changes | Agent, Admin | MVP | F-11, F-12 | Each change appears with actor and timestamp; system actions show "System" | New |
| F-16 | Concurrency safety | Version check so simultaneous edits do not silently overwrite | Agent, Admin | MVP | F-10 | Stale update returns a conflict error and the UI offers refresh | New |
| F-17 | Search | Search by ticket number, subject, requester | Agent, Admin | MVP | F-09 | Indexed search returns matches in acceptable time on a seeded large dataset | New |
| F-18 | Outbound email | Reply, confirmation, assignment, status (including a once-only Pending notice when customer action is needed, without duplicates), invitation, and auth emails, sent through an application-level email-service interface (Resend initial provider) | System | MVP | F-21 | Agent reply is emailed; email failure never blocks the reply being saved; failures are visible; provider replaceable without touching domain code; Resend initial outbound provider | Ref (replies), New (others) |
| F-19 | Inbound email to ticket | Webhook receives emails and creates tickets from unregistered senders | System | Post-MVP | F-21, F-20 | Signed webhook creates one ticket per email; unsigned or replayed requests rejected; duplicates ignored | Ref |
| F-20 | Email threading | Replies attach to the correct existing ticket | System | Post-MVP | F-19, F-08 | Reply containing ticket number or thread headers appends to that ticket | Ref (plan) |
| F-21 | Background job queue | Durable jobs with retry, backoff, and visible failures | System | MVP (email only) | Database | Failed job retries then lands in a visible failed state; graceful shutdown finishes or releases jobs | Ref |
| F-22 | AI classification | Suggest category (and priority) for new tickets | System | Post-MVP | F-13, F-21 | Suggestion stored with model and timestamp; agent can accept or override; failure leaves ticket usable | Ref |
| F-23 | AI summaries | Short summary of a long thread | Agent | Post-MVP | F-10, F-21 | Summary excludes internal notes unless viewer is staff; labeled AI-generated | Ref |
| F-24 | AI suggested replies | Draft reply the agent reviews and edits | Agent | Post-MVP | F-10, F-21 | Nothing is sent without agent action; draft never includes internal notes | Ref |
| F-26 | Knowledge base | Curated articles that can inform agent answers and AI suggestions (human-reviewed) | Admin | Future | F-22 | Admin can add and edit articles; an AI suggestion that uses an article cites it and is still reviewed by an agent | Ref (scope doc; build status unverified) |
| F-27 | Dashboard | Counts by status, unassigned, by agent; later category breakdown and recent tickets | Admin, Agent | MVP (basic) | F-09 | Numbers match list filters; each tile links to the filtered list | Ref |
| F-28 | Audit logging | Append-only record of security-relevant actions | Admin | MVP (write only) | F-03 | Logins, role changes, deactivations, deletions, assignments are recorded with actor and time | New |
| F-29 | Rate limiting | Limits on login, reset, ticket creation, webhook | System | MVP | F-01 | Exceeding limits returns 429; enforced in all environments, not only production | Ref (production-only; we change this) |
| F-30 | Error tracking | Sentry with personal data scrubbed | System | MVP | None | Test error appears in Sentry with no ticket content or emails | Ref (optional) |
| F-31 | Deployment | Docker image, CI, staging and production on Railway | System | MVP | All | Merge to main deploys to staging; production deploy is a deliberate step with rollback | Ref |
| F-32 | Automated tests | Component tests, API tests, few browser tests | All | MVP | Each feature | Critical paths covered; CI blocks merge on failure; claims of "tested" only after real runs | Ref (Vitest, RTL, Playwright; server tests unverified) |
| F-33 | File attachments | Upload and download files on tickets | All | Post-MVP | F-10 | Type and size allow-list, malware scan, private storage, signed download links | New |
| F-34 | Analytics | First-response and resolution time, volume trends | Admin | Post-MVP | F-15 | Metric definitions documented and match the numbers shown | New |
| F-35 | SLA policies, CSAT, ticket merge, SSO | Advanced support operations | Admin | Future | Several | Defined when promoted | New |
| F-36 | Invitation and account activation | Staff-created customers and invited staff activate through a secure single-use 7-day link and set their own password | Customer, Agent, Admin | MVP | F-18 | Link is random, stored hashed, single-use, invalidated after use; no session or password is created before activation | New |
| F-37 | Session security | Configurable idle and absolute timeouts, secure cookies, revocation, fixation protection, reauthentication for sensitive operations | All | MVP | F-01 | Values documented and environment-configurable; sensitive action refused without recent password confirmation | New |
| F-38 | Retention and data requests | Documented, configurable retention periods; customer data export and deletion requests; purge disabled until policy approved | Admin | MVP (configuration), Pre-production (purge and requests) | F-28 | Defaults loaded; legal-hold respected; purge logs counts only until approved | New |
| F-39 | Admin overrides | Admin-only reopen after the 7-day window, cancel on a customer's behalf, Closed to Open | Admin | MVP | F-11, F-28 | Each override needs a reason and creates an audit record | New |

---

*F-25 (AI auto-resolve) was removed in v1.0 because the approved AI policy forbids AI from closing or resolving tickets independently. IDs are not renumbered, so there is no F-25.*

## 3. User roles and permissions

The reference defines only **Admin** and **Agent**. Super Admin and Customer are our proposals.

| Role | Definition | Present in reference? |
|---|---|---|
| **Super Admin** | Platform operator across all organizations. Exists **only if** we ever become multi-tenant SaaS. **Future**; not in MVP (decision OQ-15: single organization, tenant-ready design). | No |
| **Admin** | Organization administrator. Created by seed script or another admin. | Yes |
| **Support Agent** | Handles tickets. Created by an admin. | Yes |
| **Customer** | Registered requester with access to own tickets only. | No (email sender only) |
| **System** | AI, jobs, webhooks. Always logged as "System". | Implied |

| Capability | Super Admin | Admin | Agent | Customer |
|---|---|---|---|---|
| Manage organizations / tenants | Yes | No | No | No |
| Manage users and roles | No* | Yes | No | No |
| Manage categories and settings | No* | Yes | No | No |
| View all tickets | No* | Yes | Yes | Own only |
| Create ticket | No* | Yes | Yes (on behalf) | Own |
| Public reply | No* | Yes | Yes | Own tickets |
| Internal notes | No* | Yes | Yes | No |
| Change status | No* | Yes, incl. Closed to Open, reopen after the 7-day window, and cancel override (reason required, audited) | Yes, incl. In Progress to Open | Cancel own ticket only while Open; a reply can reopen within 7 days |
| Change priority / category | No* | Yes | Yes | No |
| Assign / reassign | No* | Yes | Yes | No |
| Use AI suggestions | No* | Yes | Yes | No |
| Enable/disable AI, change retention settings | No* | Yes (reauthentication) | No | No |
| See assigned agent | No* | Full details | Full details | Display name only (no internal notes, private employee info, or internal audit info) |
| View audit log | Platform-level only | Yes (Post-MVP UI) | No | No |
| View dashboard | No* | Yes | Basic | Customer Dashboard landing page (summary, open, pending, recently resolved, recent activity, Create Ticket, link to My Tickets) |

\* A Super Admin should not read tenant ticket content by default. Any support access to tenant data must be explicit, time-limited, and logged.

All permissions are enforced on the backend for every request.

---

## 4. Functional modules

| Module | Responsibility |
|---|---|
| **Identity and Access** | Login, sessions, password flows, role checks, object-level authorization, session revocation |
| **User Management** | Admin lifecycle of staff users; deactivation side effects (sessions, assigned tickets) |
| **Ticketing Core** | Ticket entity, numbering, status workflow, priority, category, assignment, concurrency control |
| **Conversation** | Public replies, internal notes, message immutability and soft delete |
| **Search and Listing** | Filtering, sorting, pagination, indexed search |
| **Notifications (Email Out)** | Templates, sending, delivery failure visibility |
| **Email Ingestion (Email In)** | Webhook verification, parsing, spoof checks, deduplication, threading, HTML sanitization |
| **Job Processing** | Queue, workers, retry, dead-letter handling, graceful shutdown |
| **AI Assistance** | Classification, summaries, and suggested replies, all human-reviewed; redaction, data minimization, prompt-injection defence, cost caps, organization-level kill switch (AI off by default) |
| **Knowledge Base** | Article storage and retrieval to ground AI answers (Future) |
| **Dashboard and Analytics** | Aggregated counts and, later, response-time metrics |
| **Audit and Activity** | Append-only audit log (admin) and per-ticket timeline (staff) |
| **Platform / Ops** | Config, secrets, health checks, logging, error tracking, backups, deployment |

---

## 5. Technical architecture

### 5.1 Reference (as documented)

| Layer | Reference choice |
|---|---|
| Frontend | React, TypeScript, Vite, shadcn/ui, TanStack Query, React Hook Form + Zod, Axios |
| Backend | Express 5, TypeScript, Bun runtime; routers per resource; shared validation helpers |
| Shared code | `core/` package with Zod schemas, types, and constants used by client and server |
| Database | PostgreSQL via Prisma |
| Auth | Better Auth, email/password, database sessions; sign-up disabled; auth routes rate-limited in production only |
| AI | OpenAI model via Vercel AI SDK, called from background jobs |
| Email | SendGrid inbound parse (webhook, secret-verified) and outbound |
| Background workers | pg-boss (Postgres-backed); queues `classify-ticket` and `auto-resolve-ticket` (the latter is reference behaviour we do not adopt); retry with exponential backoff; stopped on SIGTERM/SIGINT |
| Testing | Vitest + React Testing Library (preferred), Playwright for few end-to-end flows; server-side test approach *(unverified)* |
| Deployment | Single Docker image; Express serves built client; Railway; Sentry optional; CI via GitHub Actions *(workflow contents unverified)* |

**Notable design ideas worth keeping:** shared Zod schemas between client and server; Postgres-backed queue (no Redis); system-managed statuses hidden from agents; component-test-first strategy.

### 5.2 Ours (baseline direction; detailed design in the architecture phase)

| Layer | Proposal | Note |
|---|---|---|
| Frontend | As per your stack: React, TS, Vite, Tailwind, shadcn/ui, TanStack Query, React Hook Form, Zod | Verify current library versions before pinning |
| Backend | Node.js, Express, TypeScript, Prisma | Node.js decided (D-08); reference uses Bun |
| Database | PostgreSQL | Indexes and pagination standards from day one |
| Auth | Better Auth; role stored in our own DB and enforced in our middleware | Verify Better Auth capabilities in Phase 3 |
| AI | OpenAI via Vercel AI SDK, server-side only, provider-swappable | Redaction, cost caps, feature flag |
| Email | Resend initial outbound provider behind an application-level `EmailService` abstraction; inbound provider unverified (OQ-11 requires external technical evaluation) | Outbound provider swappable via interface; do not treat Resend as final decision for inbound; inbound evaluated before Post-MVP |
| Workers | Postgres-based queue, separate worker process on Railway | Confirm vs Redis |
| Topology | Same-site frontend and API (one domain or reverse proxy) | Avoids cookie and CSRF complications |
| Testing | Vitest, RTL, Supertest, Playwright | Authorization test matrix in CI |
| Deployment | Docker, GitHub Actions, Railway staging + production, Sentry | Separate secrets and databases per environment |

---

## 6. Missing or additional production requirements

The reference is a good course-scale application. It does not document most of what a real SaaS needs. Recommendations below are for **our** product.

| Area | Gap in reference | Recommendation | When |
|---|---|---|---|
| **Multi-tenancy** | Single organization | Not built in the MVP. Tenant-ready design (OQ-15): `organization_id` on organization-owned tables, one scoped data-access layer, per-organization uniqueness and ticket numbering, a cross-organization isolation test. Full multi-tenancy and Super Admin are Future | Future |
| **Data isolation** | Not documented | Object-level checks on every query; tests that prove customer A cannot read customer B; no cross-user data in AI prompts | MVP |
| **Audit logging** | Not documented | Append-only events with actor (incl. "System"), action, target, time, request ID; viewer UI later | MVP (write), Post-MVP (UI) |
| **Rate limiting** | Auth only, production only | Login, reset, signup, ticket creation, webhook, AI endpoints; same in all environments; lockout/backoff on failed logins | MVP |
| **Secure file uploads** | None | Allow-list types, size caps, malware scanning, private object storage, signed short-lived URLs, never render untrusted HTML/SVG inline | Post-MVP |
| **Backup and recovery** | Not documented | Daily automated backups with 30-day retention as the provisional initial default, defined recovery targets, **a restore actually tested**, runbook | MVP |
| **Observability** | Sentry optional | Structured logs with request IDs, health check, alerts to a channel someone reads, job-failure visibility, Sentry scrubbing of personal data | MVP |
| **Data retention** | Not documented | Soft delete; configurable retention with provisional defaults (SRS 7.8: tickets 24 months after close, audit logs 12 months, backups 30 days); purge job off until policy approved; legal and privacy review before production | Defaults in MVP; review before production |
| **Privacy** | Customer text goes to an AI vendor | Strict privacy-first AI policy (SRS 7.5): backend-only calls, minimum content, redaction, no training use, no prompts in logs, organization-level AI switch, AI off by default; privacy notice; data export and deletion on request; check applicable law (for example GDPR, India's DPDP Act) with a qualified adviser | Before any AI feature |
| **Accessibility** | Not documented | WCAG 2.1 AA goal: keyboard navigation, labels, contrast, focus states, accessible error messages; automated and manual checks | MVP |
| **Email security** | Webhook secret only | SPF/DKIM/DMARC on sending domain; inbound spoof checks; HTML sanitization; loop and auto-reply detection; bounce handling | Post-MVP for inbound |
| **AI safety** | The reference can auto-resolve tickets | Human-in-the-loop only: AI suggests, agents decide and send. Treat ticket text as untrusted (prompt injection); never include internal notes; data minimization and redaction; spend caps; organization-level kill switch; record model, time, task type, and decision, not prompts | With AI |
| **Reliability** | Not documented | Email/AI failure never blocks ticket creation; idempotent jobs; dead-letter handling | MVP |
| **Session security** | Basic | Revoke sessions on deactivation, role change, password change; CSRF; secure cookies; security headers/CSP | MVP |
| **Delivery** | Docker + Railway | Staging and production, versioned migrations, rollback plan, dependency scanning in CI | MVP |
| **Admin MFA** | None | Strongly recommended soon after MVP | Post-MVP |
| **Support operations** | Minimal | SLAs, business hours, canned replies, tags, bulk actions, merge, CSAT | Future |

---

## 7. Proposed release split (summary)

**MVP:** F-01 to F-18 (without inbound email), F-21 (email jobs), F-27 (basic), F-28 (write only), F-29 to F-32, F-36, F-37, F-38 (configuration), F-39.
**Post-MVP:** F-19, F-20, F-22 to F-24, F-33, F-34, audit log viewer, MFA, canned responses, tags, full-text search.
**Future:** F-26, F-35, multi-tenancy, Super Admin, SSO, public API, multi-language. (AI auto-resolve is not planned.)

---

## 8. Decisions (all closed in v1.0 unless marked)

| ID | Decision | Final outcome | Source |
|---|---|---|---|
| D-01 | Product core | Core helpdesk first; AI and email-in after | Approved |
| D-02 | Customer access | Customer portal in the MVP; email-in later | Approved; OQ-2 |
| D-03 | Should AI resolve tickets without a human? | **Never.** AI is human-in-the-loop; auto-resolve removed | OQ-12 |
| D-04 | Single organization or multi-tenant? | Single organization with tenant-ready design; Super Admin is Future | OQ-15 |
| D-05 | Customer sign-up | Open registration with email verification | Approved default |
| D-06 | Status model | Open, In Progress, Pending, Resolved, Closed | OQ-1 |
| D-07 | Categories | Admin-managed, seeded with Billing, Technical, Account, Refund, General | SRS FR-ADMIN-004 |
| D-08 | Runtime | Node.js | Approved default |
| D-09 | Reopen window | 7 calendar days, UTC | OQ-14 |
| D-10 | Email provider | Resend initial outbound provider behind an application-level abstraction; inbound handling unverified (OQ-11 requires external technical evaluation) | OQ-11, Decision 3 |
| D-11 | AI vendor data handling | Strict privacy-first policy | OQ-12, OQ-20 |
| D-12 | Job queue | Postgres-based (design phase to confirm) | Approved default |
| D-13 | Attachments | Post-MVP | Approved default |
| D-14 | Retention | Provisional defaults; legal and privacy review before production | OQ-6, OQ-18 |
| D-15 | Verify un-inspected reference areas | **Not done; optional.** Not needed for the baseline | Open (optional) |
| D-16 | Ticket number format | NX-000001 (prefix NX-, six-digit zero-padded sequence; UUID internal identifier) | Decision 1, OQ-4 |
| D-17 | Customer landing page | Customer Dashboard (summary, open, pending, recently resolved, activity, create action, link to My Tickets) | Decision 2 |

---

## 9. Phase 1 acceptance criteria

- [ ] Reference analysis accepted, with the stated limits in section 0
- [ ] Feature inventory (section 2) approved, including priorities
- [ ] Role model (section 3) approved; Super Admin is Future only
- [ ] Module list (section 4) approved
- [ ] Architecture direction (section 5.2) acknowledged; detailed design happens in the architecture phase
- [ ] Production requirements (section 6) approved
- [ ] Decisions in section 8 reviewed (D-15 optional)

*Next, on approval of the baseline: the revised mockup, then architecture. No UI or backend work has started.*
