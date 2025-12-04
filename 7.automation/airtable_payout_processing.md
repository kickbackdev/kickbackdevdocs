# Airtable Payout Processing – Complete Operations Manual

## 📋 Prerequisites

Before you start any payout processing:

- Access to the Airtable base: **“Kickback Operations”**
- Edit permissions for the following tables:
  - `Sales`
  - `Payouts`
  - `Partners`
- Access to the **Stripe Dashboard** (required for sending transfers)
- (Important limitation) **SignWell contract template**:
  - The current SignWell agreement template does **not** yet have a dedicated field to capture **Referral information**.
  - For now, all referral relationships and referral-based commissions must be managed and recorded **only in Airtable** (e.g., in the `Partners` table and `Payouts` logic), not in the contract form itself.
  - Updating the SignWell template to include a “Referral” field is a pending task for the ops/legal team.

---

## 🔄 Step-by-Step Guide to Process Payouts

### Step 1: Create a Sales Record

**Location:** `Sales` table

1. Go to the **`Sales`** table.
2. Click **“+ Add record”**.
3. Fill in the required fields:
   - **Part Sold**: Select the sold part from the dropdown.
   - **Gross Sale Price**: Enter the sale price as a number (e.g., `100.00`).
   - **Payout Status**: Set to **`Payable`**.
   - **Payout Eligibility Date**: Enter **today’s date** or the desired date when the sale becomes eligible for payout.
4. Save the record.

> Conceptually, setting `Payout Status = Payable` is the “start signal” for the automation.

---

### Step 2: Wait for Zapier Automation

1. Zapier either:
   - Runs automatically at a scheduled time (e.g., every morning at 9:00), or  
   - Can be triggered manually if you have permission to run the Zap.
2. In the **`Sales`** table, watch the `Payout Status` of the record you just created:
   - `Payable` → `Processing` means Zapier has picked it up and is working.
   - **Do not edit** the sale while status is `Processing`.
3. When processing is complete:
   - New records will appear in the **`Payouts`** table.
   - Each commission type (Owner Share, Referral, Ambassador, etc.) will show up as a **separate payout record**.

---

### Step 3: Review Generated Payout Records

**Location:** `Payouts` table

1. Go to the **`Payouts`** table.
2. Switch to the **“Today payouts”** grid view (or equivalent operational view).
3. Filter or visually check for records where **`Status = Payable`**.
4. For each payout record, verify:
   - **Payee**: The partner or person who should receive the payout.
   - **Payout Amount**: The amount to be transferred.
   - **Commission Type**: e.g., `Owner Share`, `Referral Fee`, `Ambassador`, etc.
   - **Sale**: Linked sale record for reference.

> Note on referrals:  
> If the **Commission Type** is `Referral` but the underlying relationship is not documented in SignWell (there is no referral field yet), confirm that it is correctly captured in Airtable (e.g., partner’s referrer, referral program rule) before proceeding.

---

### Step 4: Check Stripe Account Info and Send Transfers

Repeat this for **each** payout record that needs to be paid.

1. From a payout record in **`Payouts`**, click the **`Payee`** link.
2. You will be taken to the corresponding partner record in the **`Partners`** table.
3. Locate and copy the **`Stripe Connected Account ID`** field.
   - If this field is empty:
     - **Do not** attempt to pay that payout yet.
     - Contact the partner and request that they complete **Stripe Express** onboarding.
     - Update the `Stripe Connected Account ID` field once onboarding is complete.
4. Open the **Stripe Dashboard** in a separate browser tab.
5. Navigate to **Connect → Transfers**.
6. Click **“Create transfer”**.
7. Fill in the transfer details:
   - **Destination**: Paste the **Stripe Connected Account ID**.
   - **Amount**: Enter the **Payout Amount** (match Airtable exactly, including cents).
   - **Currency**: `USD` (or your operating currency).
   - **Transfer group** (optional but recommended): Enter a payout batch ID or note (e.g., `settlement_batch_2025-12-03`).
8. Click **“Create transfer”** to execute the transfer.

> Tip: It’s often easiest to process payouts grouped by **Settlement Batch** or by date to minimize errors.

---

### Step 5: Mark Payouts as Paid in Airtable

After all related Stripe transfers are successfully sent:

1. Go back to the **`Sales`** table.
2. Find the sale(s) that have now been fully paid out.
3. Update:
   - **`Payout Status`**: Change from `Processing` → **`Paid`**.
   - **`Payout Date`**: Set to today’s date (the actual transfer date).
4. Save the changes.

> Once marked as `Paid`, this sale is considered fully settled in the system.

---

## 🚨 Troubleshooting Guide

### Zapier Is Not Running or Payouts Are Not Generated

- **Check the Sale Record:**
  - `Payout Status` must be **`Payable`**.
  - `Payout Eligibility Date` must be **today or in the past**.
- **If conditions are incorrect:**
  - Fix the fields and wait for the next scheduled run (e.g., 9:00 AM), or
  - Manually re-run the Zap if you have access.

---

### Stripe Connected Account ID Is Missing

- **In the `Partners` table:**
  - Check the partner’s record for a **`Stripe Connected Account ID`**.
- **If empty or invalid:**
  - Ask the partner to complete Stripe Express onboarding (using your standard onboarding link).
  - Once they finish, update the `Stripe Connected Account ID` field.
  - Only then proceed with the corresponding payouts.

---

### Payout Amount Is Zero or Looks Wrong

- Possible reasons:
  - Incorrect **`Gross Sale Price`** on the sale record.
  - Wrong **Part Sold** (with different commission settings).
  - A bug in the commission calculation logic.
- **Actions:**
  - Double-check the `Part Sold` and `Gross Sale Price` on the `Sales` record.
  - Verify internal commission rules (owner share %, referral %, ambassador %, etc.).
  - If needed, escalate to the person responsible for the Airtable/Zapier calculation logic.

---

### Referral Information Is Missing from Contracts (SignWell Limitation)

- **Current limitation:**
  - The SignWell contract template **does not** include a dedicated field or section for **Referral** information (e.g., who referred whom, referral code, referral tier).
- **Impact:**
  - Legal contracts do not currently show the referral linkage that is used for calculating referral commissions.
  - All referral relationships are instead maintained inside **Airtable** (e.g., partner-to-referrer mapping).
- **Operational workaround:**
  - Ensure referral relationships are always:
    - Captured in Airtable (`Partners` table and/or `Payouts` logic), and
    - Confirmed before approving referral payouts.
- **To-do (for legal/ops):**
  - Update the SignWell contract template to add:
    - A “Referral name/ID” field, or
    - A “Referral program details” section.
  - Once added, align Airtable fields and contract fields so both reflect the same referral data.

---

## 🤖 Automation Options (For Reference)

There is a separate Zap that can automate the **Stripe transfer** step.

If the automated Stripe payout Zap is enabled:

- Steps **4–5** (Stripe transfer + updating Sales status to `Paid`) can be automated.
- Operators will typically only need to:
  - Create sales (`Sales` table)
  - Verify payout records (`Payouts` table)
  - Monitor for errors
- Manual intervention is required only when:
  - Stripe transfers fail
  - Connected Account IDs are missing
  - Referral data is inconsistent or missing

**How to enable:**  
Ask the engineering or automation owner to turn on the Stripe payout Zap and confirm the error-handling rules.

---

## ✅ Payout Run Checklist

Use this checklist for every payout run:

- [ ] Sales records are correctly created (Part, Gross Sale Price, Payout Status, Eligibility Date).
- [ ] `Payout Status` has moved from `Payable` → `Processing` → payouts are generated.
- [ ] All required `Payouts` records (Owner Share, Referral, Ambassador, etc.) exist.
- [ ] For each **Payee**, a valid **Stripe Connected Account ID** is present in `Partners`.
- [ ] Payout amounts match your commission rules and look correct.
- [ ] Stripe transfers have been successfully created and sent.
- [ ] `Payout Status` in `Sales` is updated to **`Paid`** and `Payout Date` is set.
- [ ] For any **Referral** payouts:
  - [ ] Referral relationship is clearly recorded in Airtable.
  - [ ] You understand that the SignWell contract currently does **not** show referral info and this is a known limitation.

---

# 🎬 Video Walkthrough – Airtable Payout Automation Flow

This section explains the same flow as shown in the screen recording  
`화면 기록 2025-12-03 오후 4.40.46.mov`.

You can watch the video and follow along with the timestamps below.

{% embed url="https://youtu.be/2dGNPOQHbag" %}

---

## 1. Creating the Sale and Starting the Payout Flow (00:00 – 00:30)

- **Screen:** `Sales` table (sales records)
- **What you see:** A new sale (e.g., **Sale ID 46**) has been added.
- **Key field:** `Payout Status` is set to **`Payable`**.
  - This acts as the **start signal** for the payout automation.
- **System behavior:**
  - The automation scans for sales where:
    - `Payout Status = Payable`, and
    - `Payout Eligibility Date` is today or earlier.
  - Matching sales are picked up and queued for payout processing.

---

## 2. Creating a Settlement Batch (00:31 – 00:37)

- **Screen:** `Settlement Batches` table
- **System behavior:**
  - When payout processing starts, the system creates a new **Settlement Batch** record.
- **What you see:**
  - A new batch row appears with today’s date.
  - `Status` is set to **`Running`**.
- **Meaning:**
  - “All payouts generated from this run will be grouped under this batch,” making it easier to track, audit, and reconcile later.

---

## 3. Auto-Calculating Detailed Payouts (00:38 – 01:03)

- **Screen:** `Payouts` table
- **System behavior:**
  - The system takes the gross sale amount (e.g., `$100`) and automatically splits it according to your commission rules.
- **What you see:**
  - For the sale (e.g., **Sale ID 56**), multiple payout records are created. Typical examples:
    1. **Owner Share**: e.g., `$60.00` to the part owner.
    2. **Ambassador / Referral**: additional commission to a referrer, if configured.
- **Linking:**
  - Each payout record is linked to:
    - The appropriate **Settlement Batch**.
    - The correct **Payee** (partner/person).
    - The original **Sale**.
- **Note on referrals in this context:**
  - Even though the contract (SignWell) does not show referral info yet, all referral payouts you see here are derived from the Airtable configuration.
  - Always rely on Airtable relationships and commission rules when validating referral payouts.

---

## 4. Finalizing Payout Data and Sending Transfers (01:04 – 01:10)

- **Screen:** `Payouts` table, then Stripe Dashboard.
- **System behavior:**
  - At this point, the system has completed all complex math:
    - Who gets paid
    - How much
    - Which batch it belongs to
- **Operator task (critical part):**
  - Use the payout records as the **source of truth**:
    - `Payee` → Stripe Connected Account ID
    - `Payout Amount` → Transfer amount
  - Manually create transfers in Stripe (unless Stripe automation is enabled).
- **After transfer:**
  - Mark the sale as `Paid` in `Sales`.
  - Optionally, mark individual payout records as `Paid` or `Completed` if your schema supports it.

---

## 💡 High-Level Summary (What Operators Should Remember)

1. **Start the process:**  
   - Create a sale and set `Payout Status = Payable`.

2. **Let automation work:**  
   - The system:
     - Creates a **Settlement Batch**.
     - Calculates all commissions.
     - Generates detailed **Payouts** records.

3. **Execute money movement:**  
   - Use the data in `Payouts` to:
     - Send transfers via Stripe.
     - Update `Sales` (and optionally `Payouts`) to `Paid`.

4. **Referral caveat (very important):**  
   - Referral logic is correct in Airtable.
   - The SignWell contract currently does **not** show who the referrer is.
   - Until the contract template is updated, always cross-check referral relationships in Airtable, not in the signed PDF.

---
