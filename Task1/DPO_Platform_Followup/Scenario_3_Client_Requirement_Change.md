# Scenario 3 — Client Requirement Change (24h Delivery Limitation)

**Status:** ✅ Direction confirmed. Corner-case handling and the client/vendor trade-off are part of the task.

## Problem in one sentence

When a client changes the campaign criteria mid-flight, the team needs a way to deliver already-approved leads under the old rules (within 24 hours) while switching all new work to the new rules — without the two ever mixing.

## What they're likely asking

A campaign is running. Vendors are submitting leads, QA is reviewing them, delivery is shipping them. Then the client says: "Actually, stop targeting UK — only Ireland and Germany. And raise company size from 500+ to 1,000+."

The problem: some leads were already approved under the old rules. Those still need to go out within 24 hours. But from this moment forward, every team — QC, vendors, DPO Admin, email marketing — must follow the new rules. The hard part is making sure nobody accidentally checks an old lead against new rules, or lets a new lead slip through under old rules.

---

## Relationship to Scenario 5

Scenarios 3 and 5 are two versions of the same underlying problem: **the client changes something mid-campaign, and the platform needs to protect work already done while applying the new reality going forward.** Here the client changes the **qualification criteria** (e.g., target titles, company size, GEO). In Scenario 5 the client changes the **ABM list** (which companies to target). The core mechanism is the same — immutable versioning, tagging work to the version it belongs to, and deterministic rules for what happens to in-flight work at the boundary. The difference is what gets versioned (rules vs. account list), how long old and new run side by side (24 hours here vs. ongoing there), and who is affected (leads already in the pipeline here vs. companies already being worked there).

---

## Direction

The solution has three parts:

**1. Confirmation Gate (how the change starts).** The Client Success Team (CST) logs the change in the platform before anything happens. They record: what the new rules are, when they take effect, and what happens to leads already in the pipeline. The client confirms. This creates a clear record so there's no confusion later about what was agreed.

**2. Version tagging (how old and new stay separated).** Every lead gets tagged with the version of the rules it was submitted under. Old leads stay tagged "v1" and move through QA and delivery under v1 rules. New leads get tagged "v2" and follow v2 rules. Teams can filter their work by version. Reports split by version. Nothing mixes.

**3. Countdown Dashboard (how the DPO Admin keeps track).** A single screen shows how many v1 leads are still in the pipeline and how much time is left to deliver them. Example: *"v1 remaining: 26, oldest deadline in 6h 12m."* When the dashboard hits zero, the old rules are fully wound down.

**Result.** Old and new rules run side by side for up to 24 hours. No lead is checked against the wrong rules. Clients get their change applied right away for new work. Vendors know exactly what they'll be paid for. The tension between client and vendor is handled by a clear, pre-agreed rule — not last-minute negotiation.

---

## Operating Rules

| Where is the lead when the change happens? | Which rules apply? | Does the vendor get paid? | When must it be delivered? |
|---|---|---|---|
| Already approved by QA under old rules | Old rules (stays as-is) | Yes | Within 24h of the switch |
| In QA review, submitted under old rules | Old rules (finishes review under old rules) | Yes, if it passes | Within 24h of the switch |
| Failed QA under old rules after the switch | Old rules — final, not re-checked under new rules | No | N/A |
| Vendor still researching, not submitted yet | 2–4h grace window to submit under old rules, then new rules apply | Only if submitted within the grace window | Standard timeline under new rules |
| Submitted after the switch | New rules | Yes | Standard timeline |

**When does the 24h clock start?** At the moment CST confirms the switch in the platform — not when the client first sent the email. This is because the DPO team can only act from the moment the switch is live in the system.

---

## Options Considered (Summary)

1. **Confirmation Gate** ✅ *Chosen — the trigger.*
2. **Version Tagging** ✅ *Chosen — the foundation.*
3. **Countdown Dashboard** ✅ *Chosen — operational visibility.*
4. **Flag on Approved Leads** — Correct idea but incomplete on its own; absorbed into version tagging.
5. **Split Into Two Campaigns** — Rejected. Doubles admin work; version tagging achieves the same separation without the overhead.
6. **Hard Cutover With Manual Review** — Rejected. Too slow when you only have 24 hours.
7. **Two Visual Pipelines in One Campaign** — Good idea for a future UI improvement, not needed for the core logic.

**Final design = Confirmation Gate (trigger) + Version Tagging (execution) + Countdown Dashboard (visibility) + operating rules above.**

---

## Example Walkthrough

*Setup.* Campaign "EU SaaS Decision Makers." Old rules (v1): company size 500+, UK + Ireland + Germany. Three vendors are working on it. Right now there are 18 leads approved by QA but not yet delivered, 8 leads waiting in QA, and some vendor research still in progress.

1. **Client asks for a change.** Monday 2:00 PM. Client emails CST: "Drop UK. Ireland and Germany only, size 1,000+."
2. **CST logs the change.** CST opens the platform and fills in: new rules (Ireland + Germany, 1,000+), effective time (immediate on confirmation), and treatment of leads already in pipeline (deliver approved ones under old rules within 24h, give vendors a 2h grace window).
3. **Client confirms.** CST sends this back to the client for sign-off. Client confirms at 2:45 PM.
4. **Switch happens.** 2:45 PM — CST hits confirm. New rules go live. The 24h clock starts (deadline: Tuesday 2:45 PM). Platform automatically notifies all three vendors: *"Rules updated. You have until 4:45 PM to submit anything you were working on under the old rules. After that, new rules apply."*
5. **Next 24 hours — old and new run side by side.**
   - The *18 approved leads* ship to the client under old rules before Tuesday 2:45 PM. Client pays for them, vendors get paid.
   - The *8 leads in QA* get reviewed against old rules, not new ones. If they pass, they ship. If they fail, that's final — they don't get a second chance under the new rules.
   - *Vendor A* submits 2 nearly-finished leads at 3:30 PM (tagged v1, reviewed under old rules). Their other 2 leads aren't ready by 4:45 PM, so those must meet the new rules when eventually submitted.
   - *Vendor B* starts fresh research at 5:00 PM. Everything they submit from now on follows the new rules.
6. **DPO Admin watches the dashboard.** "v1 remaining: 26 … 19 … 11 … 0." By Tuesday 2:45 PM, all old-rule leads are delivered or closed out.
7. **Final report.** Delivered leads are split: *"Old rules: 32 leads (UK + Ireland + Germany, 500+) | New rules: 147 leads (Ireland + Germany, 1,000+)."* Full transparency on what was delivered under which criteria.

---

## Remaining Internal Decisions

1. Does the platform already support versioning on campaign briefs, or do we need to build this from scratch?
2. Who can authorize the switch — CST alone, or does the DPO Admin also need to approve?
3. Should vendors see what specifically changed between old and new rules, or just receive the new ruleset?
4. If the client changes their mind and wants to undo the switch, do we revert or freeze?
5. Should the final report show the old/new split to the client by default, or only if they ask?
