# NexaDesk AI: Phase 2 mockup

Frontend-only prototype. HTML5, CSS3, vanilla JS. No backend, no database, no dependencies.
All data is sample data held in memory and **resets on reload**.

## Run
Open `index.html` in a browser. Sign in with any valid email and a password of 6+ characters.

## Files
`index.html` (shell), `styles.css` (CSS variables, responsive), `app.js` (mock data, views, interactions).

## Screens
Login, Dashboard, Ticket Inbox, Ticket Details, Create Ticket, Customers, Team, AI Assistant, Analytics, Settings.

## Limitations
- Not connected to anything: "emails", AI output, invites and settings are simulated.
- AI summaries, classifications and confidence values are canned templates, not model output.
- Remember me and Forgot password are visual only. Login is not real authentication.
- Analytics charts use sample numbers (the agent and priority tables use the ticket data).
- Ticket details open only from lists; no URL routing, so Back/Refresh return to login.
- Tested by hand-checking syntax only; no automated tests or cross-browser testing has been run.

## Review checklist
- [ ] Contrast and focus rings acceptable
- [ ] Mobile menu works under 900px width
- [ ] Filters, sorting, pagination, bulk select behave correctly
- [ ] Status, priority, agent changes show in Activity history
- [ ] Reply vs internal note modes are clearly different
- [ ] Forms show clear validation messages
- [ ] Anything missing before moving to React?
