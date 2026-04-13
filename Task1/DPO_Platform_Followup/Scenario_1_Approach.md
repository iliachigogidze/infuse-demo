# Scenario 1 — Lead per Company Limit
### Proposed Approach

---

## Problem in one sentence

Multiple vendors unknowingly exceed the per-company lead cap because the system has no way to enforce it across vendors who can't see each other.

---

## Problem

A client launches a campaign targeting 100 companies and sets a cap: no more than 2 leads per company. Three vendors are assigned to the campaign. Each vendor gets the same instruction — "deliver leads, max 2 per company" — and each one follows it.

The result: Vendor A submits 2 leads from Acme, Vendor B submits 2 more, Vendor C adds another 1. The client receives 5 leads from a single company they capped at 2. The cap was never broken by any one vendor — it was broken by the system that couldn't see across all of them.

Making this harder: **vendors must never know about each other.** We can't solve this by sharing a live spreadsheet or telling vendors "someone already submitted for Acme." Any solution that reveals multi-vendor participation breaks the commercial model.

---

## How it works

### 1. Split the list — one vendor, one slice

The campaign's ABM list is divided into **exclusive chunks**. Each vendor gets their own set of companies and works only those. From their perspective, that slice *is* the campaign.

> **Example:** ABM list has 100 companies, 3 vendors.
> Vendor A → companies 1–40 · Vendor B → companies 41–75 · Vendor C → companies 76–100.
> No vendor sees the others' companies. No vendor knows the others exist.

Because each company is assigned to exactly one vendor at a time, **the cap cannot be broken** — only one vendor is working each company.

### 2. Recycle unused companies back to the pool

Companies don't stay locked forever. They return to the shared pool in two cases:

- **Vendor goes idle** — a company isn't touched for 5 business days → lock expires, company goes back to the pool.
- **Vendor hits their contracted quota** — campaign closes for that vendor → all their remaining companies go back to the pool immediately.

The next vendor who receives a recycled company just sees it appear on their list. No explanation, no vendor names disclosed.

### 3. Track remaining slots per company (the safety net)

Behind the scenes, the platform keeps a **remaining-slot counter** for every (campaign + company) pair.

> **Example continued:** Vendor A submits 1 approved lead from Company X (cap = 2).
> Counter: Company X → 1 slot remaining.
> Vendor A hits quota → Company X returns to pool → Vendor B picks it up.
> Vendor B sees: "Company X — 1 slot available." Not 2. The counter carries the real number.

If a company reaches 0 remaining slots, it is **closed** and never returns to the pool. If any edge case lets a vendor try to submit against a closed company, the platform blocks it with: *"This account is no longer available for this campaign."*

### 4. Identify companies reliably

To prevent "Acme Corp" and "Acme, Inc." from being treated as two separate companies, matching uses **normalized domain** (e.g., acme.com) as the primary key, with fuzzy legal-name matching as a fallback.

---

## Operating rules

| Rule | Detail |
|---|---|
| What vendors see | **Remaining slots per company** (live number), never the raw cap. |
| When a slot is consumed | On **QA approval**. Leads still in review are reserved so two vendors can't fill the same slot. |
| Recycled company capacity | Carries its **actual remaining slots**, not a reset to the original cap. |
| Rejected lead | Slot returns to the **same vendor** for retry. Exception: quality rejections (fabricated data) → slot goes to pool. |
| Lock duration | **5 business days** idle before auto-release. Configurable per campaign. |
| Quota hit | **Hard stop.** Campaign closes for that vendor. All remaining companies release to pool. No overflow. |
| Company matching | **Normalized domain** (primary) + **fuzzy name match** (fallback). Free-text names never trusted alone. |

---

## Open item

**How does the platform identify companies today?** If the current system relies on free-text names, the identity-matching layer above is a prerequisite and should be scoped as its own work item. To confirm with the platform team.
