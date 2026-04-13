# DPO Platform — Presentation Transcript

**Duration:** ~60 minutes
**Presenter:** Ilia Chigogidze
**Date:** April 7, 2026

---

## SLIDE: Agenda (~1 min)

> Good morning/afternoon everyone. Thank you for your time. Today I'll present my analysis and project plan for the DPO Platform — a centralized platform to replace the current Google Sheets-based workflow used by the Direct Phone Outreach department.
>
> Here's what we'll cover:
>
> First, I'll walk you through the current As-Is process — all 10 stages, the roles involved, and the systems in use. Then I'll highlight the key pain points I identified. After that, I'll present the To-Be MVP solution — what the platform will do and how it changes the workflow. I'll briefly cover the Post-MVP vision. Then we'll go through the Project Charter — goals, KPIs, limitations, and risks. Finally, I'll show the Project Plan with timeline and milestones. We'll have time for questions at the end, but feel free to interrupt if something needs clarification.

---

## SLIDE: Project Context (~3 min)

> Let me start with context. The DPO department manages lead generation campaigns for clients. Vendors make phone calls to prospects, collect leads, and those leads go through verification before being delivered to the client.
>
> Today, this entire process runs on Google Sheets. Each campaign gets its own Master sheet. Each vendor on each campaign gets their own Remote sheet. There's a central DPO Delivery Tracking sheet with Campaign Tracking, Delivery Tracking, and All Stats tabs. And campaign initialization is done through Word documents sent via email.
>
> The result is dozens of disconnected spreadsheets, a lot of manual data entry, and a workflow that's hard to scale. The DPO Platform project aims to replace this with a single centralized web application.
>
> The systems currently involved are: INFUSE LG, where the Client Success Team creates clients and campaigns; the Registry microservice, which stores client and CID data; Google Sheets, which is the main working environment; MainDB, which runs initial lead checks; and LV/LT — Lead Verification and Leads Tool — which handle manual and technical verification.
>
> There are 6 key roles: Campaign Coordinators, DPO Admin, QC/QA, Vendors, the LV team, and the automated scripts that glue everything together.

---

## SLIDE: As-Is Process — Stages 1-3 (~5 min)

> Let me walk through the current process step by step. I identified 10 stages.
>
> **Stage 1: Client and Campaign Creation.** The Client Success Team creates a new client and campaign in INFUSE LG. A CID — Campaign ID — is assigned, and this data is synced to the central Registry microservice. This part works well — it's the starting point.
>
> **Stage 2: Campaign Initialization.** This is where the manual work begins. The Campaign Coordinator creates a Word document — the campaign_init_attachment — containing all the campaign details: goal, deadline, GEO, industry criteria, target titles, custom questions, ABM and suppression lists, GDPR requirements. Then the CC emails this document to the DPO Admin. So we have a Word doc being emailed around — entirely manual, no validation, easy to have inconsistencies.
>
> **Stage 3: Campaign Setup in Tracking Sheets.** The DPO Admin takes that Word document and manually enters the data into the Campaign Tracking tab of the DPO Delivery Tracking Google Sheet. Then they create a brand new Master Google Sheet from a template for this campaign. They configure the Settings tab — sheet ID, notification emails, subject lines. They set up the Stats tab with GEO breakdowns and vendor allocations. All manual data entry, risk of typos, and no validation.

---

## SLIDE: As-Is Process — Stages 4-5 (~4 min)

> **Stage 4: Vendor Assignment and Requirements.** The DPO Admin allocates goal portions to vendors — for example, vendor V20 might get 100 out of 147 total leads. For each vendor on each campaign, the admin creates a new Remote Google Sheet from a template. They enter vendor-specific data in the Delivery Tracking tab. They send campaign requirements to vendors separately — via email or chat. The vendor also needs to get their phone script approved before they can start calling.
>
> The pain point here is sheet sprawl. One Remote sheet per vendor per campaign. If you have 10 campaigns with 3 vendors each, that's 30 Remote sheets plus 10 Master sheets. It becomes unmanageable at scale.
>
> **Stage 5: Lead Generation and Upload.** Vendors generate leads — they call prospects, ask custom questions, record the conversations. They upload the lead data into their Remote sheet: first name, last name, company, email, title, phone, country, CQ answers, recording link, prooflink, call notes. Then there's an automated script that copies leads from the Remote sheets to the campaign's Master sheet. This auto-copy is one of the few things that already works well. But there's no validation at upload time — vendor data quality varies and errors slip through.

---

## SLIDE: As-Is Process — Stage 6 (The Biggest Bottleneck) (~4 min)

> **Stage 6: Lead Transfer to MainDB.** This is the most critical stage — and the biggest bottleneck in the entire process.
>
> Looking at the BPMN diagram, there's actually a Google Apps Script designed to run every minute on the Master sheet. It checks for unprocessed and approved leads, sends them to MainDB via the POST /api/lead-upload/dpo endpoint, receives a uuid, then polls for initial statuses. If there are errors or 10+ failed attempts, it stops and sets an error status.
>
> But despite this designed automation, Campaign Coordinators also manually copy leads from the Master sheet to MainDB. Manual transfer is still common in practice. This is the step that creates the most delays — it's slow, error-prone, and blocks everything downstream. Until leads are in MainDB, nothing else can happen.

---

## SLIDE: As-Is Process — Stages 7-8 (~4 min)

> **Stage 7: MainDB Initial Checks.** Once leads are in MainDB, automated checks run. Each lead gets one of three initial status categories:
>
> PENDING — the lead is valid and moves forward.
> INIT REJECT — the lead already exists in the system. This includes statuses like "Already Accepted," "Already DPO," "Already In Collection."
> REJECTED — there's a compliance or quality issue. Things like "Complainer," "Unsubscribed," "Dangerous email," "Email not valid," "Suppressed," "Blacklisted domain," "No MX." There are 14 specific statuses in total.
>
> These statuses are sent back to the Master sheet via automation scripts. But there's a known conflict: if a CC manually set a status that differs from what MainDB returns, the script overwrites the manual correction on the next run. The CC loses their correction without even knowing. This is a documented issue in the dashboards description.
>
> **Stage 8: Lead Verification — LV/LT.** Leads that passed MainDB checks — those with PENDING status — go to Lead Verification. This is a two-part process. First, the LV platform — real people manually checking leads through LinkedIn, company websites. They assign an ov_status, which is the overall verification status, with about 30 possible values grouped into verified, rejected, questionable, and recheck categories. Second, the Leads Tool runs automated technical checks — phone validation, domain compliance, bounce detection — and assigns a pv_status, the phone verification status, with about 8 values. The combination of ov_status and pv_status determines the final status. There's a mapping table with 603 rows covering all these combinations, and each maps to one of four final statuses: accepted, rejected, pending, or returned.

---

## SLIDE: As-Is Process — Stages 9-10 (~4 min)

> **Stage 9: Status Updates and QA Review.** This is an important one. The status collection process is already implemented — I confirmed this from the BPMN diagram — but it's manually triggered. An Admin or CC has to initiate it. The script calls the LVLT endpoint — POST /api/contactInfo-by-mails — with the lead emails and CID. If LVLT returns statuses, they get updated. If LVLT doesn't have a status for some leads, the script falls back to MainDB — it calls getDpoStatusByCID for non-Email Marketing campaigns, or cs-canada-stats for Email Marketing campaigns. If MainDB also errors out, the user gets notified by email and statuses aren't updated.
>
> After statuses are collected, they sync back to the Master sheet, and from there to the Remote sheets so vendors can see results. QA/QC separately reviews call recordings — they listen to recordings and fill in the Recording QA column with their disposition and comments. This QA review is entirely manual. Rejection reasons are assigned from 16 possible categories — things like Bounced, Title, Industry, Size, Revenue, GEO, Company Limit, Duplicate, Recording Failure, and others.
>
> **Stage 10: Rework and Final Delivery.** When a lead is rejected, the vendor sees the status in their Remote sheet. They can dispute it by adding comments in the Reworked Leads column. The DPO Admin manually reviews these disputes and may update the status. Accepted leads are delivered to the client. Stats get aggregated from Master sheets through scripts into the All Stats tab, and from there via formulas into the Campaign Tracking and Delivery Tracking views.
>
> The pain point here is that the rework flow is completely unstructured — it's just comments in a sheet column.

---

## SLIDE: Pain Points Summary (~3 min)

> Let me summarize the key pain points I identified — 12 in total, prioritized by severity.
>
> The critical one for MVP is the **manual lead transfer to MainDB** — it's the single biggest bottleneck that delays the entire pipeline.
>
> High priority MVP items: **manual campaign initialization** through Word docs and email, **sheet sprawl** from creating new sheets per campaign and vendor, the **CC status overwrite conflict** where scripts overwrite manual corrections, and **no vendor portal** — vendors work directly in Google Sheets with no validation or access control.
>
> Medium priority: **manual vendor requirements communication** via email/chat, **manual stats aggregation** through triggered scripts, and **notifications requiring manual triggering**.
>
> For Post-MVP: **manual QA review**, **unstructured rework flow**, and **no real-time dashboards** — everything is formula-based Google Sheets.
>
> The DPO Platform MVP addresses the critical and high-priority items. Post-MVP focuses on analytics, dashboards, and reporting.

---

## SLIDE: To-Be MVP Process — Overview (~2 min)

> The MVP replaces the Google Sheets workflow with a centralized platform. Importantly, it keeps the existing MainDB and LV/LT integrations intact — we're not changing those systems. We're integrating with them via their existing API endpoints.
>
> The MVP compresses the 10 current stages down to 6 stages. Let me walk through them.

---

## SLIDE: To-Be MVP — Stages 1-3 (~4 min)

> **MVP Stage 1: Automated Campaign Initiation.** Instead of a Word document and email, the DPO Admin creates a campaign directly in the platform. The platform pulls client names and CIDs from Registry — that data already exists there from INFUSE LG. The admin selects the client, enters the campaign criteria — goal, GEO, industry, titles, custom questions, deadline, company limit, ABM and suppression lists — all in one form. The Campaign Tracking dashboard is auto-populated. No more Word doc, no more email, no more manual entry into multiple sheets. One form, one submission.
>
> **MVP Stage 2: Vendor Assignment via Platform.** The DPO Admin assigns vendors and allocates goal portions within the platform. Campaign requirements are shared through the platform — no more separate emails. Vendors see only their assigned active campaigns — limited visibility, which is actually described as a requirement in the dashboards description document. Phone script approval is built into the workflow.
>
> **MVP Stage 3: Vendor Lead Upload via Platform.** This replaces the Remote sheets entirely. Vendors upload leads through a portal with data validation at upload time — required fields, format checks, criteria matching. DPO ID and submission date are auto-assigned. Recording links, prooflinks, call notes, CQ answers — all captured in structured form.

---

## SLIDE: To-Be MVP — Stages 4-6 (~5 min)

> **MVP Stage 4: Automatic Lead Transfer to MainDB.** This is the big one — it eliminates the biggest bottleneck. The platform automatically sends lead batches to MainDB via the existing POST /api/lead-upload/dpo endpoint. It receives a partition uuid from MainDB, then polls the GET dpo-status endpoint for initial statuses. The 14 MainDB statuses are mapped and displayed in the platform. What used to take 2 to 4 hours of manual work now happens automatically.
>
> **MVP Stage 5: Automated Status Collection.** The platform collects statuses from LV/LT using the existing /api/contactInfo-by-mails endpoint — the same one the current GAS process uses. The key difference: instead of requiring an Admin or CC to manually trigger it, the platform does this automatically through continuous polling. The fallback logic stays the same — if LVLT doesn't have statuses, fall back to MainDB via getDpoStatusByCID or cs-canada-stats for Email Marketing. All 600+ status combinations are mapped to the four final statuses. And critically — the CC status overwrite conflict is resolved. The platform is the single source of truth, with manual override capability and a full audit log.
>
> **MVP Stage 6: Campaign Tracking and Reporting.** Campaign Tracking, Delivery Tracking, and Stats dashboards are built into the platform, replacing the current Google Sheet tabs. Calculations are real-time — pacing, percentage metrics, unallocated amounts are all automatic. Notifications are event-driven — new leads, status changes, overdue reviews, CC reminders.

---

## SLIDE: To-Be Post-MVP Vision (~4 min)

> For Post-MVP, the focus is on analytics, dashboards, and reporting — building intelligence on top of the MVP foundation.
>
> **Phase A: Analytics and Dashboards.** This is the primary focus. An advanced rejection analytics dashboard with breakdown by reason, vendor, GEO, and time period. Trend analysis with drill-down filters to identify root causes. A vendor performance scoring system — composite scores based on acceptance rate, quality, and timeliness, with a vendor leaderboard and data-driven allocation recommendations. And an executive dashboard with pipeline health indicators, campaign forecasting, and pacing trends.
>
> **Phase B: Reporting and Automation.** Automated monthly reporting with scheduled generation and email distribution. An ad-hoc report builder where users can configure their own reports with filters, date ranges, and groupings, and export to CSV or Excel. We also plan to migrate historical data from Google Sheets to establish a baseline for trend analysis.
>
> **Phase C: Workflow Improvements.** A structured rework and dispute resolution workflow — replacing the current unstructured comments-in-a-column approach with a formal process where vendors submit disputes, admins review in a queue, and the full history is logged.
>
> I want to note what's explicitly out of scope for both MVP and Post-MVP: phone call automation — vendors will continue making calls manually — and QA recording review workflows. The platform focuses on campaign management, analytics, and reporting.

---

## SLIDE: Project Charter — Goals (~3 min)

> Now let me walk you through the Project Charter. Starting with goals.
>
> We have 5 MVP goals, all quantifiable:
>
> 1. Reduce lead transfer time from 2-4 hours per batch to under 5 minutes — automated.
> 2. Centralize campaign management, reducing Google Sheets per campaign from 3-10+ down to zero.
> 3. Automate status collection, replacing the current manually-triggered process with continuous automated polling.
> 4. Provide a role-based vendor portal with data validation, reducing lead data quality errors by 50%.
> 5. Automate campaign initiation using Registry integration, reducing setup time from 1-2 hours to under 15 minutes.
>
> Post-MVP goals focus on rejection analytics, vendor performance scoring, executive dashboards, automated reporting, and historical data migration.

---

## SLIDE: Project Charter — Results and Limitations (~3 min)

> The MVP results — the concrete deliverables — are:
>
> 1. A centralized DPO Platform web application with campaign management, vendor portal, and admin dashboards.
> 2. Automated lead push to MainDB via the existing API endpoint.
> 3. Automated status collection from LV/LT and MainDB with full status mapping — 14 MainDB statuses and 600+ LV/LT status combinations.
> 4. Campaign Tracking and Delivery Tracking dashboards replacing the current Google Sheet tabs.
> 5. A role-based access system with four roles: DPO Admin, Campaign Coordinator, QC/QA, and Vendor.
> 6. An automated notification system.
>
> For limitations — what the project will NOT do:
>
> The platform will not modify INFUSE LG, Registry, or MainDB. It integrates via existing APIs only. It will not change the LV/LT verification process. No client portal. Vendor recruitment, contracts, and billing are out of scope. Historical data migration is Post-MVP. And phone call automation and QA recording review are out of scope entirely.

---

## SLIDE: Project Charter — KPIs (~3 min)

> The KPI table has 5 MVP KPIs and 3 Post-MVP KPIs, all with numeric before and after values.
>
> MVP KPIs:
>
> Lead transfer time per batch: from 2-4 hours manual to under 5 minutes automated.
> Number of Google Sheets per campaign: from 3-10+ sheets to zero.
> Status collection process: from manually triggered by Admin/CC to automated continuous polling.
> Lead data quality errors at upload: from no validation — estimated 15-20% error rate — to a 50% reduction through platform validation.
> Campaign setup time: from 1-2 hours manual to under 15 minutes.
>
> Post-MVP KPIs:
>
> Monthly report generation time: from 4-6 hours manual to fully automated.
> Rejection root-cause analysis: from manually reviewing sheets per campaign to an automated dashboard with filters.
> Vendor performance visibility: from no scoring system to composite scores with a leaderboard.

---

## SLIDE: Project Charter — Risks (~4 min)

> I identified 7 risks across 4 categories.
>
> **Technical risks — 3 items:**
>
> First, existing LV/LT and MainDB API endpoints may behave differently than expected during integration. Probability: medium. Impact: high. Prevention: early API testing with real data, request Postman collections, sandbox environment in sprint 1. Reaction: adjust mapping logic, implement manual status entry as fallback, escalate to API owners.
>
> Second, MainDB lead push endpoint behavior may differ from documentation. Probability: medium, impact: medium. Similar prevention and reaction measures.
>
> Third, status mapping errors due to 600+ LV/LT status combinations. This is a high probability risk with medium impact. Prevention: comprehensive mapping table review with the LV/LT team and automated mapping tests. Reaction: logging and alerting for unmapped statuses, plus manual override capability.
>
> **Organizational risks — 2 items:**
>
> Vendor resistance to adopting the new platform over familiar Google Sheets. We'd prevent this by involving vendor representatives in UAT and providing training. If it happens, we run a parallel operation period and allow gradual migration.
>
> Insufficient stakeholder availability for requirements validation. Prevention: recurring meetings and written requirements for async review. Reaction: escalate via project sponsor.
>
> **Schedule risk:** Scope creep from newly discovered requirements. High probability. Prevention: clear MVP scope boundaries in the charter and a change request process. Reaction: defer new requirements to Post-MVP.
>
> **Data risk:** Data integrity issues during the transition from Google Sheets. Low probability but high impact. Prevention: parallel run period with data reconciliation checks. Reaction: rollback to the Google Sheets workflow if needed.

---

## SLIDE: Project Plan — Timeline Overview (~3 min)

> The project is structured in three phases.
>
> **Pre-MVP: Planning and Analysis** — 4 weeks, May 4 to June 1, 2026. Requirements gathering, stakeholder interviews, As-Is and To-Be process documentation, technical feasibility assessment, and charter approval.
>
> **MVP Phase** — approximately 20 weeks, June 1 to October 16, 2026. This includes:
> - Design: 3 weeks — system architecture, UI/UX wireframes, status mapping specification.
> - Backend development: 9 weeks — campaign initiation, vendor management, lead upload API, MainDB integration, LV/LT integration, notifications, RBAC.
> - Frontend development: 8 weeks, overlapping with backend — campaign management UI, vendor portal, admin dashboards, lead management views.
> - Testing: ongoing during development, plus dedicated E2E testing and 2 weeks of UAT.
> - Deployment: staging, production rollout with 2-3 pilot campaigns, user training, and a parallel operation period running alongside Google Sheets.
> - A 2-week contingency buffer — 10% of the MVP timeline.
>
> **Post-MVP Phase** — approximately 15 weeks, October 19, 2026 to January 30, 2027. Analytics and dashboards first, then reporting and automation, then workflow improvements. Also includes testing, deployment, and a 2-week contingency.
>
> Total project duration: approximately 9 months from kickoff to full Post-MVP completion.

---

## SLIDE: Project Plan — Roles (~2 min)

> For the Responsible column in the project plan, I used the roles from the INFUSE RACI matrix:
>
> - **Project Manager** — accountable for planning, charter, timeline, training, and contingency management.
> - **Business Analyst** — responsible for requirements, process documentation, status mapping specification, and analytics requirements.
> - **Team** — handles all development work — backend, frontend, testing, deployment.
> - **Design Owner** — owns UI/UX wireframes, prototypes, and frontend design decisions.
> - **Product Owner** — involved in vendor performance scoring and feature prioritization.
> - **Sponsor/Customer** — participates in charter approval, UAT, and parallel operation validation.

---

## SLIDE: Key Milestones (~1 min)

> The key milestones:
>
> - June 1 — Project Charter and Plan approved, project kickoff.
> - June 19 — Design phase complete.
> - August 14 — Backend development complete.
> - August 28 — Frontend development complete.
> - September 18 — UAT complete.
> - October 2 — MVP go-live after parallel operation.
> - October 16 — MVP phase closed.
> - January 16 — Post-MVP features deployed.
> - January 30 — Post-MVP phase closed.

---

## SLIDE: Summary (~1 min)

> To summarize: the DPO Platform replaces a fragmented Google Sheets workflow with a centralized web platform. The MVP eliminates the biggest bottleneck — manual lead transfer — and automates campaign initiation, vendor management, status collection, and dashboards. Post-MVP builds analytics, reporting, and vendor intelligence on top of that foundation. The project is scoped for approximately 9 months with clear boundaries, quantifiable KPIs, and identified risks with mitigation plans.
>
> Thank you. I'm happy to take any questions.

---

## Q&A — Anticipated Questions and Answers

**Q: Why not just fix the existing Google Sheets scripts instead of building a new platform?**
> The core issue isn't that the scripts don't work — some of them do. The issue is architectural. Each campaign creates new sheets, each vendor gets their own sheet, and the system doesn't scale. There's no access control, no validation at upload, no audit trail. Fixing scripts doesn't solve sheet sprawl, doesn't give you a vendor portal, and doesn't give you dashboards. A platform solves the structural problem.

**Q: What happens to the existing Google Sheets during transition?**
> We planned a parallel operation period — 1-2 weeks where both the platform and Google Sheets run side-by-side with data reconciliation checks. No hard cutover until validated. If there are issues, we can roll back to Google Sheets.

**Q: How confident are you in the timeline?**
> The timeline includes a 10% contingency buffer for both MVP and Post-MVP. The biggest schedule risk is scope creep, which is why the charter has clear scope boundaries and a change request process. The technical risks — API behavior and status mapping — are mitigated by early testing in sprint 1.

**Q: Why is phone call automation out of scope?**
> The platform focuses on campaign management, lead workflow, and analytics. Phone calls are a separate domain with different technical requirements. Including them would significantly expand scope without addressing the core pain points — which are all about data flow, manual entry, and lack of dashboards.

**Q: What about the 600+ status combinations — how do you handle that?**
> There are approximately 30 ov_status values crossed with 8 pv_status values, producing 603 mapped combinations in the leads_statuses_mapping spreadsheet. Each combination maps to one of 4 final statuses. The platform implements this mapping table and includes automated tests. For any unmapped combination that appears in the future, we'd have logging, alerting, and manual override capability.

**Q: Who are the main stakeholders for UAT?**
> Campaign Coordinators, DPO Admin, and vendor representatives — they're the daily users. The Sponsor/Customer also participates to validate that the platform meets business requirements.
