# Scenario 4 — Cross-Channel Contacts
### Proposed Approach

---

## Problem in one sentence

The same person gets contacted — and submitted as a lead — by multiple channels (DPO calls, email marketing, etc.) on the same campaign, producing both duplicate outreach and duplicate lead delivery to the client.

---

## Problem

A client runs Campaign X. The DPO team is calling prospects from the target list. At the same time, the email marketing team is running an automated sequence hitting the same list for the same campaign. All channels share one database, so the contact records exist in one place — but there is no rule today that prevents two channels from working the same person simultaneously.

Two things go wrong. First, **duplicate outreach**: John Doe at Acme gets a DPO call on Tuesday morning and a marketing email Tuesday afternoon — for the same campaign. He feels spammed, the contact is burned, and both teams wasted effort on someone the other team was already handling.

Second, **duplicate lead delivery**: the DPO vendor submits John Doe as a lead for Campaign X. Meanwhile, the email team's sequence nurtures John Doe to the point where he's also delivered as a lead for Campaign X. The client receives the same lead twice from two different channels. This is the same duplicate lead problem as Scenario 2, but crossing channel boundaries instead of vendor boundaries.

The team confirmed the rule is absolute: **"Never contact through another channel for the same campaign."** Not "wait a few days" — *never*. If DPO is working John Doe on Campaign X, email marketing must not touch him on Campaign X. Period.

Today, the only way to enforce this is manual coordination — syncing spreadsheets, daily standups between teams. That doesn't scale, and it breaks the moment someone forgets.

---

## How it works

### 1. Tag the person the moment a channel starts working them

When any channel begins actively working a person on a campaign, the platform writes an **exclusion tag** on that person's record.

> **Example:** The DPO team dials John Doe for Campaign X.
> The platform writes: *"in-flight: Campaign X: DPO"* on John Doe's record.
> From this moment, no other channel can touch John Doe on Campaign X.

The tag is created on a **real action** — actually dialed, actually queued into an email sequence, actually added to a LinkedIn outreach list. Just loading a working list or setting up a campaign does **not** create a tag.

### 2. Every channel checks the tag before reaching out

Before any outreach, each channel queries the tag store. If the person already has a tag for that campaign, they are **automatically skipped**. No manual check, no coordination meeting.

> **Example continued:** The email platform is about to send John Doe a Campaign X email.
> It checks the tag store → finds *"in-flight: Campaign X: DPO"* → skips John Doe.
> John Doe never receives the email. No human had to intervene.

### 3. Release the tag when the channel is done

The tag is removed when the person exits that channel's pipeline — delivered, disqualified, unresponsive past a threshold, or the campaign ends. Once the tag is removed, **other channels become eligible to pick them up** — but still only one at a time.

> **Example continued:** The DPO team marks John Doe as unresponsive after 3 weeks.
> The tag is released → John Doe becomes available again.
> The email marketing team can now pick him up for Campaign X.
> A new tag is written: *"in-flight: Campaign X: Email"* — and DPO is now blocked from contacting him.

### 4. Break ties with a priority order

If two channels try to claim the same person at the exact same moment, a **pre-set priority order** decides who wins. Priority is configured once per campaign at launch (e.g., *DPO > email > LinkedIn*), not decided case by case.

### 5. Log every touch for audit

Every outreach attempt — successful or skipped — is written to a **central touch log**: who was contacted, when, by which channel, for which campaign. The tag enforces the rule going forward; the log proves it was enforced after the fact. It powers reporting and gives the client evidence of contact discipline if they ever ask.

---

## Operating rules

| Rule | Detail |
|---|---|
| Scope of exclusion | **Per campaign.** John Doe can be in DPO for Campaign A and email for Campaign B at the same time — that's allowed. |
| When a tag is created | On a **real pipeline action** — dialed, queued into a sequence, added to an outreach list. Campaign setup alone does not create a tag. |
| When a tag is released | Delivered, disqualified, unresponsive past threshold, channel drops the contact, or campaign ends. Person becomes available to other channels immediately. |
| Tie-break | **Per-campaign priority order** (e.g., DPO > email > LinkedIn). Set once at launch. |
| What gets logged | **Every attempt, every channel** — including skipped ones. Skipped attempts are logged as *skipped-by-rule* so the audit trail is complete. |
| Enforcement point | **At the moment of outreach**, not at list-load time. Guarantees freshness. |
| Cross-campaign | The same person in different campaigns is **not** blocked. The exclusion only applies within one campaign. |

---

## Resolved items

1. ~~**Does a shared contact identity exist across DPO and email marketing today?**~~ **Yes — confirmed.** All channels use one shared database. No identity resolution layer needs to be built.
3. ~~**Is there a central touch log today?**~~ **Answered implicitly.** Since all channels share one database, the foundation for a central touch log exists. The log structure may still need to be built, but the shared data layer is already in place.

## Open items

1. **Can outreach platforms check an external tag store before sending?** Platforms that support a pre-send API check get real-time enforcement. Platforms that don't will need a periodic suppression list sync — and the sync frequency determines how tight the enforcement actually is.
