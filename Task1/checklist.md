# DPO Platform — Task Checklist

> **Deadline:** EOD April 6, 2026
> **Presentation:** April 7, 2026 (1 hour)
> Status key: `[ ]` = to do, `[~]` = in progress, `[x]` = done

---

## Phase 1: Research & Understanding (Day 1–2)

- [x] Read the task brief (`task.md`)
- [x] Read and organize the instructions / process overview (`instructions.md`)
- [x] Review the Document Register (`readme.xlsx`)
- [x] Watch the introduction video
- [x] Review `dashboards_description.xlsx` — all 7 tabs documented in research_notes.md
- [x] Study `delivery_tracking_copy.xlsx` — 14 sheets, 4516 rows of real data
- [x] Study `provider_remote_gsheet_example_copy.xlsx` — Rubrik campaign V20, 32 columns, 3 CQs
- [x] Study `master_sheet_example_copy.xlsx` — 40 columns, status conflict issue noted
- [x] Study `dpo_rejections.xlsx` — 6,597 rejections catalogued
- [x] Review `Project Management Rules.docx` — charter structure, phases, 5-10% contingency
- [x] Review `Bootcamp - Project Charter Template.docx` — all sections noted
- [x] Review `Bootcamp - Project Plan Template.xlsx` — columns and MVP/Post-MVP structure
- [x] Review `campaign_init_attachment.docx` — Rubrik example with all criteria fields
- [x] Review `leads_flow_diagram` (`Untitled Diagram.drawio.png`) — two BPMN processes
- [x] Review `bootcamp_project documentation (for senior).docx` — KPIs must be numeric
- [x] Study `leads_statuses_mapping (bootcamp).xlsx` — 5-step API flow, 600+ status combos
- [ ] Postman endpoints documentation — request when needed for technical depth
- [ ] Compile list of questions for stakeholders (Astghik Sargsyan) — see open questions in `instructions.md` + any new ones
- [ ] Schedule call with Astghik (max 30 min/day) to clarify business logic

---

## Phase 2: Analysis & Requirements (Day 2–3)

- [x] Map the current **As-Is** process end-to-end (10 stages documented in `analysis.md`)
- [x] Identify pain points and manual steps to eliminate (12 pain points prioritized in `analysis.md`)
- [x] Define the **To-Be MVP** process (6 stages + scope boundaries in `analysis.md`)
- [x] Define the **To-Be Post-MVP** process (3 phases: Advanced Ops, Automation, Ecosystem in `analysis.md`)
- [x] Define roles involved in each process step (6 roles mapped in `analysis.md`)
- [x] Define scope boundaries — in-scope vs. out-of-scope (documented in `analysis.md`)

---

## Phase 3: Deliverable Creation (Day 3–5)

### 3A. Project Charter (`.docx`)

Using the `Bootcamp - Project Charter Template.docx` + tips from `bootcamp_project documentation (for senior).docx`:

- [x] **Project Name** — DPO Platform (Direct Phone Outreach Platform)
- [x] **Context Description** — fragmented Google Sheets workflow, manual bottlenecks
- [x] **Goals** — 5 MVP goals + Post-MVP goals (all quantifiable with numeric targets)
- [x] **Results** — 6 MVP results + Post-MVP results (specific deliverables)
- [x] **Limitations** — 6 explicit exclusions defined
- [x] **KPIs** — 5 MVP + 3 Post-MVP KPIs with numeric Before/After values
- [x] **Risks** — 7 risks (Technical x3, Organizational x2, Schedule x1, Data x1) with all columns filled

### 3B. Project Plan (`.xlsx`)

Using the `Bootcamp - Project Plan Template.xlsx`:

- [x] Break down into **MVP phase** steps:
  - [x] Pre-MVP: Planning & Analysis (5 tasks)
  - [x] Design (3 tasks: architecture, UI/UX, status mapping)
  - [x] Backend development (7 tasks: campaign init, vendor mgmt, lead upload, MainDB, LV/LT, notifications, RBAC)
  - [x] Frontend development (4 tasks: campaign UI, vendor portal, dashboards, lead views)
  - [x] Testing (3 tasks: unit/integration, E2E, UAT)
  - [x] Deployment & Launch (4 tasks: staging, production, training, parallel ops)
- [x] Break down into **Post-MVP phase** steps (9 tasks):
  - [x] QA review workflow, rework/dispute, phone integration R&D
  - [x] Analytics, vendor scoring, automated reporting, data migration
  - [x] Testing & deployment
- [x] Assign states, start/end dates, responsible parties, comments for each step
- [x] Add contingency (10% buffer for both MVP and Post-MVP)
- [x] Set baseline dates (MVP: May 4 – Oct 16, 2026; Post-MVP: Oct 19, 2026 – Jan 30, 2027)

### 3C. Business Process Visualizations (Diagrams)

- [x] **As-Is diagram** — 10 stages, all 6 roles, manual pain points highlighted (HTML/Mermaid)
- [x] **To-Be MVP diagram** — 6 stages with improvement annotations vs. As-Is
- [x] **To-Be Post-MVP diagram** — fully automated future state with all enhancements labeled
- [x] Ensure diagrams show: sequence of steps, results after each step, all participating roles
- [x] Choose notation (Flowchart, BPMN, or UML) and keep it consistent — BPMN-style flowchart via Mermaid

---

## Phase 4: Review & Polish (Day 5)

- [x] Cross-check Project Charter against Project Management Rules (goals aligned with strategy, KPIs measurable, risks have prevention+reaction)
- [x] Cross-check Project Plan for completeness (all phases covered, dates realistic, responsibilities assigned)
- [x] Verify diagrams are clear, consistent, and match the written deliverables
- [ ] Proofread all documents (final review with user)
- [x] Prepare presentation structure for the April 7 meeting:
  - [x] Project overview & context
  - [x] As-Is process walkthrough
  - [x] Pain points identified
  - [x] To-Be MVP solution
  - [x] To-Be Post-MVP vision
  - [x] Project Charter highlights (goals, KPIs, risks)
  - [x] Project Plan summary & timeline
  - [x] Q&A

---

## Phase 5: Submission (EOD April 6)

- [ ] Send all materials via email reply
- [ ] Upload to the shared folder
- [ ] Confirm receipt

---

## Open Items / Questions to Resolve

*(Move answered items to a "Resolved" section below as you go)*

1. [ ] What does "permission to participate in the campaign" mean?
2. [x] Who creates a new client/campaign in INFUSE LG? → **Client Success Team (CST)**
3. [ ] What is the ABM list exactly?
4. [ ] Difference between "APPOINTMENT SETTING" and "DPO PLATFORM - statuses upd 3/4" in delivery tracking?
5. [ ] What does "Unallocated" mean in delivery tracking Column W?
6. [ ] What can be the goal of a campaign other than GEO?
7. [ ] Confirm: Vendors only access remote sheets, not master sheets?
8. [ ] Missing document: `leads_statuses_mapping (bootcamp)` — statuses mapping from endpoints (not yet in folder, request from Astghik)
9. [ ] Postman endpoints documentation — request when needed for technical depth
10. [ ] Understand the two processes in leads_flow_diagram: (a) leads sending to MainDB for initial statuses (endpoints exist but not used as-is) vs. (b) leads statuses collecting (already in use)

---

## Resolved Questions

*(Move items here once answered)*
