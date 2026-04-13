# Design System — INFUSE DPO Platform

A single source of truth for all visual decisions. Every HTML file in this project must use these tokens and patterns.

---

## Brand Identity

INFUSE is a B2B demand generation company. The visual language should feel like a modern ops dashboard — precise, clean, and confident. Inspiration: Airtable, Linear, Notion. No decoration for decoration's sake. Every element carries information.

---

## Color Tokens

### Brand Core
| Token | Hex | Usage |
|---|---|---|
| `--brand-primary` | `#0052CC` | Primary actions, active states, key accents |
| `--brand-secondary` | `#00A3BF` | Supporting accents, charts, secondary highlights |
| `--brand-dark` | `#0A1628` | Page headers, hero backgrounds |

> INFUSE brand leans into deep navy + electric blue — professional, tech-forward, B2B.

### Neutrals
| Token | Hex | Usage |
|---|---|---|
| `--neutral-900` | `#0A1628` | Primary text, headings |
| `--neutral-700` | `#2D3748` | Body text |
| `--neutral-500` | `#64748B` | Secondary text, labels, metadata |
| `--neutral-300` | `#CBD5E1` | Borders, dividers |
| `--neutral-100` | `#F1F5F9` | Page background |
| `--neutral-0` | `#FFFFFF` | Card / surface background |

### Semantic Colors
| Token | Hex | Usage |
|---|---|---|
| `--green` | `#059669` | Success, approved, resolved |
| `--yellow` | `#D97706` | Warning, in-progress, pending |
| `--red` | `#DC2626` | Error, blocked, problem |
| `--purple` | `#7C3AED` | Process, logic, system |
| `--teal` | `#0891B2` | Data, tracking, suppression |

### Scenario Accent Colors
Each scenario gets a consistent accent used for its badge, card border, and step numbers.

| Scenario | Token | Hex |
|---|---|---|
| S1 — Lead Limit | `--s1` | `#0052CC` |
| S2 — Duplicates | `--s2` | `#059669` |
| S3 — Req. Change | `--s3` | `#D97706` |
| S4 — Cross-Channel | `--s4` | `#7C3AED` |
| S5 — ABM History | `--s5` | `#0891B2` |

---

## Typography

One font stack across everything. No web font imports — system fonts only for reliability and speed.

```css
font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', 'Inter', Helvetica, Arial, sans-serif;
```

### Scale
| Role | Size | Weight | Color |
|---|---|---|---|
| Page title | `24px` | `700` | `--neutral-900` |
| Section heading | `18px` | `700` | `--neutral-900` |
| Card heading | `15px` | `700` | `--neutral-900` |
| Body | `14px` | `400` | `--neutral-700` |
| Label / overline | `12px` | `700` | `--neutral-500` |
| Caption / meta | `12px` | `400` | `--neutral-500` |

### Overline labels
Section headers use ALL CAPS with letter-spacing. This is the only case for uppercase text.

```css
font-size: 12px;
font-weight: 700;
text-transform: uppercase;
letter-spacing: 0.8px;
color: var(--neutral-500);
```

---

## Spacing

Base unit: `4px`. All spacing is a multiple of 4.

| Token | Value | Usage |
|---|---|---|
| `--space-1` | `4px` | Icon gaps, tight internal padding |
| `--space-2` | `8px` | Inline element gaps |
| `--space-3` | `12px` | List item gaps |
| `--space-4` | `16px` | Card internal padding (compact) |
| `--space-5` | `20px` | Default element margin |
| `--space-6` | `24px` | Card padding standard |
| `--space-8` | `32px` | Section spacing |
| `--space-12` | `48px` | Major section breaks |
| `--space-16` | `64px` | Page bottom padding |

---

## Surfaces & Elevation

Two levels of elevation. No heavy drop shadows — keep it flat-first.

### Page background
```css
background: #F1F5F9; /* --neutral-100 */
```

### Card (default surface)
```css
background: #FFFFFF;
border-radius: 12px;
box-shadow: 0 1px 3px rgba(0, 0, 0, 0.06);
```

### Card with scenario accent border
```css
border-left: 4px solid var(--sX); /* scenario color */
```

### Header / hero band
```css
background: #0A1628; /* --brand-dark */
color: white;
```

---

## Components

### Card
The primary content container. All scenario content lives in cards.

```html
<div class="card">
  <div class="card-label">OVERLINE LABEL</div>
  <h2 class="card-title">Card Title</h2>
  <p class="card-body">Body text here.</p>
</div>
```

Variants via modifier classes:
- `.card--problem` → `border-left-color: var(--red)`
- `.card--solution` → `border-left-color: var(--brand-primary)`
- `.card--s1` through `.card--s5` → scenario accent borders

### Badge / Tag
Inline pill for labels, scenario numbers, statuses.

```css
display: inline-block;
padding: 2px 10px;
border-radius: 20px;
font-size: 11px;
font-weight: 700;
text-transform: uppercase;
letter-spacing: 0.4px;
```

Use `background` at 12% opacity of the accent color with matching text color for soft badges.

### Step / Flow Node
Used in decision flows and process steps.

```css
/* Circle step number */
width: 32px;
height: 32px;
border-radius: 50%;
background: var(--sX);
color: white;
font-size: 13px;
font-weight: 700;
```

Connector line between steps:
```css
position: absolute;
left: 23px;
top: 48px;
width: 2px;
background: var(--neutral-300);
```

### Tab Navigation
Used for top-level scenario switching.

```css
/* Tab bar */
border-bottom: 1px solid rgba(255,255,255,0.1);

/* Tab item */
padding: 12px 20px;
font-size: 13px;
font-weight: 600;
color: rgba(255,255,255,0.5);
border-bottom: 3px solid transparent;

/* Active tab */
color: white;
border-bottom-color: var(--brand-primary);
```

### Section Divider
```css
font-size: 12px;
font-weight: 700;
text-transform: uppercase;
letter-spacing: 0.8px;
color: var(--neutral-500);
padding-bottom: 8px;
border-bottom: 1px solid var(--neutral-300);
margin: 32px 0 16px;
```

### Status Dot
Small inline indicator for state.

```css
display: inline-block;
width: 8px;
height: 8px;
border-radius: 50%;
/* Colors: --green, --yellow, --red per state */
```

### Data Grid Row
For side-by-side comparisons (before/after, vendor scopes, etc.).

```css
display: grid;
grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
gap: 16px;
```

---

## Layout

### Max content width
`1060px`, centered, with `24px` horizontal padding.

```css
max-width: 1060px;
margin: 0 auto;
padding: 0 24px;
```

### Page structure
```
[Header band — dark navy]
  Logo/title + subtitle
  [Tab bar]
[Content area — light gray bg]
  [Panel per scenario]
    Cards, grids, flow diagrams
```

---

## Motion

Minimal. Only transitions on interactive state changes.

```css
/* Tab / button hover */
transition: color 0.15s ease, border-color 0.15s ease, background 0.15s ease;

/* Panel switch — no animation, instant */
display: none / display: block;
```

No keyframe animations. No scroll animations. No loading spinners.

---

## CSS Custom Properties Boilerplate

Paste at the top of every `<style>` block:

```css
:root {
  /* Brand */
  --brand-primary: #0052CC;
  --brand-secondary: #00A3BF;
  --brand-dark: #0A1628;

  /* Neutrals */
  --neutral-900: #0A1628;
  --neutral-700: #2D3748;
  --neutral-500: #64748B;
  --neutral-300: #CBD5E1;
  --neutral-100: #F1F5F9;
  --neutral-0: #FFFFFF;

  /* Semantic */
  --green: #059669;
  --yellow: #D97706;
  --red: #DC2626;
  --purple: #7C3AED;
  --teal: #0891B2;

  /* Scenarios */
  --s1: #0052CC;
  --s2: #059669;
  --s3: #D97706;
  --s4: #7C3AED;
  --s5: #0891B2;

  /* Spacing */
  --space-1: 4px;
  --space-2: 8px;
  --space-3: 12px;
  --space-4: 16px;
  --space-5: 20px;
  --space-6: 24px;
  --space-8: 32px;
  --space-12: 48px;
  --space-16: 64px;
}
```

---

## Do / Don't

| Do | Don't |
|---|---|
| Use system font stack | Import Google Fonts |
| Use CSS custom properties | Hardcode hex values inline |
| Keep card borders to left-only accents | Use full colored card backgrounds |
| Use overline labels for section headers | Use `<h4>` or smaller for section headers |
| Keep shadows to `0 1px 3px rgba(0,0,0,0.06)` | Add heavy or layered box-shadows |
| Stick to the 4px spacing grid | Use arbitrary pixel values like `7px`, `13px` |
| Use semantic colors for status states | Use brand blue for error states |
| Tab navigation in the dark header band | Floating or sidebar navigation |
