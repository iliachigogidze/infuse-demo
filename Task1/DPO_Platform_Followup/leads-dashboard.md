# Leads Dashboard — Documentation

Single-file interactive dashboard (`index.html`) demonstrating 5 DPO operational scenarios. Self-contained HTML/CSS/JS — no build tools, no dependencies. Hosted on GitHub Pages.

---

## Layout

```
Role Bar (sticky top)
Header + Tab Bar
Content area (max-width: 1600px)
  └── rendered by getTabContent(tab)
Modal overlays (campaign creation, S1 lead submission)
```

---

## Role System

Four roles selectable from the top bar. Role is stored in `STATE.role` and controls what data, tabs, and actions are visible.

| Role | ID | Color | Access |
|---|---|---|---|
| DPO Admin | `admin` | `#0984e3` | Full platform — all vendors, all tabs |
| Vendor | `vendor` | `#00b894` | Own companies and leads only; S4/S5 tabs hidden |
| Lead Delivery | `ld` | `#6c5ce7` | All campaigns; ABM-focused callouts |
| Client Success | `cst` | `#e17055` | All campaigns; can create campaigns |

Active vendor identity is `STATE.currentVendor` (default `v1` = DataReach). To simulate a different vendor, change this value in the JS.

Tab visibility per role is controlled by `updateTabVisibility()` via the `grayMap` object.

---

## Tabs

| Tab ID | Label | Scenario |
|---|---|---|
| `campaigns` | Campaigns | Campaign list + detail view |
| `s1` | Company Limit | Scenario 1 — Lead per company cap |
| `s2` | Duplicates | Scenario 2 — Duplicate lead prevention |
| `s3` | Req Change | Scenario 3 — 24h requirement change |
| `s4` | Cross-Channel | Scenario 4 — Email marketing suppression |
| `s5` | ABM History | Scenario 5 — Target list versioning |

---

## Mock Data

### `VENDORS`
Three vendors — each scoped to one geography to prevent overlap.

| ID | Name | Geo | Color |
|---|---|---|---|
| `v1` | DataReach | UK | `#0984e3` |
| `v2` | LeadForge | Ireland | `#00b894` |
| `v3` | ProspectIQ | Germany | `#6c5ce7` |

### `COMPANIES`
8 target accounts split across vendors. Each has `slots` (max leads allowed) and `used` (leads already consumed). Used to enforce the company limit in S1.

### `CAMPAIGNS`
8 campaigns across Live / Paused / Closed statuses. Each has a `script` field (plain-text qualification notes) and optionally a `qualQuestions` array of structured questions with answer options and disqualify flags. Campaign questions drive the CQ column headers in the S1 table.

| ID | Name | Status | Vendors |
|---|---|---|---|
| `cam1` | EU SaaS Decision Makers | Live | v1, v2, v3 |
| `cam2` | UK FinTech C-Suite | Live | v1 |
| `cam3` | DACH Manufacturing IT Leaders | Live | v3 |
| `cam4` | Nordics Cloud Transformation | Live | v2 |
| `cam5` | Healthcare IT Decision Makers | Paused | v1, v2 |
| `cam6` | EU Energy Sector CxOs | Paused | v3 |
| `cam7` | UK Retail Digital Transformation | Closed | v1 |
| `cam8` | EMEA MarTech Buyers | Closed | v1, v2 |

Each campaign also carries a `vendorGoals` map (`{ vendorId: number }`) specifying each vendor's individual lead target within the campaign. Used by `vendorStatsForCampaign()` to compute per-vendor progress.

### `S1_LEADS`
24 pre-seeded leads across `cam1`, `cam2`, and `cam3`. Used exclusively in the S1 tab. Each lead carries a `campaign` field for filtering, plus all 32 Remote Sheet fields: `dpoId`, `firstName`, `lastName`, `company`, `email`, `jobTitle`, `tel`, `country`, `city`, `state`, `postalCode`, `address`, `compSize`, `industry`, `revenue`, `guide`, `optin`, `cq1`, `cq2`, `cq3`, `prooflink`, `prospectProoflink`, `recordingLink`, `reworkedLeads`, `reworkStatus`, `reworkRejReason`, `vendor`, `status`, `ccStatus`, `rejReason`, `submDate`, `notes`.

---

## Scenario 1 — Company Limit (`s1` tab)

### Concept
Vendors submit leads against a per-company slot cap. The cap is enforced centrally — vendors never learn that other vendors are participating.

### Remote Sheet (32 columns)

Matches `provider_remote_gsheet_example_copy.xlsx`. Column header colours reflect the real spreadsheet palette.

| # | Column | Color group | Who fills |
|---|---|---|---|
| 1 | # (row index) | Grey (`#b2bec3`) | System |
| 2 | Rejection Reason | Purple (`#8e7cc3`) | DPO |
| 3 | Prooflink | Purple (`#8e7cc3`) | DPO |
| 4 | Report Submission Date | Green (`#93c47d`) | DPO |
| 5 | Reworked Leads | Green (`#93c47d`) | DPO |
| 6 | Status of Reworked Leads | Green (`#93c47d`) | DPO |
| 7 | Rejection Reason (Rework) | Green (`#93c47d`) | DPO |
| 8 | Call Notes | Green (`#93c47d`) | DPO |
| 9 | Prospect Prooflink | Green (`#93c47d`) | DPO |
| 10 | Recording Link | Green (`#93c47d`) | DPO |
| 11 | First Name | Blue (`#6d9eeb`) | Vendor |
| 12 | Last Name | Blue (`#6d9eeb`) | Vendor |
| 13 | Company | Blue (`#6d9eeb`) | Vendor |
| 14 | Email | Blue (`#6d9eeb`) | Vendor |
| 15 | Job Title | Blue (`#6d9eeb`) | Vendor |
| 16 | Telephone | Blue (`#6d9eeb`) | Vendor |
| 17 | Guide | Blue (`#6d9eeb`) | Vendor |
| 18 | Country (Region) | Blue (`#6d9eeb`) | Vendor |
| 19 | Company Size | Blue (`#6d9eeb`) | Vendor |
| 20 | Address | Blue (`#6d9eeb`) | Vendor |
| 21 | City | Blue (`#6d9eeb`) | Vendor |
| 22 | State | Blue (`#6d9eeb`) | Vendor |
| 23 | Postal Code | Blue (`#6d9eeb`) | Vendor |
| 24 | Industry | Blue (`#6d9eeb`) | Vendor |
| 25 | Revenue | Blue (`#6d9eeb`) | Vendor |
| 26 | Timestamp | Blue (`#6d9eeb`) | System |
| 27 | CQ1 | Blue (`#6d9eeb`) | Vendor |
| 28 | CQ2 | Blue (`#6d9eeb`) | Vendor |
| 29 | CQ3 | Blue (`#6d9eeb`) | Vendor |
| 30 | Status | Purple (`#8e7cc3`) | DPO |
| 31 | Opt-in | Blue (`#6d9eeb`) | Vendor |
| 32 | Comments | Green (`#93c47d`) | DPO |

**CQ column headers** are dynamic — derived from the active campaign's `qualQuestions` array (structured) or parsed from the `script` text field (split on `?` / `;`, prefix `"Qualify: "` stripped). Falls back to `CQ1` / `CQ2` / `CQ3` if no questions are defined.

- Vendor role: rows filtered to own leads only; `+ Add Lead` button visible
- Admin role: all rows visible; `+ Add Lead` button visible

### Master Sheet (12 columns)

Internal DPO view (Admin / LD / CST only). All vendors' rows combined. Columns: `#`, Vendor chip, Contact + email, Company + slot badge, Job Title, Country, Status, CC Status, Rejection Reason, Submitted date, Opt-in, CQ1 / CQ2 (combined, labelled from campaign questions).

### View toggle

Removed. The table always shows the appropriate view for the active role — Remote Sheet for vendors, with the Master Sheet accessible only to non-vendor roles directly via the same table.

### Slot badge
Inline badge on the Company cell: `X/Y slots` in green (slots available), amber (1 left), or red (full). Recalculates live from `S1_SLOTS`.

### Stats row
| Stat | Source |
|---|---|
| Total Leads | `S1_LEADS` filtered for campaign + role |
| Accepted | `status === 'accepted'` |
| In Review | `status === 'in-review'` |
| Submitted | `status === 'submitted'` |
| Rejected | `status === 'rejected'` (non-vendor only) |
| Slots Left | Sum of remaining slots across visible companies |

### Adding leads — inline row (Airtable-style)

Clicking **+ Add Lead** (top-right of the table section) inserts an editable row directly at the bottom of the table body. Every column has an input or select field. CQ1/CQ2/CQ3 inputs use the campaign's question text as placeholder.

A **Save row / Cancel** bar appears fixed to the bottom-right of the viewport (`position: fixed; bottom: 24px; right: 32px`) — always visible regardless of how far the wide table has been scrolled horizontally.

On save:
1. Validates First Name, Last Name, Company, Email (required)
2. Checks for duplicate email in `S1_LEADS`
3. Pushes new lead object with all 32 fields; increments `S1_SLOTS` if company matched
4. Calls `render()` — no page reload

On cancel: edit row and the fixed bar are both removed.

---

## Campaigns Tab

### List view
- Table with columns: Campaign, Client, Vendors, **Vendor Progress** (Admin/CST only), Progress, Goal, In Review, Deadline, ABM
- Admin / CST: `+ New Campaign` button opens creation modal
- **Vendor Progress column**: nested table per row — one line per assigned vendor showing colour dot, name, inline bar, and % toward that vendor's individual goal. Hidden for Vendor role.
- Vendor: sees own campaigns only; Vendors and Vendor Progress columns hidden

### Detail view
- Triggered by clicking any campaign row (`openCampaignDetail`)
- Layout: stat cards → **Vendor Performance table** (Admin/CST only) → Companies table → Leads table → sidebar
- **Vendor Performance table** columns:
  - Admin: Vendor | Delivered/Goal | Progress | Accepted | In Review | Submitted | Accept Rate
  - CST: Vendor | Delivered/Goal | Progress (+ footnote that breakdown is Admin-only)
- Vendor: company and lead tables filtered to own data only; Vendor Performance section not shown

### Campaign creation modal
Multi-section form: Campaign Info, Target Audience, Lists, Qualification. Includes:
- Searchable client dropdown
- Multi-select tag inputs (countries, job titles, job areas)
- Sub-goals by country with total validation
- Company size range builder

---

## Styling

| Token | Value |
|---|---|
| Font | `-apple-system, BlinkMacSystemFont, 'Segoe UI'` |
| Background | `#f5f6fa` |
| Text | `#2d3436` |
| Dark header | `#1a1a2e` |
| Content max-width | `1600px` |
| Card radius | `12px` |
| Card shadow | `0 1px 3px rgba(0,0,0,0.06)` |

Badge classes — colors sourced from the actual spreadsheet palette:

| Class | Background | Text | Meaning |
|---|---|---|---|
| `badge-done` / `badge-live` | `#d9ead3` | `#274e13` | Accepted / Live |
| `badge-new` / `badge-pre` | `#c9daf8` | `#1155cc` | Submitted / Pre-launch |
| `badge-inprog` / `badge-paused` | `#fce5cd` | `#b45f06` | In Review / Pending / Paused |
| `badge-blocked` | `#f4cccc` | `#cc0000` | Rejected / Blocked |
| `badge-vendor` | `#d9d2e9` | `#8e7cc3` | Vendor chip |
| `badge-closed` | `#efefef` | `#636e72` | Closed / none |

Button classes: `btn-primary` (blue), `btn-success` (green), `btn-danger` (red), `btn-warning` (yellow), `btn-ghost` (grey). Add `btn-sm` for compact size.

---

## Files

| File | Purpose |
|---|---|
| `index.html` | Main dashboard — single self-contained file |
| `Scenario_1_Decision_Tree.html` | Original S1 decision tree visual (style reference) |
| `leads-dashboard.md` | This documentation file |

### Source reference files (Task1 root)
| File | Role |
|---|---|
| `master_sheet_example_copy.xlsx` | Internal DPO Master Sheet — 40 cols, QA + CC status layers |
| `provider_remote_gsheet_example_copy.xlsx` | Vendor Remote Sheet — 32 cols, prospect data + submission status |
