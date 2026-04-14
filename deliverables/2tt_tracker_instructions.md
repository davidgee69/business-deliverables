# 2 Tier Treasures (2TT) Referral Tracker: Operations Manual

**Owner:** Rizza (Director of Operations)
**Reviewers:** Founder, Gonzalo (Business Partner)
**Last Updated:** 2026-04-11

---

## Summary Metrics (Formulas to Implement in Your Spreadsheet Tool)

When you import the CSV into Google Sheets or Excel, build these calculations in a summary tab or dashboard section above the data.

### Counts

- **Total Referrals This Month:** Count rows where Date of Referral falls within the current calendar month.
- **Total Referrals This Quarter:** Count rows where Date of Referral falls within Q1 (Jan-Mar), Q2 (Apr-Jun), Q3 (Jul-Sep), or Q4 (Oct-Dec) of the current year.
- **Total Referrals This Year:** Count rows where Date of Referral falls within the current calendar year.

### Conversion Rate

- **Formula:** (Number of rows with Status = "Converted" or "Commission Pending" or "Commission Paid") divided by (Total rows excluding Status = "Lost") multiplied by 100.
- Only count referrals that have had at least 30 days to convert. Do not penalize fresh referrals.

### Commission Totals

- **Total Commission Owed (Unpaid):** Sum of Commission Owed ($) where Commission Paid ($) = 0 and Status is "Commission Pending" or "Converted."
- **Total Commission Collected:** Sum of Commission Paid ($) across all rows.

### Timing Averages

- **Average Days to Conversion:** Average of the Days to Conversion column for all converted referrals (Status = Converted, Commission Pending, or Commission Paid). Exclude blanks.
- **Average Days to Payment:** Average of the Days to Payment column for all paid referrals (Status = Commission Paid). Exclude blanks.

### Partner and Source Rankings

- **Top 5 Partners by Commission Generated:** Group by Referred To. Sum Commission Owed ($) + Commission Paid ($) for each partner. Sort descending. Show top 5.
- **Top 5 Referral Sources by Volume:** Group by Referral Source Event. Count total referrals per source. Sort descending. Show top 5.

---

## How to Add a New Referral

1. Open the tracker CSV or spreadsheet.
2. Go to the next empty row.
3. Assign the next Referral ID in sequence. If the last ID was 2TT-005, the new one is 2TT-006.
4. Fill in these fields at the time of referral:
   - **Date of Referral:** The date the intro was made. Use YYYY-MM-DD format.
   - **Referred By:** The person who initiated the referral. Use "Marcus (Founder)" for direct referrals. For bird dogs and connectors, use their full name with a label, like "Ray Delgado (Bird Dog)."
   - **Referred To:** The partner company name exactly as it appears in previous rows. Consistency matters for grouping.
   - **Partner Category:** Use one of these categories: Insurance, Legal, Financial Services, Real Estate, Tech, Marketing, Home Services, Tax/Accounting, Consulting, Other.
   - **Client Name:** Full name of the person or couple being referred.
   - **Client Contact Info:** Email and phone, separated by a slash.
   - **Referral Source Event:** Pick one: IMParty, Founders Lounge, 3DayThinkTank, Direct, Other.
   - **Status:** Set to "Sent."
5. Leave these fields blank for now: Conversion Date, Deal Value, Commission Rate, Commission Owed, Commission Paid, Commission Paid Date, Days to Conversion, Days to Payment.
6. Add anything relevant to the Notes field. Include context that will help with follow-up.

---

## Status Definitions

| Status | Meaning |
|---|---|
| Sent | Referral has been made. Waiting for the partner to engage the client. |
| In Progress | Partner has contacted the client. Conversations or proposals are underway. |
| Converted | Client closed a deal with the partner. Commission has not yet been invoiced or discussed. |
| Lost | Client did not proceed. No deal. |
| Commission Pending | Deal closed. Commission has been invoiced or is expected. Payment has not been received. |
| Commission Paid | Commission received in full. |

---

## Rizza's Weekly Workflow

Do this every Monday morning. Should take 15 to 30 minutes once the system is established.

### Step 1: Update Active Referrals

- Filter for Status = "Sent" or "In Progress."
- Contact each partner (or check CRM/email) for an update on the client.
- Update the Status column if anything has changed.
- If a deal closed, update: Status to "Converted," Conversion Date, Deal Value, Commission Rate (%), and calculate Commission Owed ($) as Deal Value times Commission Rate divided by 100.
- Calculate Days to Conversion: Conversion Date minus Date of Referral.

### Step 2: Chase Pending Commissions

- Filter for Status = "Commission Pending."
- Check if payment has been received.
- If paid, update: Commission Paid ($), Commission Paid Date, Status to "Commission Paid."
- Calculate Days to Payment: Commission Paid Date minus Conversion Date.
- If not paid, check how long it has been since the Conversion Date. Follow the escalation timeline below.

### Step 3: Log New Referrals

- Check with the founder and any active bird dogs for referrals made during the past week.
- Add new rows following the "How to Add a New Referral" section above.

### Step 4: Flag Problems

- Run through the Red Flags checklist (see below).
- If any red flags exist, send a short summary to the founder and Gonzalo by end of day Monday.

---

## Commission Follow-Up and Escalation Timeline

Use this timeline starting from the Conversion Date for any referral in "Commission Pending" status.

| Days Since Conversion | Action |
|---|---|
| 0 to 14 days | No action needed. Normal processing window. |
| 15 days | Send a friendly check-in email to the partner contact. Reference the client name and deal. Ask for expected payment date. |
| 30 days | Send a second follow-up. Copy the founder if the partner has not responded to the first message. |
| 45 days | Escalate to the founder directly. He will call the partner. Flag this row in the Notes column with "ESCALATED" and the date. |
| 60+ days | This is now a red flag. Add to the Monday report for the founder and Gonzalo. The founder will decide next steps. This may affect the partnership. |

---

## How to Flag a Slow Partner

Track patterns over time. A partner is considered slow if any of the following are true:

**Slow to Convert:**
- Average Days to Conversion for that partner exceeds 45 days across 3 or more referrals.
- Action: Note this in the partner's rows. Mention it in the monthly report. The founder and Gonzalo will decide whether to continue, renegotiate, or replace the partner.

**Slow to Pay:**
- Average Days to Payment for that partner exceeds 45 days across 2 or more referrals.
- Any single commission unpaid past 60 days.
- Action: Add "SLOW PAYER" to the Notes column on all rows for that partner. Include in the Monday report and the monthly report. The founder will address it directly.

---

## Monthly Reporting

Pull these numbers on the first business day of each month. Send to the founder and Gonzalo in a short email or shared doc.

### Numbers to Include

1. **Total new referrals last month** (count of rows with Date of Referral in the prior month).
2. **Total conversions last month** (count of rows where Conversion Date falls in the prior month).
3. **Conversion rate for the trailing 90 days** (converted referrals divided by total referrals sent 90+ days ago).
4. **Total commission collected last month** (sum of Commission Paid ($) where Commission Paid Date falls in the prior month).
5. **Total commission outstanding** (sum of Commission Owed ($) minus Commission Paid ($) for all rows where Status is Commission Pending or Converted).
6. **Average days to conversion** (trailing 90 days).
7. **Average days to payment** (trailing 90 days).
8. **Top 3 partners by commission generated last month.**
9. **Top 3 referral source events by volume last month.**
10. **Any red flags** (see below).

### Format

Keep it to one page or one screen. Use a table or bullet list. No narrative needed. The founder wants numbers, not paragraphs.

---

## Red Flags to Surface Immediately

Do not wait for the Monday report or the monthly report if any of these conditions are true. Message the founder and Gonzalo the same day you discover it.

1. **Commission owed for 60+ days with no response from the partner.** This is a relationship and cash flow problem.
2. **Conversion rate drops below 30% over a rolling 90-day window.** Something is wrong with either the partner quality or the referral fit.
3. **A partner disputes a commission or changes the agreed rate.** This needs founder involvement immediately.
4. **A referred client complains about the partner's service.** The founder's reputation is on the line. He needs to know within 24 hours.
5. **A bird dog or connector is sending low-quality referrals.** If 3 or more consecutive referrals from the same source are marked "Lost," flag it.
6. **Any single commission owed exceeds $10,000 and is unpaid past 30 days.** High-dollar items get priority follow-up.

---

## Field Reference

| Field | Format | Example | Notes |
|---|---|---|---|
| Referral ID | 2TT-XXX | 2TT-001 | Sequential. Never reuse an ID. |
| Date of Referral | YYYY-MM-DD | 2026-01-14 | Date the intro was made. |
| Referred By | Name (Role) | Marcus (Founder) | Always include the role label. |
| Referred To | Company Name | Pinnacle Risk Group | Use the exact same name every time for the same partner. |
| Partner Category | Category | Insurance | Pick from the standard list. |
| Client Name | Full Name | David Chen | Person or couple being referred. |
| Client Contact Info | Email / Phone | dchen@gmail.com / 512-555-0183 | Slash-separated. |
| Referral Source Event | Event Name | IMParty | Pick from: IMParty, Founders Lounge, 3DayThinkTank, Direct, Other. |
| Status | Status Label | Sent | Pick from: Sent, In Progress, Converted, Lost, Commission Pending, Commission Paid. |
| Conversion Date | YYYY-MM-DD | 2026-02-02 | Date the client closed the deal. Blank until converted. |
| Deal Value | Dollar amount | 48000 | Total value of the deal. No dollar sign in the cell. |
| Commission Rate (%) | Percentage | 8 | Just the number. No percent sign in the cell. |
| Commission Owed ($) | Dollar amount | 3840 | Deal Value times Commission Rate divided by 100. |
| Commission Paid ($) | Dollar amount | 3840 | Amount actually received. |
| Commission Paid Date | YYYY-MM-DD | 2026-03-01 | Date payment was received. |
| Days to Conversion | Number | 19 | Conversion Date minus Date of Referral. Auto-calculate if using a spreadsheet. |
| Days to Payment | Number | 27 | Commission Paid Date minus Conversion Date. Auto-calculate if using a spreadsheet. |
| Notes | Free text | Auto-converted after intro call. | Anything relevant. Use this for escalation flags too. |
