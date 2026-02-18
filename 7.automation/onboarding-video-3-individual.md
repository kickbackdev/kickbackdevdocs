# Kickback Individual Partner Onboarding Manual

This document is a guide for operators responsible for the Kickback Individual Partner Onboarding process. It is designed to allow anyone to perform and verify the onboarding tasks by following this manual.

---

## 1. Concept and Workflow Summary

### Onboarding Flowchart
**Parts Inquiries (Inquiry Submission)**
`Partner Type = "Individual" Selected`
↓
**Contracts (Contract Generation & Signing)**
↓
**Partners (Partner Record Creation)**
↓
**Parts (Part Record Creation, Intake Class = "Long Term")**

### Core Process Summary
1.  **Submission:** The partner submits the intake form on the website.
2.  **Review & Proposal:** The admin reviews the data in Airtable and proposes a price (Email sent).
3.  **Contract:** When the partner accepts the proposal, the contract is automatically sent for signing.
4.  **Registration:** Upon signing, partner and part information are automatically registered.
5.  **QR Issuance:** A QR code for the part is sent via email.

---

## 2. Airtable Schema & Views Description

Description of the main tables and fields used in the operation.

### 2.1 Parts Inquiries (Onboarding Starting Point)
Where initial data submitted by the partner via the website accumulates.

| Field Name | Type | Description |
| :--- | :--- | :--- |
| **Partner Type** | Single select | Select "Individual" or "Shop" |
| **Email** | Single line text | Partner contact email |
| **Partner Legal Name** | Single line text | Partner legal name |
| **Phone Number** | Phone number | Phone number |
| **Part Name / Description** | Single line text | Part name and detailed description |
| **Proposed Floor Price** | Currency | Floor price proposed by Kickback |
| **Onboarding Stage** | Formula | Onboarding progress stage (Auto-calculated) |
| **Contracts** | Linked record | Linked contract |
| **Parts** | Linked record | Linked part |
| **Referral Code** | Single line text | Referral code |

**Onboarding Stage Levels:**
* `Inquiry Submitted`: Inquiry submitted
* `Offer Accepted / Contract Not Sent`: Offer accepted / Contract not yet sent
* `Contract Sent / Awaiting Completion`: Contract sent / Awaiting signature
* `Contract Completed / Part Not Created`: Contract completed / Part record not created
* `Part Created`: Part creation completed

### 2.2 Contracts
Table integrated with SignWell to manage contract progress.

| Field Name | Type | Description |
| :--- | :--- | :--- |
| **Contract ID** | Single line text | Contract unique ID |
| **Contract Type** | Single select | "Individual", "Individual Partner", etc. |
| **Date Signed** | Date | Date signature was completed |
| **Document Link** | Single line text | Signed document link |
| **Contract Packet Type** | Formula | Packet type (Auto-calculated) |
| **Signed Partner Legal Name** | Single line text | Signed partner name |
| **Signed Authorized Floor Price** | Currency | Signed authorized floor price |
| **SignWell Document ID** | Single line text | SignWell document ID |

**Contract Packet Type:**
* `Individual Full Packet`: Initial full packet for individuals
* `Follow-On Only`: Follow-up contract only
* `Waiting for Approval`: Waiting for approval

### 2.3 Partners
Table where information on partners with completed contracts is stored.

| Field Name | Type | Description |
| :--- | :--- | :--- |
| **Partner Name** | Single line text | Partner name |
| **Partner Type** | Single select | Set to "Individual", etc. |
| **Onboarding Status** | Checkbox | Onboarding completion check |
| **Partner Onboarding Stage** | Formula | Partner stage (Needs Review, Attributed, etc.) |
| **Stripe Connected Account ID** | Single line text | Stripe account ID |
| **Payout Enabled** | Checkbox | Payout enabled status |

### 2.4 Parts
Final registered part information.

| Field Name | Type | Description |
| :--- | :--- | :--- |
| **Part ID** | Autonumber | Part unique ID |
| **Owner** | Linked record | Owner (Partner) link |
| **Intake Class** | Formula | Set to "Long Term" for Individuals |
| **Status** | Single select | Part status |
| **Contract** | Linked record | Linked contract |
| **Parts Inquiry** | Linked record | Initial inquiry link |

---

## 3. Detailed Onboarding Procedures & Demo Guide

### 3.1 Process Detailed Flow
1.  **Submission:** The individual partner accesses the dedicated menu on the Kickback website, fills out the **Intake Form**, and submits it.
2.  **Proposal:** The admin reviews the submitted information in the `Parts Inquiries` table in Airtable. The admin enters the price in `Proposed Floor Price` and checks the **Send Proposal Email checkbox** to send the email.
3.  **Acceptance:** When the partner clicks the **'Accept'** button in the received proposal email, the **Individual Packet Contract** is automatically sent via email.
4.  **Signing:** The partner completes the contract signing.
5.  **Reflection:** The contract completion status and referral information are reflected in the `Partners` and `Parts` tables in Airtable.
6.  **QR Issuance:** Emails containing the **Part QR Code** are sent to the partner's email. (Confirmation of receiving 2 normal emails is required).
7.  **Follow-up:** Afterwards, when the partner scans the issued QR code and inputs information, a **Per-product Contract** is sent, and the remaining registration procedures are executed.

### 3.2 Demo Timeline (Video Guide)

| Time | Action | Where to verify |
| :--- | :--- | :--- |
| **00:00 – 00:10** | **Open Parts Inquiry Submission Form** (Individual Intake Form) | Website Form Screen |
| **00:10 – 00:55** | **Input Information** (Contact, Partner Type, Part Info, etc.) | Form Fields |
| **00:55 – 01:05** | **Submit Form** (Check submission completion screen) | Completion Page |
| **01:05 – 01:20** | Switch to email inbox and wait for automated email | Email Inbox |
| **01:20 – 01:30** | **Check Airtable & Process Proposal** (Check new record in `Parts Inquiries` & propose price) | Airtable |
| **01:30 – 01:45** | **Open Floor Price / Offer Email** | Email Content |
| **01:45 – 01:55** | **Submit Response Form (Accept)** and check confirmation | Completion Page |
| **01:55 – 02:10** | Return to email inbox and wait for SignWell contract | Email Inbox |
| **02:10 – 02:45** | **Open SignWell Contract** and review document | SignWell Document Screen |
| **02:45 – 02:55** | Enter required info in signature fields and **Complete Signing** | SignWell Signature Section |
| **02:55 – 03:05** | Check signature completion confirmation screen | SignWell Completion Page |
| **03:05 – 03:30** | **Confirm Airtable Data Creation** (Check `Partners`, `Contracts`, `Parts` records) | Airtable |
| **03:30 – 04:20** | **Validate Linked Data** (Check Contract↔Partner, Part↔Contract links & signed data) | Airtable |
| **04:20 – 05:05** | **Confirm QR Label Email Arrival** (Check for QR image attachment) | Email Inbox |
| **05:05 onwards** | Scan received QR → Submit Pre-filled Form → Proceed with follow-up | Form & Email |

---

## 4. Operator Checklist & Policy

Items that must be verified during operation.

### ✅ Operator Checklist

**Parts Inquiries**
- [ ] Has a new Row been created?
- [ ] Is the Offer/Price field entered correctly?
- [ ] Does the proposal email trigger (checkbox) work normally?
- [ ] Does the Individual Packet send trigger work normally after "Accepted"?

**Contracts**
- [ ] Has the Individual Packet Contract been created/updated?
- [ ] Has the Document Link (`signing_url`) been saved?
- [ ] Are Part and Inquiry links connected correctly?

**Parts**
- [ ] Has the Part record been created/updated?
- [ ] If referral attribution information exists, has it been reflected?
- [ ] Does the Per-product follow-on contract loop work normally?

**Email (Verification)**
- [ ] **Did 2 QR-related emails arrive in the individual partner's email?**
    * **Normal Criteria:** ① Document signing completion notification, ② Part QR label issuance email (Total 2 emails).
    * **Abnormal Criteria:** If a **"Referral QR issuance email"** (for Ambassadors or Shops) arrives, it is an **ERROR**.

### 🚨 Core Policy (Policy P1 Core)

* **Core Policy:** Individual partners **must NEVER receive the "Referral QR email".**
* **Allowed Scope:** By policy, only **"Parts-related QR and emails"** are allowed for Individual partners.
* **Standard for 2 Normal Emails:**
    1.  **Mail A:** SignWell document signing completion notification
    2.  **Mail B:** Part QR label issuance email
