# Campaign Creation Wizard — Documentation

**File:** `DPO_Platform_Followup/index.html`  
**Access:** Admin and Client Success roles only (Vendor and Lead Delivery cannot create campaigns)  
**Trigger:** "+ New Campaign" button on the Campaigns tab

---

## Overview

The campaign creation UI is a 5-step wizard modal. It collects all configuration needed to launch a DPO campaign — identity, audience targeting, lists, qualification rules, and vendor distribution logic.

Steps are shown in a left sidebar with visual state: active (blue dot), completed (green dot), incomplete (grey dot). All steps except Step 1 are optional from a validation standpoint — the only hard-required fields are Campaign Name and Client.

```
Step 1 — Campaign Info        Required
Step 2 — Target Audience      Required (Countries)
Step 3 — Lists                Optional
Step 4 — Qualification        Optional
Step 5 — Vendor Split         Optional
```

**Footer:** Cancel is always on the left. ← Back, Next →, and Create Campaign are grouped together on the right. Back is hidden on step 1; Create Campaign replaces Next on step 5.

---

## Step 1 — Campaign Info

Divided into three logical groups.

### Group: Campaign Identity

| Field | ID | Type | Required | Notes |
|---|---|---|---|---|
| Campaign Name | `f-name` | Text input | Yes | e.g. "EU SaaS Decision Makers Q3" |
| Client | `f-client-search` / `f-client-id` | Searchable dropdown | Yes | Filters `CLIENTS` array by name or CID; stores hidden `f-client-id` |

### Group: Volume & Timeline

| Field | ID | Type | Required | Notes |
|---|---|---|---|---|
| Goal (total leads) | `f-goal` | Number input | Yes | Total across all vendors; triggers `syncGoalTotal()` |
| Sub-Goals by Country | `subgoal-list` | Dynamic rows (nested under Goal) | No | Indented subsection below Goal field; each row: country select + leads number; added via `addSubgoal()`; validated against total goal |
| Deadline | `f-deadline` | Date picker | No | Hard end date; clicking anywhere on the field (not just the calendar icon) opens the picker via `openDeadlinePicker()` |

Sub-goal rows are visually subordinate to the Goal field — indented with a left border. Warning (`subgoal-warning`) appears if sub-goal totals don't match the campaign goal.

### Group: Delivery Settings

| Field | ID | Type | Required | Notes |
|---|---|---|---|---|
| Company Limit | `f-company-limit` | Number input | No | Max leads accepted per company; drives the slot counter in S1 |
| Pacing | `f-pacing` | Select | No | Even-pacing only |

---

## Step 2 — Target Audience

Divided into two groups: Geography & Firmographics, and Persona. The Exclusions section has been removed — job title and job function exclusions are handled inline via include/exclude toggles.

### Group: Geography & Firmographics

| Field | ID | Type | Notes |
|---|---|---|---|
| Countries | `f-geo` (hidden) | Multi-select tag picker | Required; drives `_geoSelected[]` |
| Industry | `f-industry` | Select | Any / SaaS / Financial Services / Healthcare / Manufacturing / Retail / Professional Services / Telecom / Energy / Education |
| Company Size | `size-range-list` | Dynamic rows | Each row is a size range select; added via `addSizeRange()` |
| Revenue | `f-revenue` | Select | Any / Under $10M / $10M–$100M / $100M–$1B / $1B+ |

### Group: Persona

Job Titles and Job Function / Area use an **include/exclude toggle** system. When a value is selected from the search dropdown, it appears as a row below the input with two toggle buttons: **Include** (green) and **Exclude** (red). Default is Include.

| Field | ID | Type | Notes |
|---|---|---|---|
| Target Job Titles | `f-titles` (hidden) | Multi-select with IE toggles | Drives `_titlesSelected[]` and `_titlesMode{}`; sourced from `JOB_TITLES` |
| Job Function / Area | `f-jobareas` (hidden) | Multi-select with IE toggles | Drives `_jobareasSelected[]` and `_jobareasMode{}`; sourced from `JOB_AREAS` |

#### Include/Exclude toggle behavior

- Selected items render in `ie-tag-list` below the search input (not as chips inside the input)
- Each row: item name · Include / Exclude toggle · ✕ remove
- Include = green active state; Exclude = red active state
- Hidden field value encodes mode: excluded items are prefixed with `-` (e.g. `-Intern, CTO`)

### Data saved to campaign object

```js
geo:      'UK, Germany, Ireland'     // comma-joined _geoSelected
industry: 'SaaS / Software'
size:     '201-500, 1000+'
revenue:  '$10M–$100M'
titles:   '-Intern, CTO, VP Engineering'   // '-' prefix = exclude
jobareas: 'IT & Technology, -Operations'
```

---

## Step 3 — Lists

Both lists use a **file upload simulation** UI. No actual file is parsed — clicking the upload zone shows a brief uploading animation then reveals dummy data.

### ABM List

| State | Behavior |
|---|---|
| Empty | Dashed upload zone with 📂 icon; click to simulate upload |
| Uploading | 600ms spinner animation |
| Uploaded | File header (filename + ✕ remove), scrollable preview table of target companies, metadata line |

Dummy data: 10 companies (Salesforce, HubSpot, SAP, Workday, ServiceNow, Zendesk, Monday.com, Freshworks, Pipedrive, Intercom) with columns: Company, Domain, Country, Industry, Size.

### Suppression List

Same upload flow. Dummy data: 7 contact records with columns: First Name, Last Name, Email, Company, Reason.

Clicking ✕ in the file header removes the upload and restores the upload zone. State resets when the modal is closed.

### Functions

| Function | Description |
|---|---|
| `simulateUpload(type)` | `type`: `'abm'` or `'suppression'`; shows spinner then renders dummy file preview |
| `removeUpload(type)` | Hides uploaded block, restores upload zone |

---

## Step 4 — Qualification

### Standard Fields

| Field | ID | Type | Notes |
|---|---|---|---|
| Language | `f-language` | Select | English / German / French / Spanish / Dutch |
| OPT-IN / GDPR | `f-gdpr` | Select | Required / Optional / Not Required |
| Phone Script Notes | `f-script` | Textarea | Objection handling and general qualification guidance |

### Qualification Questions Builder

Below the standard fields, a **Qualification Questions** section allows adding structured questions with answer options and disqualification flags.

**Adding a question:** Click "+ Add Question". Each question block contains:
- Numbered header (auto-incremented)
- Question text input
- List of answer option rows (2 auto-added on creation)
- "+ Add answer option" button
- ✕ to remove the whole question

**Answer option rows:** Each row contains:
- Answer text input
- **Disqualify** toggle button — click to mark as disqualifying (turns red, text changes to "Disqualified"); click again to unmark
- ✕ to remove that answer

### Functions

| Function | Description |
|---|---|
| `addQualQuestion()` | Adds a question block with 2 pre-populated answer rows |
| `removeQualQuestion(qid)` | Removes question block by ID |
| `addQualAnswer(qid)` | Adds an answer row to the specified question |
| `toggleAnswerDisq(btn)` | Toggles `is-disq` class on button; updates label |

---

## Step 5 — Vendor Split (Optional)

Configures how the campaign's company pool is divided between vendors. Each vendor receives an exclusive scope — they cannot see each other's assignments or know other vendors exist.

**Skipping this step** (selecting no rule) defaults the campaign to counter-only enforcement with `splitRule: null`.

### Split Rule Cards

Five cards displayed in a 2-column grid. Click to select — the selected card highlights with a colored border. Only one rule can be active at a time.

| Rule | Color | Dimension | When to use |
|---|---|---|---|
| Rule 1 — ABM List | `#0984e3` Blue | ABM accounts | Client has a named target account list; divide it between vendors |
| Rule 2 — Geography | `#00b894` Green | Regional territories | No ABM list; campaign has distinct geo sub-regions (e.g. UK, Germany) |
| Rule 3 — Industry | `#6c5ce7` Purple | Industry verticals | No ABM, broad geo, multiple industries present |
| Rule 4 — Company Size | `#e17055` Orange | Employee size bands | No ABM, broad geo/industry; size range is wide enough to split |
| No Split | `#b8860b` Gold | None | No clean split dimension; cap ≥ 3; slot counter is primary enforcement |

Selecting Rules 1–4 reveals the **Vendor Scope Assignments** section along with a contextual hint. Selecting **No Split** hides the scope section and instead shows the **Vendor Assignment** section — a flat checkbox list of all vendors. At least one vendor must be checked before the campaign can be saved.

### Vendor Assignment (No Split only)

When rule 5 is selected, a flat checkbox list renders under `#vendor-nosplit-list` (inside `#vendor-nosplit-section`). Each row shows:
- Vendor name with a colored dot and geo label
- A checkbox (`id="nosplit-vendor-{v.id}"`, `value="{v.name}"`)

All vendors source freely across the full campaign pool. The per-company slot counter is the only enforcement mechanism — no exclusive territories apply.

On save, `submitCampaign()` reads checked boxes and pushes `{ vendor: name, scope: '' }` entries into `vendorScopes[]`, same shape as scoped assignments.

### Vendor Scope Assignments

Dynamic rows added via `addVendorScopeRow()`, removed via ✕ (`removeVendorScopeRow(id)`). Each row contains a **Vendor dropdown** (sourced from `VENDORS[]`) and a **scope input** whose type varies by rule:

| Rule | Scope input type | Options / behaviour |
|---|---|---|
| Rule 1 — ABM List (Random) | Number input | Enter account count to auto-distribute (e.g. 50) |
| Rule 1 — ABM List (Specific) | Multi-select | Pick individual companies from `ABM_COMPANIES[]`; hold Ctrl/Cmd for multiple |
| Rule 2 — Geography | Multi-select | Countries from `COUNTRIES[]` |
| Rule 3 — Industry | Multi-select | Industry verticals |
| Rule 4 — Company Size | Multi-select | Size bands from `SIZE_OPTIONS[]` |

Multi-select values are joined with `, ` when saved to the campaign object.

#### ABM Assignment Mode (Rule 1 only)

When Rule 1 is selected, an **ABM Assignment Mode** radio group appears above the scope rows:

- **Random (auto-distribute)** — enter the number of accounts to assign; the platform splits them automatically
- **Specific (pick accounts per vendor)** — a multi-select list of named companies appears; admin manually assigns accounts to each vendor

Switching mode clears all existing rows.

### Supporting Settings

| Field | ID | Options | Default | Notes |
|---|---|---|---|---|
| Idle Release Window | `f-idle-release` | 3 / 5 / 7 / 10 business days | 5 days | Companies auto-return to pool if vendor goes idle |

### Recycling Rules (always applied)

- Companies auto-release after the idle window if vendor goes inactive
- A vendor hitting their contracted quota triggers immediate release of remaining companies
- Reassigned companies carry their **actual remaining slots** (not reset to original cap)
- The receiving vendor gets a suppression list of already-contacted leads (no vendor identifiers)

### Data saved to campaign object

```js
splitRule:    1 | 2 | 3 | 4 | 5 | null
vendorScopes: [{ vendor: string, scope: string }]
preCheck:     'optional' | 'required' | 'off'
idleRelease:  3 | 5 | 7 | 10
```

---

## Full Campaign Object Schema

All fields collected by `submitCampaign()` and pushed to `CAMPAIGNS[]`:

```js
{
  id:           'cam2',               // auto-generated
  name:         string,              // Step 1
  client:       string,              // Step 1
  clientId:     string | null,       // Step 1
  status:       'Pre',               // always 'Pre' on creation
  goal:         number,              // Step 1
  subgoals:     [{ country, leads }],// Step 1
  delivered:    0,
  inReview:     0,
  companyLimit: number,              // Step 1
  pacing:       string,              // Step 1
  deadline:     string,              // Step 1

  // Step 2 — Target Audience
  geo:          string,              // comma-joined included countries
  industry:     string,
  size:         string,
  revenue:      string,
  titles:       string,              // '-' prefix = exclude
  jobareas:     string,              // '-' prefix = exclude

  // Step 3 — Lists
  abm:          'yes' | 'no',        // file upload simulated
  suppression:  'yes' | 'no',        // file upload simulated

  // Step 4 — Qualification
  language:     string,
  gdpr:         string,
  script:       string,
  // qualQuestions not yet persisted to campaign object (wizard-only)

  // Step 5 — Vendor Split
  splitRule:    number | null,
  vendorScopes: [{ vendor, scope }],
  idleRelease:  number,

  vendors:      [],                  // populated later by Admin
}
```

---

## JavaScript Functions Reference

### Dummy data fill

Each step has a **⚡ Complete with dummy** button (yellow, top-right of the step panel) that pre-fills all fields with realistic sample data. Useful for demos and testing.

| Function | Step | What it fills |
|---|---|---|
| `fillDummy(1)` | Campaign Info | Name: "EU SaaS Decision Makers Q3-2026"; Client: TechCorp Global (CID-0001); Goal: 300; Deadline: 2026-09-30; Company limit: 2; Pacing: Even-pacing |
| `fillDummy(2)` | Target Audience | Countries: UK, Germany, Netherlands, France (with subgoal amounts 100/80/70/50); Industries: SaaS / Software, Financial Services, Analytics; Revenue: $10M–$100M; Titles: CTO, VP Engineering, IT Director; Job areas: IT & Technology, Engineering |
| `fillDummy(3)` | Lists | Triggers `simulateUpload('abm')` and `simulateUpload('suppression')` — shows dummy file previews for both lists |
| `fillDummy(4)` | Qualification | Language: English; GDPR: Required; script notes pre-filled; 2 qualification questions with answer options (one answer marked disqualifying) |
| `fillDummy(5)` | Vendor Split | Selects Rule 1 — ABM List, Random assignment; adds 3 vendor rows: DataReach → 6 accounts, LeadForge → 6 accounts, ProspectIQ → 5 accounts |

Fields that already have a value are not overwritten (idempotent per field).

---

### Wizard navigation

| Function | Description |
|---|---|
| `openModal()` | Resets form, goes to step 1, opens modal |
| `closeModal()` | Closes modal, clears dropdowns |
| `goStep(n)` | Navigates to step n; updates dot states, shows/hides Back/Next/Submit |
| `stepNav(dir)` | Called by Back (dir=-1) and Next (dir=1) buttons |
| `resetModalForm()` | Clears all fields, resets all state arrays and upload zones |
| `submitCampaign()` | Validates, collects all fields, pushes to CAMPAIGNS[], closes modal |
| `openDeadlinePicker()` | Opens native date picker on the deadline input |

### Step 2 — Target Audience (IE toggles)

| Function | Description |
|---|---|
| `addTitle(title)` | Adds title to `_titlesSelected[]`, defaults mode to `'include'`, re-renders |
| `removeTitle(title)` | Removes title from state, re-renders |
| `toggleTitleMode(title, mode)` | Sets `_titlesMode[title]` to `'include'` or `'exclude'`, re-renders |
| `renderTitlesTags()` | Rebuilds the IE tag list for job titles |
| `addJobarea(area)` | Adds job area to `_jobareasSelected[]`, defaults to `'include'` |
| `removeJobarea(area)` | Removes job area from state |
| `toggleJobareaMode(area, mode)` | Sets `_jobareasMode[area]` to `'include'` or `'exclude'` |
| `renderJobareasTags()` | Rebuilds the IE tag list for job areas |

### Step 3 — Lists

| Function | Description |
|---|---|
| `simulateUpload(type)` | Simulates file upload for `'abm'` or `'suppression'`; shows spinner then dummy table |
| `removeUpload(type)` | Resets upload zone to empty state |

### Step 4 — Qualification Questions

| Function | Description |
|---|---|
| `addQualQuestion()` | Creates a new question block with 2 default answer rows |
| `removeQualQuestion(qid)` | Removes a question block by element ID |
| `addQualAnswer(qid)` | Appends an answer row to the specified question |
| `toggleAnswerDisq(btn)` | Toggles disqualified state on an answer button |

### Step 5 — Vendor Split

| Function | Description |
|---|---|
| `selectSplitRule(rule)` | Highlights selected rule card; shows/hides vendor scope section or No Split vendor assignment section; inserts contextual hint and ABM mode selector (rule 1 only); clears existing rows |
| `setAbmMode(mode)` | Sets `_abmMode` to `'random'` or `'specific'`; clears existing rows so new rows render with the correct input type |
| `addVendorScopeRow()` | Adds a row with a vendor dropdown + context-appropriate scope input (number / multi-select) based on `_selectedSplitRule` and `_abmMode` |
| `removeVendorScopeRow(id)` | Removes a specific scope row by ID |
| `_vendorSelectHtml(id)` | Returns HTML for the vendor `<select>` populated from `VENDORS[]` |
| `_scopeValueHtml(id, rule)` | Returns HTML for the scope input — varies by rule and ABM mode |

### Global state variables

```js
_currentStep            // integer 1–5, current wizard step
TOTAL_STEPS             // 5 (constant)
_selectedClient         // client object from CLIENTS[] or null
_subgoalCount           // counter for subgoal row IDs
_sizeRangeCount         // counter for size range row IDs
_geoSelected[]          // included countries
_titlesSelected[]       // job titles (include or exclude)
_titlesMode{}           // map of title -> 'include' | 'exclude'
_jobareasSelected[]     // job areas (include or exclude)
_jobareasMode{}         // map of area -> 'include' | 'exclude'
_qualQuestionCount      // counter for question block IDs
_qualAnswerCount        // counter for answer row IDs
_selectedSplitRule      // integer 1–5 or null
_vendorScopeCount       // counter for vendor scope row IDs
_abmMode                // 'random' | 'specific' — ABM assignment mode for Rule 1
_exclGeoSelected[]      // excluded countries (legacy, not surfaced in UI)
_exclIndustrySelected[] // excluded industries (legacy, not surfaced in UI)
_exclTitlesSelected[]   // excluded job titles (legacy, replaced by _titlesMode)
```

---

## Splitter Logic — Decision Tree Reference

The decision tree determines which split rule should be applied. Walk rules top to bottom and apply the first match.

```
Is there a client ABM (named account) list?
  YES → Rule 1 (ABM List split)

  NO → Does the geo have distinct sub-regions?
         YES → Rule 2 (Geography split)

         NO → Are multiple distinct industries targeted?
                YES → Rule 3 (Industry split)

                NO → Is the company size range wide enough to segment?
                       YES → Rule 4 (Company Size split)

                       NO → Is the per-company cap ≥ 3?
                              YES → No Split (counter-only enforcement)
                              NO  → Rule 6 (STOP — escalate, do not launch)
```

**Rule 6** is a platform guard, not a wizard option. If cap ≤ 2 with no viable split dimension, campaign parameters must be renegotiated with the client before creation.

### What vendors see vs. what the platform stores

| Aspect | Vendor sees | Platform stores |
|---|---|---|
| Their scope | "Your territory: United Kingdom" | Full split config with all vendor scopes |
| Other vendors | Nothing — other vendors invisible | All vendor scopes in `vendorScopes[]` |
| Company cap | Remaining slots per company (live number) | `slots`, `used` per company |
| Why company unavailable | "This company has reached its campaign limit" | Full slot history |
| Recycled company | Appears as normal available company | `status: 'reassigned'`, original vendor data intact |

---

## UI / CSS Reference

### Modal dimensions

| Property | Value |
|---|---|
| Max width | 940px |
| Max height | calc(100vh - 32px) |
| Sidenav width | 168px |
| Body padding | 24px 28px |

### Form group block structure

```html
<div class="form-group-block">
  <div class="form-group-block-label">Group Name</div>
  <!-- form-row(s) inside -->
</div>
```

### Include/Exclude tag row structure

```html
<div class="ie-tag-list" id="titles-ie-list">
  <div class="ie-tag-row">
    <span class="ie-tag-name">CTO</span>
    <div class="ie-toggle">
      <button class="ie-toggle-btn active-include">Include</button>
      <button class="ie-toggle-btn">Exclude</button>
    </div>
    <button class="ie-tag-remove">−</button>
  </div>
</div>
```

| Class | State | Color |
|---|---|---|
| `.ie-toggle-btn.active-include` | Include active | Green `#00b894` |
| `.ie-toggle-btn.active-exclude` | Exclude active | Red `#d63031` |

### File upload states

| Class | Usage |
|---|---|
| `.file-upload-zone` | Empty state — dashed border upload target |
| `.file-uploaded-block` | Filled state — header + preview table + meta |
| `.file-preview-table` | Scrollable data table inside uploaded block |
| `.qual-answer-disq-toggle.is-disq` | Disqualified answer — red background/border |

### Split rule colors

| Rule | Color | Hex |
|---|---|---|
| Rule 1 — ABM List | Blue | `#0984e3` |
| Rule 2 — Geography | Green | `#00b894` |
| Rule 3 — Industry | Purple | `#6c5ce7` |
| Rule 4 — Company Size | Orange | `#e17055` |
| No Split | Gold | `#b8860b` |
| Rule 6 — Blocked | Red | `#d63031` (Decision Tree only) |
