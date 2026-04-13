# DPO Platform — Research Notes

> These notes document everything learned from each project document. They serve as the knowledge base for creating the Project Charter, Project Plan, and Business Process Diagrams.

---

## 1. dashboards_description.xlsx

This file describes the **existing dashboards and sheets** that the DPO platform must eventually replace or integrate with. It's essentially the requirements source for "what we have now."

### 1.1 INTRODUCTION tab

Lists 6 key components of the current system:

| Dashboard/Sheet | Description | Current Tool | Suggestion for New Platform |
|---|---|---|---|
| **Campaign Tracking** | Tracks all current campaigns | Google Sheet (DPO Delivery Tracking, tab Campaign Tracking) | — |
| **Delivery Tracking** | Tracks volumes per vendor | Google Sheet (DPO Delivery Tracking, tab Delivery Tracking) | Vendors should get limited visibility — when logged in, they see only ACTIVE campaigns assigned to them |
| **Stats Collection** | Collects delivery stats per campaign/vendor monthly, used to generate monthly reports | Google Sheet (DPO Delivery Tracking, tab All Stats) | — |
| **DPO Rejections** | Collects rejections across all campaigns monthly, for rejection reason analysis | Separate Google Sheet | — |
| **Master Gsheet** | Template per campaign. CC collects leads here. Aggregates from Remote sheets. Multiple Remote sheets → one Master. Statuses go back to Remote sheets | Google Sheet template | — |
| **Remote Gsheet** | Template per vendor per campaign. Vendors upload leads here. Each vendor has their own per campaign | Google Sheet template | — |

**Key insight:** Master sheet collects data from Remote sheets (starting from column K). Multiple Remote sheets connect to one Master. DPO ID identifies which vendor delivered the leads. Once leads are processed, CC updates statuses per lead, and those statuses flow back to corresponding Remote sheets.

### 1.2 Campaign Tracking tab

26 columns describing the unified campaign overview. Key details:

**Manual input fields (that the platform must support):**
- Client Name, CID — from CC DPO/Canada campaign request
- DPO Goal — from campaign request, **must be editable**
- Master ID — piece of a link used to pull data per campaign from Master sheet
- Status — 5 values: **Live, Closed, Pre, Canceled, Paused** (must be editable)
- Start Date — day request received by DPO department
- Deadline — from campaign request, **must be editable**
- Comment — internal use, **must be editable**
- Campaign Type — from campaign request
- Assigned — 3 values: **No, Yes, Partially**
- Assigned date — when campaign assigned to team/vendor
- Responsible CC — from campaign request, **must be editable**
- Responsible LD (= DPO admin) — internal, **must be editable**

**Formula/automated fields:**
- Delivered, Accepted, Rejected, Pending — pulled from All Stats tab
- Left = Goal − Accepted
- % Time passed — networking days formula
- % Accepted = Accepted / DPO Goal
- % Rejected = Rejected / Delivered
- % Delivered = Delivered / DPO Goal
- Pacing — on track or behind, based on Goal, Accepted, Pending
- # vendors — count of unique vendors from Delivery Tracking
- Unallocated — from All Stats (total goal minus allocated amount)

### 1.3 Delivery Tracking tab

21+ columns tracking per-vendor per-campaign delivery. Key details:

**Manual fields:**
- CQ (Custom Questions) — **subject for discussion** on automation
- SL - Delivery Chain — subject line of email to vendor (contains vendor name, client name, CID)
- Vendor name — **hidden, maybe not needed? Subject for discussion**
- DPO ID — depends on vendor allocation
- CID — from campaign request
- Start date — when campaign chain sent to particular vendor
- Deadline — must be editable
- Phone script approval — date when lead delivery approved phone script from vendor
- Campaign Status — 6 values: **Live, Closed, Pre, Canceled, Paused, Expired** (must be editable)
- Comments — must be editable

**Formula fields:**
- Time Passed, % leads accepted, % leads rejected, % leads delivered
- Allocation (Goal), Total delivered, Total Accepted, LEFT, Pending, Pending QA
- **14 columns with dates** — number of leads delivered by vendor on specific days for 14 calendar-day period. Pulled from hidden DailyDelivery tab. **Subject for discussion.**

### 1.4 Master Gsheet tab

The core operational sheet per campaign. **Two sections: Master tab + Settings tab + Stats tab.**

#### Master tab columns (22+ columns):

**Mixed manual/formula (50/50) — important conflict issue:**
- CC status — pulled from Leads Tool, BUT if CC manually sets a different status, the script overwrites it next run. **Needs to be fixed.** Must be editable.
- CC Final status — same conflict issue as CC status
- CC commentary — same conflict issue

**Manual fields:**
- CC TY Email — not all CCs fill it out, **subject for discussion** if needed
- CC submission date — same issue, not all CCs fill it out
- Recording QA — QA puts disposition and comment here. Must be editable.
- Status — 4 values: **accepted, rejected, pending, returns** (dropdown). Must be editable.
- Rejection reason — 16 values: **Bounced, Title, Industry, Size, Revenue, GEO, Company Limit, NWC, Duplicate, Recording Failure, Unsubscribed, in Suppression, not in NAC, n/a CQ, Invalid ph number, Other** (dropdown). Must be editable.
- Rejection Comment — explanation. Must be editable.
- Status of the Reworked Leads — 4 values (same as Status dropdown). Must be editable.
- Rejection Reason for the Reworked Leads — explanation. Must be editable.
- Category — chosen from dropdown based on Stats tab column D. Must be editable.

**Formula fields (pulled from Remote sheet):**
- DPO ID ← Remote column D
- Report submission date ← Remote column E
- Reworked Leads ← Remote column F
- Call Notes ← Remote column I
- Prooflink ← Remote column J
- Recording link ← Remote column K
- Notification status — auto-added after "Send New Leads Notification" script
- Columns T+ — lead data from Remote. **Number of columns is dynamic** (depends on CQs). Columns AJ+ must be editable.

#### Settings tab:
- New leads subject line (format: DPO/EM_New_Leads_Were_Added_[client]_[CID])
- Reworked leads subject line (format: DPO/EM_Reworked_Leads_Were_Added_[client]_[CID])
- New Leads notification emails — list of recipients
- Reworked Leads notification emails — list of recipients
- Vendor management team emails
- CC emails — for reminder if leads without statuses for too long
- CC reminder subject line (format: DPO/EM leads not checked - [CID])

**Insight:** There are automated **email notification scripts**: "Send New Leads Notification", "Send Reworked Leads Notification", "Send reminders to CC". These are operational workflows the platform should replicate.

#### Stats tab:
- Month — upper "pink" part is always "Overall", white part added manually
- Status — 6 values: **Cancelled, Closed, EXCLUDE, Live, Paused, Pre**
- DPO ID, Category (GEO or other goal breakdown)
- Goal: # — 1st row auto-SUM of all "Total" goals; green fields filled by lead delivery member
- Delivered, Accepted, Rejected, Returned, Pending, Pending QA — all formula-based using Master columns (DPO ID, Category, Status)
- Left = Goal − Accepted
- Unallocated = total goal − allocated amount
- Left To Generate — **subject for discussion** if needed
- Type — 9 values: **Email Marketing, Appointment Setting, BANT, CALL Back, Soft BANT, MQL, SQL, Webinar, Survey** + need to add "LIVE event"

### 1.5 Remote Gsheet tab

**Two sections: Remote tab + Count table tab + Email tests tab.**

#### Remote tab columns:

**Formula fields (pulled FROM Master):**
- Status ← Master column G
- Rejection reason ← Master column H
- Prooflink ← Master column I (note: description says "Rejection Comment" — possible label mismatch)
- Status of the Reworked Leads ← Master column M
- Rejection Reason for the Reworked Leads ← Master column N

**Manual fields (vendor inputs):**
- DPO ID — ideally auto-added
- Report submission date — ideally auto-added
- Reworked Leads — vendor's note on why lead should be accepted. Must be editable.
- Call Notes — description of call. Must be editable.
- Prooflink — link added by vendor. Must be editable.
- Recording link — link to folder with recordings. Ideally one folder per delivery.
- Columns T+ — lead data (First Name, Last Name, Company, Email, Title, Phone, Guide/Asset, Country, Company Size, Address, City, State, Postal Code, Industry, Revenue, Timestamp, CQ1, CQ2, CQ3, Opt-in, Comments). Dynamic columns for CQs.

#### Count table tab:
- GEO (optional, needed if >1 GEO or category)
- Goal, Delivered, Rejected, Accepted, Left, Pending, Returns — formulas counting from Remote tab

#### Email tests tab (only for Email Marketing campaigns):
- Asset name, Test SL, Status of the test (approved/pending/fix!), Date, Comment

### 1.6 All Stats tab

A tab inside the DPO Delivery Tracking Google Sheet (our local copy: `delivery_tracking_copy.xlsx`, sheet "All_Stats"). Aggregated stats pulled via **script** from individual Master Gsheet Stats tabs across all campaigns. 16 columns: CID, Month, Status, DPO ID, Category, Goal, Delivered, Accepted, Rejected, Returned, Pending, Pending QA, Left, Unallocated, Left to generate, Campaign Type. The actual data contains **4,516 rows**.

### 1.7 DPO Rejections tab

Two tabs: Data + Pivot. Data tab has 5 columns: CID, DPO ID, Rejection reason, Report submission date, Email, Month. All pulled via script. **Need to check with Roman Tomenko** (noted in doc).

---

## 2. delivery_tracking_copy.xlsx

**The actual data file** with 14 sheets. This is the real DPO Delivery Tracking spreadsheet.

### Key sheets and observations:

- **All_Stats** (4516 rows) — real data with CIDs like DC58772, campaign types (Appointment Setting, Email Marketing, etc.), vendors (T1, V5, V20, V36, V38, etc.), categories (AG, US, UK, IE, etc.)
- **Campaign Stats** (86 rows) — summary by campaign type showing Live/Paused counts, goals, accepted, left, returns, pending
- **Monthly Stats** — vendor performance per month (delivered, rejected, rejection rate, accepted, pending, unique CIDs)
- **Delivery Tracking** — per-vendor per-campaign tracking
- **Campaign Tracking** — the unified campaign dashboard
- **Vendor Directory** — list of vendors
- **Vendors performance** — vendor performance analysis
- **Error log** — error tracking
- **DailyDelivery** — daily delivery data (source for the 14-day columns)
- **Backend_Days, Backend_Months** — backend data aggregations

**Key data insight:** Campaign types include: Appointment Setting, Email Marketing, BANT, CALL Back, Soft BANT, MQL, SQL, Webinar, Survey, LIVE EVENT, Custom Questions. There are many vendors (T1, V1-V38+). Categories are typically GEOs (AG, US, UK, IE, etc.).

---

## 3. provider_remote_gsheet_example_copy.xlsx

Real example of a vendor's Remote sheet for one campaign (Rubrik, CID DU59067, Vendor V20).

### Remote tab (7 rows of sample data):
- All leads from UK and Ireland
- Columns: Status, Rejection reason, Prooflink, DPO ID (V20), Report submission date, Reworked Leads, Status of Reworked, Rejection Reason for Reworked, Call Notes, Prospect Prooflink, Recording link, First Name, Last Name, Company, Email, Job Title, Telephone, Guide (asset name), Country, Company Size, Address, City, State, Postal Code, Industry, Revenue, Timestamp, CQ1, CQ2, CQ3, Opt-in, Comments
- **32 columns total** in this example (3 CQs)
- Guide = "Why Data Security Is the New Cybersecurity Priority" (the asset/content piece)
- CQ1 = "Yes" (backup >20TB), CQ2 = "0-6 months" / "9-12 months" (ransomware review timing), CQ3 = "Yes" (right person)

### Count table tab:
- Two GEOs: United Kingdom (goal 147) and Ireland (goal 23)
- Shows Delivered, Rejected, Accepted, Left, Pending, Returns per GEO

### Email tests tab:
- Empty template (Asset name, Test SL, Status of the test, Date, Comment)

---

## 4. master_sheet_example_copy.xlsx

Real example of a Master sheet for the same Rubrik campaign (CID DU59067).

### Master tab (3 rows of sample leads):
- 40 columns
- Shows the full lead lifecycle: CC status (accepted/rejected), CC Final status (Submitted/Not Submitted), CC submission date, CC commentary, Recording QA, Status, Rejection reason, DPO ID, etc.
- Example: Lead 1 = accepted → Submitted. Lead 2 = rejected → Not Submitted, commentary "LT: Unsubscribe", rejection reason "Unsubscribed"

### Settings tab:
- Master Sheet ID, Name, First dynamic column (T), Master "DPO ID" Column (J)
- (Note: only 4 settings visible, but the full template has email notification settings too)

### Stats tab:
- Overall stats for V20 per GEO (Ireland: goal 23, UK: goal 147)
- Total rows summing up all vendors
- Campaign Type = "Custom Questions"
- Shows Delivered, Accepted, Rejected, Returned, Pending, Pending QA, Left, Unallocated, Left To Generate

### REJ tab:
- Rejection tracking (likely mirrors DPO Rejections structure)

---

## 5. dpo_rejections.xlsx

**6,597 rejection records** across all campaigns. This is the rejection analysis dataset.

### Data tab columns:
- CID, DPO ID, Rejection reason, Report submission date, Month
- Copy of Data also includes Email column (1,002 rows)

### Observed rejection reasons:
- Other, Bounced, Title, Industry, Size, Revenue, GEO, Duplicate, NWC, Recording Failure, Unsubscribed, in Suppression, not in NAC, n/a CQ, Invalid ph number, Company Limit, Not reachable via phone

### Key insight for KPIs:
- 6,597 rejections is a significant volume
- Can calculate rejection rates by reason, vendor, campaign, time period
- This data can inform KPIs for the new platform (e.g., reduce rejection rate by X%)

---

## 6. leads_statuses_mapping (bootcamp).xlsx

**Critical technical document** — maps API endpoint responses to platform statuses. 4 sheets.

### 6.1 DRAFT Flow (push + statuses collecting)

Defines the **5-step integration flow** between DPO Platform, MainDB (MainDB), and LV/LT:

| # | Trigger | Interaction | Method | Description |
|---|---|---|---|---|
| 1 | Provider approved a new batch | DPO Platform → MainDB | `POST /api/lead-upload/dpo` | Send leads with Auto status = "WAITING". Distinguishes email marketing vs. other campaign types via `is_email_marketing` flag |
| — | (response) | MainDB → DPO Platform | — | MainDB returns uuid of the partition |
| 2 | MainDB sent uuid (from step 1) | DPO Platform → MainDB | `GET /api/lead-upload/dpo-status?uuid={uuid}` | Get upload statuses of leads |
| — | (response) | MainDB → DPO Platform | — | MainDB returns lead statuses for the partition |
| 3 | User pushed "Check status of leads" button | DPO Platform → LV/LT | `POST /api/contactInfo-by-mails` (listed as TBD LVM-2873 in source doc, but BPMN diagram shows this endpoint is already implemented) | Send emails & CIDs for leads with Auto status = WAITING, PENDING, REJECTED, or RETURNED |
| — | (response) | LV/LT → DPO Platform | — | LV/LT returns lead statuses |
| 4 | Leads not found on LV/LT (step 3), campaign type ≠ Email Marketing | DPO Platform → MainDB | `POST /api/getDpoStatusByCID` | Send emails & CIDs for leads with status "NotFound" from LV/LT and Auto status WAITING/PENDING/REJECTED/RETURNED |
| — | (response) | MainDB → DPO Platform | — | MainDB returns statuses |
| 5 | Leads not found on LV/LT (step 3), campaign type = Email Marketing | DPO Platform → MainDB | `POST /api/campaign/uploads/cs-canada-stats` | Same as step 4 but different endpoint for Email Marketing |
| — | (response) | MainDB → DPO Platform | — | MainDB returns statuses |

**Key insight:** The leads push (steps 1-2) has a designed GAS automation (runs every 1 min from Master sheets per BPMN diagram) but manual transfer by CC is still common. The statuses collecting (steps 3-5) is **already implemented** (per BPMN diagram) but is **manually triggered** by Admin or CC — not automatic. The platform MVP should make both fully automated.

### 6.2 MainDB push contacts

Maps MainDB responses when pushing leads (step 1-2):

| MainDB Message | Mapped Status | Reason |
|---|---|---|
| Valid | **PENDING** | — |
| Already Accepted | **INIT REJECT** | Already Accepted |
| Already DPO | **INIT REJECT** | Already DPO |
| Already In Collection | **INIT REJECT** | Already In Collection |
| Complainer | **REJECTED** | Complainer |
| Unsubscribed | **REJECTED** | Unsubscribed |
| Dangerous email | **REJECTED** | Dangerous email |
| Dangerous contact | **REJECTED** | Dangerous contact |
| Email not valid | **REJECTED** | Email not valid |
| Already Rejected | **REJECTED** | Already Rejected |
| Suppressed | **REJECTED** | Suppressed |
| Blacklisted company | **REJECTED** | Blacklisted company |
| Blacklisted domain | **REJECTED** | Blacklisted domain |
| No MX | **REJECTED** | No MX |

**Three status categories from MainDB push:** PENDING (valid), INIT REJECT (duplicate/already exists), REJECTED (compliance/quality issues).

### 6.3 MainDB statuses collecting

Maps MainDB statuses when collecting non-initial statuses (steps 4-5). 44 possible statuses, all mapping to either **pending** or **rejected**:

- **Pending:** only "Valid" → pending
- **Rejected:** 42 different reasons including HB/SB validation, Role-based, Bad words, Soft-Bounced, Unsubscribed variants, Spamtrap, Bad data, Out of business, Hard-Bounce, Banned Company, Syntax error, No MX, Blacklisted (domain/company/email), Suppressed variants, Already in Collection/Accepted/Rejected, Complainer, GDPR Complainer, Contact/Email Dangerous, Briteverify failure
- **NotFound** → rejected (no description)

### 6.4 LVLT statuses collecting

**Massive mapping** — 600+ rows covering statuses from two sources:

**Leads Tool (LT) statuses:** 47 unique statuses mapping to 4 categories:
- **accepted:** Accepted, Submitted, Accepted - Bonus, Accepted - Buying Committee
- **pending:** Processed, Unprocessed, Processing, Not submitted - Have Enough/Pacing, Not submitted - Other
- **returned:** Return - Bounce, Unsubscribe, Title, Industry, GEO, NAC, SUP, Bad Data/PV Comment, Wrong Asset, Company size/Revenue, Duplicate, Other, Client Company Limit (Phone), Criteria Change, No Explanation
- **rejected:** Reject by CC - Title/Company size/Industry/GEO/NAC/SUP/Bad Data/Reengagement/Custom Questions/Criteria Change/Buying Committee/Other, Not submitted - Pause/Cancelled/Wrong Asset/Company Limit, Bounce, Unsubscribe, System reject, Duplicate, Nurturing Unsubscribe
- **special:** NotFound → "-" (dash, meaning check MainDB next)

**LV Platform statuses:** Combination of `ov_status` × `pv_status`. The ov_status values include:
- Y variants (accepted): Y: accepted (old), Y: Data Checked, Y1: linkedin/company website, Y2: PL Summary, Y3: Facebook, Y4: Suspicious Linkedin, Y5: 3rd Party Prooflink, Y+C: Company accepted
- N/A variants (rejected): Emp. Size, Industry, Revenue, Title/PL Summary, Country/GEO, GEO, NAC/SUP, Prooflink, Duplicate, Wrong email/General domain, PV Tool, Other, Probably NWC, Back-up (verified/unverified), Backup (list preparation)
- N variants (rejected): N1: NWC, N2: Out of Business/Bad data (+ company variant)
- Q variants: Q1: Questionable Title (rejected), Q2: Questionable Company (rejected), Q3: Other (pending)
- Other (pending): Recheck, R+C: Company Recheck, Returned

The pv_status values include:
- lead confirmed (direct/transferred) → Y variants = **pending**, N/A variants = **rejected**
- lead confirmed (CD) → same pattern
- lead confirmed (operator/not transferred) → same pattern
- lead confirmed (operator/transfer failed) → same pattern
- company confirmed → same pattern
- company confirmed (no info) → same pattern
- company confirmed (not in CD) → same pattern
- company confirmed (operator/likely NWC) → same pattern

**Critical insight:** The LV platform's final mapped status depends on BOTH ov_status and pv_status. A lead can be "accepted" at verification (Y status) but still be "pending" — it needs both verification approval AND phone confirmation to be fully accepted. This is the two-stage verification process.

---

## 7. Untitled Diagram.drawio.png (Leads Flow Diagram)

BPMN diagram showing **two processes** (hard to read at resolution, but structure is visible):

**Top process:** Leads sending to MainDB — shows the flow from DPO Platform through MainDB with decision gateways, intermediate events, and end events. Multiple parallel paths and error handling.

**Bottom process:** Leads statuses collecting — shows the flow of checking statuses from LV/LT and MainDB, with conditional branching based on whether leads are found on LV/LT or not, and whether campaign type is Email Marketing.

These correspond exactly to the 5 steps documented in leads_statuses_mapping DRAFT Flow tab.

---

## 8. campaign_init_attachment.docx

Real campaign creation template for **Rubrik — Deep Funnel Campaign - UKI - DU59067**.

### Campaign details:
- Status: LIVE
- Goal: 147 total (Ireland: 23, UK: 124)
- Deadline: 01/25/2024
- Pacing: Even-pacing
- Company Limit: 2 (max 2 leads per company)

### Target Audience:
- GEO: UK & Ireland
- Industry: any
- Revenue: any
- Company Size: any
- Target Titles/Roles table:
  - Include | Manager+ | IT/Computers/Electronics: IT Infrastructure, Cyber Security, Computer/Network Security, Data Center, General Management, Information Systems Management
  - Include | C-Level | IT/Computers/Electronics | Keywords: CIO, CISO
  - Include | Director+ | IT/Computers/Electronics | Keywords: IT Engineering
  - Include | Exact titles | IT/Computers/Electronics | Keywords: Cloud Architect, Enterprise Architect, Back Up Admin, Systems Admin

### ABM list: Yes
### Suppression list: Yes

### Phone script requirements:
- Language: English
- Each lead must agree to answer 3 custom questions AND agree to download the attached asset
- **CQ1:** "Do you have more than 20 Tbs of backup data?" → Yes / No (disqualified)
- **CQ2:** "When do you next plan to review and optimise your ransomware recovery solution?" → 0-6 months / 6-12 months / 12-24 months (disqualified) / 24+ months (disqualified)
- **CQ3 (Profiling):** "Are you the right person to contact regarding Backup/Data Security or Cyber Resilience?" → Open field, accept if they are the right person
- OPT-IN: GDPR Compliant

---

## 9. Project Management Rules.docx

Defines INFUSE's project management methodology. Key elements for our deliverables:

### Project Charter must contain:
1. **Goals** — why the project is needed, aligned with company strategy (e.g., increase sales, decrease expenditures, improve quality, decrease risks)
2. **Results** — material outcomes (e.g., new channel, automated systems, increased feedback time)
3. **Limitations** — what we're NOT doing
4. **KPIs** — measurable indicators of goal achievement
5. **Plan** — WBS with dates and accountable members, milestones
6. **Risks** — prevention + reaction plans

### Project Phases:
1. **Initiation** — first charter version (Goals, Results, Limitations, Team register) + initial plan approved
2. **Planning** — deep analysis, final approval, budget. For business process change: As-Is → To-Be → preferred variant selected → plan development + implementation
3. **Execution & Monitoring** — plan vs. actual reporting. Task resolution: R&D task first → answer question → update plan
4. **Completion** — submit results for final approval

### Roles:
- Sponsor, Customer(s), Process owner, PM, Resource manager, Experts/assignees, Product owner, Design owner

### Other rules:
- Baseline = snapshot after approval; includes Baseline-Start and Baseline-End
- Monthly reporting to stakeholders
- Change requests for any approved parameter changes
- Contingency: **5-10% buffer** for time and budget
- Budget expressed in man-hours, can use ranges (e.g., 800-1000 hours)
- Timeline can use ranges (e.g., 3-4 months)

---

## 10. bootcamp_project documentation (for senior).docx

Additional tips specific to this bootcamp:

- Goals & Results must be **specific and quantifiable** — no vague terms like "significant decrease" or "major improvement"
- It's acceptable to **duplicate KPI values** in Goals section for visibility
- KPIs must be **countable** with numeric Before and After values **calculated based on data**
- Use the Bootcamp Charter Template, NOT the one from Project Management Rules

---

## Key Insights & Observations

### Data-driven KPI opportunities (from dpo_rejections.xlsx):
- 6,597 rejections tracked — can calculate current rejection rates
- Top rejection reasons can be quantified
- Vendor performance varies significantly (Monthly Stats shows different rejection rates)

### Automation opportunities (pain points to solve):
1. **Manual lead transfer** from Master sheets to MainDB — the biggest manual bottleneck
2. **CC status conflicts** — Leads Tool overwrites CC manual statuses on next script run
3. **Multiple disconnected Google Sheets** — one Master per campaign, one Remote per vendor per campaign
4. **Manual campaign setup** — CC creates campaign_init_attachment and sends manually
5. **Email notification scripts** — manual triggering of notification scripts
6. **Status sync delays** — statuses don't flow back in real-time
7. **Dynamic columns** — CQ columns vary per campaign, making standardization hard
8. **Manual stats aggregation** — All Stats pulled via script from individual Master sheets

### Existing integrations to preserve/enhance:
- LG → Registry → MainDB (client & CID data)
- DPO Platform → MainDB: `POST /api/lead-upload/dpo` (leads push)
- DPO Platform → MainDB: `GET /api/lead-upload/dpo-status?uuid={uuid}` (push statuses)
- DPO Platform → LV/LT: `POST (TBD - LVM-2873)` (status check)
- DPO Platform → MainDB: `POST /api/getDpoStatusByCID` (non-EM fallback)
- DPO Platform → MainDB: `POST /api/campaign/uploads/cs-canada-stats` (EM fallback)

### Status lifecycle of a lead:
1. **Vendor uploads** → appears in Remote sheet
2. **Auto-copied** to Master sheet
3. **CC transfers** to MainDB → gets **initial status** (PENDING / INIT REJECT / REJECTED from MainDB)
4. **Passes MainDB checks** → sent to **LV/LT** for verification
5. **LV checks** (LinkedIn, data verification) → ov_status assigned
6. **LT checks** (phone, domain, technical) → pv_status assigned
7. **Combined status** mapped back → accepted / rejected / pending / returned
8. **Statuses sync** back to Master → back to Remote (vendor sees result)
9. **If rejected/returned** → vendor can **rework** (comment in Reworked Leads column)
10. **QA/QC reviews** recordings → additional verdict

### Answered Questions:
- **Q2:** New clients/campaigns in INFUSE LG are created by the **Client Success Team (CST)**, not the DPO team. CC creates the campaign_init_attachment and sends it to DPO, but the LG entry is done by CST.

### Corrections/clarifications to instructions.md:
- The video says "permission to participate" — this likely refers to **GDPR consent / opt-in** (confirmed by campaign_init_attachment showing OPT-IN and GDPR Compliant fields)
- "INIT REJECT" is a distinct status from REJECTED — it means the lead already exists in the system (duplicate, already accepted, already in collection)
- The Stats tab Status has 6 values (Cancelled, Closed, EXCLUDE, Live, Paused, Pre) — different from Master tab Status which has 4 (accepted, rejected, pending, returns)
- Campaign Type has at least 10 types (Email Marketing, Appointment Setting, BANT, CALL Back, Soft BANT, MQL, SQL, Webinar, Survey, LIVE EVENT, Custom Questions)
- "Unallocated" (from open questions) = total goal minus the allocated amount across vendors
