# DPO Platform — Phase 2 Analysis

---

## 1. As-Is Process (Current State)

### 1.1 End-to-End Flow

The current DPO process involves **6 roles** (CC, DPO Admin, QC/QA, Vendors, LV team, System/Scripts) and **5 systems** (INFUSE LG, Registry, Google Sheets, MainDB, LV/LT). It runs across **10 stages**:

---

**Stage 1: Client & Campaign Creation (INFUSE LG)**
- A new client and campaign are created in **INFUSE LG** by the **Client Success Team (CST)**
- Client name and CID are assigned
- This data is synced to the central **Registry** microservice so other systems can access it

**Stage 2: Campaign Initialization (Manual)**
- The **Campaign Coordinator (CC)** creates a **campaign_init_attachment** document containing:
  - Campaign name, CID, status, goal (possibly broken by GEO or other categories), deadline, pacing, company limit
  - Target audience criteria: GEO, Industry, Revenue, Company Size, Target Titles/Roles (include/exclude)
  - ABM list, Suppression list
  - Promotion language, Custom Questions (0 to 10+), GDPR opt-in requirements
- CC sends this document to the **DPO team** (DPO Admin)
- **Pain point:** Entirely manual — CC fills a Word doc and emails it

**Stage 3: Campaign Setup in Tracking Sheets (Manual)**
- DPO Admin manually enters campaign data into the **Campaign Tracking** tab of the DPO Delivery Tracking Google Sheet (Client Name, CID, DPO Goal, Deadline, Campaign Type, Responsible CC, Responsible LD, etc.)
- DPO Admin creates a **new Master Google Sheet** from template for this campaign
- Configures the Master sheet's Settings tab (Sheet ID, name, dynamic column, notification emails, subject lines)
- Sets up the Stats tab (GEO/category breakdown, vendor allocations, campaign type)
- **Pain point:** Manual data entry into multiple sheets, risk of typos, no validation

**Stage 4: Vendor Assignment & Requirements (Manual)**
- DPO Admin allocates goal portions to vendors (e.g., V20 gets 100 of 147 leads)
- Creates a **Remote Google Sheet** from template for each vendor per campaign
- Enters vendor-specific data in the **Delivery Tracking** tab (CQ, DPO ID, CID, Start date, Allocation/Goal, Deadline)
- Sends campaign requirements to vendors (criteria, CQs, assets, phone script)
- Vendor must get **phone script approved** by DPO team before starting
- Updates the **Delivery Tracking** tab with phone script approval date
- **Pain point:** One Remote sheet per vendor per campaign = massive sheet sprawl. Manual setup for each. Requirements communicated separately (email/chat).

**Stage 5: Lead Generation & Upload by Vendors (Vendor-side + Auto-copy)**
- Vendors generate leads according to campaign criteria (GEO, industry, titles, CQs, etc.)
- Vendors call leads by phone, ask custom questions, record conversations
- Vendors upload leads into their **Remote sheet**: First Name, Last Name, Company, Email, Title, Phone, Guide/Asset, Country, Company Size, Address, Industry, Revenue, CQ answers, Opt-in, Recording link, Prooflink, Call Notes
- Leads are **automatically copied** from Remote sheets to the campaign's **Master sheet** (starting from column K onwards). DPO ID identifies the source vendor.
- **What works:** The auto-copy from Remote → Master is already automated
- **Pain point:** Vendor data quality varies. No real-time validation at upload time.

**Stage 6: Lead Review & Transfer to MainDB (Manual / Semi-automated)**
- CC reviews leads in the Master sheet
- A **GAS (Google Apps Script)** on the Master sheet runs every 1 minute: checks for unprocessed & approved leads, sends them via `POST /api/lead-upload/dpo` to MainDB, receives a uuid, then polls `GET /api/lead-upload/dpo-status?uuid={uuid}` for initial statuses (per BPMN diagram). On error or 10+ failed attempts, sets error status and stops.
- Despite this designed automation, CC **also manually copies/transfers** leads to **MainDB** — manual transfer remains the biggest bottleneck
- **Pain point:** Manual transfer still common — slow, error-prone, creates delays.

**Stage 7: Initial Checks in MainDB (Automated)**
- MainDB runs automated checks on each lead and returns an initial status:
  - **PENDING** — lead is valid, proceeds to next stage
  - **INIT REJECT** — lead already exists (Already Accepted, Already DPO, Already In Collection)
  - **REJECTED** — compliance/quality issue (Complainer, Unsubscribed, Dangerous email/contact, Email not valid, Already Rejected, Suppressed, Blacklisted company/domain, No MX)
- These initial statuses are sent back to the Master sheet via **automation** (existing scripts)
- CC status and CC Final status columns get updated
- **Conflict issue:** If CC manually set a status that differs from the Leads Tool status, the script **overwrites** it on the next run. This is a known problem that needs fixing.
- **What works:** The status sync from MainDB → Master is automated
- **Pain point:** The CC status overwrite conflict. Also, CC submission date and CC commentary not consistently filled.

**Stage 8: Lead Verification — LV/LT (Semi-automated)**
- Leads that passed MainDB checks (PENDING status) are sent to **Lead Verification (LV)** module
- **Manual verification (LV platform):** People check leads through LinkedIn, company websites, and other platforms. They assign an `ov_status` (verification outcome):
  - Y variants (verified): Data Checked, LinkedIn/company website, PL Summary, Facebook, 3rd Party Prooflink, etc.
  - N/A variants (rejected): Emp. Size, Industry, Revenue, Title, GEO, NAC/SUP, Duplicate, Wrong email, etc.
  - Q variants (questionable): Questionable Title, Questionable Company
  - R variants (recheck): Recheck, Company Recheck, Returned
- **Technical checks (Leads Tool / LT):** Automated checks for phone validation, domain compliance, bounce detection, etc. Returns statuses like: Accepted, Submitted, Processed, Unprocessed, various Reject/Return reasons
- The `pv_status` (phone verification) is also captured: lead confirmed (direct/transferred), lead confirmed (CD), lead confirmed (operator/not transferred), company confirmed, etc.
- Combined `ov_status` + `pv_status` determines the **final mapped status**: accepted, rejected, pending, or returned
- **What works:** LV/LT endpoints exist and status collection automation is already in use
- **Pain point:** 600+ status combinations to map. Two-stage process (LV manual + LT technical) adds complexity.

**Stage 9: Status Updates & QA Review (Semi-automated)**
- **Admin or CC manually triggers** the status collection process (per BPMN diagram — this is already implemented)
- Script calls LVLT via `POST /api/contactInfo-by-mails?api_key={api-key}&cid={cid}&emails[0]={email-0}`
- If LVLT returns non-empty statuses → statuses are updated. If not found → falls back to MainDB: `POST /api/getDpoStatusByCID` (non-Email Marketing) or `POST /api/campaign/uploads/cs-canada-stats` (Email Marketing)
- If MainDB also errors → user notified via email, statuses not updated
- Final statuses (accepted/rejected/pending/returned) are synced back to Master sheet
- Statuses propagate from Master → Remote sheets (vendors see updates)
- **QA/QC reviews** selected call recordings (Recording link column) and adds disposition + comment in the Recording QA column
- **Master sheet Status** column is updated (4 values: accepted, rejected, pending, returns)
- **Rejection reason** is assigned from 16 possible reasons (Bounced, Title, Industry, Size, Revenue, GEO, Company Limit, NWC, Duplicate, Recording Failure, Unsubscribed, in Suppression, not in NAC, n/a CQ, Invalid ph number, Other)
- **Pain point:** QA review is fully manual. Recording QA column must be manually filled.

**Stage 10: Rework Flow & Final Delivery (Manual)**
- If a lead is rejected or returned, the vendor sees the updated status in their Remote sheet
- Vendor can **dispute** by adding comments in the **Reworked Leads** column explaining why the lead should be accepted
- DPO Admin reviews rework requests and may update the **Status of the Reworked Leads** column (same 4 statuses)
- Accepted leads are delivered to the client
- Stats are aggregated: Master Stats → (script) → All Stats → (formula) → Campaign Tracking / Delivery Tracking
- DPO Rejections sheet collects rejection data monthly for analysis
- **Notification scripts** run to alert team members about new leads, reworked leads, and overdue status checks
- **Pain point:** Rework flow is slow — vendor comments in sheet, admin manually reviews. No structured workflow.

---

### 1.2 Data Flow Summary

```
INFUSE LG → Registry → (manual) → Campaign Tracking sheet
                                 → Master sheet (new per campaign)
                                 → Remote sheets (new per vendor per campaign)
                                          ↓ (vendor uploads leads)
                                   Remote sheet
                                          ↓ (auto-copy)
                                   Master sheet
                                          ↓ (CC manual transfer) ← BIGGEST BOTTLENECK
                                      MainDB
                                          ↓ (automated checks)
                                   Initial statuses
                                          ↓ (automation)
                                   Master sheet ← status sync
                                          ↓ (leads that pass)
                                       LV / LT
                                          ↓ (verification + technical checks)
                                   Final statuses
                                          ↓ (automation)
                                   Master sheet ← status sync
                                          ↓ (formula sync)
                                   Remote sheet (vendor sees result)
                                          ↓
                                   Campaign Tracking / Delivery Tracking / All Stats
                                          ↓
                                   Client delivery
```

---

## 2. Pain Points & Manual Steps to Eliminate

| # | Pain Point | Current State | Impact | Priority |
|---|---|---|---|---|
| 1 | **Manual lead transfer to MainDB** | CC copies leads from Master sheet to MainDB by hand | Slowest step, delays entire pipeline, error-prone | Critical (MVP) |
| 2 | **Manual campaign initialization** | CC creates Word doc, emails it, then DPO admin manually enters data into multiple sheets | Duplicate data entry, risk of inconsistency | High (MVP) |
| 3 | **Sheet sprawl** | New Master sheet per campaign + new Remote sheet per vendor per campaign | Unmanageable at scale, hard to track, scripts break | High (MVP) |
| 4 | **CC status overwrite conflict** | Script overwrites CC manual status with Leads Tool status on next run | CC loses manual corrections, data integrity issue | High (MVP) |
| 5 | **No vendor portal** | Vendors use Google Sheets, can accidentally edit wrong cells, no validation | Data quality issues, no access control granularity | High (MVP) |
| 6 | **Manual vendor requirements communication** | Requirements sent via email/chat, phone script approval tracked in sheet | Requirements can be lost/misunderstood, no audit trail | Medium (MVP) |
| 7 | **Manual QA review tracking** | QA fills Recording QA column by hand after listening to recordings | No structured workflow, easy to miss leads | Medium (Post-MVP) |
| 8 | **Rework flow is unstructured** | Vendor comments in a column, admin manually checks and updates | Slow turnaround, disputes hard to follow | Medium (Post-MVP) |
| 9 | **Manual stats aggregation** | Script pulls from each Master sheet's Stats tab into All Stats | Script must be triggered, fails if sheet structure changes | Medium (MVP) |
| 10 | **No real-time dashboards** | All dashboards are formula-based Google Sheets | No live reporting, delayed visibility into pipeline health | Low (Post-MVP) |
| 11 | **Phone call process is manual** | Vendors call leads, manually document results | Time-consuming, inconsistent documentation | Low (Post-MVP — future automation) |
| 12 | **Notification scripts require manual triggering** | Email notifications for new leads, reworked leads, CC reminders | Delays in communication, can be forgotten | Medium (MVP) |

---

## 3. To-Be MVP Process

The MVP replaces the Google Sheets workflow with a centralized platform while keeping existing MainDB and LV/LT integrations intact.

### 3.1 MVP Stages

**Stage 1: Automated Campaign Initiation**
- DPO Platform pulls client names and CIDs from **Registry** (which gets them from LG)
- DPO Admin creates a campaign in the platform by selecting client/CID from Registry and entering criteria (goal, GEO, industry, titles, CQs, deadline, company limit, ABM/suppression lists, etc.)
- No more Word doc + email. All data entered once in the platform.
- Campaign Tracking dashboard is auto-populated.

**Stage 2: Vendor Assignment & Requirements via Platform**
- DPO Admin assigns vendors to campaigns within the platform, allocating goal portions
- Campaign requirements (criteria, CQs, assets, phone script) are shared with vendors through the platform
- Vendors see only their assigned active campaigns (limited visibility, as suggested in dashboards_description)
- Phone script approval workflow built into the platform

**Stage 3: Vendor Lead Upload via Platform**
- Vendors upload leads directly to the DPO Platform (replacing Remote sheets)
- Platform validates lead data at upload time (required fields, format checks, criteria matching)
- DPO ID and submission date are auto-assigned
- Recording links, prooflinks, call notes, CQ answers captured in structured form

**Stage 4: Automatic Lead Transfer to MainDB**
- Platform automatically sends approved lead batches to MainDB via existing `POST /api/lead-upload/dpo` endpoint
- Leads sent with Auto status = "WAITING"
- Platform receives partition uuid from MainDB
- Platform polls `GET /api/lead-upload/dpo-status?uuid={uuid}` to get initial statuses
- Statuses (PENDING / INIT REJECT / REJECTED) mapped and displayed in platform
- **Eliminates the biggest manual bottleneck**

**Stage 5: Automated Status Collection**
- Platform collects statuses from LV/LT via existing `POST /api/contactInfo-by-mails` endpoint (already implemented in current GAS process)
- For leads not found on LV/LT: platform falls back to MainDB (`POST /api/getDpoStatusByCID` or `/api/campaign/uploads/cs-canada-stats` for Email Marketing)
- All 600+ status combinations mapped to 4 final statuses (accepted/rejected/pending/returned)
- Statuses displayed in platform with full audit trail
- Vendors see their lead statuses in their portal view
- **Resolves the CC status overwrite conflict** — platform is the single source of truth with manual override capability and audit log

**Stage 6: Campaign Tracking & Reporting**
- Campaign Tracking, Delivery Tracking, and Stats dashboards built into the platform
- Real-time calculations (no more script-based aggregation)
- Pacing, % metrics, unallocated amounts calculated automatically
- Automated notifications (new leads, reworked leads, CC reminders) triggered by platform events

### 3.2 MVP Scope Boundaries

**In scope:**
- Campaign creation with Registry integration (client/CID lookup)
- Campaign criteria configuration (all fields from campaign_init_attachment)
- Vendor assignment and goal allocation
- Vendor portal for lead upload with validation
- Automatic lead push to MainDB (existing endpoint)
- Automatic status collection from MainDB and LV/LT (existing endpoints)
- Status mapping (all MainDB + LVLT mappings from leads_statuses_mapping)
- Campaign Tracking dashboard (replaces Google Sheet tab)
- Delivery Tracking dashboard (replaces Google Sheet tab)
- Basic notification system (new leads, status changes)
- Role-based access: DPO Admin, CC, QC, Vendor (with limited visibility)
- Lead status management with manual override + audit trail
- Basic rework flow (vendor can flag leads, admin can review)

**Out of scope (MVP — deferred to Post-MVP):**
- Advanced analytics and rejection analysis dashboards
- Vendor performance scoring/analytics
- Executive dashboards and pipeline forecasting
- Monthly reporting automation and ad-hoc report builder
- Structured rework/dispute workflow
- Historical data migration from existing Google Sheets

**Out of scope (both MVP and Post-MVP):**
- Phone call automation (calls remain manual by vendors)
- QA recording review workflow (QA reviews externally)
- ABM list and Suppression list management within the platform (uploaded as files for now)
- Integration with email marketing tools
- Mobile app

---

## 4. To-Be Post-MVP Process

### 4.1 Post-MVP Enhancements

**Phase A: Analytics & Dashboards (Primary Focus)**
- **Advanced Rejection Analytics:** Rejection breakdown dashboard by reason, vendor, GEO, and time period. Trend analysis with drill-down filters. Root-cause identification to enable data-driven decisions on vendor management and campaign targeting.
- **Vendor Performance Scoring:** Composite scoring system based on acceptance rate, quality metrics, and timeliness. Vendor leaderboard and ranking. Performance alerts for underperforming vendors. Data-driven vendor allocation recommendations.
- **Executive Dashboard:** Pipeline health indicators, campaign forecasting, pacing trends, and goal attainment tracking across all campaigns. Cross-campaign comparison views for management oversight.

**Phase B: Reporting & Automation**
- **Automated Monthly Reporting:** Scheduled report generation per campaign/vendor. Email distribution to stakeholders. Customizable report templates.
- **Ad-hoc Report Builder:** User-configurable reports with filters, date ranges, and groupings. CSV/Excel export capability. Saved report templates for recurring analysis.
- **Historical Data Migration:** Import past campaign data from Google Sheets to establish analytics baseline and enable long-term trend analysis.

**Phase C: Workflow Improvements**
- **Structured Rework Flow:** Formal dispute workflow. Vendor submits rework request → notification to admin → admin reviews → resolution logged with full history.

### 4.2 Post-MVP To-Be Process

The same 6 stages as MVP, enhanced with:
- Stage 5 (Status Collection): Fully automated, no manual trigger needed — runs on schedule
- Stage 6 (Reporting): Advanced rejection analytics, vendor scoring dashboards, executive pipeline views, automated monthly reports, ad-hoc report builder
- New workflow: **Structured rework** with formal dispute resolution
- **Historical data** migrated from Google Sheets for trend baseline

---

## 5. Roles in the Process

| Role | Current Responsibilities | MVP Platform Role |
|---|---|---|
| **Campaign Coordinator (CC)** | Creates campaign_init_attachment, manually transfers leads to MainDB, updates statuses, manages campaign lifecycle | Creates campaigns in platform, monitors pipeline, resolves status conflicts, manages campaign settings |
| **DPO Admin** | Controls and administrates campaigns, assigns vendors, manages Delivery Tracking, reviews rework requests | Manages platform settings, assigns vendors, approves rework, oversees all campaigns |
| **QC / QA** | Reviews call recordings, provides verdict on lead quality | Reviews recordings externally, enters verdict in platform. QA recording workflow is out of scope. |
| **Vendor** | Generates leads, calls prospects, uploads to Remote sheet, disputes rejections | Uploads leads via vendor portal, views own campaign statuses, submits rework requests |
| **LV Team** | Manually verifies leads via LinkedIn and other platforms | No change in MVP — platform collects their statuses via API |
| **System / Scripts** | Auto-copies Remote→Master, syncs statuses, sends notifications, aggregates stats | Replaced by platform's built-in automation |

---

## 6. Scope Boundaries Summary

### In Scope (Overall Project)
- Replace Google Sheets-based workflow with a centralized web platform
- Automate campaign initiation with Registry integration
- Provide vendor portal for lead upload
- Automate lead transfer to MainDB
- Automate status collection from MainDB and LV/LT
- Provide real-time dashboards (Campaign Tracking, Delivery Tracking, Stats)
- Role-based access control
- Notification system
- Rework/dispute workflow
- QA review workflow (Post-MVP)
- Phone call integration (Post-MVP)

### Out of Scope (Entire Project)
- Changes to INFUSE LG or Registry (they remain as-is, platform only reads from them)
- Changes to MainDB logic (platform uses existing API endpoints)
- Changes to LV/LT verification process (platform only collects statuses)
- Client-facing portal (leads are delivered to clients through existing channels)
- Vendor recruitment or vendor contract management
- Financial/billing management for campaigns
- Changes to the phone calling process itself (Post-MVP automates documentation, not the calls themselves)
