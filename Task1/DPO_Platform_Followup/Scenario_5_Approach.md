# Scenario 5 — ABM List Change History
### Proposed Approach

---

## Problem in one sentence

When a client sends an updated ABM list, the Lead Delivery team has no way to see what changed, protect work already done, or handle dropped companies — so things get re-processed, effort gets wasted, and nobody knows what the list looked like last month.

---

## Problem

A client sends "Target Accounts v1" with 500 companies. Work starts — vendors begin contacting, leads come in. A month later the client sends "v2" with 800 companies. Some are brand new. Some were already in v1 and have been worked or delivered. Some v1 companies are gone entirely.

Today, the Lead Delivery team has to manually compare the two files to figure out what changed. That means:

- Companies already worked get re-assigned to vendors. Effort is wasted.
- Companies the client removed stay in the work queue because nobody caught the deletion.
- Nobody can answer "what did the target list look like on March 1st?" — the old file was overwritten.

One account = one company. The ABM list is a list of **companies** (not people), and the goal is to make every version trackable, every change visible, and every past decision auditable.

---

## Relationship to Scenario 3

Scenarios 3 and 5 are two versions of the same underlying problem: **the client changes something mid-campaign, and the platform needs to protect work already done while applying the new reality going forward.** In Scenario 3 the client changes the **qualification criteria** (e.g., target titles, company size, GEO). Here the client changes the **ABM list** (which companies to target). The core mechanism is the same — immutable versioning, tagging work to the version it belongs to, and deterministic rules for what happens to in-flight work at the boundary. The difference is what gets versioned (rules there vs. account list here), how long old and new run side by side (24 hours there vs. ongoing here), and who is affected (leads already in the pipeline there vs. companies already being worked here).

---

## How it works

### 1. Identify companies by domain

Before anything else, every company is matched on **normalized domain** (e.g., acme.com), not company name. This prevents "Acme Corp" and "Acme, Inc." from showing up as two different companies in the diff.

> **Example:** v1 has "Acme Corporation" with domain acme.com. v2 has "Acme, Inc." with domain acme.com. The platform recognizes them as the same company. No false "new account" created.

### 2. Save every version, compute the diff automatically

Every upload creates a **new version** — the old one is never overwritten. The platform compares the new version against the previous one and classifies every company:

- **Added** — in v2 but not in v1.
- **Removed** — was in v1, not in v2.
- **Unchanged** — in both.

The Lead Delivery member sees this breakdown before committing the new version. Nothing changes in the work queue until they confirm.

> **Example:** Client uploads v2 (800 companies).
> Platform shows: *300 added · 50 removed · 450 unchanged. Review and commit?*
> LD reviews the list, confirms → v2 goes live.

### 3. Track status on every company

Each company carries a **work status**: *new*, *in-progress*, *worked*, or *delivered*. These flags drive the daily workflow — the Lead Delivery team filters by status instead of re-reading the raw list every time.

> **Example:** Company X was in v1 and has status *delivered* (lead already sent to client).
> v2 arrives and Company X is still on the list.
> Platform sees: unchanged + already delivered → **no action**. Company X is not re-queued.

### 4. Only act on what changed

When a new version is committed, the platform only touches the differences:

- **New companies** → enter the work queue.
- **Unchanged companies** → keep their current status. Already-worked companies are **never re-processed**.
- **Removed companies** → handled by the rules in step 5.

This is the structural guarantee that uploading v2 cannot accidentally undo v1 work.

### 5. Handle removed companies by rule

When v2 drops a company that was in v1, the platform applies a fixed rule based on its current status:

| Company status | What happens |
|---|---|
| **Delivered** | No action — lead already belongs to the client. |
| **In progress** | Flagged for Lead Delivery review. LD decides: halt now or give the vendor a short grace window (default 5 business days) to finish. |
| **Not started** | Quietly removed from the work queue. No effort wasted. |

Every decision is logged against the company's history.

> **Example:** v2 drops Company Y, which a vendor is actively working.
> Platform flags it: *"Company Y removed by client in v2 — vendor in progress. Halt or allow 5-day grace?"*
> LD chooses grace window → vendor finishes → lead delivered → Company Y marked *delivered + removed*.

---

## Operating rules

| Rule | Detail |
|---|---|
| Company matching | **Normalized domain** (primary). Alias table for companies with multiple domains. Company name is display-only. |
| Version storage | Every upload is a new version. Previous versions are **never overwritten**. Rollback = commit an older version as the new current. |
| Commit gate | Diff must be **reviewed and confirmed by the Lead Delivery member** before the new version goes live. No silent commits. |
| Already-worked protection | Companies with status *worked* or *delivered* are **never re-queued**, even if they appear in a later version. |
| Grace window for in-progress removals | **5 business days** default, configurable per client. |
| Company re-appears after removal | If Company Z was dropped in v2 but re-appears in v3 — treated as a new opportunity **only if not previously delivered**. If already delivered, skipped. |
| Metadata changes (e.g., company size) | Latest values used for filtering. Historical values preserved on the company record. |
| Frequent uploads | LD can commit only the latest version rather than every intermediate one. All uploaded files retained for audit. |

---

## Open item

**Does the platform already store uploaded client files with version history today?** If not, versioned storage needs to be scoped as its own work item. Domain normalization is assumed to carry over from Scenario 1 — to confirm with the platform team.
