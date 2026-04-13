# Left Navigation — DPO Platform Dashboard

## Overview

The left sidebar is a fixed 220px panel (`#1a1a2e` dark background) present on all pages. It contains the brand header, role-scoped navigation links, and a user identity footer. Navigation is role-aware: certain items are hidden or disabled depending on the active role (Admin / Vendor / Client Success).

---

## Structure

```
┌─────────────────────┐
│  ⚡ DPO Platform    │  ← Brand header
│     Operations Suite│
├─────────────────────┤
│  MAIN               │  ← Section label
│  ▤  Dashboard       │
│  ◈  Campaigns    8  │
│  ◎  Vendors      4  │  ← Hidden for Vendor role
│  ↓  Lead Inbox  12  │
├─────────────────────┤
│  OPERATIONS         │
│  ⚙  Rules Engine    │
│  ⊘  Suppression Lists│
│  ◧  ABM Lists     3 │
│  ⇄  Integrations    │
├─────────────────────┤
│  REPORTING          │
│  ↗  Analytics       │
│  ≡  Audit Log       │
├─────────────────────┤
│  ⚙  Settings        │
│  [Avatar] User Name │  ← Reflects active role
└─────────────────────┘
```

---

## Sections & Items

### Main

| Item | Icon | Badge | Description |
|---|---|---|---|
| Dashboard | ▤ | — | Overview stats, campaign health, recent activity |
| Campaigns | ◈ | `8` (blue) | Campaign list and detail views |
| Vendors | ◎ | `4` (muted) | Vendor management — **Admin/CST only** |
| Lead Inbox | ↓ | `12` (orange) | Incoming leads pending review/acceptance |

### Operations

| Item | Icon | Badge | Description |
|---|---|---|---|
| Rules Engine | ⚙ | — | Lead qualification rules, slot counters, limits |
| Suppression Lists | ⊘ | — | Global and campaign-level suppression management |
| ABM Lists | ◧ | `3` (muted) | Target account lists with version history |
| Integrations | ⇄ | — | CRM, MAP, and data connector configuration |

### Reporting

| Item | Icon | Badge | Description |
|---|---|---|---|
| Analytics | ↗ | — | Performance dashboards and delivery metrics |
| Audit Log | ≡ | — | Full change history, version diffs, user actions |

### Footer

| Item | Description |
|---|---|
| Settings | Platform and account settings |
| User block | Displays avatar, name, and role label — updates dynamically when role is switched |

---

## Role-Based Visibility

| Nav Item | Admin | Vendor | CST |
|---|:---:|:---:|:---:|
| Dashboard | ✓ | ✓ | ✓ |
| Campaigns | ✓ | ✓ | ✓ |
| **Vendors** | ✓ | **✗** | ✓ |
| Lead Inbox | ✓ | ✓ | ✓ |
| Rules Engine | ✓ | ✓ | ✓ |
| Suppression Lists | ✓ | ✓ | ✓ |
| ABM Lists | ✓ | ✓ | ✓ |
| Integrations | ✓ | ✓ | ✓ |
| Analytics | ✓ | ✓ | ✓ |
| Audit Log | ✓ | ✓ | ✓ |
| Settings | ✓ | ✓ | ✓ |

**Vendors page is hidden from the Vendor role** — vendors must never know who else is working on the same campaign. This is enforced in `setRole()` in the JS layer by toggling `display:none` on the `[data-nav="vendors"]` sidebar item.

---

## Implementation Notes

### CSS classes

| Class | Purpose |
|---|---|
| `.sidebar` | Fixed 220px left panel, dark background |
| `.sidebar-brand` | Top brand/logo block |
| `.sidebar-nav-scroll` | Scrollable middle section (hides scrollbar) |
| `.sidebar-section` | Groups nav items under a labelled section |
| `.sidebar-section-label` | Uppercase 10px muted group header |
| `.sidebar-item` | Individual nav link row |
| `.sidebar-item.active` | Highlighted state — left blue accent bar |
| `.sidebar-item-icon` | Icon character span |
| `.sidebar-item-badge` | Count pill (blue default, orange `.warn`, muted `.muted`) |
| `.sidebar-footer` | Fixed-to-bottom settings + user block |
| `.sidebar-user` | Avatar + name + role row |
| `.sidebar-avatar` | Single-letter avatar circle |

### JavaScript functions

| Function | What it does |
|---|---|
| `sidebarNav(el)` | Removes `.active` from all items, adds it to clicked item |
| `setRole(role)` | Switches active role, updates user block, hides Vendors item for vendor role, calls `updateTabVisibility()` |
| `updateTabVisibility()` | Grays out top-tab items that are off-limits for the current role (tabs s4, s5 for vendor) |

### Data attribute

The Vendors sidebar item carries `data-nav="vendors"` so `setRole()` can target it precisely without relying on text content:

```html
<div class="sidebar-item" data-nav="vendors" onclick="sidebarNav(this)">
  <span class="sidebar-item-icon">◎</span>
  Vendors
  <span class="sidebar-item-badge muted">4</span>
</div>
```

```js
const vendorsNavItem = document.querySelector('.sidebar-item[data-nav="vendors"]');
if (vendorsNavItem) vendorsNavItem.style.display = (role === 'vendor') ? 'none' : '';
```

---

## Responsive Behaviour

The sidebar collapses (`display: none`) at viewports below ~768px (handled in the media query block at the bottom of the `<style>` section). On mobile, the top tab bar serves as the primary navigation.
