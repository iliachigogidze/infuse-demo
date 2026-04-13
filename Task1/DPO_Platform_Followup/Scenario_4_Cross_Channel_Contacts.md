# Scenario 4 — Cross-Channel Contacts

**Status:** ✅ Rule confirmed by the team: **"Never contact through another channel for the same campaign."** This is a hard rule, not a time-window cooldown.

**Confirmed:** All channels (DPO, email marketing, etc.) share **one database**. The infrastructure for cross-channel visibility exists. The solution builds on top of it.

**Confirmed:** This is a **cross-channel version of the duplicate lead problem** (Scenario 2). In Scenario 2, two vendors within DPO submit the same person. Here, a person contacted by DPO also gets contacted by email marketing for the same campaign. The outcome is the same — a duplicate contact and potentially a duplicate lead — but the duplication crosses channel boundaries.

## Problem in one sentence

When a DPO vendor calls a prospect and adds them as a contact, the email marketing team has no way to know — so they email the same person for the same campaign, producing duplicate outreach and potentially a duplicate lead.

## Problem

A client runs Campaign X — "EU SaaS Decision Makers." The DPO team has 3 vendors calling prospects. When a vendor calls someone, they add that person as a contact in the platform — that's how the contact enters the system. At the same time, the email marketing team is running an automated nurture sequence for the same campaign, pulling from their own target lists.

Here's what goes wrong. Vendor A calls John Doe, VP of IT at Acme Corp, on Tuesday morning and adds him as a contact for Campaign X. That afternoon, the email marketing platform sends John Doe a Campaign X nurture email. John Doe has now been contacted twice for the same campaign — once by phone, once by email. He feels spammed, the contact is burned, and two teams spent effort on the same person without knowing about each other.

It gets worse if both channels produce a lead. The DPO vendor qualifies John Doe and submits him as a lead. Meanwhile, John Doe clicks through the email sequence, fills out a form, and is also delivered as a lead for Campaign X through email marketing. The client receives John Doe twice.

The team confirmed the rule is absolute: **"Never contact through another channel for the same campaign."** If a vendor has called someone for Campaign X, email marketing must not email that person for Campaign X. Period.

Today, there's no automatic connection between "vendor added a contact in DPO" and "email marketing's suppression list." The only way to prevent this is manual — someone exports the DPO contact list, someone else imports it as a suppression list in the email tool. That happens once a week at best, and every contact added between syncs is exposed to duplicate outreach.

---

## Solution

The solution has two parts: automatic suppression when contacts are added, and a central log for audit.

### 1. Auto-Suppression on Contact Entry (how contacts are protected)

When a vendor adds a contact to a campaign in the DPO platform — meaning they've already called that person — the platform automatically writes a **suppression flag** on that contact's record for that campaign. The email marketing platform checks this flag before sending. If the flag exists, it skips that person. No manual export, no weekly sync, no spreadsheet.

The flag is written at the moment the **vendor adds the contact** — this is the natural trigger, because adding the contact means the call already happened. The email marketing platform reads from the same shared database, so the suppression is visible immediately.

The same works in reverse: if the email marketing team has already been emailing someone for Campaign X and a vendor later tries to add that same person as a DPO contact for Campaign X, the platform warns the vendor that this person is already being worked through email. The vendor skips them and moves on.

### 2. Shared Touch Log (how every action is recorded)

Every contact addition and every email send is written to a **central touch log**: who was contacted, when, by which channel, for which campaign. If an email is suppressed because the person was already added by a DPO vendor, that's logged too — *"skipped-by-rule."* The log proves the rule was enforced, powers reporting, and gives the client evidence of contact discipline.

**Result.** When a vendor adds a contact, email marketing automatically knows not to email them. When email marketing is already working someone, vendors see a warning before duplicating effort. No manual syncing, no coordination meetings. The shared database does the work.

---

## Operating Rules

| Rule | Detail |
|---|---|
| Scope of suppression | **Per campaign.** John Doe can be a DPO contact for Campaign A and receive emails for Campaign B — that's allowed. |
| When the flag is created | When a **vendor adds a contact** to a campaign (meaning the call already happened) or when **email marketing sends a first email** to that person for a campaign. |
| What the email platform does | Before every send, it checks: does this person already have a DPO contact record for this campaign? If yes → **skip**. |
| What the DPO platform does | When a vendor adds a contact, it checks: is this person already in an email sequence for this campaign? If yes → **warn the vendor**, contact is not added. |
| What gets logged | Every contact addition, every email send, every suppression. Suppressed sends are logged as *skipped-by-rule*. |
| Cross-campaign | Not blocked. The suppression only applies within one campaign. |

---

## Process Flow

**Step 1 — Campaign Launches.** Campaign X is set up. The DPO team begins vendor outreach. The email marketing team sets up their nurture sequence for the same campaign. Both operate from the shared database.

**Step 2 — Vendor Calls and Adds a Contact.** Vendor A calls John Doe at Acme Corp. The call happens. Vendor A then adds John Doe as a contact in the DPO platform for Campaign X. At that moment, the platform writes a suppression flag: *"DPO contact: Campaign X"* on John Doe's record.

**Step 3 — Email Marketing Checks Before Sending.** The email platform is about to send John Doe a Campaign X nurture email. Before sending, it checks the shared database → finds *"DPO contact: Campaign X"* → skips John Doe automatically. John Doe never receives the email. No human had to intervene.

**Step 4 — Reverse Check: Email Was First.** Sarah Chen has been receiving Campaign X emails for a week. Vendor B finds Sarah Chen during their research and tries to add her as a DPO contact for Campaign X. The platform checks → Sarah Chen is already in the email sequence for Campaign X → warns Vendor B: *"This contact is already being worked through email marketing for this campaign."* Vendor B skips her.

**Step 5 — Contact Exits the Pipeline.** John Doe is qualified by the DPO vendor and delivered as a lead. His record shows: contacted by DPO, delivered as a lead, email marketing was suppressed throughout. Clean, single-channel journey.

**Step 6 — Reporting.** The DPO Admin or campaign manager pulls a report from the touch log: how many contacts were added by DPO vendors, how many were suppressed from email marketing as a result, and whether any duplicates slipped through. The client sees proof that no one was contacted by both channels.

---

## Example Walkthrough

*Setup.* Campaign "EU SaaS Decision Makers" launches on Monday. Three DPO vendors start calling. The email marketing team has a 4-week nurture sequence ready for the same campaign, pulling from the campaign's target account list.

1. **Monday — Vendors start calling.** Over the first week, the three vendors call 60 prospects and add them as contacts in the DPO platform. Each addition writes a suppression flag for Campaign X.

2. **Tuesday — Email sequence fires its first batch.** The email platform pulls the target list — 500 people. Before sending, it checks the shared database for each one. 12 of those 500 were already added as DPO contacts on Monday → skipped. The remaining 488 receive the first email.

3. **Wednesday — Vendor adds someone already in email.** Vendor B researches Lisa Park and is about to add her as a contact. The platform checks → Lisa Park received a Campaign X email yesterday → warns Vendor B: *"Lisa Park is already in the email sequence for Campaign X."* Vendor B skips her, moves to the next prospect.

4. **Week 2 — More vendors, more contacts.** Vendors add another 45 contacts throughout the week. Each one is automatically suppressed from the next email batch. The email platform's Tuesday send skips all 45 without anyone having to do anything.

5. **Week 3 — DPO delivers leads.** Vendor A qualifies John Doe and submits him as a lead. He passes QA, gets delivered to the client. His touch log shows: called by Vendor A on Monday of Week 1, followed up Tuesday and Thursday, qualified Week 3, delivered. Email marketing never touched him — suppressed from day one.

6. **Campaign end — Final report.** Touch log summary: *"DPO vendors added 150 contacts. 150 were auto-suppressed from email marketing. Email marketing worked 350 contacts (500 minus the 150 DPO contacts). 0 people were contacted by both channels. 0 duplicate leads."*

---

## Platforms / Systems Involved

- **DPO Platform** — where vendors add contacts. Writes the suppression flag to the shared database on every contact addition.
- **Email Marketing Platform** — reads the shared database before every send. Skips anyone flagged as a DPO contact for the same campaign.
- **Shared Database** — the single source of truth. Both platforms read from and write to it. The suppression flag lives here.
- **CRM** — consumes the touch log for reporting and client-facing evidence.

---

## Options Considered (Summary)

1. **Auto-Suppression on Contact Entry** ✅ *Chosen — primary enforcement mechanism.*
2. **Shared Touch Log** ✅ *Chosen — audit layer.*
3. **Manual Suppression List Export/Import** — Rejected. This is what happens today — weekly at best, error-prone, and every contact added between syncs is exposed.
4. **Cooldown Timer Between Channels** — Rejected. Team confirmed the rule is "never," not "not within X days."
5. **Pre-Campaign Channel Split** — Rejected. Doesn't match reality — DPO contacts are added as vendors call, not assigned upfront.

**Final design = Auto-Suppression (real-time, triggered by contact entry) + Shared Touch Log (audit).**

---

## Corner Cases

1. **Vendor adds a contact who is already in email** — platform warns the vendor before adding. Vendor skips or, if override is needed, the contact is reassigned from email to DPO with a log entry.
2. **Cross-campaign contact** — the suppression is *per campaign*, so the same person can be a DPO contact for Campaign A and receive emails for Campaign B. This is allowed.
3. **Email already sent before vendor added the contact** — if the vendor adds John Doe at 10:15 AM but the email batch ran at 9:00 AM, John Doe already received one email. The touch log captures this. From the next batch onward, he's suppressed. The first email is a known limitation of batch timing.
4. **Retroactive discovery** — if a duplicate slipped through (e.g., email batch ran before the vendor added the contact), the touch log surfaces it in reporting and the batch timing can be tightened.
5. **Contact removed from DPO mid-campaign** — if a vendor's contact is disqualified or marked unresponsive and removed, the suppression flag is released. Email marketing can pick them up in the next batch if they're still on the target list.

---

## Remaining Internal Decisions

1. **How often does the email platform check the shared database?** Real-time (before every individual send) is ideal. If the email tool works in batches, the batch frequency determines the gap — contacts added between batches could still receive one email.
2. **Can a vendor override the warning and add a contact already in email?** If yes, the contact should be pulled from the email sequence and logged. If no, the warning is a hard block.
3. **Should the suppression flag be released when a DPO contact is disqualified?** This would let email marketing try reaching them. Alternatively, once DPO has called someone, they stay suppressed for the rest of the campaign regardless of outcome.
