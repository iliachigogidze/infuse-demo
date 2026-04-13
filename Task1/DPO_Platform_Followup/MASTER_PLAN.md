# DPO Platform Interactive Dashboard — Implementation Plan

## Context
Ilia needs a **functional prototype dashboard** (not a slide deck) for a 30-minute presentation. The dashboard simulates the real DPO Platform with role-based views, campaign creation, and interactive demonstrations of 5 operational scenarios. Published on GitHub Pages at `iliachigogidze/infuse-demo-dashboard`.

---

## Architecture

### Navigation: Two-Axis
- **Top bar**: Role switcher — `DPO Admin` | `Vendor` | `Lead Delivery` | `Client Success`
- **Secondary tabs**: `Campaigns` | `S1: Company Limit` | `S2: Duplicates` | `S3: Req Change` | `S4: Cross-Channel` | `S5: ABM History`
- Switching roles changes what's visible. Some tabs are hidden/grayed per role.

### The "I get it" moment
Click **Vendor** and watch half the data disappear. That single interaction demonstrates the core business constraint (vendors never see each other) better than any document.

---

## Campaign Creation (based on campaign_init_attachment.docx)

The Campaigns tab includes a **"+ New Campaign"** button (DPO Admin / CST roles) that opens a multi-section creation form matching the real campaign initiation document:

### Form Sections

**1. Campaign Info**
- Campaign Name (text)
- Client (text)
- Status (dropdown: Pre / Live / Paused / Closed / Canceled)
- Goal (number) — with optional geo sub-goals (e.g., UK: 124, Ireland: 23)
- Deadline (date)
- Pacing (dropdown: Even-pacing / Front-loaded / Back-loaded)
- Company Limit (number, optional)

**2. Target Audience**
- GEO (multi-select or text)
- Industry (dropdown or "any")
- Revenue (dropdown or "any")
- Company Size (dropdown or "any")
- Target Titles/Roles — table with columns: Action (Include/Exclude), Job Level, Job Area, Keywords

**3. Lists**
- ABM List (toggle yes/no, file upload simulation)
- Suppression List (toggle yes/no)

**4. Qualification**
- Language (dropdown)
- Phone Script Requirements (textarea)
- Qualification Questions — add/remove rows, each with question text + answer options + disqualify flag
- OPT-IN / GDPR toggle

On submit: campaign appears in the campaign list with all metrics initialized. The mock data includes one pre-created campaign ("EU SaaS Decision Makers") so the dashboard isn't empty on first load.

---

## Mock Data (1 pre-loaded campaign, 3 vendors, 8 companies, ~12 leads)

**Campaign:** "EU SaaS Decision Makers" — TechCorp Global, Goal: 200, Company Limit: 2, GEO split (UK/Ireland/Germany), ABM list: yes

**Vendors:** DataReach (UK), LeadForge (Ireland), ProspectIQ (Germany) — each with 2-3 assigned companies

**Companies:** 8 total with slot counters (mix of delivered/in-progress/new)

**Leads:** ~12 with statuses (submitted/in-review/accepted/rejected), tagged v1/v2 for S3

**ABM versions:** v1 (500 companies) → v2 (800: 300 added, 50 removed, 450 unchanged) for S5

---

## Per-Role Views

### DPO Admin (sees everything)
- **Campaigns**: Full campaign list + detail view + "New Campaign" button + vendor assignment panel
- **S1**: Full company table with slot counters, split rule indicator, vendor assignments
- **S2**: Submission log with block reasons, suppression list
- **S3**: v1/v2 rule comparison, countdown timer, lead matrix by version
- **S4**: All-channel touch log, exclusion tags, priority config
- **S5**: Full diff view, removed-company queue

### Vendor (sees only their slice)
- **Campaigns**: "Your Campaign" card — their companies only, no other vendors, no create button
- **S1**: Their 3 companies, slot availability check (click to see remaining)
- **S2**: Lead submission form with live validation (accepted/blocked)
- S3: Notification banner about rule change + grace window
- S4/S5: Not visible (grayed out tabs)

### Lead Delivery
- **Campaigns**: Campaign list with ABM list status, companies by status
- **S5** (main): Version diff viewer, "Commit v2" button, removed-company decision queue with Halt/Grace/Remove buttons
- S1-S4: Read-only or filtered views

### Client Success Team
- **Campaigns**: Campaign list + "New Campaign" button + requirement change history
- **S3** (main): "Log Requirement Change" form, confirmation workflow, version history
- Other tabs: Read-only summaries

---

## Per-Scenario Interactive Elements

| Scenario | Key Interaction | What User Sees |
|----------|----------------|----------------|
| S1 | Click "Submit Lead" on a company | Slot counter decrements 2→1→0, row turns red at 0 |
| S1 | Toggle campaign params | Decision rules cascade (ABM→Geo→Industry→etc) |
| S2 | Type name/email in submission form | "Accepted" or "Blocked" with reason (scope/suppression/fingerprint) |
| S3 | Click "Confirm Switch" | 24h countdown starts, leads split into v1/v2 columns |
| S4 | Click "Claim" on a contact for DPO | Lock icons appear on Email/LinkedIn lanes |
| S4 | Click "Release" | Locks disappear, contact available to other channels |
| S5 | Click "Commit v2" | Diff stats appear (300/50/450), decision queue populates |
| S5 | Click "Halt" or "Grace 5 Days" on removed company | Company status updates |

---

## Technical Approach

**Single HTML file, ~2200 lines, no dependencies.**

```
Lines 1-400:      CSS (design system + role colors + forms + interactive states)
Lines 400-450:    Static header + nav HTML
Lines 450-460:    <div id="content"></div> render target
Lines 460-600:    Mock data constants (campaign fields, companies, leads, vendors)
Lines 600-700:    State management + render() engine + shared helpers
Lines 700-950:    Campaign list + detail + creation form renderers
Lines 950-1700:   Scenario renderer functions (S1-S5, role-aware)
Lines 1700-1950:  Action handlers (submit-lead, create-campaign, confirm-switch, etc.)
Lines 1950-2050:  Event delegation + init
```

**JS pattern**: STATE object → renderer functions return HTML strings → `render()` injects into `#content` → event delegation on `[data-action]` attributes → action handlers modify STATE and re-render.

**Shared helpers** to reduce repetition: `card()`, `table()`, `flowStep()`, `statCard()`.

**Role accent colors**: Admin=#0984e3, Vendor=#00b894, LD=#6c5ce7, CST=#e17055

---

## Build Order

1. **Scaffold**: HTML boilerplate, CSS design system, header/nav, role switcher, tab bar, render engine, mock data
2. **Campaigns tab**: Campaign list view, campaign detail view, metrics cards — proves the render system works
3. **Campaign Creation**: Multi-section form based on campaign_init_attachment.docx fields, form validation, add to campaign list on submit
4. **Role switching**: Vendor sees filtered campaign data, CST sees create button, LD sees ABM status — proves role-based views work
5. **S1 - Company Limit**: Company table with slot counters, submit lead interaction, decision rule cascade
6. **S2 - Duplicates**: Submission form with validation, block reasons (scope/suppression/fingerprint)
7. **S3 - Requirement Change**: Timeline, countdown, version-tagged lead matrix, CST confirmation form
8. **S4 - Cross-Channel**: Channel lanes, claim/release, touch log
9. **S5 - ABM History**: Diff stats, commit flow, removed-company decisions
10. **Polish**: Transitions, empty states, responsive tweaks

---

## Files to Modify
- **Create**: `/Users/iliachigogidze/Documents/Development/INFUSE/Task1/DPO_Platform_Followup/index.html` (overwrite existing presentation version)

## Style Reference
- Reuse design language from `Scenario_1_Decision_Tree.html`: system fonts, #f5f6fa bg, card-based, colored left borders, circular step numbers

## Verification
1. Open index.html in browser — all 4 roles load, tabs switch
2. Click through each scenario's interactive elements
3. Verify vendor view hides other vendors' data
4. Test responsive (works on projector ~1280px)
5. Push to GitHub Pages and verify live URL works
