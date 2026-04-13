# Vendor Switcher & Campaign Switcher — Documentation

---

## Vendor Switcher

### Overview

The vendor switcher is a dropdown in the role bar that allows switching the dashboard view between individual vendors. It is **only visible when the "Vendor" role is active**, keeping the UI clean for Admin and Client Success views.

### Where It Lives

| Layer | Location | Detail |
|-------|----------|--------|
| CSS | `index.html` — `.vendor-switcher`, `.vendor-switcher-label`, `.vendor-select` | Styles for the switcher container and dropdown |
| HTML | `index.html` — inside `.role-bar` | `<div id="vendor-switcher">` with `<select id="vendor-select">` |
| JS | `index.html` — `switchVendor()`, `setRole()` | Logic to show/hide and handle selection |
| Data | `index.html` — `const VENDORS` (line ~2170) | Source of truth for vendor list |

### Vendors

Defined in `const VENDORS`:

| ID | Name | Geo | Color |
|----|------|-----|-------|
| `v1` | DataReach | UK | `#0984e3` |
| `v2` | LeadForge | Ireland | `#00b894` |
| `v3` | ProspectIQ | Germany | `#6c5ce7` |

### Show / Hide Behaviour

Controlled by the `visible` CSS class:

```css
.vendor-switcher          { display: none; }   /* hidden by default */
.vendor-switcher.visible  { display: flex; }   /* shown when vendor role active */
```

`setRole(role)` adds/removes `.visible` on `#vendor-switcher`:

```js
const switcher = document.getElementById('vendor-switcher');
if (switcher) switcher.classList.toggle('visible', role === 'vendor');
```

### Switching Vendors

`switchVendor(vid)` is called by the `<select>` `onchange` handler:

```js
function switchVendor(vid) {
  STATE.currentVendor = vid;          // update global state
  const v = VENDORS.find(x => x.id === vid);
  if (v) {
    // Update sidebar user block
    document.getElementById('sidebar-user-name').textContent = v.name;
    document.getElementById('sidebar-user-role').textContent = v.geo;
    document.getElementById('sidebar-avatar').textContent   = v.name[0];
    // Update role-bar descriptor (top-right)
    document.getElementById('role-desc').textContent = 'Viewing as ' + v.name + ' (' + v.geo + ')';
  }
  render();  // re-render all tabs with scoped data
}
```

#### What updates on vendor switch

| Element | What changes |
|---------|-------------|
| `#role-desc` (top-right of role bar) | "Viewing as DataReach (UK)" → reflects selected vendor |
| `#sidebar-user-name` | Vendor name |
| `#sidebar-user-role` | Vendor geo region |
| `#sidebar-avatar` | First letter of vendor name |
| Campaigns tab | Only shows campaigns assigned to selected vendor |
| Leads / slot data | Filtered to selected vendor's companies and leads |
| Vendor callout banner | Updates to selected vendor name |

### Vendor Isolation

Each vendor only sees their own data regardless of which vendor is selected in the switcher. This mirrors the real-world constraint: **vendors must never know about each other**.

- `COMPANIES` filtered by `company.vendor === STATE.currentVendor`
- `LEADS` filtered by `lead.vendor === STATE.currentVendor`
- `CAMPAIGNS` filtered by `campaign.vendors.includes(STATE.currentVendor)`

The switcher is an **admin/demo tool** — in a real deployment, a vendor user would be authenticated to exactly one vendor ID and this switcher would not exist.

### Adding a New Vendor

1. Add an entry to `const VENDORS` in `index.html`:
   ```js
   { id: 'v4', name: 'VendorName', geo: 'Region', color: '#hexcolor' }
   ```
2. Add a corresponding `<option>` to `#vendor-select` in the HTML:
   ```html
   <option value="v4">VendorName (Region)</option>
   ```
3. Add sample companies/leads with `vendor: 'v4'` to `COMPANIES` and `S1_LEADS`.
4. Add `'v4'` to the `vendors` array on any campaigns they should appear in.

---

## Campaign Switcher

### Overview

The campaign switcher is a pill-button bar that appears at the top of the **Scenario 1 (Company Limit)** tab. It lets any role switch between campaigns to see that campaign's leads and custom questions. A vendor only sees campaigns they are assigned to — isolating them from campaigns they should not know about.

### Where It Lives

| Layer | Location | Detail |
|-------|----------|--------|
| State | `index.html` — `STATE.currentCampaign` | Currently selected campaign ID (default: `'cam1'`) |
| JS | `index.html` — `switchCampaign(camId)` | Sets `STATE.currentCampaign` and calls `render()` |
| JS | `index.html` — `renderS1()` (campaign switcher section) | Builds the pill buttons and filters leads |
| Data | `index.html` — `S1_LEADS[*].campaign` | Every lead carries its campaign ID |
| Data | `index.html` — `CAMPAIGNS[*].qualQuestions` | Per-campaign custom question definitions |

### Campaigns with Dummy Data

| ID | Name | Client | Vendors | Status |
|----|------|--------|---------|--------|
| `cam1` | EU SaaS Decision Makers | TechCorp Global | v1, v2, v3 | Live |
| `cam2` | UK FinTech C-Suite | FinAxis Partners | v1 | Live |
| `cam3` | DACH Manufacturing IT Leaders | Meritech Solutions | v3 | Live |
| `cam4` | Nordics Cloud Transformation | Apex Cloud Systems | v2 | Live |
| `cam5` | Healthcare IT Decision Makers | Celera Health | v1, v2 | Paused |
| `cam6` | EU Energy Sector CxOs | Praxis Energy | v3 | Paused |
| `cam7` | UK Retail Digital Transformation | Landmark Retail Group | v1 | Closed |
| `cam8` | EMEA MarTech Buyers | Orion Digital | v1, v2 | Closed |

Campaigns **cam1, cam2, cam3** have full `S1_LEADS` dummy data. cam4–cam8 are shown in the switcher but have no leads in `S1_LEADS` (demonstrating an empty state cleanly).

### Custom Questions (CQ Columns)

Each campaign defines its own qualifying questions via `qualQuestions: [ ... ]` in `CAMPAIGNS`. These drive the CQ1/CQ2/CQ3 column **headers** (tooltip text) in the leads table on the Scenario 1 tab.

```js
// Example: cam1
qualQuestions: [
  'Are you evaluating SaaS solutions in the next 6 months?',
  'Do you have budget authority for this decision?',
  'How many users would the solution need to support?',
]
```

When the campaign is switched, `camQsActive` (derived in `renderS1`) updates, and the CQ headers re-render with the new tooltips. The CQ values in each lead row (`cq1`, `cq2`, `cq3`) are pre-populated to match the questions of their respective campaign.

#### CQ values per campaign

| Campaign | CQ1 question | CQ2 question | CQ3 question |
|----------|-------------|-------------|-------------|
| cam1 | Evaluating SaaS in next 6 months? | Budget authority? | User count range |
| cam2 | Responsible for tech procurement? | Currently reviewing fintech vendors? | Annual tech budget range |
| cam3 | Actively digitising manufacturing? | Who leads tech decisions (role)? | Evaluating ERP/MES this year? |

### Vendor-Campaign Scoping

When the **Vendor** role is active and the vendor is switched:

- The campaign switcher only shows campaigns where `campaign.vendors.includes(STATE.currentVendor)`.
- If `STATE.currentCampaign` is not in the vendor's campaign list, it is auto-reset to the first valid campaign.
- Leads are then double-filtered: `l.campaign === STATE.currentCampaign && l.vendor === STATE.currentVendor`.

This ensures a vendor working on multiple campaigns (e.g. DataReach v1 is on cam1, cam2, cam5, cam7, cam8) can only see their own leads on each campaign they are assigned to.

### Adding a New Campaign with Leads

1. Add the campaign object to `CAMPAIGNS` with a `qualQuestions` array.
2. Add leads to `S1_LEADS` with `campaign: 'camN'` and `vendor: 'vX'`.
3. The campaign switcher pill will appear automatically for any role that has access.
