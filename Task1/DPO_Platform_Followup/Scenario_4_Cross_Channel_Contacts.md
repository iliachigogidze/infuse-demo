# Scenario 4 - Cross-Channel Contacts

**Status:** ✅ Rule confirmed by the team: **"Never contact through another channel for the same campaign."** This is a hard rule, not a time-window cooldown.

**Confirmed:** All channels (DPO, email marketing, etc.) share **one database**. The infrastructure for cross-channel visibility exists. The solution builds on top of it.

**Confirmed:** This is a **cross-channel version of the duplicate lead problem** (Scenario 2). In Scenario 2, two vendors within DPO submit the same person. Here, a person contacted by DPO also gets contacted by email marketing for the same campaign. The outcome is the same - a duplicate contact and potentially a duplicate lead - but the duplication crosses channel boundaries.

## Problem in one sentence

When a DPO vendor calls a prospect and adds them as a contact, the email marketing team has no way to know - so they email the same person for the same campaign, producing duplicate outreach and potentially a duplicate lead.

## Problem

A client runs Campaign X - "EU SaaS Decision Makers." The DPO team has 3 vendors calling prospects. When a vendor calls someone, they add that person as a contact in the platform - that's how the contact enters the system. At the same time, the email marketing team is running an automated nurture sequence for the same campaign, pulling from their own target lists.

Here's what goes wrong. Vendor A calls John Doe, VP of IT at Acme Corp, on Tuesday morning and adds him as a contact for Campaign X. That afternoon, the email marketing platform sends John Doe a Campaign X nurture email. John Doe has now been contacted twice for the same campaign - once by phone, once by email. He feels spammed, the contact is burned, and two teams spent effort on the same person without knowing about each other.

It gets worse if both channels produce a lead. The DPO vendor qualifies John Doe and submits him as a lead. Meanwhile, John Doe clicks through the email sequence, fills out a form, and is also delivered as a lead for Campaign X through email marketing. The client receives John Doe twice.

The team confirmed the rule is absolute: **"Never contact through another channel for the same campaign."** If a vendor has called someone for Campaign X, email marketing must not email that person for Campaign X. Period.

Today, there's no automatic connection between "vendor added a contact in DPO" and "email marketing's suppression list." The only way to prevent this is manual - someone exports the DPO contact list, someone else imports it as a suppression list in the email tool. That happens once a week at best, and every contact added between syncs is exposed to duplicate outreach.

---

## Solution

The solution has two parts: automatic suppression when contacts are added, and a central log for audit.

### 1. Pre-Call Check + Auto-Suppression (how contacts are protected)

**Before calling,** vendors can view a suppression list for their assigned companies showing who is already in the email sequence for this campaign. If a person is on the list, the vendor skips them and never makes the call. This prevents the overlap before it happens.

**After calling,** when a vendor adds a contact to a campaign in the DPO platform, the platform automatically writes a **suppression flag** on that contact's record for that campaign. The email marketing platform checks this flag before sending. If the flag exists, it skips that person. No manual export, no weekly sync, no spreadsheet.

**The suppression flag is permanent for the campaign.** Once written, it is never released, even if the contact is later disqualified or marked unresponsive. The rule is absolute: if DPO touched someone on this campaign, email marketing never contacts them for this campaign.

The flag is written at the moment the **vendor adds the contact**. The email marketing platform reads from the same shared database, so the suppression is visible immediately.

### 2. Shared Touch Log (how every action is recorded)

Every contact addition and every email send is written to a **central touch log**: who was contacted, when, by which channel, for which campaign. If an email is suppressed because the person was already added by a DPO vendor, that's logged too - *"skipped-by-rule."* The log proves the rule was enforced, powers reporting, and gives the client evidence of contact discipline.

**Result.** When a vendor adds a contact, email marketing automatically knows not to email them. When email marketing is already working someone, vendors see a warning before duplicating effort. No manual syncing, no coordination meetings. The shared database does the work.

---

## Operating Rules

| Rule | Detail |
|---|---|
| Scope of suppression | **Per campaign.** John Doe can be a DPO contact for Campaign A and receive emails for Campaign B. |
| Pre-call check | Vendors can view who is already in the email sequence for their assigned companies **before calling**. If a person is on the list, vendor skips them. |
| When the flag is created | When a **vendor adds a contact** to a campaign (meaning the call already happened) or when **email marketing sends a first email** to that person for a campaign. |
| Flag duration | **Permanent for the campaign.** Once written, the flag is never released, even if the contact is disqualified or unresponsive. |
| What the email platform does | Before every send, it checks: does this person already have a DPO contact record for this campaign? If yes, **skip**. |
| What the DPO platform does | When a vendor adds a contact, it checks: is this person already in an email sequence for this campaign? If yes, **warn the vendor**, contact is not added. |
| What gets logged | Every contact addition, every email send, every suppression, every pre-call skip. Suppressed sends are logged as *skipped-by-rule*. |
| Cross-campaign | Not blocked. The suppression only applies within one campaign. |

---

## Process Flow

**Step 1 - Campaign Launches.** Campaign X is set up. The DPO team begins vendor outreach. The email marketing team sets up their nurture sequence for the same campaign. Both operate from the shared database.

**Step 2 - Pre-Call Check.** Before starting their calling session, Vendor A views the suppression list for their assigned companies. The list shows who is already in the email sequence for Campaign X. Vendor A skips those people and only calls prospects not already being worked by email.

**Step 3 - Vendor Calls and Adds a Contact.** Vendor A calls John Doe at Acme Corp (confirmed not in email sequence via pre-call check). The call happens. Vendor A adds John Doe as a contact in the DPO platform for Campaign X. The platform writes a permanent suppression flag: *"DPO contact: Campaign X"* on John Doe's record.

**Step 4 - Email Marketing Checks Before Sending.** The email platform is about to send John Doe a Campaign X nurture email. Before sending, it checks the shared database, finds *"DPO contact: Campaign X"*, and skips John Doe automatically. No human had to intervene.

**Step 5 - Reverse Check: Email Was First.** Sarah Chen has been receiving Campaign X emails for a week. Vendor B finds Sarah Chen during their research and tries to add her as a DPO contact for Campaign X. The platform warns Vendor B: *"This contact is already being worked through email marketing for this campaign."* Vendor B skips her.

**Step 6 - Contact Exits the Pipeline.** John Doe is qualified by the DPO vendor and delivered as a lead. His record shows: contacted by DPO, delivered as a lead, email marketing was suppressed throughout. The suppression flag stays permanently - even after delivery, email marketing will never contact John Doe for this campaign.

**Step 7 - Reporting.** The DPO Admin or campaign manager pulls a report from the touch log: how many contacts were added by DPO vendors, how many were suppressed from email marketing as a result, how many were skipped via pre-call check, and whether any duplicates slipped through.

---

## Example Walkthrough

*Setup.* Campaign "EU SaaS Decision Makers" launches on Monday. Three DPO vendors start calling. The email marketing team has a 4-week nurture sequence ready for the same campaign, pulling from the campaign's target account list.

1. **Monday - Pre-call check and calling.** Vendors check the suppression list before calling. 8 prospects on their lists are already in the email sequence - vendors skip them. Over the first week, the three vendors call 60 prospects (all confirmed clean via pre-call check) and add them as contacts. Each addition writes a permanent suppression flag for Campaign X.

2. **Tuesday - Email sequence fires its first batch.** The email platform pulls the target list - 500 people. Before sending, it checks the shared database for each one. 12 of those 500 were already added as DPO contacts on Monday - skipped. The remaining 488 receive the first email.

3. **Wednesday - Pre-call check catches overlap.** Vendor B runs the pre-call check and sees Lisa Park is in the email sequence. Vendor B skips her without making the call.

4. **Week 2 - More vendors, more contacts.** Vendors add another 45 contacts throughout the week. Each one is automatically suppressed from the next email batch. The email platform's Tuesday send skips all 45 without anyone having to do anything.

5. **Week 3 - DPO delivers leads.** Vendor A qualifies John Doe and submits him as a lead. He passes QA, gets delivered to the client. His touch log shows: pre-call check (clean), called by Vendor A on Monday of Week 1, followed up Tuesday and Thursday, qualified Week 3, delivered. Email marketing never touched him - suppressed from day one. The flag stays permanently.

6. **Campaign end - Final report.** Touch log summary: *"DPO vendors added 150 contacts. 8 prospects skipped via pre-call check. 150 auto-suppressed from email. Email marketing worked 342 contacts. 0 contacted by both channels. 0 duplicate leads."*

---

## Platforms / Systems Involved

- **DPO Platform** - where vendors add contacts. Writes the suppression flag to the shared database on every contact addition.
- **Email Marketing Platform** - reads the shared database before every send. Skips anyone flagged as a DPO contact for the same campaign.
- **Shared Database** - the single source of truth. Both platforms read from and write to it. The suppression flag lives here.
- **CRM** - consumes the touch log for reporting and client-facing evidence.

---

## Options Considered (Summary)

1. **Pre-Call Check + Auto-Suppression on Contact Entry** ✅ *Chosen - primary enforcement mechanism. Pre-call check prevents overlap before the call. Auto-suppression catches anything that slips through.*
2. **Shared Touch Log** ✅ *Chosen - audit layer.*
3. **Manual Suppression List Export/Import** - Rejected. This is what happens today - weekly at best, error-prone, and every contact added between syncs is exposed.
4. **Cooldown Timer Between Channels** - Rejected. Team confirmed the rule is "never," not "not within X days."
5. **Pre-Campaign Channel Split** - Rejected. Doesn't match reality - DPO contacts are added as vendors call, not assigned upfront.

**Final design = Pre-Call Check (prevention) + Auto-Suppression (real-time, triggered by contact entry) + Shared Touch Log (audit).**

---

## Corner Cases

1. **Pre-call check not used** - if a vendor skips the pre-call check and calls someone already in the email sequence, the platform warns them when they try to add the contact. The call already happened, but email is suppressed from that point forward. One overlap, but contained.
2. **Cross-campaign contact** - the suppression is *per campaign*, so the same person can be a DPO contact for Campaign A and receive emails for Campaign B. This is allowed.
3. **Email already sent before vendor added the contact** - if the vendor adds John Doe at 10:15 AM but the email batch ran at 9:00 AM, John Doe already received one email. The touch log captures this. From the next batch onward, he's suppressed. The first email is a known limitation of batch timing. Pre-call check minimizes this risk.
4. **Contact disqualified or unresponsive** - the suppression flag stays permanently. Even if the DPO contact goes nowhere, email marketing never contacts that person for this campaign. The rule is absolute.

---

## Remaining Internal Decisions

1. **How often does the email platform check the shared database?** Real-time (before every individual send) is ideal. If the email tool works in batches, the batch frequency determines the gap - contacts added between batches could still receive one email.
2. **Can a vendor override the warning and add a contact already in email?** If yes, the contact should be pulled from the email sequence and logged. If no, the warning is a hard block.
