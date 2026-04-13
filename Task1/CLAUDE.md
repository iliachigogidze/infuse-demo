# DPO Platform Follow-Up Task — CLAUDE.md

## Project Overview
This is INFUSE Task 1 — a follow-up submission for the DPO (Demand Performance Optimization) Platform assignment. The goal is to demonstrate approach to 5 operational scenarios from process, data, and system perspectives, then create a visual HTML dashboard deliverable published on GitHub.

## The 5 Scenarios
1. **Lead per Company Limit** — Enforcing max leads per company across multiple vendors who can't see each other. Solution: vendor scope splitting (ABM list > geo > industry > size) + slot counter + optional pre-check.
2. **Duplicate Leads from Different Vendors** — Preventing duplicate lead submissions. Solved by Scenario 1's scope split + suppression lists + fingerprint check for same-person-different-company edge case.
3. **Client Requirement Change (24h Delivery)** — Managing mid-campaign criteria changes while delivering approved leads under old rules within 24h. Solution: confirmation gate + version tagging + countdown dashboard.
4. **Cross-Channel Contacts** — Preventing same person being contacted by DPO and email marketing on same campaign. Solution: auto-suppression on contact entry + shared touch log.
5. **ABM List Change History** — Tracking versions of client target account lists, protecting completed work, handling dropped companies. Solution: domain-first identity + versioned list with auto-diff + status flags + delta-only processing + removed-company rules.

## Key Constraints
- Vendors must NEVER know about each other
- Documents must be brief, well-processed, informative (executive audience)
- One page per scenario target
- Visual deliverables required

## Current Deliverable Goal
Build a single-page HTML dashboard that presents all 5 scenarios visually, suitable for GitHub Pages publishing.

## Working Directory Structure
- `DPO_Platform_Followup/` — All scenario analysis, approaches, and working docs
- `Deliverables/` — Final deliverable outputs
- `DPO_Platform_Followup/Scenario_1_Decision_Tree.html` — Existing visual for Scenario 1 (reference for style)
- `DPO_Platform_Followup/index.html` — Main single-page dashboard app

## Tech Stack for Dashboard
- Single HTML file, self-contained (inline CSS/JS)
- No build tools, no dependencies
- Clean, professional design matching existing Scenario 1 Decision Tree style
- Must work on GitHub Pages (static HTML)

## Role-Based Views
The dashboard has three role modes (toggled via the top bar):
- **DPO Admin** — Full access. Vendors page shows all vendors with lead counts, account assignments, and summary stat cards.
- **Client Success** — Same as Admin for navigation, but Vendors page omits lead counts and account assignments (shown only to Admin). A scope note explains the restriction.
- **Vendor** — Scoped access. Vendors nav item is hidden entirely (vendors must never see each other). If on the Vendors tab when switching to Vendor role, auto-redirects to Campaigns.

## Vendors Page (`renderVendors()`)
- Accessible from the left sidebar ("Vendors" nav item, badge shows vendor count)
- Table columns: Vendor name (colored dot), Region, Status, Campaigns active
- Admin-only columns: Leads (accepted/pending), Accounts assigned
- Admin also sees 4 summary stat cards above the table (Vendors, Total Leads, Accepted, In Review)
- CST sees a scope note callout in place of admin-only columns

## Campaign Selector (Leads Tab)
The campaign switcher in `renderS1()` is a custom dropdown (not pill buttons):
- Trigger button: shows active campaign name + status dot + chevron, min-width 220px, white card with border
- Dropdown menu (`#cam-dd-menu`): absolute-positioned below trigger, lists each campaign with status dot, name, client name, and a status badge pill
- Active campaign is highlighted with blue left border + `#f0f7ff` tint
- Click-outside closes via the global `document.addEventListener('click', ...)` handler at bottom of file
- Secondary metadata (client + deadline) shown inline next to the trigger, not inside it

## Style Reference
The existing `Scenario_1_Decision_Tree.html` uses:
- System font stack (-apple-system, BlinkMacSystemFont, 'Segoe UI')
- Light background (#f5f6fa), dark text (#2d3436)
- Card-based layout with colored left borders
- Section headers with uppercase labels
- Clean, information-dense design
