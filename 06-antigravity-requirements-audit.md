# 06 - Requirements Consistency Audit: NexaDesk AI (Post-Decision Freeze Audit)

**Auditor:** Antigravity (Senior Software Architect / Engineering Lead)  
**Date:** 2026-10-05  
**Version:** 1.0 (Requirements Freeze Audit)  
**Status:** Requirements Audit Complete — Requirements Baseline Frozen  
**Scope:** Comprehensive cross-document consistency audit across `01-product-discovery.md` (v1.1), `02-software-requirements-specification.md` (v1.0), `03-user-stories.md` (v1.0), `04-acceptance-criteria.md` (v1.0), and `05-decision-log.md` (v1.0).

---

## 1. Executive Summary & Freeze Readiness

A comprehensive consistency audit was conducted across the five project requirements documents following the application of Product Owner Decisions 1 through 4 and the resolution/mapping of all 13 open questions (OQ items) originally listed in SRS Appendix B.2.

### Freeze Verdict: **APPROVED FOR REQUIREMENTS FREEZE**

1. **Zero Open Business Contradictions:** All previous contradictions (C-01, C-02, C-03, C-04, C-05) have been completely reconciled across all five documents.
2. **Complete OQ Mapping:** All 13 OQ items are formally classified as **APPROVED**, **PROVISIONAL**, or **REQUIRES EXTERNAL REVIEW**. None remain silently unaddressed.
3. **No Premature Implementation:** Architecture, database schema design, API implementation, React code, and backend code have NOT been started.
4. **Next Milestone:** The project is fully unblocked to proceed to the **Phase 2 Mockup Revision** (SRS Appendix C), which remains the mandatory gate prior to frontend or backend implementation.

---

## 2. Final Decision Mapping (Decisions 1–4 and 13 OQ Items)

| Decision / OQ # | Topic & Summary | Decision Applied | Status | Affected Documents |
|---|---|---|---|---|
| **DECISION 1 / OQ-4** | **Ticket ID Format** | Customer-facing ticket number format `NX-000001` (prefix `NX-`, six-digit zero-padded sequence, e.g., NX-000001, NX-000042, NX-001042). Uniquely generated per organization. Decoupled from database primary key (internal stable identifier is UUID). | **APPROVED** | `01-product-discovery.md`<br>`02-software-requirements-specification.md`<br>`03-user-stories.md`<br>`04-acceptance-criteria.md`<br>`05-decision-log.md` |
| **DECISION 2** | **Customer Landing Page** | The authenticated Customer landing page is the Customer Dashboard. It provides: ticket summary (counts by five statuses), open tickets, pending tickets ("Needs your reply"), recently resolved tickets, recent activity, Create Ticket action, and navigation to "My Tickets". "My Tickets" remains the full, paginated ticket-list view. | **APPROVED** | `01-product-discovery.md`<br>`02-software-requirements-specification.md`<br>`03-user-stories.md`<br>`04-acceptance-criteria.md`<br>`05-decision-log.md` |
| **DECISION 3 / OQ-11** | **Email Provider Architecture** | **Outbound:** Resend is the initial outbound transactional email provider, accessed strictly behind an application-level email service abstraction (`EmailService`) so the provider can be replaced without domain logic impact.<br>**Inbound:** OQ-11 specifically governs inbound email processing. Resend is NOT the final architectural decision for inbound; inbound handling requires dedicated technical review before Post-MVP. | Outbound: **APPROVED**<br>Inbound: **REQUIRES EXTERNAL REVIEW** | `01-product-discovery.md`<br>`02-software-requirements-specification.md`<br>`04-acceptance-criteria.md`<br>`05-decision-log.md` |
| **DECISION 4 / OQ-1** | **Five Ticket Statuses Only** | Exactly five ticket statuses: `Open`, `In Progress`, `Pending`, `Resolved`, `Closed`. Enforce valid transitions via state machine. System rejects illegal transitions. | **APPROVED** | `01-product-discovery.md`<br>`02-software-requirements-specification.md`<br>`03-user-stories.md`<br>`04-acceptance-criteria.md`<br>`05-decision-log.md` |
| **DECISION 4 / OQ-2** | **Mockup Revision** | Rebuild mockup before code implementation: 8 customer screens; separate Admin, Agent, and Customer interfaces with role-based routing. No React until approved. | **APPROVED** | `02-software-requirements-specification.md`<br>`03-user-stories.md`<br>`04-acceptance-criteria.md`<br>`05-decision-log.md` |
| **DECISION 4 / OQ-3** | **Unknown Customer Inbound/Email** | Unknown inbound email or staff ticket creation creates a customer record with status `INVITATION_PENDING` in a single transaction. No password and no authenticated session created. Secure single-use invitation token with 7-day lifetime. Unique `(organization_id, normalized_email)`. | **APPROVED** | `01-product-discovery.md`<br>`02-software-requirements-specification.md`<br>`03-user-stories.md`<br>`04-acceptance-criteria.md`<br>`05-decision-log.md` |
| **DECISION 4 / OQ-5 & OQ-16** | **Session Security Parameters** | Idle timeout 8 hours; absolute lifetime 30 days with Remember me, 24 hours without Remember me; Secure, HttpOnly, SameSite=Lax cookies; session fixation protection (new session ID on sign-in, role change, password change); reauthentication window 15 minutes for sensitive operations (FR-AUTH-010). Environment-configurable. Better Auth capabilities to be verified during technical design. | **PROVISIONAL** | `02-software-requirements-specification.md`<br>`04-acceptance-criteria.md`<br>`05-decision-log.md` |
| **DECISION 4 / OQ-6 & OQ-18** | **Data Retention & Backup Schedule** | Retention schedule: tickets/timeline 24 months after close, audit logs 12 months, outbound email records 90 days, AI results 90 days, tokens 7 days, unactivated accounts 90 days. Database backups 30 daily backups (provisional). Purge job disabled until legal/privacy policy approved. Legal review required prior to production. | **REQUIRES EXTERNAL REVIEW** | `01-product-discovery.md`<br>`02-software-requirements-specification.md`<br>`04-acceptance-criteria.md`<br>`05-decision-log.md` |
| **DECISION 4 / OQ-7** | **Pending Customer Notification** | Pending customer notification occurs when customer action is required ("Needs your reply"), without duplicate notifications. | **APPROVED** | `02-software-requirements-specification.md`<br>`04-acceptance-criteria.md`<br>`05-decision-log.md` |
| **DECISION 4 / OQ-8** | **Availability Target** | 99.5% monthly uptime is an internal operational target, not a contractual SLA. | **APPROVED** | `02-software-requirements-specification.md`<br>`04-acceptance-criteria.md`<br>`05-decision-log.md` |
| **DECISION 4 / OQ-9** | **Customer Visibility of Staff** | Customers see the assigned agent's display name only. Customers must NOT see internal notes, private employee information, or internal audit information. | **APPROVED** | `01-product-discovery.md`<br>`02-software-requirements-specification.md`<br>`03-user-stories.md`<br>`04-acceptance-criteria.md`<br>`05-decision-log.md` |
| **DECISION 4 / OQ-10** | **Customer Cancellation** | Customer cancellation is allowed only while the ticket is Open. Admin may override where explicitly allowed with audit logging. | **APPROVED** | `01-product-discovery.md`<br>`02-software-requirements-specification.md`<br>`03-user-stories.md`<br>`04-acceptance-criteria.md`<br>`05-decision-log.md` |
| **DECISION 4 / OQ-12 & OQ-20** | **AI Privacy-First Policy & Exclusions** | AI is privacy-first, human-in-the-loop, and disabled by default. AI may classify, summarize, suggest priority, and suggest replies. AI must NOT autonomously close tickets, send customer replies, issue refunds, or perform administrative actions. AI input excludes internal notes and attachments. Retain AI results for 90 days. AI can be disabled per organization. Provider no-training terms verified before enablement. | OQ-12: **APPROVED**<br>OQ-20: **PROVISIONAL** | `01-product-discovery.md`<br>`02-software-requirements-specification.md`<br>`03-user-stories.md`<br>`04-acceptance-criteria.md`<br>`05-decision-log.md` |
| **DECISION 4 / OQ-13** | **In Progress to Open Transition** | In Progress -> Open is allowed for Agents and authorized Admins, and must be audited with previous/new status, actor, timestamp, and optional reason. Assignee and messages preserved. | **APPROVED** | `02-software-requirements-specification.md`<br>`03-user-stories.md`<br>`04-acceptance-criteria.md`<br>`05-decision-log.md` |
| **DECISION 4 / OQ-14 & OQ-19** | **Customer Reopen Window & Admin Override** | Customer reply reopens Resolved ticket to Open within 7 calendar days (UTC). After the 7-day customer reopen window, only Admin can reopen (audited with actor, timestamp, reason). Customer replies after window are blocked and offered a linked follow-up ticket (`parent_ticket_id`). | **APPROVED** | `01-product-discovery.md`<br>`02-software-requirements-specification.md`<br>`03-user-stories.md`<br>`04-acceptance-criteria.md`<br>`05-decision-log.md` |
| **DECISION 4 / OQ-15** | **Organization Model** | The MVP uses a single Organization/workspace. The schema and authorization boundaries carry `organization_id` (tenant-ready) to avoid blocking future multi-tenancy, but full multi-tenancy and Super Admin are NOT implemented in MVP. | **APPROVED** | `01-product-discovery.md`<br>`02-software-requirements-specification.md`<br>`03-user-stories.md`<br>`04-acceptance-criteria.md`<br>`05-decision-log.md` |
| **DECISION 4 / OQ-17** | **Pending to Open on Customer Reply** | Pending -> Open occurs when the customer replies. No pending_reason field in MVP. | **APPROVED** | `02-software-requirements-specification.md`<br>`03-user-stories.md`<br>`04-acceptance-criteria.md`<br>`05-decision-log.md` |
| **DECISION 4 / OQ-21** | **Invitation Token Lifetime** | Invitation activation uses a secure, single-use invitation token with a 7-day lifetime. | **APPROVED** | `01-product-discovery.md`<br>`02-software-requirements-specification.md`<br>`03-user-stories.md`<br>`04-acceptance-criteria.md`<br>`05-decision-log.md` |

---

## 3. Remaining Provisional Decisions

The following requirements remain **PROVISIONAL** by design and will be validated during upcoming engineering spikes:

| ID | Item | Provisional Baseline | Validation Plan & Timing |
|---|---|---|---|
| **OQ-16** | **Session Durations & Reauthentication** | - Idle timeout: 8 hours<br>- Absolute lifetime (Remember me): 30 days<br>- Absolute lifetime (no Remember me): 24 hours<br>- Reauthentication window: 15 minutes for sensitive operations (FR-AUTH-010) | **Phase 4 Architecture Spike:** Verify Better Auth session revocation, cookie management, and step-up reauthentication hooks against latest Better Auth documentation. Fallback to custom Express middleware if Better Auth lacks native support. |
| **OQ-20** | **AI Provider Settings & Result Retention** | - 90-day retention for `AiSuggestion`<br>- Strict exclusion of internal notes and attachment contents from AI prompts<br>- Organization-level disable toggle | **Post-MVP AI Gate (Acceptance Gate 6.4):** Author `docs/ai-data-handling.md`, verify OpenAI/vendor enterprise data processing agreements (confirming zero-training on customer data), and pass prompt injection test suite before enabling. |

---

## 4. Items Requiring External Review

The following requirements are explicitly flagged as **REQUIRES EXTERNAL REVIEW** and do not block MVP implementation, but represent gates prior to future releases:

| ID | Item | Current Working Baseline | Review Required & Timing | Release Gate Affected |
|---|---|---|---|---|
| **OQ-11** | **Inbound Email Architecture** | Inbound email handling is unverified. Resend is used for outbound only. No inbound provider is assumed or invented. | **Technical Provider Spike:** Perform an evaluation of inbound webhook providers (e.g., Mailgun, Postmark, AWS SES, Resend inbound webhooks) prior to Post-MVP email-to-ticket implementation. | Gate 6.5 (Post-MVP: email-to-ticket) |
| **OQ-18** | **Data Retention & 30-Day Backup Retention** | - Tickets & timeline: 24 months post-close<br>- Audit logs: 12 months<br>- Backups: 30 daily backups (provisional)<br>- Email & AI logs: 90 days<br>- Purge job disabled in MVP (counts-only dry run) | **Legal & Privacy Review:** Qualified privacy counsel review against applicable regulations (GDPR, DPDP Act) before production deployment. Purge job remains disabled until policy sign-off. | Gate 6.3 (Production Release) |

---

## 5. Audit of Previous Contradictions

All contradictions identified during the preliminary audit have been systematically reviewed and resolved:

| ID | Issue | Initial State | Resolution Applied | Current Status |
|---|---|---|---|---|
| **C-01** | **Ticket ID Format** | Doc 01 specified zero-padded `NX-000001`; Doc 02 gave example `NX-1042`. | **Decision 1 applied:** Format is standardized to `NX-000001` (prefix `NX-`, six-digit zero-padded sequence). Database primary key is UUID. Applied across all docs. | **RESOLVED** |
| **C-02** | **F-25 Status in Decision Log** | Doc 05 noted doc 01 still listed F-25 as Future, but doc 01 line 109 correctly noted F-25 was removed in v1.0. | **Doc 05 corrected:** Aligned Section 2.3 to acknowledge that Doc 01 correctly documented the removal of F-25. | **RESOLVED** |
| **C-03** | **Customer Landing Page** | US-002 stated customers land on "My tickets"; FR-PORTAL-002/003 stated Customer Dashboard. | **Decision 2 applied:** Authenticated customer landing page is the Customer Dashboard. "My Tickets" remains the full list view. US-002, US-038, and FR-PORTAL aligned. | **RESOLVED** |
| **C-04** | **Throttling 5 vs 6 Attempts** | Potential wording variance between 5 failed attempts vs 6th attempt blocked. | **Verified consistent:** 5 failed attempts trigger lockout; the 6th attempt (even with valid password) is blocked. | **RESOLVED** |
| **C-05** | **Email Provider & OQ-11 Scope** | Doc 01 stated Resend was decided for outbound; Doc 05 marked OQ-11 open. | **Decision 3 applied:** Clarified that Resend is approved for outbound behind `EmailService` abstraction; OQ-11 governs inbound processing only and requires external technical review. | **RESOLVED** |

---

## 6. Coverage & Minor Gaps Audit

1. **M-01 (Bot Protection for Registration):** Open registration rate limiting and neutral responses are specified in FR-AUTH-002. Turnstile/CAPTCHA is documented as a design-phase consideration if abuse is observed.
2. **M-02 (Email Template Management):** Confirmed as code-managed templates for MVP in FR-NOTIF-001.
3. **M-04 (Pending to In Progress Story Coverage):** Story coverage added to US-014 in `03-user-stories.md` and documented in coverage notes. Transition matrix is 100% covered.
4. **M-05 (Password Policy):** NIST SP 800-63B alignment (10-character minimum, common password denylist) confirmed in NFR-SEC-004.
5. **Customer Permissions Isolation:** Strict verification performed across all documents confirming that customers see assigned agent display name only and never see internal notes, private employee info, or internal audit info (OQ-9).

---

## 7. Unresolved Blockers for Next Phase

| Blocker ID | Item | Impact | Next Action / Resolution |
|---|---|---|---|
| **IB-01** | **Phase 2 Mockup Revision** | Implementation in React is strictly blocked until the Phase 2 mockup is revised and approved per SRS Appendix C. | Proceed immediately to Phase 2 mockup revision (vanilla HTML/CSS/JS shell in workspace). |
| **IB-02** | **Technical Spikes (Architecture Phase)** | Auth module and search strategy need confirmation. | Better Auth capability spike & Prisma search strategy during Phase 4 architecture. |
| **IB-03** | **Production Legal Review** | Production launch blocked until retention periods and privacy notice are approved. | Does NOT block mockup revision or MVP development. Scheduled prior to production launch. |

---

## 8. Requirements Freeze Conclusion

All requirements across:
- `docs/01-product-discovery.md` (v1.1)
- `docs/02-software-requirements-specification.md` (v1.0)
- `docs/03-user-stories.md` (v1.0)
- `docs/04-acceptance-criteria.md` (v1.0)
- `docs/05-decision-log.md` (v1.0)

are **100% consistent, complete, and frozen**. No code implementation or architecture modifications should take place until the revised mockup is completed and approved.
