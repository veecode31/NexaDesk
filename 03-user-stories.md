# 03 - User Stories: NexaDesk AI

**Status:** v1.0 (Requirements Freeze Candidate - All OQ Items Resolved & Mapped). Derived from `02-software-requirements-specification.md` (SRS v1.0) with Decisions 1-4 and all OQ resolutions applied (see `05-decision-log.md`).
**Statuses used:** Open, In Progress, Pending, Resolved, Closed.
**Format:** As a [user], I want [functionality], so that [benefit]. Acceptance criteria in Given/When/Then.
**Release:** MVP, Post-MVP, or Future. **FR** = linked requirement in the SRS.
Roles: Customer, Agent, Admin, System.

---

## Epic 1: Accounts and access

### US-001 Register as a customer (MVP, FR-AUTH-002)
As a **customer**, I want to create an account and verify my email, so that I can raise and track tickets securely.
- **Given** I am on the registration page, **When** I submit a valid name, email, and password, **Then** I see a message to check my email and cannot sign in until I verify.
- **Given** I received a verification link, **When** I open it before it expires, **Then** my account becomes verified and I can sign in.
- **Given** I register with an email that already has an account, **When** I submit, **Then** I see the same confirmation as a new user and the existing owner is emailed (the page does not reveal the account exists).

### US-002 Sign in and out (MVP, FR-AUTH-001, FR-AUTH-003)
As a **user**, I want to sign in and out, so that only I can access my data.
- **Given** valid credentials and a verified active account, **When** I sign in, **Then** staff land on the staff dashboard and customers land on the Customer Dashboard (Decision 2).
- **Given** a wrong password or unknown email, **When** I sign in, **Then** I see the same neutral error in both cases.
- **Given** I signed out, **When** the old session is reused, **Then** the server rejects it.

### US-003 Reset my password (MVP, FR-AUTH-004)
As a **user**, I want to reset a forgotten password, so that I can regain access without support.
- **Given** I request a reset for any email, **When** I submit, **Then** I see the same confirmation message either way.
- **Given** I open a valid reset link, **When** I set a compliant password, **Then** the old password stops working and my other sessions end.
- **Given** the link is expired or already used, **When** I open it, **Then** I see an error and an option to request a new one.

### US-004 Protect against repeated login attempts (MVP, FR-AUTH-006)
As an **admin**, I want repeated failed logins to be slowed or blocked, so that accounts resist guessing attacks.
- **Given** 5 failed attempts within 15 minutes (proposed threshold), **When** another attempt is made, **Then** it is rejected with a generic "Try again later" message.
- **Given** the window has passed, **When** the user signs in correctly, **Then** access is granted and the counter resets.

### US-005 Change my password and profile (MVP, FR-AUTH-005)
As a **user**, I want to edit my name and change my password, so that I keep my account current and secure.
- **Given** I am signed in, **When** I enter a wrong current password, **Then** the change is refused and nothing is modified.
- **Given** I change my password correctly, **When** it saves, **Then** my other sessions are signed out.

---

## Epic 2: Creating and finding tickets

### US-006 Create a ticket (MVP, FR-TICKET-001)
As a **customer**, I want to submit a ticket with a subject, description, and category, so that support can help me.
- **Given** I am signed in, **When** I submit valid fields, **Then** the ticket is created as Open with a customer-facing ticket number `NX-000001` (six-digit zero-padded sequence; Decision 1, OQ-4) and I receive a confirmation email.
- **Given** my subject is shorter than 5 characters, **When** I submit, **Then** I see a field-level error and nothing is created.
- **Given** the email provider is down, **When** I submit, **Then** the ticket is still created.

### US-007 Create a ticket for a customer (MVP, FR-TICKET-002, FR-AUTH-011)
As an **agent**, I want to create a ticket on a customer's behalf, even if they have no account yet, so that issues reported by phone or chat are tracked.
- **Given** I enter an existing customer's email (any letter case or surrounding spaces), **When** I submit, **Then** the ticket is linked to that customer and the timeline notes I created it.
- **Given** I enter an email that does not exist, **When** I submit, **Then** a customer record with status "invitation pending" is created in the same organization, the ticket is linked to it, no password and no session are created, and an invitation email is queued through the configured email service.
- **Given** two agents create tickets for the same new email at the same moment, **When** both save, **Then** only one customer record exists.
- **Given** the email service is down, **When** I submit, **Then** the ticket and customer are still created and the invitation is retried.

### US-008 See my tickets only (MVP, FR-TICKET-004, FR-AUTH-007)
As a **customer**, I want to see only my own tickets, so that my information stays private.
- **Given** I am signed in, **When** I open "My tickets", **Then** I see only tickets where I am the requester.
- **Given** I change a ticket ID in the address bar to another customer's ticket, **When** the page loads, **Then** I get a "not found" response.

### US-009 Work the inbox (MVP, FR-TICKET-004, FR-TICKET-005)
As an **agent**, I want to filter, sort, search, and page through tickets, so that I find what to work on quickly.
- **Given** tickets of mixed statuses, **When** I filter by status Open and priority High, **Then** only tickets matching both are shown.
- **Given** I search by ticket number or subject words, **When** I submit, **Then** matching tickets appear and others do not.
- **Given** no tickets match, **When** results load, **Then** I see an empty state with a "clear filters" action.

### US-010 Find unassigned tickets (MVP, FR-TICKET-004)
As an **agent**, I want a view of unassigned tickets, so that nothing is left without an owner.
- **Given** some tickets have no assignee, **When** I filter Agent = Unassigned, **Then** only those appear.

---

## Epic 3: Working a ticket

### US-011 Read a ticket (MVP, FR-TICKET-006)
As an **agent**, I want to see the full conversation, internal notes, details, and history of a ticket, so that I have complete context.
- **Given** I open a ticket, **When** it loads, **Then** I see messages, internal notes, current status, priority, assignee, and timeline.

### US-012 Reply to a customer (MVP, FR-CONV-001, FR-NOTIF-001)
As an **agent**, I want to send a public reply, so that the customer gets my answer.
- **Given** I am on a ticket, **When** I send a non-empty reply, **Then** it appears in the conversation and the customer is emailed.
- **Given** this is the first public staff reply, **When** it saves, **Then** first-response time is recorded.
- **Given** I send an empty reply, **When** I submit, **Then** I see a validation message.

### US-013 Add an internal note (MVP, FR-CONV-002)
As an **agent**, I want private notes on a ticket, so that colleagues see context the customer should not.
- **Given** the composer is in "Internal note" mode, **When** I save, **Then** the note is visible to staff only and no email is sent.
- **Given** I am the customer, **When** I view or fetch the ticket, **Then** no internal note is present in the page or the API response.

### US-014 Change status (MVP, FR-TICKET-007)
As an **agent**, I want to move a ticket through its workflow, so that everyone knows where it stands.
- **Given** a ticket is Open, **When** I set In Progress, **Then** the status updates and the timeline records previous status, new status, me, and the time.
- **Given** a ticket is In Progress, **When** I move it back to Open with or without a reason, **Then** it succeeds, the assignee and all messages are unchanged, and the reason (if given) appears in the staff timeline.
- **Given** a ticket is Pending, **When** an Agent or Admin sets In Progress, **Then** it moves to In Progress and the timeline records the transition.
- **Given** a ticket is Closed, **When** an Agent tries to set Open, **Then** it is refused (Admin only, audited).
- **Given** I set Resolved, **When** it saves, **Then** the resolution time is stored in UTC and the customer is emailed.
- **Given** I am a customer, **When** I try any status change other than cancelling my own Open ticket, **Then** it is refused.

### US-015 Set priority and category (MVP, FR-TICKET-008)
As an **agent**, I want to set priority and category, so that urgent work is handled first and routed correctly.
- **Given** I am staff, **When** I change priority to Urgent, **Then** it saves and a timeline event is written.
- **Given** I am a customer, **When** I try to change priority, **Then** the request is refused.

### US-016 Assign a ticket (MVP, FR-TICKET-009)
As an **agent**, I want to take or assign a ticket, so that one person is accountable.
- **Given** a ticket is unassigned, **When** I choose "Take it", **Then** I become the assignee.
- **Given** I assign it to another agent, **When** it saves, **Then** that agent is emailed and the timeline records the change.
- **Given** an agent is deactivated, **When** I open the assignee list, **Then** they are not selectable.

### US-017 Avoid overwriting a colleague (MVP, FR-TICKET-010)
As an **agent**, I want to be warned when someone else changed the ticket, so that I don't silently overwrite their work.
- **Given** two agents opened the same ticket, **When** both change the status and the second saves after the first, **Then** the second sees a conflict message and keeps their draft text.

### US-018 Follow my ticket's progress (MVP, FR-NOTIF-001, FR-TICKET-006)
As a **customer**, I want email updates and a clear status, so that I know when to act.
- **Given** an agent replies, **When** the reply is sent, **Then** I receive an email with the ticket number and a link.
- **Given** I view my ticket, **When** it loads, **Then** I see status, priority, and the assigned agent's display name only.

### US-019 Reply, reopen, or cancel as a customer (MVP, FR-TICKET-011, FR-CONV-001)
As a **customer**, I want to reply to or cancel my ticket, so that I stay in control of my request.
- **Given** my ticket is Pending, **When** I reply, **Then** it returns to Open (OQ-17 approved).
- **Given** my ticket was Resolved 3 days ago, **When** I reply, **Then** it reopens to Open and the reopening is recorded in the timeline and audit log.
- **Given** my ticket was resolved exactly 7 x 24 hours ago (UTC), **When** I reply at that moment, **Then** it still reopens; **When** I reply one second later, **Then** the reply is blocked.
- **Given** the window has passed or the ticket is Closed, **When** I open it, **Then** I see an explanation and a "Create follow-up ticket" action that links to the old ticket.
- **Given** my ticket is Open, **When** I cancel it, **Then** it becomes Closed with reason "Cancelled by customer".

### US-019a Admin override of the reopen window (MVP, FR-TICKET-007, FR-TICKET-011)
As an **admin**, I want to reopen a ticket after the 7-day window, so that I can help a customer in exceptional cases.
- **Given** a ticket was resolved 10 days ago, **When** an Admin reopens it, **Then** it moves to Open and an audit record shows who did it and why.
- **Given** I am an Agent, **When** I try the same after the window, **Then** it is refused (OQ-19 approved).

### US-020 Delete an inappropriate message (MVP, FR-CONV-003)
As an **admin**, I want to remove a message from view, so that abusive or sensitive content can be handled.
- **Given** a message exists, **When** I soft-delete it, **Then** customers no longer see it, staff see "Deleted by [Admin]", and the action is audited.
- **Given** I am an Agent, **When** I try to delete a message, **Then** it is refused.

---

## Epic 4: Administration

### US-021 Invite and manage staff (MVP, FR-ADMIN-002, FR-ADMIN-003)
As an **admin**, I want to invite agents, change roles, and deactivate users, so that the right people have the right access.
- **Given** I invite an email as Agent, **When** I submit, **Then** an invitation email is sent and the user (status invitation pending) cannot sign in until they activate through the invitation link.
- **Given** I deactivate an agent, **When** it saves, **Then** their sessions end and their open tickets become unassigned.

### US-022 Keep at least one admin (MVP, FR-ADMIN-003)
As an **admin**, I want the system to stop me removing the last admin, so that the workspace is never locked out.
- **Given** only one active Admin exists, **When** I try to demote or deactivate that Admin, **Then** the system refuses and nothing changes.

### US-023 Manage categories (MVP, FR-ADMIN-004)
As an **admin**, I want to maintain ticket categories, so that routing reflects how we work.
- **Given** a category is in use, **When** I try to delete it, **Then** I can only deactivate it and old tickets keep it.
- **Given** I add a name that already exists (any letter case), **When** I save, **Then** I see an error.

### US-024 Trace important actions (MVP, FR-ADMIN-005)
As an **admin**, I want sensitive actions recorded, so that I can investigate what happened.
- **Given** a role change, deactivation, assignment, or message deletion occurs, **When** it completes, **Then** an audit record exists with actor, time, and target.
- **Given** an automated job caused the change, **When** the record is written, **Then** the actor is "System", never blank.

### US-025 See workload at a glance (MVP, FR-DASH-001)
As an **admin or agent**, I want a dashboard of ticket counts, so that I can see the backlog and who is busy.
- **Given** tickets exist, **When** I open the dashboard, **Then** I see totals by status, unassigned count, and counts per agent, and each number opens the matching filtered list.

---

## Epic 5: Reliability and operations

### US-026 Emails never block work (MVP, FR-NOTIF-002)
As a **customer**, I want my ticket saved even if email is down, so that my request isn't lost.
- **Given** the email provider is failing, **When** a ticket or reply is created, **Then** it saves and the email retries with backoff.
- **Given** all retries fail, **When** the final attempt ends, **Then** the email is marked FAILED and an Admin can see it.

### US-027 Know when something breaks (MVP, NFR-OBS-001..003)
As an **admin**, I want alerts on errors and failed jobs, so that I can act before customers complain.
- **Given** the error rate spikes or a backup fails, **When** the threshold is hit, **Then** an alert reaches the monitored channel.

---

## Epic 6: AI assistance (Post-MVP, strict privacy-first policy, decision OQ-12)
All AI stories require the AI gate in `04-acceptance-criteria.md` (section 6.4). AI stays off until then.

### US-028 Get a category and priority suggestion (Post-MVP, FR-AI-001)
As an **agent**, I want a suggested category and priority on new tickets, so that triage is faster.
- **Given** AI is enabled for the organization and a ticket is created, **When** the job finishes, **Then** I see a suggestion labeled AI-generated with a confidence value.
- **Given** a suggestion is shown, **When** I reject it, **Then** the ticket's fields are unchanged and my decision is recorded.
- **Given** the AI provider fails, **When** the job errors, **Then** the ticket works normally with no suggestion.

### US-029 Read an AI summary (Post-MVP, FR-AI-002)
As an **agent**, I want a short summary of a long thread, so that I catch up quickly.
- **Given** a ticket has several messages, **When** I open it, **Then** I see a summary labeled AI-generated, visible only to users who may see that ticket's internal content.
- **Given** I am the customer, **When** I open the ticket, **Then** no AI output is shown.

### US-030 Edit an AI-drafted reply (Post-MVP, FR-AI-003)
As an **agent**, I want a draft reply I can edit, so that I answer faster without losing control.
- **Given** a draft is available, **When** I choose "Use AI suggestion", **Then** it fills the editor and nothing is sent until I review it and press send.
- **Given** a ticket has internal notes, **When** the draft is generated, **Then** the notes are not used as input.
- **Given** the draft proposes a refund, account change, or closure, **When** I send, **Then** no such action happens; only the text is sent.

### US-031 Switch AI off for the organization (Post-MVP, FR-AI-004)
As an **admin**, I want an organization-level AI switch and a spend cap, so that cost and risk stay bounded.
- **Given** I disable AI (after reauthentication), **When** the next job runs, **Then** no request goes to the provider and the change is audited.
- **Given** the monthly cap is reached, **When** another job starts, **Then** AI pauses and I am alerted.

### US-044 Keep personal data out of AI requests (Post-MVP, FR-AI-006)
As a **customer**, I want only the minimum necessary text of my ticket sent to the AI provider, so that my data stays private.
- **Given** a ticket contains a password, token, or other personal data, **When** an AI request is built, **Then** it is redacted or omitted and only the minimum content for that task is sent.
- **Given** an AI request completes, **When** I (or an Admin) look at the logs, **Then** the full prompt and ticket content are not in them; only model, time, task type, and outcome are recorded.
- **Given** a ticket contains instructions aimed at the AI ("ignore your rules"), **When** AI processes it, **Then** the output is still validated as a suggestion and cannot trigger any action.

## Epic 7: Later releases (Post-MVP)

### US-032 Create tickets by email (Post-MVP, FR-EMAIL-001, FR-EMAIL-002)
As a **customer**, I want to email support and have a ticket created, so that I don't need the portal.
- **Given** a valid signed inbound email, **When** it is received, **Then** a ticket is created once, even if the provider retries.
- **Given** a reply that quotes a ticket number from the requester's address, **When** received, **Then** it is added to that ticket.
- **Given** an invalid signature, **When** received, **Then** nothing is stored.

### US-033 Attach files (Post-MVP, FR-ATT-001)
As a **customer**, I want to attach a screenshot, so that support understands my issue.
- **Given** a PNG under the size limit, **When** I upload, **Then** it is scanned, stored privately, and linked to my message.
- **Given** a disallowed file type, **When** I upload, **Then** it is refused with a reason.

### US-034 Review analytics (Post-MVP, FR-DASH-002)
As an **admin**, I want volume and response-time trends, so that I can plan staffing.
- **Given** enough historical tickets, **When** I open Analytics, **Then** each metric matches its documented definition.

### US-035 Act on many tickets at once (Post-MVP, FR-TICKET-013)
As an **agent**, I want bulk status or assignee changes, so that I can clear queues quickly.
- **Given** I select five tickets and choose Resolved, **When** one transition is illegal, **Then** four succeed and I am told which one failed and why.

### US-036 Review the audit log (Post-MVP, FR-ADMIN-007)
As an **admin**, I want to search the audit log, so that I can answer "who changed this and when".
- **Given** audit records exist, **When** I filter by actor and date, **Then** only matching records appear.

---

## Epic 8: Customer portal and activation (MVP)

### US-037 Activate an invited account (MVP, FR-AUTH-011)
As an **invited customer or staff member**, I want to set my own password from a secure link, so that I can use the account without anyone else knowing my password.
- **Given** I received an invitation, **When** I open the link before it expires and set a compliant password, **Then** my account becomes active and I can sign in.
- **Given** the link is expired or already used, **When** I open it, **Then** I see an error and can ask for a new invitation.
- **Given** staff created a ticket for me, **When** I activate and sign in, **Then** I see that ticket in My tickets.

### US-038 Customer Dashboard landing page (MVP, FR-PORTAL-003)
As a **customer**, I want an authenticated landing page dashboard of my tickets, so that I immediately see what needs my attention and can take action.
- **Given** I am signed in as a Customer, **When** I land on my home view, **Then** I see the Customer Dashboard providing: a ticket summary (counts by the five statuses), open tickets, pending tickets ("Needs your reply"), recently resolved tickets, recent activity, a "Create Ticket" action, and navigation to "My Tickets" (Decision 2).
- **Given** I navigate to "My Tickets", **When** the page opens, **Then** I have access to the full, paginated ticket-list view with search and filters ("My Tickets" remains the full ticket-list view).
- **Given** I am viewing the Customer Dashboard, **When** data loads, **Then** no staff-only notes, private staff information, or internal audit information is visible.

### US-039 Submit and follow tickets in my portal (MVP, FR-PORTAL-004, 005, 006, 007)
As a **customer**, I want to submit tickets, list mine, read the conversation, and reply, so that I never need to email support separately.
- **Given** I am in the portal, **When** I submit a valid ticket, **Then** it appears in My tickets as Open.
- **Given** I filter My tickets by status, **When** the list loads, **Then** I see only my tickets with that status.
- **Given** the reply window is closed, **When** I open the ticket, **Then** the reply box is replaced by an explanation and the follow-up action.

### US-040 Manage my profile (MVP, FR-PORTAL-008, FR-AUTH-005)
As a **customer**, I want to edit my name and change my password, so that my details stay correct.
- **Given** I change my password, **When** I confirm my current password, **Then** the change saves and my other sessions end.

### US-041 Use the right interface for my role (MVP, FR-UI-001)
As a **user**, I want an interface made for my role, so that I only see what I need.
- **Given** I sign in as Customer, Agent, or Admin, **When** I land, **Then** I reach the Customer portal, Agent workspace, or Admin console respectively.
- **Given** a customer opens a staff URL directly, **When** the request is made, **Then** the server refuses it (the interface split is never the only protection).

## Epic 9: Session security, privacy, and retention (MVP)

### US-042 Stay signed in safely (MVP, FR-AUTH-009)
As a **user**, I want sessions that expire sensibly and end when I sign out or change my credentials, so that a stolen or old session cannot be misused.
- **Given** I am idle beyond the configured idle timeout (provisional 8 hours), **When** I act, **Then** I must sign in again.
- **Given** I sign in, **When** the session is created, **Then** a new session identifier is issued (no session fixation).
- **Given** I changed my password elsewhere, **When** my old session makes a request, **Then** it is rejected.

### US-043 Confirm my password for sensitive actions (MVP, FR-AUTH-010)
As an **admin**, I want to be asked for my password again before sensitive actions, so that an unattended session cannot be abused.
- **Given** I last authenticated more than 15 minutes ago (provisional), **When** I change a role, deactivate a user, change retention settings, or disable AI, **Then** I am asked to confirm my password first.
- **Given** I confirm successfully, **When** I repeat the action within the window, **Then** I am not asked again.

### US-045 Configure retention (MVP, FR-ADMIN-008)
As an **admin**, I want retention periods that are documented and configurable, so that the organization can meet its obligations.
- **Given** the default periods are loaded, **When** I view retention settings, **Then** I see each period with a note that legal and privacy review is required before production.
- **Given** the purge job has not been approved, **When** the schedule runs, **Then** it deletes nothing and logs counts only.

### US-046 Ask for my data to be exported or deleted (Pre-production, FR-PRIV-001)
As a **customer**, I want to request export or deletion of my data, so that I control my personal information.
- **Given** I submit a request, **When** an Admin reviews it, **Then** personal fields are removed or anonymized unless law or security requires retention, and the decision is recorded.

## Coverage note
Every MVP functional requirement in the SRS v1.0 maps to at least one story or to the acceptance tests in `04-acceptance-criteria.md`. FR-TICKET-003 (ticket numbers NX-000001), FR-TICKET-012 (timeline), FR-CONV-004 (visibility filtering), FR-AUTH-007/008 (authorization, session revocation), FR-ADMIN-001 (seed admin), FR-ADMIN-006 (organization settings) and FR-UI-002 (required UI states) are covered mainly through acceptance tests. FR-AI-005 (auto-resolve) was withdrawn and has no story. Pending to In Progress is covered in US-014.
