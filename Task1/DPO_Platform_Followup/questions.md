# Questions

## 1. Sharing a suppression list with vendors

When a company is reassigned from one vendor to another, is it permissible to share a suppression list of already-contacted leads with the new vendor, so they can avoid duplicating outreach?

**Example scenario:**
- Vendor 1 was originally assigned Company A and generated a lead for *John Doe* (an employee at Company A).
- Company A is later redistributed to Vendor 2.
- Without knowing that John Doe has already been contacted, Vendor 2 may spend time pursuing the same lead.

**The question:** Can we provide Vendor 2 with a suppression list identifying John Doe (and other already-contacted contacts at Company A) so that Vendor 2 knows to skip these leads and focus their effort on net-new contacts? Are there any data protection, contractual, or confidentiality concerns with sharing contact-level information between vendors in this way?

**Answer:** Yes — already-contacted leads can be added to a suppression list and shared with the new vendor.

---

## 2. Duplicate leads for the same person under different emails

Does it ever happen that the same individual is recorded as two separate leads because they were submitted with different email addresses?

**Example scenario:**
- Vendor 1 submits *John Doe* using his work email (e.g., `john.doe@company.com`).
- Vendor 2 later submits the same *John Doe* using a different address (e.g., a personal email, an alias, or an updated work email).
- Because the email addresses don't match, the system may treat these as two distinct leads even though they refer to the same person.

**The question:** Is this considered lead duplication?

**Answer:** No — as long as the email domains are different, it is not considered duplication. Additionally, personal emails are not taken into account; only work emails are used for deduplication.

---

## 3. Shared contact list across channels (Scenario 4)

To prevent the same person from being contacted by two channels on the same campaign, we need a way to look up a person and see — across all channels — whether someone is already working them on that campaign.

**Example:**
- DPO is calling John Doe for Campaign X.
- Email marketing is about to email John Doe for Campaign X.
- We need to block that email. But to do that, the email platform has to know that DPO is already working John Doe on Campaign X.
- (This only applies within Campaign X — the email team can still email John Doe on other campaigns.)

**The question:** Is there one shared contact list (or database) that DPO, email marketing, and all other channels use today? Or does each channel keep its own separate list with no connection between them?

**Why it matters:** If a shared list already exists, we can build the cross-channel exclusion on top of it. If each channel has its own isolated list, we would need to build that shared layer first — before any of the Scenario 4 logic can work.

**Answer:** Yes — there is one shared database across all channels.

---

## 4. Cross-channel duplicate leads (Scenario 4)

In Scenario 2, two vendors submit the same person as a lead — duplication happens within one channel. The same problem can also happen across channels: one channel already has John Doe as a lead, and another channel produces the same lead independently. The result is the same — a duplicate lead delivered to the client — but this time the duplication comes from different channels rather than different vendors.

**Example:**
- The email team is already working John Doe for Campaign X — he's in their active sequence.
- At the same time, a DPO vendor contacts John Doe and submits him as a lead for Campaign X.
- Both channels are now producing the same lead for the same campaign, and both will eventually deliver to the same client.

**The question:** Is this understanding correct — that this is essentially a cross-channel version of the duplicate lead problem? And if so, how should it be handled? Should one channel have priority over the other, should the first to start working the lead "own" it, or is there a different rule?

The answer determines whether the Scenario 4 exclusion rule needs to block not just outreach, but also lead submission across channels.

**Answer:** Yes — this is confirmed as a cross-channel version of the duplicate lead problem. A DPO vendor should not call contacts that are being contacted by email marketing, and vice versa. The example given (email team working John Doe while a DPO vendor independently submits him as a lead for the same campaign) is exactly the issue to solve.
