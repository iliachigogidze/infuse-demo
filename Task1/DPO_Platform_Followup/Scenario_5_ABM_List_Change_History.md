# Scenario 5 — ABM List Change History

**Status:** ✅ Direction confirmed by the team.

**Confirmed:** One account = one company. An ABM list is a list of **companies** with metadata (name, domain, industry, size, etc.). When the client sends a new version, they are changing which *companies* are targets — not which people inside them.

**Confirmed:** This is the **account-list version of Scenario 3**. In Scenario 3 the client changes the qualification criteria (who counts as a valid lead). Here the client changes the ABM list (which companies to target). The core mechanism is the same — immutable versioning, tagging work to the version it belongs to, and deterministic rules for what happens to in-flight work at the boundary. The difference is what gets versioned (rules there vs. account list here), how long old and new run side by side (24 hours there vs. ongoing here), and who is affected (leads already in the pipeline there vs. companies already being worked here).

## Problem in one sentence

When a client sends an updated ABM list, the Lead Delivery team has no way to see what changed, protect work already done, or handle dropped companies — so things get re-processed, effort gets wasted, and nobody knows what the list looked like last month.

## Problem

A client sends "Target Accounts v1" with 500 companies. Work starts — vendors begin contacting people at those companies, leads come in. A month later the client sends "v2" with 800 companies. Some are brand new. Some were already in v1 and have been worked or delivered. Some v1 companies are gone entirely.

Today, the Lead Delivery team has to manually compare the two files to figure out what changed. The old file gets overwritten or saved somewhere with a date in the filename. The comparison happens in Excel — side by side, vlookup, eyeballing differences. Three things go wrong:

1. **Companies already worked get re-assigned to vendors.** Company X was in v1, a vendor already delivered a lead from it. v2 arrives with Company X still on the list. Nobody catches that it's already done, so it goes back into the work queue. The vendor wastes time researching a company that's already delivered.

2. **Companies the client removed stay in the work queue.** Company Y was in v1 but the client dropped it from v2. Nobody caught the deletion because the comparison was done manually. A vendor is still calling into Company Y a week later — effort wasted, and the client didn't even want that company anymore.

3. **Nobody can answer "what did the target list look like on March 1st?"** The old file was overwritten. When the client disputes what was agreed, there's no record. When ops needs to audit what was worked under which version, they have to reconstruct it from emails and file dates.

The manual process doesn't scale. A 500-company list is manageable. A 2,000-company list with monthly updates and three active campaigns per client — that's where things break.

---

## Solution

The solution stacks five layers. Each one does something the others don't — they work together.

### 1. Domain-First Account Identity (how companies are matched)

Before any versioning logic runs, every incoming company is normalized and keyed on **company domain** (e.g., acme.com). Name variations, capitalization, whitespace, legal suffixes ("Inc.", "Ltd.") no longer cause the same company to appear as a "new" account. Domain is the primary key; company name is a secondary display field.

> **Example:** v1 has "Acme Corporation" with domain acme.com. v2 has "Acme, Inc." with domain acme.com. The platform recognizes them as the same company. No false "new account" created.

### 2. Versioned ABM List with Auto-Diff (how changes are detected)

Every upload creates a **new immutable version** — the old one is never overwritten. The platform compares the new version against the previous one and classifies every company:

- **Added** — in v2 but not in v1.
- **Removed** — was in v1, not in v2.
- **Unchanged** — in both.

The Lead Delivery member sees this breakdown before committing the new version. Nothing changes in the work queue until they confirm.

> **Example:** Client uploads v2 (800 companies). Platform shows: *"300 added · 50 removed · 450 unchanged. Review and commit?"* LD reviews the list, confirms → v2 goes live.

### 3. Status Flags on Every Company (how work history is preserved)

Each company carries a **work status**: *new*, *in-progress*, *worked*, or *delivered*. These flags drive the daily workflow — the Lead Delivery team filters by status instead of re-reading the raw list every time.

> **Example:** Company X was in v1 and has status *delivered* (lead already sent to client). v2 arrives and Company X is still on the list. Platform sees: unchanged + already delivered → **no action**. Company X is not re-queued.

### 4. Delta-Only Processing (how re-work is prevented)

When a new version is committed, the platform only touches the differences:

- **New companies** → enter the work queue.
- **Unchanged companies** → keep their current status. Already-worked companies are **never re-processed**.
- **Removed companies** → handled by the rules in step 5.

This is the structural guarantee that uploading v2 cannot accidentally undo v1 work.

### 5. Removed-Company Handling Rules (how dropped companies are managed)

When v2 drops a company that was in v1, the platform applies a fixed rule based on its current status:

| Company status | What happens |
|---|---|
| **Delivered** | No action — lead already belongs to the client. |
| **In progress** | Flagged for Lead Delivery review. LD decides: halt now or give the vendor a short grace window (default 5 business days) to finish. |
| **Not started** | Quietly removed from the work queue. No effort wasted. |

Every decision is logged against the company's history.

> **Example:** v2 drops Company Y, which a vendor is actively working. Platform flags it: *"Company Y removed by client in v2 — vendor in progress. Halt or allow 5-day grace?"* LD chooses grace window → vendor finishes → lead delivered → Company Y marked *delivered + removed*.

**Result.** The ABM list becomes a versioned, auditable entity. Every change is tracked automatically. Already-worked companies are never touched again. Dropped companies are handled deterministically instead of case-by-case. Ops always knows what the list looked like at any point in time.

---

## Operating Rules

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

## Process Flow

**Step 1 — Client Sends Updated List.** The client emails a new version of the ABM list to the Client Success Team or uploads it directly. The platform stores the file as a new version (e.g., v2) without overwriting the previous one.

**Step 2 — Domain Normalization.** The platform normalizes every company in the new file by domain. "Acme Corporation" and "Acme, Inc." both resolve to acme.com → same company. Companies with no domain or an ambiguous domain are flagged for manual review.

**Step 3 — Auto-Diff.** The platform compares v2 against v1, company by company. It produces a summary: *"300 added · 50 removed · 450 unchanged."* Each company is classified into one of those three buckets.

**Step 4 — Lead Delivery Reviews and Commits.** The Lead Delivery member sees the diff breakdown. They review the added and removed lists, verify nothing looks off (e.g., a key account accidentally dropped), and hit confirm. Until they confirm, **nothing changes in the work queue.**

**Step 5 — Delta Processing Kicks In.** The platform acts only on the differences:
- The 300 **added** companies enter the work queue with status *new*.
- The 450 **unchanged** companies keep their current status. Companies already *delivered* or *worked* are not touched.
- The 50 **removed** companies are handled by status: *delivered* → no action; *in-progress* → flagged for LD review with grace window option; *not started* → quietly removed from the queue.

**Step 6 — Work Continues on the Updated List.** Vendors and teams now work from the updated list. Their filters show only actionable companies — new ones to pick up, in-progress ones to continue. Already-delivered companies don't appear in the work queue.

**Step 7 — Audit and Reporting.** At any point, ops can pull the history: what the list looked like at any version, which companies were added or removed when, what work was done under each version, and what decisions were made about removed companies. The full version history is always available.

---

## Example Walkthrough

*Setup.* Client "TechCorp" runs Campaign "Cloud Adoption Leaders." They send their first ABM list (v1) on March 1st — 500 companies. Three vendors start working the list.

1. **March 1 — v1 uploaded and committed.** 500 companies enter the platform. All normalized by domain. All set to status *new*. Vendors begin contacting.

2. **March 15 — Mid-month progress.** Vendors have been working for two weeks. Current status across the 500 companies: 40 *delivered* (leads sent to client), 80 *in-progress* (vendors actively working), 120 *worked* (contacted but no lead), 260 *new* (not yet picked up).

3. **April 1 — Client sends v2.** Client emails a new file: 800 companies. LD uploads it. The platform normalizes domains and runs the auto-diff against v1:
   - **300 added** — brand new companies not in v1.
   - **50 removed** — were in v1, not in v2.
   - **450 unchanged** — present in both files.
   
   LD sees the summary: *"300 added · 50 removed · 450 unchanged. Review and commit?"*

4. **April 1 — LD reviews the diff.** LD scans the removed list. They notice Company Y (a large enterprise account) was dropped. They check — a vendor is actively working Company Y (status: *in-progress*). LD confirms the diff.

5. **April 1 — Platform processes the delta.**
   - The **300 new companies** enter the work queue with status *new*. Vendors can start picking them up.
   - The **450 unchanged companies** keep their current status. The 40 already *delivered* are not re-queued. The 80 *in-progress* continue as-is. The 120 *worked* stay closed. The 210 still *new* remain in the queue.
   - The **50 removed companies** are handled by rule:
     - 5 were *delivered* → no action, work is done.
     - 3 were *in-progress* (including Company Y) → flagged. LD grants a 5-day grace window. Vendors have until April 8 to finish.
     - 42 were *not started* → quietly removed from the queue. No effort wasted.

6. **April 8 — Grace window closes.** Of the 3 in-progress companies that were removed: Vendor finishes Company Y and delivers a lead (status: *delivered + removed*). The other 2 are halted — vendors stop work, companies marked *removed*.

7. **May 1 — Client sends v3.** 900 companies. Platform diffs against v2. Company Z, which was dropped in v2, re-appears in v3. Platform checks: Company Z was *not started* when removed — never delivered. It's treated as a new opportunity and enters the work queue. Meanwhile, Company Y (delivered in v1, removed in v2) also re-appears — platform sees status *delivered*, skips it.

8. **Audit request.** Client asks: "What companies were you working on March 15th?" LD pulls up v1, filters by status as of that date. Full answer in seconds — no digging through emails or old spreadsheets.

---

## Corner Cases

1. **Domain missing or ambiguous** — fallback to fuzzy match on normalized company name + country; flagged for manual review if confidence is low.
2. **Company owns multiple domains** — maintain a small alias table so both domains resolve to the same account.
3. **Company re-appears after being removed** — treated as a new opportunity only if not previously delivered; otherwise skipped.
4. **Metadata changed across versions** (e.g., company size went from 500 to 600) — latest values used for filtering, but historical values preserved on the company record.
5. **Very frequent uploads** — LD can commit only the latest version rather than each intermediate one. All uploaded files retained for audit.

---

## Options Considered (Summary)

1. **Domain-First Account Identity** ✅ *Chosen — foundation for matching.*
2. **Versioned ABM List with Auto-Diff** ✅ *Chosen — versioning mechanism.*
3. **Status Flags on Every Account** ✅ *Chosen — workflow layer.*
4. **Delta-Only Processing** ✅ *Chosen — efficiency guarantee.*
5. **Removed-Account Handling Rules** ✅ *Chosen — deterministic rulebook.*
6. **Master Account Registry per Client** — Powerful phase-2 extension; held for future scope.
7. **Client-Facing Upload Portal with Preview** — Strong improvement candidate; proposed as a phase-2 enhancement.

**Final design = Domain Identity (matching) + Versioned List with Auto-Diff (versioning) + Status Flags (workflow) + Delta-Only Processing (efficiency) + Removed-Account Rules (handling).**

---

## Remaining Internal Decisions

1. **Does the platform already store uploaded client files with version history today?** If not, versioned storage needs to be scoped as its own work item.
2. **Does domain normalization carry over from Scenario 1?** To confirm with the platform team whether this is already built or needs to be added.
3. **Should the client see the diff before it's committed?** Could be a future portal feature — for now, LD reviews and commits internally.
