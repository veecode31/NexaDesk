# 07 - HTML Mockup V2 Review & Verification Report

**Reviewer:** Antigravity (Senior Software Architect / Engineering Lead)  
**Date:** 2026-10-05  
**Version:** 2.0 (Phase 2 Mockup)  
**Status:** Mockup V2 Complete & Verified  
**Artifact Location:** `mockup/` (`index.html`, `styles.css`, `app.js`, `README.md`)

---

## 1. Executive Summary

The Phase 2 prototype has been completely redesigned and rebuilt from scratch into **HTML Mockup V2** under `mockup/`, fulfilling all requirements established in SRS Appendix C, User Stories Epic 8, and the frozen decisions.

The prototype is a zero-dependency, pure frontend application implemented using **HTML5**, **Vanilla CSS3**, and **Vanilla JavaScript**. It incorporates realistic in-memory state, comprehensive role-based isolation between Staff and Customer portals, and strict behavioral enforcement of the 5-status ticket lifecycle.

---

## 2. Screens Implemented

### 2.1 Entry & Authentication Screens

| # | Screen Name | DOM ID | Key Capabilities & Verification |
|---|---|---|---|
| 1 | **Unified Sign In / Login** | `#view-login` | Unified authentication interface for all personas (Admin, Agent, Customer) with auto-routing by authenticated user's role (Staff to Staff Dashboard, Customers to Customer Dashboard per Decision 2). Features 1-click demo persona quick-select (`Admin: Priya`, `Agent: Marcus`, `Customer: Sarah`), 30-day Remember me toggle, neutral error simulation, forgot password link, and activation link. |
| 2 | **Customer Invitation Activation** | `#view-customer-activation` | Activation screen for invited customer (`alex.m@newclient.co`); masked email; token validation; minimum 10-character password enforcement (NIST SP 800-63B); activates account and routes directly into Customer Dashboard. |

---

### 2.2 Staff Screens (Admin & Agent)

| # | Screen Name | DOM ID | Key Capabilities & Verification |
|---|---|---|---|
| 1 | **Operational Dashboard** | `#view-staff-dashboard` | Real-time counts across all five statuses (`Open`, `In Progress`, `Pending`, `Resolved`, `Closed`); "Requires Attention" unassigned & urgent queue; live audit activity stream. |
| 2 | **Ticket Inbox** | `#view-staff-inbox` | Multi-criteria filtering (Status, Priority, Category, Assignee); search by ticket number (`NX-000001`), subject, or customer; sorting; pagination; empty state handling. |
| 3 | **Ticket Details** | `#view-staff-ticket-details` | Customer-facing ticket number header (`NX-000001`), status/priority/category badges; AI Copilot Assistant box with confidence scores, summary, and draft reply insertion; conversation thread distinguishing public replies vs internal notes; staff workflow properties editor; per-ticket audit timeline. |
| 4 | **Create Ticket (on Behalf)** | `#view-staff-create-ticket` | Staff creation flow. Features live customer email detection: existing emails link automatically; unknown emails trigger the **OQ-3 Flow** (`INVITATION_PENDING` customer created in single transaction, 7-day secure token generated, no authenticated session created). |
| 5 | **Customer Management** | `#view-staff-customers` | Complete directory of customers; status filtering (`ACTIVE`, `INVITATION_PENDING`, `DEACTIVATED`); open ticket counters; link to Customer Profile. |
| 6 | **Customer Profile** | `#view-staff-customer-profile` | Detailed customer profile with organization data, invitation token status and expiration countdown, full ticket history, and privacy data export trigger (FR-PRIV-001). |
| 7 | **Team Management** | `#view-staff-team` | Admin-only view listing staff members, roles (`ADMIN`, `AGENT`), active ticket workloads, and user deactivation toggles. |
| 8 | **Analytics** | `#view-staff-analytics` | Post-MVP preview: first response time targets (&lt; 2h), resolution averages, CSAT scores, and volume-by-category breakdown charts. |
| 9 | **AI Assistant & Privacy** | `#view-staff-ai` | Post-MVP preview: workspace kill-switch toggle, transparent privacy-first rules (zero training, 90-day retention, prompts never stored), and interactive prompt classification tester. |
| 10 | **Workspace Settings** | `#view-staff-settings` | Single organization configuration (`NexaDesk Global Support`), Resend outbound email service abstraction display, OQ-11 inbound unverified status note, and data retention policy schedule. |

---

### 2.3 Customer Portal Screens

| # | Screen Name | DOM ID | Key Capabilities & Verification |
|---|---|---|---|
| 1 | **Customer Dashboard** | `#view-customer-dashboard` | **Approved Decision 2 Landing Page:** Status summary cards, **"Needs Your Reply" Spotlight** for Pending tickets, open tickets list, recently resolved tickets with countdown timers for the 7-day reopen window, recent activity stream, Create Ticket button, and link to My Tickets. |
| 2 | **Create Ticket** | `#view-customer-create-ticket` | Customer ticket submission form: subject (min 5 chars), category, description. Priority field is intentionally omitted (assigned automatically per FR-PORTAL-004). |
| 3 | **My Tickets** | `#view-customer-my-tickets` | Full paginated list of tickets submitted by the authenticated customer only. Filter by status, search by number or subject. |
| 4 | **Customer Ticket Details** | `#view-customer-ticket-details` | Customer ticket view: displays assigned agent display name only; shows allowed contextual actions (Cancel if Open, Reopen if Resolved within 7 days, Follow-up if Closed or past window). |
| 5 | **Conversation & Reply** | `#view-customer-ticket-details` | Public message thread. **Strict Isolation:** Internal staff notes are never rendered. Reply composer available when active; automatically updates Pending -> Open upon customer reply. |
| 6 | **Customer Profile** | `#view-customer-profile` | Name edit, read-only email display, password change, data export request, and account deletion request (FR-PRIV-001). |

---

## 3. Workflows Tested & Verified

1. **Ticket ID Format (`NX-000001` through `NX-000008`):**
   - Verified that all tickets use the prefix `NX-` followed by a six-digit zero-padded number.
   - Verified that new tickets created by staff or customers increment sequentially (`NX-000009`, etc.) and are decoupled from internal UUID primary keys.

2. **Customer Landing Page (Decision 2):**
   - Verified that when a customer logs in or switches to the Customer persona, they immediately land on the **Customer Dashboard**.
   - Verified that the "Needs Your Reply" spotlight card displays ticket `NX-000003` (Pending status).
   - Verified that "My Tickets" remains available as the full ticket-list view.

3. **Status Workflow Lifecycle (BR-LC-001):**
   - `In Progress -> Open`: Tested on ticket `NX-000002`. Selecting Open triggers the modal prompt for an audited business reason, records the event in the timeline, and updates status.
   - `Pending -> Open on Customer Reply (OQ-17)`: Tested on ticket `NX-000003`. Submitting a customer reply automatically moves the ticket to `Open` and adds an audit timeline event.
   - `Customer Reopen Window (7 Calendar Days, OQ-14)`:
     - Ticket `NX-000004` (resolved 2 days ago): Customer view shows active countdown (5 days remaining) and "Reopen Ticket" action. Replying reopens the ticket to Open.
     - Ticket `NX-000005` (resolved 12 days ago, past 7 days): Customer reply is blocked; "Create Follow-up Ticket" button is displayed. In staff view, an Agent is blocked from reopening, but an Admin can use **Admin Override Reopen** with a required reason.
   - `Customer Cancellation Only When Open (OQ-10)`: Tested on ticket `NX-000001`. "Cancel Ticket" button is present and prompts for confirmation; sets status to `Closed` with reason `Cancelled by customer`. On non-Open tickets, the cancel button is absent.

4. **Unknown Customer Email Flow (OQ-3):**
   - Entering an unknown email in Staff Create Ticket triggers an amber alert explaining the `INVITATION_PENDING` flow.
   - Submitting provisions a customer record with status `INVITATION_PENDING`, links the ticket, and logs the invitation email without creating an authenticated session.

5. **Customer Invitation Activation Flow (FR-AUTH-011):**
   - Navigating to the activation view for Alex Morgan allows entering a new password with client-side 10-character validation.
   - Successful activation transitions the user to `ACTIVE` and routes to the Customer Dashboard.

6. **Privacy & Internal Notes Isolation (OQ-9):**
   - Verified that on ticket `NX-000001`, staff can view and create yellow-tinted `INTERNAL NOTE` items.
   - Verified that in the Customer view for `NX-000001`, internal notes are completely excluded from the conversation thread.
   - Verified that customers see only the agent's display name ("Marcus Vance"), with private employee emails or audit logs hidden.

7. **AI Copilot Interaction (OQ-12):**
   - Tested AI panel on Ticket Details: clicking "Insert into Composer" copies the suggested draft reply into the public reply composer for staff review and editing before sending.

---

## 4. Issues Found & Fixed During Implementation

| Issue ID | Description | Resolution Applied |
|---|---|---|
| **FIX-01** | Initial draft allowed customer to attempt cancellation on In Progress tickets. | Enforced strict rule that customer cancellation is allowed **only while in Open status** (OQ-10); button is hidden for In Progress, Pending, Resolved, and Closed. |
| **FIX-02** | Customer dashboard was missing dedicated visual countdown for the 7-day reopen window. | Added a clear countdown badge (`⏳ Reopen window: X days left`) on resolved tickets in the Customer Dashboard and Ticket Details views. |
| **FIX-03** | Status dropdown in staff view showed invalid transitions when ticket was Closed. | Updated `renderStaffStatusSelect()` to restrict transitions based strictly on role (only Admin can reopen Closed tickets with an audited reason). |
| **FIX-04** | Form validation on ticket creation allowed empty descriptions. | Added client-side validation requiring minimum length and non-empty input before dispatching state events. |

---

## 5. Remaining Prototype Limitations

1. **In-Memory Storage Only:** State resets to initial sample data upon browser page reload. Data is not persisted to a database or local storage.
2. **Simulated Email & AI Services:** Outbound transactional emails and AI completions are simulated with realistic mock content and toast notifications; no live external API connections exist.
3. **Client-Side Role Enforcement:** The prototype enforces role visibility and actions in JavaScript for UI demonstration purposes. Full cryptographic server-side authorization enforcement will be implemented in the backend phase.

---

## 6. Conclusion & Readiness

The **HTML Mockup V2** is complete, stable, accessible, and responsive. It fulfills all requirements in SRS Appendix C and serves as the visual and interaction reference for the upcoming architecture and backend phases.
