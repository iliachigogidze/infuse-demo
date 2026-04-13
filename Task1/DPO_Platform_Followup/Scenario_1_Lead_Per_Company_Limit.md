# Scenario 1 — Lead per Company Limit

**Status:** Rules defined. Admin flow added. Visual pending update.

---

## Campaign Initiation Parameters (reference)

From the campaign initiation document — the full set of fields that define a campaign at launch:

| Parameter | Example Value | Notes |
|---|---|---|
| Campaign name / ID | Rubrik — Deep Funnel Campaign - UKI - DU59067 | |
| Status | LIVE | |
| Goal | 147 (broken down by geo: UK 124, Ireland 23) | Lead delivery target |
| Deadline | 01/25/2024 | |
| Pacing | Even-pacing | |
| **Company Limit** | **2** | **Max leads per company — the field this scenario is about** |
| GEO | UK & Ireland | |
| Industry | any | |
| Revenue | any | |
| Company Size | any | |
| Target Titles/Roles | Include/exclude rules by job level, job area, keywords | Tiered: C-Level, Director+, Manager+, exact titles |
| **ABM list** | **Yes / No** | **Not all campaigns have one** |
| Suppression list | Yes / No | Pre-existing contacts to exclude |
| Language | English | |
| Qualification questions | Custom per campaign; some answers disqualify | |
| OPT-IN / GDPR | Compliant | |

---

## Problem

When multiple vendors work the same campaign, they can collide in two ways:

1. **They exceed the company limit** — each vendor stays within the cap individually, but in aggregate they deliver more leads from one company than the client allows.
2. **They contact the same person** — even without a company limit, two vendors calling the same prospect on the same campaign creates duplicate outreach and a poor experience for the client and the prospect.

Splitting vendors into non-overlapping scopes solves both problems. A company limit makes splitting more urgent (the cap can be broken), but separation is valuable even without one.

The constraint that makes this hard: **vendors must never know about each other.** Any solution that shares a live tracker, names another vendor, or signals that someone else is working the same company breaks the commercial model.

---

## Solution — Three Layers

### Layer 1 — Vendor Scope Split

The goal: give each vendor a non-overlapping scope so they don't collide on the same companies. **Vendors always know the company limit** (if one is set) — it's in their campaign brief, they can plan their outreach around it.

How we split depends on what the campaign gives us. We go from the most precise dimension to the least, and use the first one that works. See **Rules** below.

### Layer 2 — Per-Company Slot Counter (enforcement)

When a campaign has a company limit, every (campaign + company) pair gets a **remaining-slot counter.** This is the cap enforcement layer.

- When a lead is approved through QA, the counter decrements.
- When a company reaches 0 remaining slots, it is **closed** — no further submissions accepted.
- If a vendor attempts to submit against a closed company, the platform blocks it: *"This account is no longer available for this campaign."*
- Leads still in QA review are **reserved** (slot held but not yet consumed) so two vendors can't fill the same slot simultaneously.

In Rules 1–4, the counter is a safety net — the split already prevents most collisions. In Rule 5, the counter is the primary mechanism. In Rule 6, the counter alone isn't good enough, so we stop and fix the campaign setup.

### Layer 3 — Optional Pre-Check for Vendors

The platform offers vendors the ability to check whether a company is still available **before** reaching out — not just at submission time.

- The vendor can look up a company and see: "available (2 slots remaining)" or "unavailable."
- This is **optional** — vendors are not required to check. They can still submit first and find out at submission time if they prefer.
- The value is highest in Rule 5 (no split, counter only), where vendors source companies freely and collisions are possible. A vendor who checks first avoids wasting a call on a company that's already full.
- In Rules 1–4 (split), the pre-check is less critical because the vendor's scope is already exclusive — but it's still useful for seeing remaining slots before planning outreach at a specific company.

Over time, vendors who keep getting blocked at submission will naturally start using the pre-check.

---

## Rules

### Rule 1 — ABM list exists → split the list

The campaign's ABM list is divided into exclusive chunks. Each vendor gets their own set of companies. From their perspective, that slice *is* the campaign. Because each company is assigned to exactly one vendor at a time, collisions are prevented structurally.

**Recycling:** Companies don't stay locked forever. They return to a shared pool if the vendor goes idle (default: 5 business days) or hits their contracted quota (hard stop — all remaining companies release immediately). The next vendor who receives a recycled company sees it appear on their list with its actual remaining slots — no explanation, no vendor names.

**Suppression on reassignment:** When a company moves from one vendor to another, the receiving vendor gets a suppression list of already-contacted leads at that company (names/emails only, no vendor identifiers). Confirmed as permissible (see `questions.md`, Q1).

**Vendor POV:** You receive a list of 40 companies and your brief says "company limit: 2." That's your world. You can check any company's remaining slots before calling, or just submit and find out. A week later, 5 new companies appear on your list. One shows "1 slot available" instead of 2, and there's a note: "Skip these contacts." You don't ask questions, you just work the list. You never knew another vendor existed.

---

### Rule 2 — No ABM list, GEO has sub-regions → split by geography

If the campaign breaks down by region (e.g., UK vs Ireland, or city-level), assign each vendor a geographic territory. A company's HQ location determines which vendor owns it. Counter runs underneath.

**Vendor POV:** Your brief says "Your territory: Scotland. Company limit: 3." You go find companies headquartered in Scotland and start calling. You can check availability before reaching out, or just submit. You never see England or Wales. You think this is a Scotland-only campaign.

---

### Rule 3 — No ABM list, GEO is broad, multiple industries → split by industry

If the campaign targets multiple industries but GEO is too broad to split, assign each vendor one or more industry verticals. A company belongs to one industry, so no overlap. Counter runs underneath.

**Vendor POV:** Your brief says "Target Financial Services companies. Company limit: 2." You go find banks and insurers. Another vendor got Healthcare — from your chair, this was always a Financial Services campaign.

---

### Rule 4 — No ABM list, GEO broad, industry broad, company size range is wide → split by size band

If neither geo nor industry provides a clean split, but the campaign covers a wide company size range, assign each vendor a size band (e.g., enterprise 5000+, mid-market 500–5000). Employee count or revenue defines the boundary. Counter runs underneath.

**Vendor POV:** Your brief says "Target companies with 5,000+ employees. Company limit: 2." You go after large enterprises. Another vendor is working mid-market — you thought this was an enterprise campaign.

---

### Rules 1–4: common pattern

In all four rules, the vendor experience is the same: they have a clear scope, they know the cap, and they manage their own work. The dimension of the split is different (list, geo, industry, size), but from the vendor's chair it all feels identical — "here's my territory, here's my limit, go." The counter runs silently underneath as a safety net but rarely triggers because the split prevents collisions structurally.

---

### Rule 5 — No usable split dimension → counter only

When the campaign is fully broad — any geo, any industry, any size, no ABM list — there's nothing to split on. Vendors go find companies on their own. The slot counter is the only enforcement mechanism.

**Vendor POV:** Your brief says "Any geo, any industry, any size. Company limit: 3." You check Siemens on the platform — available, 3 slots. You call, qualify, submit — goes through. You submit a second — goes through. You check again before your third — "unavailable." Someone else took the last slot while you were working. But you found out *before* making the call, not after. You pick a different company.

**If the vendor doesn't pre-check:** They call, qualify, submit — platform says *"This account is no longer available for this campaign."* They did the work for nothing. This is why pre-check is most valuable in Rule 5.

**Why splitting matters:** In Rules 1–4, collisions don't happen — the vendor has exclusive scope. In Rule 5, the lower the cap and the more vendors on the campaign, the more collisions occur. Pre-check reduces wasted effort but doesn't eliminate it entirely (two vendors can check at the same time, both see "available," both start calling).

---

### Rule 6 — Low cap + no split possible → flag at campaign setup

If the company limit is low (1–2), no ABM list exists, and no campaign dimension provides a clean split, the collision risk is too high for Rule 5 — even with pre-check, a cap of 1 means every collision is 100% wasted work. The operations team catches this before launch and flags it: *"This campaign needs an ABM list or a tighter targeting split before we can go live."*

The campaign doesn't start until the split is sorted out. Vendors never experience this scenario — it's resolved at setup.

**Threshold:** Rule 6 triggers when company limit ≤ 2 and no split dimension is available. For company limit ≥ 3 with few vendors, Rule 5 is acceptable.

---

## DPO Admin Flow

This is the process for the DPO admin who receives a campaign and sets it up for vendors.

### Step 1 — Review campaign parameters

Open the campaign initiation document. Note: company limit (if any), ABM list (yes/no), GEO, industry, company size, number of vendors assigned.

### Step 2 — Determine the split

Walk through the rules in order. Use the first one where the condition is met:

**Check Rule 1:** Is there an ABM list?
→ Yes: Divide the list into exclusive vendor chunks. The split can be even or weighted by vendor capacity or geo sub-goals. Load each vendor's chunk into the platform so they only see their assigned companies. Move to Step 3.

**Check Rule 2:** No ABM list. Does GEO have sub-regions?
→ Look at the GEO field and sub-goals. Can the geography be broken into non-overlapping territories with enough companies per territory? Example: campaign says "UK & Ireland" with sub-goals UK 124, Ireland 23 — give Ireland to one vendor, split UK by region between the other two. Define territories, move to Step 3.

**Check Rule 3:** GEO too broad. Does the campaign target multiple industries?
→ Does the industry field list specific sectors, or does the campaign's focus imply distinct verticals? If yes, assign each vendor one or more verticals. Move to Step 3.

**Check Rule 4:** Industry is broad. Is the company size range wide enough to split?
→ If the campaign spans from mid-market to enterprise, define size bands (e.g., 5000+, 1000–5000, 500–1000). Assign each vendor a band. Move to Step 3.

**Check Rule 5:** Nothing to split on. Is the company limit ≥ 3?
→ Accept the collision risk. Assign vendors to the campaign without splitting. The counter is the only enforcement. Move to Step 3.

**Rule 6:** Company limit ≤ 2 and nothing to split on.
→ **Stop.** Flag to the campaign manager: *"This campaign has a low company limit and no way to separate vendor scope. We need an ABM list or tighter targeting criteria before we can launch."* The campaign is held until resolved.

### Step 3 — Configure the platform

- Set the company limit on the campaign (if applicable) so the slot counter is active.
- If Rule 1: load vendor-specific company lists.
- If Rules 2–4: configure vendor scope restrictions (territory, vertical, size band).
- If Rule 5: no scope restrictions — vendors work freely.
- Enable vendor pre-check so vendors can look up company availability before outreach.

### Step 4 — Brief the vendors

Each vendor gets a campaign brief that includes:

- Their targeting scope (their company list, their geo territory, their industry vertical, their size band, or "open" for Rule 5).
- The company limit (always disclosed).
- The suppression list if one exists.
- Qualification questions and other campaign requirements.

**The vendor never sees the full campaign picture.** They see only their slice. The brief is written so that from the vendor's perspective, their slice *is* the campaign.

### Step 5 — Monitor during the campaign

**For all rules:**
- Monitor the slot counter dashboard — see which companies are approaching the cap, which are closed.
- Respond to vendor questions. If a vendor asks "why is this company unavailable?", the answer is always neutral: *"This company has reached its campaign limit."* Never mention other vendors.

**For Rules 1–4 (split):**
- Monitor vendor activity. If a vendor goes idle on assigned companies (5 business days, no activity), the platform auto-releases those companies to the pool.
- Review auto-released companies. Either let the platform auto-assign them to another vendor, or manually decide who gets them.
- When a company is reassigned, ensure the suppression list (already-contacted leads) is attached before the new vendor starts working it.
- If a vendor hits their quota, confirm the redistribution of their remaining companies.

**For Rule 5 (counter only):**
- Watch for high collision rates. If too many vendors are hitting blocked submissions, consider whether a retroactive split is possible mid-campaign (e.g., "from now on, Vendor A focuses on Financial Services, Vendor B on Healthcare").
- Check whether pre-check adoption is reducing wasted effort.

---

## Operating Rules

| Rule | Decision |
|---|---|
| Company limit visibility | **Vendors always know the company limit.** It's in the campaign brief. |
| Pre-check | **Optional.** Vendors can check company availability before outreach, but are not required to. |
| What vendors see during campaign | **Remaining slots per company** (live number). |
| When a slot is consumed | On **QA approval**. Leads still in review are reserved. |
| Recycled company capacity | Carries its **actual remaining slots**, not a reset to the original cap. |
| Rejected lead | Slot returns to the **same vendor** for retry. Exception: quality rejections (fabricated data) → slot goes to pool. |
| Lock duration (Rules 1–4) | **5 business days** idle before auto-release. Configurable per campaign. |
| Quota hit (Rules 1–4) | **Hard stop.** Campaign closes for that vendor. All remaining companies release to pool. No overflow. |
| Company matching | **Normalized domain** (primary) + **fuzzy name match** (fallback). Free-text names never trusted alone. |
| Suppression on reassignment | Receiving vendor gets contact-level suppression list. No vendor identifiers disclosed. |

---

## Company Identity Resolution

To prevent "Acme Corp" and "Acme, Inc." from being treated as two separate companies, matching uses **normalized domain** (e.g., acme.com) as the primary key, with fuzzy legal-name matching as a fallback. Free-text names are never trusted alone.

---

## Resolved Decisions

These were surfaced as corner cases during design and have been locked in.

| Decision | Resolution | Rationale |
|---|---|---|
| When does a cap slot count? | On **QA approval** | Prevents double-booking while leads are in review. Leads under review are reserved. |
| Rejected lead — where does the freed slot go? | Back to the **same vendor** for retry. Exception: fabricated data → slot goes to pool. | Gives the vendor a fair chance to fix legitimate rejections. Bad-faith submissions shouldn't get a second chance from the same vendor. |
| How long before auto-release? | **5 business days** idle (configurable per campaign). | Long enough for normal vendor workflow, short enough to avoid dead inventory. |
| Quota hit — auto-release all remaining? | **Yes, hard stop.** All remaining companies release to pool immediately. No overflow/bonus option. | Keeps vendor scope clean. Overflow creates ambiguity around contractual targets. |
| Do vendors know the company limit? | **Yes, always.** It's in the campaign brief for all rules. | Helps vendors plan outreach. No confidentiality risk — knowing the cap doesn't reveal other vendors. |
| Is vendor pre-check mandatory? | **No, optional.** Platform offers it; vendors choose whether to use it. | Avoids forcing a workflow change. Vendors who get blocked frequently will adopt it naturally. |

---

## Items to Flag During Review

1. **"Remaining slots" visibility** — Vendors see the live remaining-slot count. In Rules 1–4 (split), the count only changes from the vendor's own submissions, so no information leaks. In Rule 5 (counter only), a vendor who submitted 2 of 3 and sees "0 remaining" can infer someone else submitted — this is an acceptable trade-off since Rule 5 is already the fallback for broad campaigns.

2. **Suppression list scope** — Confirmed that contact-level suppression is permissible. The list contains names/emails only, no activity timestamps or vendor identifiers. Whether to include "reason for suppression" (already contacted vs. already converted) is a separate decision.

3. **Rule 6 threshold** — Proposed at company limit ≤ 2 with no split. This is a starting point — may need adjustment based on how many vendors are typically assigned to broad campaigns.

4. **Splitting even without a company limit** — The rules apply even when no company limit is set. Separation prevents duplicate outreach to the same person. If no company limit exists and no split is possible, the risk shifts from cap breach to wasted effort and poor prospect experience.

---

## Open Items (require input from the platform team)

1. **How does the platform identify companies today?** If the current system relies on free-text names, the identity-matching layer (normalized domain + fuzzy name) is a prerequisite and should be scoped as its own work item.

2. **Which campaign dimensions are reliably available for splitting?** Rules 2–4 depend on GEO, industry, and company size being granular enough to create non-overlapping vendor scopes. Need to confirm how often these dimensions are usable in practice vs. set to "any."

3. **Can the platform support a vendor pre-check?** The pre-check requires a real-time query against the slot counter. Need to confirm the platform can expose this to vendors without performance issues or confidentiality leaks.
