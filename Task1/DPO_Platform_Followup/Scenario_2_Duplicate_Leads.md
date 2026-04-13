# Scenario 2 — Duplicate Leads from Different Vendors

**Status:** Fully covered by Scenario 1. No additional mechanism needed.

---

## The problem in one sentence

Two vendors independently find the same person and both submit them to the same campaign — but duplicates are only caught during QA cleanup, wasting vendor effort and eroding client trust.

## The constraint

No vendor may ever learn that another vendor is on the campaign. A block must look like a neutral, vendor-agnostic rejection.

---

## Why Scenario 1 already solves this

The vendor scope split from Scenario 1 eliminates the duplicate problem at its root. Each vendor works an exclusive set of companies — Vendor A owns companies 1–40, Vendor B owns 41–75. They cannot contact the same person at the same company because they're never working the same company at the same time.

When a company is recycled (vendor goes idle or hits quota), the receiving vendor gets a **suppression list** of already-contacted leads — names and emails only, no vendor identifiers. The second vendor knows who to skip before they start calling.

> **Example:** Vendor A submits 2 leads from Acme, then hits quota. Acme moves to Vendor B's list with a suppression file: "Skip Jane Doe (j.doe@acme.com) and Mark Lee (m.lee@acme.com)." Vendor B works Acme's remaining slots with fresh contacts. No duplicate is possible.

Between the exclusive split and suppression on reassignment, duplicates are structurally prevented.

---

## Summary

| Layer | What it does | Where it comes from |
|---|---|---|
| Vendor scope split | Prevents two vendors from working the same company | Scenario 1 |
| Suppression list on reassignment | Prevents re-contacting leads when a company changes hands | Scenario 1 |

Both layers are inherited from Scenario 1 at zero additional cost. No net-new mechanism is needed for Scenario 2.
