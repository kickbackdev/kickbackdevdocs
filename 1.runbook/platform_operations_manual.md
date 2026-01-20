# Kickback Platform Automation Operations Manual

<img src="../images/logokick.avif" alt="Kickback logo" width="120" />

> **Audience:** Operations lead, settlement team, partner support team  \
> **Scope:** Partner onboarding, inventory intake, settlement/subscription/refund automations

## Table of Contents
1. [Key Summary & Daily Checklist](#1-key-summary--daily-checklist)  
2. [Platform Overview](#2-platform-overview)  
3. [Airtable Data Architecture](#3-airtable-data-architecture)  
4. [Workflow Playbooks](#4-workflow-playbooks)  
5. [Settlement Model & Examples](#5-settlement-model--examples)  
6. [Monitoring & Incident Response](#6-monitoring--incident-response)  
7. [Appendix: Schema & Change Control](#7-appendix-schema--change-control)

---

## 1. Key Summary & Daily Checklist

| Area | Owner | Action | Tool/Location |
| --- | --- | --- | --- |
| **Alert review** | All ops | Monitor `#automation-alerts` for onboarding/settlement success, plus `#ops-alerts`, `#finance-alerts`, `#settlement-reports` for exceptions | Slack |
| **Inventory intake** | Warehouse ops | Check Airtable `Parts` view **"Awaiting Receipt"** and update status after physical arrival | Airtable |
| **Settlement batch** | Finance | Ensure today’s `Settlement Batches` record is `Completed`; investigate any `Error` payouts | Airtable |
| **Subscription gap check** | Partner ops | Weekly review of "Shops needing subscription" view (Non-Charter & missing Stripe Customer ID) | Airtable |
| **If an issue occurs** | Respective owner | Follow Section 6 steps: inspect Zapier Task History → `Replay Task` if applicable → escalate | Zapier/Slack |

---

## 2. Platform Overview

### 2.1 SaaS Stack
- **Contract intake:** SignWell (templates with machine-readable custom fields)
- **Orchestration:** Zapier (Paths, Code, Webhooks, Looping, Delay)
- **Data hub:** Airtable `Kickback Operations` base (single source of truth)
- **Payments:** Stripe (Connect transfers + Customer Subscriptions)
- **Comms:** Slack alert channels (success, ops, finance, settlement)

### 2.2 End-to-End Flow (Ops View)
1. Partner completes SignWell template → Zapier populates Partner/Contract/Part records in Airtable.  
2. Daily 09:00 UTC settlement Zap aggregates payable sales, calculates commissions/clawbacks, and sends Stripe transfers.  
3. Refunds trigger automatic negative `Clawback` payouts to offset future runs.  
4. Non-Charter shops enter subscription automation that creates Stripe customer + subscription.  
5. Every success/failure emits a Slack alert so ops can take action immediately.

---

## 3. Airtable Data Architecture

> Full schema lives in `schema.md`. This section highlights the fields ops teams manipulate often.

### 3.1 Core Tables & Purpose

| Table | Purpose | Must-know Fields |
| --- | --- | --- |
| **Partners** | Manage shops/individuals/ambassadors/referrals | `Partner Type`, `Email`, `Stripe Connected Account ID`, `Stripe Customer ID`, `Referred By`, `Shop Type`, `Subscription Status`, `Charter Status End Date` |
| **Parts** | Track part lifecycle | `QR Code ID`, `Owner`, `Status`, `Location Code`, `Sale Record` |
| **Sales** | Financial event per sale | `Part Sold`, `Marketplace Source`, `Gross Sale Price`, `Payout Status`, `Refund Status`, `Payout Batch` |
| **Payouts** | Every payment/recoup line item | `Sale`, `Payee`, `Payout Amount`, `Commission Type`, `Stripe Transfer ID`, `Status`, `Error Message` |
| **Contracts** | Signed SignWell docs | `Contract Type`, `Date Signed`, `Document Link` |
| **Settlement Batches** | Each settlement execution | `Run Date`, `Status`, `Total Payouts Processed`, `Total Amount Paid` |

### 3.2 States & Views to Monitor
- `Parts.Status`: stays at "Contract Signed - Awaiting Receipt" until warehouse confirms arrival and manually switches to `In Stock`/`Listed for Sale`.
- `Sales.Payout Status`: Zap sets to `Processing`; after Stripe success it becomes `Paid`. Keep an "Errors" view filtered on `Payout Status='Error'`.
- `Partners.Subscription Status`: when `Charter Status End Date` is reached, ensure automation flips from "N/A" to "Active" and Stripe subscription exists.

---

## 4. Workflow Playbooks

### 4.1 Partner Onboarding & Parts Intake (SignWell → Zapier → Airtable)
**Trigger:** SignWell `Document Completed`  
**Paths:** Individual, Shop, Ambassador, Parts Addendum

| Step | Description | Ops Notes |
| --- | --- | --- |
| 1. Send SignWell template | Choose one of four official templates | Custom field names must match Airtable fields exactly |
| 2. Zapier Path routing | Template name drives Path A/B/C/D | Shop (Path B) updates `Shop Type` and calculates `Charter Status End Date` |
| 3. Airtable record writes | Partners/Contracts/Parts records are created or updated | Parts Addendum **only searches** for existing partner → emails must be accurate |
| 4. Slack confirmation | `#automation-alerts` posts success summary | If duplicate warning appears, manually verify in Airtable |
| 5. Physical intake | Warehouse updates `Parts.Status` after receiving part | Clear the "Awaiting Receipt" view daily to avoid backlog |

> **Failure handling:** If Path D raises "Partner Not Found", ensure the base Individual/Shop contract is completed, then `Replay Task`.

### 4.2 Settlement & Payout Automation (Daily)
**Trigger:** Zapier Schedule at 09:00 UTC  
**Key channels:** `#settlement-reports`, `#ops-alerts`

1. Create `Settlement Batches` record with `Status=Running`.  
2. Pull `Sales` where `Payout Status='Payable'` and `Payout Eligibility Date <= TODAY()`.  
3. Loop through sales, set `Payout Status='Processing'`, link to current batch.  
4. Code by Zapier computes multi-party commissions (Owner/Shop/Ambassador/Referral tiers).  
5. Create `Payouts` rows (status `Payable`).  
6. Loop all payable payouts tied to the batch and call Stripe `POST /v1/transfers` via Webhook.  
7. Update `Payouts.Status` to `Paid` or `Error`, capturing Transfer ID or failure details; send Slack alerts per outcome.  
8. Mark batch `Completed` and post summary to `#settlement-reports`.

**Ops checklist**
- Monitor Stripe platform balance before large batches.
- Review the `Payouts` view filtered on `Status='Error'` and follow Section 6 actions per error code.
- Commission percentages live inside the Code step—request dev support for changes.

### 4.3 Refund & Clawback Automation
**Trigger:** Airtable `Sales` record where `Refund Status` becomes `Processed`.  
**Result:** Locate all prior `Payouts` linked to the sale, create negative `Clawback` payouts (`Status=Payable`). They net against each payee during the next settlement run.  
**Ops action:** Check `#finance-alerts` message for total clawbacks and contact partners if needed.

### 4.4 Non-Charter Shop Subscription Automation
1. View "Shops needing subscription" (Partner Type=Shop, `Shop Type=Non-Charter`, empty `Stripe Customer ID`) triggers the Zap.  
2. Create Stripe Customer → store `Stripe Customer ID` on the partner.  
3. Create Stripe Subscription ($149.95/mo price) → set `Subscription Status='Active'`.  
4. Charter-expiry Zap runs daily; when `Charter Status End Date` equals today, it launches steps 1–3.

**Ops reminders**
- Failed subscription creation includes the Stripe error in Slack; follow up with the shop for updated payment method.  
- Proactively email Charter partners ~14 days before expiry.

---

## 5. Settlement Model & Examples

### 5.1 Default Revenue Split
- Part owner: **≈65%**
- Kickback share: **≈35%**, distributed as Shop Partner 5%, Ambassador 5%, Referral Partner 3%, remainder retained by Kickback.

### 5.2 Tiered Fees by Sale Price

| Gross Price | Kickback Share | Notes |
| --- | --- | --- |
| ≤ $250 | 40% | Covers low-ticket handling cost |
| $250.01–$500 | 35% | Standard tier |
| $500.01+ | 30% | Incentive for higher-value parts |

### 5.3 Scenario Examples
1. **Ambassador → Shop → Owner ($1,000 sale)**  
   - Owner $650, Shop $50, Ambassador $50, Kickback net $250.
2. **Shop → Owner (no ambassador)**  
   - Owner $650, Shop $50, Kickback $300.
3. **Referral Partner → Owner (direct intro)**  
   - Owner $650, Referral $30, Kickback $320.

> Every amount exists as a `Payouts` record; refunds create equal negative `Clawback` rows to reverse prior payments.

---

## 6. Monitoring & Incident Response

### 6.1 Slack Alert Matrix

| Channel | Sample Message | Meaning / Action |
| --- | --- | --- |
| `#automation-alerts` | `✅ New partner onboarded ...` | Success. If duplicate warning is included, validate email uniqueness. |
| `#ops-alerts` | `⚠️ F1 Input Missing`, `⚠️ Unknown Template`, `🚨 F1 Step Failed` | Follow Section 6.2 flow to diagnose and replay/fix. |
| `#finance-alerts` | `🔄 Refund (Clawback) created for Sale ID ...` | Refund processed. Confirm amount and notify finance/partner if large. |
| `#settlement-reports` | `✅ Settlement batch complete. Paid X, Failed Y.` | If failures >0, inspect `Payouts.Status='Error'` immediately. |

### 6.2 Standard Response Flow
1. In **Zapier Task History**, locate the failed run using timestamp from Slack.  
2. Identify failing step + error message (e.g., Airtable auth expired, Stripe `balance_insufficient`).  
3. Check Airtable to confirm whether partial data exists to avoid duplicates.  
4. **Transient issues:** Re-auth the affected app in Zapier `My Apps`, then `Replay Task`.  
5. **Logic/data issues:** Capture payload + error details and file to system owner/dev team.  
6. **Stripe payout errors:**  
   - `balance_insufficient`: top up platform balance, then retry errored payouts.  
   - `account_invalid` / `verification_failed`: contact partner to re-verify their Stripe Connect account.  
   - Other codes: share full message with engineering for diagnosis.

### 6.3 Manual Fallback Plan
- If repeated failures occur, toggle the impacted Zap **OFF**.  
- Use SignWell completion emails to manually enter data into `Partners/Contracts/Parts`.  
- For settlements, export `Sales` & `Payouts`, calculate in spreadsheets, and issue transfers directly in Stripe Dashboard.  
- After automation is restored, reconcile manual entries to prevent duplicates.

---

## 7. Appendix: Schema & Change Control

### 7.1 Schema Reference
- Full field definitions: `schema.md`. Update that doc whenever new fields/dependencies are introduced.

### 7.2 Change Log Template

| Date | Owner | Change Summary | Ticket |
| --- | --- | --- | --- |
| 2025-11-07 | Jeong Yuhyeon | System initial build complete per KKB-F1-14 | KKB-F1-14 |
| (YYYY-MM-DD) | (Name) | (Description of change) | (Jira/Tag) |

### 7.3 Glossary
- **Charter Partner:** Launch cohort receiving 12-month subscription waiver; auto-convert to paid once `Charter Status End Date` passes.  
- **Clawback:** Negative payout created when a post-payment refund happens; netted on the next batch.  
- **Single Source of Truth:** Airtable `Kickback Operations` base—avoid overwriting via other spreadsheets.

---

> This document is an **operations manual**; obtain ops owner approval before distributing updates.
