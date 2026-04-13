# Campaigns Page — DPO Platform Dashboard

## Overview

The Campaigns page is the primary operational view in the DPO Platform dashboard. It lists all demand generation campaigns, organised by status, and applies role-based visibility so that vendors only see campaigns they are assigned to.

---

## Access & Role Visibility

| Role | What they see |
|---|---|
| **Admin** | All campaigns across all clients, vendors, and statuses |
| **Client Success (CST)** | All campaigns; can create new campaigns |
| **Vendor** | Only campaigns where their vendor ID appears in the `vendors` array |

Vendors **cannot** see other vendors, their companies, or their leads — even within a shared campaign. This directly supports **Scenario 1 (Lead per Company Limit)** and **Scenario 2 (Duplicate Prevention)**.

---

## Campaign Statuses

### Active (Live)
Campaigns currently running and accepting lead submissions.

| Campaign | Client | Vendors | Goal | Delivered | Region |
|---|---|---|---|---|---|
| EU SaaS Decision Makers | TechCorp Global | DataReach, LeadForge, ProspectIQ | 200 | 74 | UK, Ireland, Germany |
| UK FinTech C-Suite | FinAxis Partners | DataReach | 120 | 41 | United Kingdom |
| DACH Manufacturing IT Leaders | Meritech Solutions | ProspectIQ | 150 | 29 | Germany, Austria, Switzerland |
| Nordics Cloud Transformation | Apex Cloud Systems | LeadForge | 80 | 12 | Sweden, Denmark, Norway, Finland |

### Paused
Campaigns temporarily halted — typically awaiting a client approval, ABM list revision, or criteria update. This maps directly to **Scenario 3 (Client Requirement Change)** — paused campaigns display the pause reason inline on the card.

| Campaign | Client | Vendors | Reason |
|---|---|---|---|
| Healthcare IT Decision Makers | Celera Health | DataReach, LeadForge | ABM list revision in progress (Scenario 3 active) |
| EU Energy Sector CxOs | Praxis Energy | ProspectIQ | Pending ABM list v2 approval from client |

### Completed (Closed)
Campaigns that have met their lead goal and been closed. Rows are dimmed (65% opacity) and show a ✓ instead of an in-review count. Read-only for all roles.

| Campaign | Client | Vendors | Goal | Delivered | Deadline |
|---|---|---|---|---|---|
| UK Retail Digital Transformation | Landmark Retail Group | DataReach | 50 | 51 | 2026-02-28 |
| EMEA MarTech Buyers | Orion Digital | DataReach, LeadForge | 90 | 92 | 2026-01-31 |

---

## Vendor Scope Logic

When the role switcher is set to **Vendor** (e.g. DataReach, UK), the platform:

1. Filters `CAMPAIGNS` to only those where `campaign.vendors.includes(vendorId)`
2. Shows a scoped banner: _"Viewing as DataReach (UK) — you can only see campaigns assigned to your vendor"_
3. In campaign detail, filters `COMPANIES` and `LEADS` to the vendor's own rows only
4. Hides the vendor assignment card and vendor column from all tables
5. Updates the sidebar badge to reflect only in-flight campaigns visible to that vendor

**DataReach (v1)** is assigned to: cam1, cam2, cam7, cam8 → sees 3 active/paused campaigns  
**LeadForge (v2)** is assigned to: cam1, cam4, cam5, cam8 → sees 3 active/paused campaigns  
**ProspectIQ (v3)** is assigned to: cam1, cam3, cam6 → sees 3 active/paused campaigns  

---

## Dummy Data Fields

Each campaign object contains:

| Field | Type | Description |
|---|---|---|
| `id` | string | Unique campaign identifier (cam1–cam8) |
| `name` | string | Campaign display name |
| `client` | string | Client company name |
| `status` | `Live` \| `Paused` \| `Closed` | Current campaign state |
| `goal` | number | Total lead target |
| `delivered` | number | Leads accepted and delivered |
| `inReview` | number | Leads currently in QA queue |
| `companyLimit` | number | Max leads per company (Scenario 1) |
| `geo` | string | Target geography |
| `industry` | string | Target industry vertical |
| `size` | string | Target company size band |
| `revenue` | string | Target revenue band |
| `titles` | string | Target job titles |
| `abm` | `yes` \| `no` | Whether an ABM target list is attached (Scenario 5) |
| `suppression` | `yes` \| `no` | Whether a cross-channel suppression list is active (Scenario 4) |
| `pacing` | string | Lead delivery pacing strategy |
| `language` | string | Required language for qualification |
| `gdpr` | string | GDPR consent requirement |
| `deadline` | date | Campaign end date |
| `vendors` | string[] | Vendor IDs assigned to this campaign |
| `vendorGoals` | `{ [vendorId]: number }` | Per-vendor lead targets (must sum to ≤ `goal`) |
| `pauseReason` | string (optional) | Displayed on card when status is Paused |

### Per-vendor goal allocations

| Campaign | v1 (DataReach) | v2 (LeadForge) | v3 (ProspectIQ) |
|---|---|---|---|
| cam1 — EU SaaS Decision Makers | 70 | 60 | 70 |
| cam2 — UK FinTech C-Suite | 120 | — | — |
| cam3 — DACH Manufacturing IT Leaders | — | — | 150 |
| cam4 — Nordics Cloud Transformation | — | 80 | — |
| cam5 — Healthcare IT Decision Makers | 55 | 45 | — |
| cam6 — EU Energy Sector CxOs | — | — | 60 |
| cam7 — UK Retail Digital Transformation | 50 | — | — |
| cam8 — EMEA MarTech Buyers | 50 | 40 | — |

---

## UI Behaviour Notes

- **Status tabs**: A segmented pill control above the table — `Active`, `Paused`, `Completed` — filters the single table to one status at a time. Each tab shows its campaign count. Defaults to Active on load, role switch, and navigation.
- **Single table**: One `data-table` with columns: Campaign, Client, Vendors (hidden for Vendor role), Vendor Progress (Admin/CST only), Progress, Goal, In Review, Deadline, ABM.
- **Vendor Progress column** (Admin/CST only): A nested table per row with one line per assigned vendor — colour dot, name, inline progress bar, and percentage toward that vendor's individual goal. Colour-coded: green ≥ 90%, amber ≥ 60%, red < 60%.
- **Progress bar**: Slim 6px inline bar per row. Colour adapts to status — green (Live), orange (Paused), grey (Closed).
- **Paused rows**: Show the `pauseReason` as a small orange line beneath the campaign name.
- **Completed rows**: Dimmed to 65% opacity; In Review cell shows ✓ instead of a number.
- **No left-border colour stripes** on the table wrapper — clean box shadow only.
- **Create Campaign** button visible to Admin and CST roles only.
- **Sidebar badge** shows count of in-flight campaigns (Active + Paused) for the current role/vendor scope; updates dynamically on role switch.

---

## Campaign Detail — Vendor Performance Section

Visible to **Admin** and **Client Success** only. Appears between the top stat cards and the Companies table.

Rendered as a full table with one row per assigned vendor:

| Column | Admin | Client Success |
|---|---|---|
| Vendor (name + colour dot) | ✓ | ✓ |
| Delivered / Goal | ✓ | ✓ |
| Progress bar + % | ✓ | ✓ |
| Accepted | ✓ | — |
| In Review | ✓ | — |
| Submitted | ✓ | — |
| Accept Rate | ✓ | — |

Client Success sees a footnote: _"Detailed acceptance breakdown visible to DPO Admin only."_

Progress colour thresholds: green ≥ 90%, amber ≥ 60%, red < 60%.  
Accept rate colour thresholds: green ≥ 70%, amber ≥ 40%, red < 40%.

### `vendorStatsForCampaign(cam)` helper

Computes per-vendor stats at render time:
- **For cam1–cam3**: counts from `S1_LEADS` filtered by `campaign` and `vendor` fields
- **For all other campaigns**: proportional split of `cam.delivered` and `cam.inReview` across assigned vendors (fallback when no S1_LEADS data exists)

Returns an array of `{ vid, v, goal, delivered, accepted, inReview, submitted, rejected, acceptRate }` — one entry per vendor in `cam.vendors`.

---

## Scenario Connections

| DPO Scenario | Where it appears on Campaigns page |
|---|---|
| S1 — Lead per Company Limit | `companyLimit` shown in detail view; slot counter in company table |
| S2 — Duplicate Prevention | Vendor scope isolation — vendors can't see each other's campaigns, companies, or leads |
| S3 — Criteria Change (24h) | Paused tab — campaigns show inline `pauseReason` beneath the name |
| S4 — Cross-Channel Contacts | `suppression` flag shown in campaign detail sidebar |
| S5 — ABM List Versioning | ABM column (✓ / —) visible in all tabs; ABM version shown in lead table in detail view |
