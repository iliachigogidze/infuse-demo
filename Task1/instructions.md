# INFUSE DPO Platform — Instructions

## 1. Company Overview

INFUSE is a demand generation company that provides leads to its clients. Clients define criteria for the leads they want, and INFUSE generates and verifies leads that match those criteria.

---

## 2. Lead Criteria

Clients provide the following criteria for their desired leads (these can be **included** or **excluded**):

- **Geo Location** — Country, Address, Zip Code
- **Industry**
- **Revenue** (in millions)
- **Company Size**
- **Target Titles/Roles** — Job Level, Job Area, Keywords

> Refer to the `campaign_init_attachment` document for full details on criteria fields.

---

## 3. Key Terminology

| Term | Definition |
|------|-----------|
| **Campaign** | INFUSE's term for a "contract" with a client |
| **CID** | Campaign Identification Number |
| **DPO** | Direct Phone Outreach — leads are contacted by phone, asked questions, and results are documented |
| **CC** | Campaign Coordinator — creates and manages campaigns |
| **DPO Admin** | Controls and administrates the campaign |
| **QC (QA)** | Quality Coordinator — controls the quality of the leads |
| **Vendor** | External lead provider that partners with INFUSE |
| **CQ** | Custom Question — qualifying questions asked to leads |

---

## 4. Systems & Modules

| System | Description |
|--------|-------------|
| **INFUSE LG (LG)** | Where new clients and campaigns are initialized |
| **Registry** | Central microservice that stores initialized clients and CIDs so other services can retrieve them |
| **Campaign Management Module (MainDB)** | Where lead generation happens — initial checks and preparation of contacts against client criteria (target audience, consent, etc.) |
| **Lead Verification Module (LV)** | Where leads are verified manually (e.g., via LinkedIn and other platforms) before delivery to the client |
| **Leads Tool (LT)** | Part of LV — handles technical checks such as phone validation, domain compliance, etc. |

---

## 5. Campaign Initialization

When a campaign is created, the Campaign Coordinator (CC) creates a **Campaign Creation Template** (`campaign_init_attachment`) and sends it to the DPO team.

### Fields in `campaign_init_attachment`:

1. **Campaign Name and CID**
2. **Status**
3. **Goal** — Can be broken down by GEO or other dimensions. Goals are set higher than the final target to account for leads that may fail verification. *Example: Goal = 70 leads. Sub-goals: 23 from Ireland + 147 from UK = 170 total to ensure margin.*
4. **Deadline**
5. **Pacing** — Not critical for now
6. **Company Limit** — Maximum number of leads from the same company
7. **GEO**
8. **Industry**
9. **Revenue**
10. **Company Size**
11. **Target Titles/Roles** — Table with Job Level, Job Title, and Keywords (include/exclude)
12. **ABM List**
13. **Suppression List**
14. **Promotion Language** — e.g., English
15. **Custom Questions (CQs)** — Qualifying questions for leads. A campaign can have 0, 1, or 10+ CQs. *Example: "Do you have more than 20 TB of backup data?" Option A: Yes; Option B: No — disqualified.*

---

## 6. Dashboards & Sheets

Refer to the `dashboards_description` document for the full list of dashboards used in the process.

### 6.1 Campaign Tracking Sheet

- **Source:** `delivery_tracking_copy` document, "Campaign Tracking" tab
- **Purpose:** Unified dashboard with all campaigns
- **Notes:**
  - In the comment section, T1, V2, etc. are **Vendor IDs**
  - "Responsible LD" = DPO Admin

### 6.2 Remote Sheet (Vendor Sheet)

- **Source:** `provider_remote_gsheet_example_copy` document, "Remote sheet" tab
- **Purpose:** Each campaign may have multiple vendors. Each vendor has their own copy of this sheet per campaign where they add leads.
- **Key columns:**
  - **DPO ID** = Vendor ID
  - **Guide** = Asset used to attract the lead (e.g., social media post)
  - **CQ1, CQ2** = Custom Questions
- **Access:** One sheet per vendor per campaign. Only the vendor and INFUSE team can see it.

### 6.3 Master Sheet

- **Source:** `master_sheet_example_copy` document, "Master sheet" tab
- **Purpose:** Aggregates leads from multiple vendors per campaign
- **Access:** Vendors do **not** have access to master sheets — only to their own remote sheets
- **Flow:**
  1. Leads are **automatically copied** from remote sheets to the master sheet
  2. CC **manually moves** leads from the master sheet to MainDB
  3. Based on checks, the following columns are updated: **CC Status**, **CC Final Status**, **CC Submission Date**, **CC Commentary**
  4. QA/QC reviews recordings (via **Recording Link**) and gives a verdict on verification
- **Status Sync:**
  - CC Status may not always be in sync with the Status column (admin can override system decisions)
  - When Status is updated, vendors see the update in their remote sheets
  - If a vendor disagrees, they can comment in the **Reworked Leads** column. The admin may then update the status of reworked leads accordingly.

---

## 7. Current Process (End-to-End)

1. **Campaign Initiation** — DPO team initiates the campaign and provides requirements to vendors
2. **Lead Generation by Vendors** — Vendors generate leads according to the criteria
3. **Lead Upload** — Vendors upload leads to their remote sheets, which are automatically copied to the master sheet
4. **Transfer to MainDB** — Campaign coordinators manually move leads from the master sheet to MainDB
5. **Lead Checks in MainDB** — Initial status is created; automations send statuses back to the master sheets
6. **Lead Verification (LV)** — Leads that pass MainDB checks are sent to Lead Verification and technical checks before client delivery
7. **Status Updates** — At each verification stage, statuses (good/bad) are formed and pushed back to the master sheets via existing automations

---

## 8. DPO Platform — Automation Goals

The new DPO Platform aims to automate the entire process through the following stages:

1. **Campaign Initiation** — Automate campaign creation using Registry (which pulls client names and CIDs from LG)
2. **Vendor Communication** — Streamline how the DPO team delivers requirements to vendors
3. **Lead Upload** — Provide vendors with the ability to upload leads directly to the new platform
4. **MainDB Integration** — Automatic transfer of leads from the platform to MainDB
5. **Status Collection** — Automated collection of statuses from the lead checking and verification pipeline

---

## 9. Open Questions

1. What does "permission to participate in the campaign" mean? *(Ref: Video at 05:35)*
2. Who creates a new client and campaign in INFUSE LG? Is it the DPO team? *(Video says CC creates and sends `campaign_init_attachment` to DPO, but who creates it in LG?)*
3. What is the ABM list? Is it a specific, curated list of targeted companies (as opposed to a broad audience)?
4. What is the difference between "APPOINTMENT SETTING" and "DPO PLATFORM - statuses upd 3/4"? *(Ref: `delivery_tracking_copy`, Rows 3 and 6)*
5. What does "Unallocated" mean? *(Ref: `delivery_tracking_copy`, Column W)*
6. What can be the goal of a campaign other than GEO?
7. Vendors only have access to remote sheets, not master sheets — is that correct?
