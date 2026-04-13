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

| ID | Name | Status | ABM | Vendors |
|---|---|---|---|---|
| `cam1` | EU SaaS Decision Makers | Live | yes | v1, v2, v3 |
| `cam2` | UK FinTech C-Suite | Live | yes | v1 |
| `cam3` | DACH Manufacturing IT Leaders | Live | no | v3 |
| `cam4` | Nordics Cloud Transformation | Live | no | v2 |
| `cam5` | Healthcare IT Decision Makers | Paused | yes | v1, v2 |
| `cam6` | EU Energy Sector CxOs | Paused | yes | v3 |
| `cam7` | UK Retail Digital Transformation | Closed | no | v1 |
| `cam8` | EMEA MarTech Buyers | Closed | yes | v1, v2 |

Each campaign also carries a `vendorGoals` map (`{ vendorId: number }`) specifying each vendor's individual lead target within the campaign. Used by `vendorStatsForCampaign()` to compute per-vendor progress.

#### `abmList` — per-campaign ABM company assignments

Campaigns with `abm: 'yes'` carry an `abmList` object keyed by vendor name. Each key holds the exact list of companies the DPO Admin has assigned to that vendor for the campaign. This is the authoritative source for the Company dropdown in the Add Lead modal.

```js
abmList: {
  'DataReach':  ['Novatera Group', 'Harmon Inc', 'Veltrix Ltd'],
  'LeadForge':  ['Celtic Analytics', 'Brindlewood Co'],
  'ProspectIQ': ['Berlinium GmbH', 'Rhine Digital', 'Hanseatic Tech'],
}
```

- The lists must mirror what the DPO Admin entered in `vendorScopes[].scope` — they are the ground truth for scope enforcement.
- `abmList` is a separate field from `vendorScopes` so it is not overwritten when the campaign edit modal saves a new `vendorScopes` array.
- Non-ABM campaigns (`abm: 'no'`) do not have this field; the Company field in the Add Lead modal renders as a free-text input for those campaigns.

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

### Adding leads — Add Lead modal

Clicking **+ Add Lead** (top-right of the table section) or the **Add Row** trigger at the bottom of the table opens the Add Lead modal (`#modal-add-lead`). Opened by `openAddLeadModal()`.

#### Company field — ABM vs free-text

The Company field switches between two modes depending on the active campaign:

| Campaign `abm` | Role | Company field |
|---|---|---|
| `yes` | Vendor | `<select>` — only that vendor's assigned companies from `cam.abmList[vendorName]` |
| `yes` | Admin / CST | `<select>` — all companies across all vendors (`Object.values(cam.abmList).flat()`) |
| `no` | Any | `<input type="text">` — free entry |

Both elements (`#al-company` input and `#al-company-select` select) exist in the DOM at all times; `openAddLeadModal()` shows one and hides the other based on `cam.abm`. `submitAddLead()` reads from whichever is currently visible.

Logic in `openAddLeadModal()`:
1. Find active campaign via `STATE.currentCampaign`
2. If `cam.abm === 'yes'` and `cam.abmList` exists:
   - Vendor role → resolve vendor name from `STATE.currentVendor` → `cam.abmList[vendorName]`
   - Admin / CST → `Object.values(cam.abmList).flat()`
3. If campaign has no `abmList` → fallback to `COMPANIES` filtered by vendor

#### Other fields

CQ1/CQ2/CQ3 labels and visibility are set from the campaign's `qualQuestions` array (or parsed from `script`). Hidden if the campaign has no questions for that slot.

On submit (`submitAddLead()`):
1. Validates First Name, Last Name, Email, Company (required)
2. Checks for duplicate email in `S1_LEADS`
3. Pushes new lead object; increments `S1_SLOTS` if company matched in `COMPANIES`
4. Calls `render()` — no page reload

### Filter bar (Admin / CST only)

Two dropdown filters above the leads table, hidden from Vendor role.

| Filter | Source | Behaviour |
|---|---|---|
| **DPO ID** | Distinct non-empty `dpoId` values from leads in the current campaign + vendor scope | Filters table to leads matching the selected DPO ID |
| **Client ID** | All `clientId` values across non-Closed campaigns | Filters table to leads whose campaign matches the selected client |

Both default to **All** (no filter). State is held in `S1_FILTER_DPO` and `S1_FILTER_CID`. Both reset to empty when switching campaigns (`switchCampaign()`). A results count appears when either filter is active.

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
