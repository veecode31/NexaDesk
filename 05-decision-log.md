# 05 - Decision Log and Summary: NexaDesk AI

**Status:** Authoritative decision record for NexaDesk AI requirements baseline. Frozen for requirements freeze. No implementation has started.
**Applies to:** `01-product-discovery.md` (v1.1), `02-software-requirements-specification.md` (v1.0), `03-user-stories.md` (v1.0), `04-acceptance-criteria.md` (v1.0), `06-antigravity-requirements-audit.md` (v1.0).

---

## 1. Authoritative Decisions & Mapping

All 13 open questions from SRS Appendix B.2 and product owner decisions have been resolved and mapped:

| ID | Decision / Topic | Applied as | Status | Where Applied |
|---|---|---|---|---|
| **DECISION 1 / OQ-4** | **Ticket ID Format** | Customer-facing ticket number format `NX-000001` (prefix `NX-`, six-digit zero-padded sequence, e.g., NX-000001, NX-000042, NX-001042). Must be unique per organization. Internal stable identifier is UUID (not the database primary key). | **APPROVED** | SRS FR-TICKET-003, Section 8.1; 01 F-08; 03 US-006; 04 FR-TICKET-003 |
| **DECISION 2** | **Customer Landing Page** | The authenticated Customer landing page is the Customer Dashboard. The Dashboard provides: ticket summary (counts by five statuses), open tickets, pending tickets ("Needs your reply"), recently resolved tickets, recent activity, Create Ticket action, and navigation to "My Tickets". "My Tickets" remains the full ticket-list view. | **APPROVED** | SRS FR-PORTAL-002, FR-PORTAL-003; 01 F-05, Section 3; 03 US-002, US-038; 04 FR-PORTAL |
| **DECISION 3 / OQ-11** | **Email Provider Architecture** | **Outbound:** Resend is the initial outbound transactional email provider, accessed through an application-level email service abstraction (`EmailService`) so the provider can be replaced later.<br>**Inbound:** OQ-11 refers specifically to inbound email handling. Inbound email processing remains unverified and requires external technical review before Post-MVP. Do NOT treat Resend as final for inbound. | Outbound: **APPROVED**<br>Inbound: **REQUIRES EXTERNAL REVIEW** | SRS C-6; 01 F-18, Section 5.2; 04 section 3.4, 6.5 |
| **DECISION 4 / OQ-1** | **Five Ticket Statuses Only** | Exactly five ticket statuses: Open, In Progress, Pending, Resolved, Closed. Enforce valid transitions via state machine. | **APPROVED** | SRS 1.4, BR-LC-001..010; 01 F-11, D-06; 03 line 4; 04 section 3.1 |
| **DECISION 4 / OQ-2** | **Mockup Revision** | Revise mockup before implementation: 8 customer screens; separate Admin, Agent, Customer interfaces with role-based landing. No React until approved. | **APPROVED** | SRS FR-PORTAL-001..008, FR-UI-001, Appendix C; 03 Epic 8; 04 section 6.1 |
| **DECISION 4 / OQ-3** | **Unknown Customer Inbound/Email** | Unknown inbound email/customer creates a customer record with status INVITATION_PENDING in one transaction. No password, no authenticated session. Invitation activation via secure single-use token (7-day lifetime). Duplicate prevention by unique `(organization_id, normalized_email)`. | **APPROVED** | SRS FR-TICKET-002, FR-AUTH-011, Section 8; 01 F-36; 03 US-007, US-037; 04 section 3.7 |
| **DECISION 4 / OQ-5 & OQ-16** | **Session Security Parameters** | Idle timeout 8 hours; absolute lifetime 30 days with Remember me, 24 hours without Remember me; Secure, HttpOnly, SameSite=Lax cookies; session fixation protection (new session ID on sign-in, role change, password change); reauthentication window 15 minutes for sensitive operations (FR-AUTH-010). Environment-configurable. Better Auth capabilities to be verified during technical spike. | **PROVISIONAL** | SRS FR-AUTH-009/010, BR-SES-001..004, NFR-SEC-005, 012..014; 03 US-042, 043; 04 section 3.9 |
| **DECISION 4 / OQ-6 & OQ-18** | **Data Retention & Backup Schedule** | Retention periods: tickets/timeline 24 months after close, audit logs 12 months, outbound email records 90 days, AI results 90 days, tokens 7 days, unactivated accounts 90 days. Database backups 30 daily backups (provisional). All retention periods require legal/privacy review prior to production launch. Purge job disabled until policy approved. | **REQUIRES EXTERNAL REVIEW** | SRS FR-ADMIN-008, FR-PRIV-001, 7.8, BR-RET; 03 US-045, 046; 04 section 3.11, 6.3 |
| **DECISION 4 / OQ-7** | **Pending Customer Notification** | Pending customer notification occurs when customer action is required ("Needs your reply"), without duplicate notifications. | **APPROVED** | SRS BR-NT-001, FR-NOTIF-001; 04 section 3.4 |
| **DECISION 4 / OQ-8** | **Availability Target** | 99.5% monthly uptime is an internal operational target, not a contractual SLA. | **APPROVED** | SRS NFR-AVAIL-001; 04 section 5 |
| **DECISION 4 / OQ-9** | **Customer Visibility of Staff** | Customers see the assigned agent's display name only. Customers must NOT see internal notes, private employee information, or internal audit information. | **APPROVED** | SRS FR-PORTAL-006, BR-RBAC-001; 01 F-05, Section 3; 03 US-018; 04 section 4 |
| **DECISION 4 / OQ-10** | **Customer Ticket Cancellation** | Customer cancellation is allowed only while the ticket is Open. Admin may override where explicitly allowed with audit logging. | **APPROVED** | SRS BR-LC-001, BR-LC-009, BR-RBAC-001; 01 Section 3; 03 US-019; 04 section 3.1 |
| **DECISION 4 / OQ-12 & OQ-20** | **AI Privacy-First & Exclusions** | AI is privacy-first, human-in-the-loop, and disabled by default. AI may classify, summarize, suggest priority, and suggest replies. AI must NOT autonomously close tickets, send customer replies, issue refunds, or perform administrative actions. AI must not expose secrets or credentials. AI input excludes internal notes and attachments. Retain AI results for 90 days. AI can be disabled per organization. Provider no-training terms verified before enablement. | OQ-12: **APPROVED**<br>OQ-20: **PROVISIONAL** | SRS 3.5, 7.5 (BR-AI-001..015), 9.3; 01 Section 6; 03 Epic 6; 04 section 3.10, 6.4 |
| **DECISION 4 / OQ-13** | **In Progress to Open Transition** | In Progress -> Open is allowed for Agents and authorized Admins, and must be audited with previous/new status, actor, timestamp, and optional reason. Assignee and messages preserved. | **APPROVED** | SRS BR-LC-001, BR-LC-007, FR-TICKET-007; 04 section 3.1; 03 US-014 |
| **DECISION 4 / OQ-14 & OQ-19** | **Customer Reopen Window & Admin Override** | Customer reply reopens Resolved ticket to Open within 7 calendar days (UTC). After the 7-day customer reopen window, only Admin can reopen (audited with actor, timestamp, reason). Customer replies after window are blocked and offered a linked follow-up ticket (`parent_ticket_id`). | **APPROVED** | SRS BR-LC-008, FR-TICKET-011, BR-RBAC-001; 01 F-39; 03 US-019, 019a; 04 section 3.8 |
| **DECISION 4 / OQ-15** | **Organization Model** | The MVP uses a single Organization/workspace. The schema and authorization boundaries carry `organization_id` (tenant-ready) to avoid blocking future multi-tenancy, but full multi-tenancy and Super Admin are NOT implemented in MVP. | **APPROVED** | SRS 2.4 C-4, Section 8; 01 D-04; 03 US-041; 04 section 4 |
| **DECISION 4 / OQ-17** | **Pending to Open on Customer Reply** | Pending -> Open occurs when the customer replies. No pending_reason field in MVP. | **APPROVED** | SRS BR-LC-010; 03 US-019; 04 section 3.1 |
| **DECISION 4 / OQ-21** | **Invitation Token Lifetime** | Invitation activation uses a secure, single-use invitation token with a 7-day lifetime. | **APPROVED** | SRS FR-AUTH-011, Section 8.1; 01 F-36; 03 US-037; 04 section 3.7 |

---

## 2. Key Clarifications and Alignment Notes

1. **Ticket ID Format (Decision 1):** Customer-facing format is `NX-000001` (six digits zero-padded, prefix `NX-`). This is purely a customer-facing display sequence, completely decoupled from the database primary key (UUID).
2. **Customer Landing Page (Decision 2):** Authenticated customers land on the Customer Dashboard (providing ticket summary, open tickets, pending tickets, recently resolved tickets, recent activity, Create Ticket action, and link to My Tickets). "My Tickets" remains the full paginated ticket-list view.
3. **Email Provider & Inbound Scope (Decision 3):** Resend is confirmed as the initial outbound transactional email provider behind an application-level `EmailService` abstraction. OQ-11 refers specifically to inbound email ingestion, which is unverified and deferred to a dedicated technical evaluation before Post-MVP. Resend is not assumed to be the inbound solution.
4. **Pending to In Progress Workflow:** Clarified in SRS BR-LC-001, acceptance criteria section 3.1, and user story US-014 that an Agent or Admin can manually transition a ticket from Pending to In Progress.
5. **AI Auto-Resolve Withdrawn:** `01-product-discovery.md` line 109 correctly notes F-25 was removed in v1.0 because the approved AI policy forbids AI from closing or resolving tickets independently.
6. **Admin Overrides:** Admin may override where explicitly allowed (e.g. reopening after 7 days, Closed to Open, cancellation on customer behalf), and the action must always be audited with actor, timestamp, and reason.

---

## 3. What Changed Across Documents

| Document | Applied Changes | Version / Status |
|---|---|---|
| `01-product-discovery.md` | Aligned F-05 (Customer Dashboard landing), F-08 (NX-000001 format), F-18 (Resend behind EmailService abstraction), Section 3 (staff display name only, hidden internal notes/audit info), and Section 8 decisions table (D-10, D-16, D-17). | v1.1 Baseline |
| `02-software-requirements-specification.md` | Updated FR-TICKET-003 & Section 8.1 (`NX-000001` six-digit format, UUID PK); FR-PORTAL-002 & FR-PORTAL-003 (Customer Dashboard landing page); FR-PORTAL-006 (agent display name only); C-6 & NFR-EML (EmailService abstraction); NFR-AVAIL-001 (99.5% operational target); BR-NT-001 (Pending notification once); Appendix B (all 13 OQ items mapped with APPROVED/PROVISIONAL/REQUIRES EXTERNAL REVIEW). | v1.0 Freeze Candidate |
| `03-user-stories.md` | Updated US-002 (customers land on Customer Dashboard); US-006 (NX-000001 format); US-014 (Pending to In Progress transition covered); US-019/US-019a (approved OQ status notes); US-038 (detailed Customer Dashboard story with link to My Tickets); coverage note updated. | v1.0 Freeze Candidate |
| `04-acceptance-criteria.md` | Updated FR-TICKET-003 test criteria (NX-000001 format decoupled from UUID PK); FR-PORTAL portal row (Customer Dashboard landing); Section 3.4 notification matrix (Pending notification once without duplicates); Section 6.1 prototype release gate; Section 7 review log (all 13 OQ items mapped). | v1.0 Freeze Candidate |
| `05-decision-log.md` | Completely updated to record Decisions 1 to 4 and full 13 OQ mapping with formal statuses. | v1.0 Freeze Record |
| `06-antigravity-requirements-audit.md` | Cross-document consistency audit verifying zero remaining contradictions and readiness for requirements freeze. | v1.0 Freeze Audit |

---

## 4. Requirements Freeze & Review Status

### 4.1 Approved Items (Ready for Architecture & Implementation)
- OQ-1: Five ticket statuses (Open, In Progress, Pending, Resolved, Closed)
- OQ-2: Mockup revision with 8 customer screens & 3 separate interfaces
- OQ-3: Unknown customer invitation flow (INVITATION_PENDING, single-use token)
- OQ-4 / Decision 1: Ticket ID format `NX-000001` (six digits zero-padded, UUID PK)
- OQ-7: Pending customer notification once when action required
- OQ-8: Availability target 99.5% monthly (operational target, not SLA)
- OQ-9: Staff display name only to customers; internal notes/audit hidden
- OQ-10: Customer cancellation allowed only while Open
- OQ-12: Strict privacy-first AI policy (15 rules, human-in-the-loop, disabled by default)
- OQ-13: In Progress -> Open transition allowed with audit and optional reason
- OQ-14: 7 calendar days customer reopen window (UTC)
- OQ-15: Single Organization workspace with tenant-ready schema
- OQ-17: Customer reply on Pending moves to Open
- OQ-19: Only Admin can reopen after 7 days (audited)
- OQ-21: Invitation token lifetime 7 days
- Decision 2: Customer Dashboard authenticated landing page
- Decision 3: Resend initial outbound email provider behind application-level abstraction

### 4.2 Provisional Items (Subject to Technical Spikes During Design)
- **OQ-16 (Session Security):** 8h idle timeout, 30d Remember me, 24h no Remember me, 15 min reauthentication window. Technical spike on Better Auth capabilities will verify native vs middleware enforcement.
- **OQ-20 (AI Provider Settings & Retention):** 90-day AI result retention, exclusion of internal notes and attachments. Provider contractual terms (no-training policy) and `docs/ai-data-handling.md` to be confirmed before Post-MVP AI enablement.

### 4.3 Items Requiring External Review (Tracked Non-Blocking Gates)
- **OQ-11 (Inbound Email Ingestion):** Technical evaluation of inbound email provider options deferred to Post-MVP design phase. (Does not block MVP helpdesk).
- **OQ-18 (Data Retention & Backup Retention):** Retention periods (including 30-day provisional backup retention) flagged for legal/privacy review before production launch. Purge job remains disabled until policy is approved. (Does not block MVP development).

---

## 5. Next Steps
Requirements are officially **frozen**. The next milestone is the **revised Phase 2 mockup** (SRS Appendix C). No React or backend application implementation begins until the revised mockup is approved.
