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

## 2. Process Flow Details

1.  The individual partner accesses the Kickback website, fills out the **Intake Form** in the dedicated menu, and submits it.
2.  The admin reviews the submitted information in the `Parts Inquiries` table in Airtable. The admin enters the **Proposed Floor Price** and checks the **Send Proposal Email** checkbox to trigger the email.
3.  When the partner selects **'Accept'** in the received proposal email, the **Individual Packet Contract** is automatically sent via email.
4.  The partner completes the Individual Packet contract signing.
5.  The contract completion status and referral information are reflected in the `Partners` and `Parts` tables in Airtable.
6.  **Two emails** containing the **Part QR Code** are sent to the individual partner's email. (Verification of 2 normal emails required).
7.  Afterwards, when the partner scans the issued Individual QR code to fill out and submit the pre-filled intake form, a **Per-product Contract** is sent, and the remaining part registration procedures are executed.

---

## 3. Policy (P1 Core)

* **Core Policy:** Individual partners **must NOT** receive the "Referral QR email".
* **Allowed Scope:** By policy, only **"Parts-related QR/emails"** are allowed for Individual partners.
* **Standard for 2 Normal Emails:**
    1.  **Mail A:** SignWell document signing completion notification email.
    2.  **Mail B:** Part QR label issuance email.

---

## 4. Demonstration Timeline

| Time | Action | Where to Look / Verify |
| :--- | :--- | :--- |
| **00:00 – 00:10** | Open the Parts Inquiry Submission Form (Individual intake form). | Form screen |
| **00:10 – 00:55** | Fill out contact and part details (partner type, email/phone, part info, etc.). | Form fields |
| **00:55 – 01:05** | Submit the form; confirmation screen appears. | Form submission completion page |
| **01:05 – 01:20** | Switch to email inbox to watch automated emails arrive. | Email inbox |
| **01:20 – 01:30** | Open Airtable `Parts Inquiries` view to review the new record and process the proposal. | Airtable |
| **01:30 – 01:45** | Open the **Floor Price / Offer email**. | Email content |
| **01:45 – 01:55** | Submit the response form (**Accept**) and check the confirmation screen. | Response form completion page |
| **01:55 – 02:10** | Return to email inbox and wait for the **SignWell agreement email**. | Email inbox |
| **02:10 – 02:45** | Open the SignWell agreement and review the document. | SignWell document screen |
| **02:45 – 02:55** | Enter required information in signature fields and **complete signing**. | SignWell signature section |
| **02:55 – 03:05** | Check the signature completion confirmation screen. | SignWell completion page |
| **03:05 – 03:30** | Go back to Airtable and confirm `Partners`, `Contracts`, and `Parts` records were created. | Airtable |
| **03:30 – 04:20** | Validate linked fields (Contract ↔ Partner, Part ↔ Contract, Part ↔ Inquiry) and signed data. | Airtable |
| **04:20 – 05:05** | Confirm the **Part QR label email** arrived with the QR Label image attached. | Email inbox |
| **05:05 onwards** | Scan the received QR code, submit the pre-filled form, and proceed with follow-on steps. | Form and email |

---

## 5. Operator Checklist

### Parts Inquiries
- [ ] New row created.
- [ ] Offer/Price proposal field entered correctly.
- [ ] Proposal email trigger checkbox works normally.
- [ ] Individual Packet send trigger works normally after "Accepted".

### Contracts
- [ ] Individual Packet Contract created/updated.
- [ ] Document Link (`signing_url`) saved.
- [ ] Part and Inquiry links reflected correctly.

### Parts
- [ ] Part record created/updated.
- [ ] Referral attribution snapshot reflected (if applicable).
- [ ] Per-product follow-on contract loop works normally.

### Email
- [ ] **Did 2 QR-related emails arrive in the individual's email?**
    * **Normal Criteria:** Only the document completion email and part QR issuance email should arrive.
    * **Abnormal Criteria:** If a **referral QR issuance email** (for ambassadors or shops) arrives, it is considered an **ERROR**.

---

## 6. Individual Onboarding Schema Overview

### 6.1 Parts Inquiries (Onboarding Starting Point)
| Field Name | Type | Description |
| :--- | :--- | :--- |
| **Partner Type** | Single select | Select "Individual" or "Shop" |
| **Email** | Single line text | Contact email |
| **Partner Legal Name** | Single line text | Partner's legal name |
| **Phone Number** | Phone number | Phone number |
| **Part Name / Description** | Single line text | Part name and description |
| **Proposed Floor Price** | Currency | Proposed floor price |
| **Onboarding Stage** | Formula | Auto-calculated onboarding stage |
| **Contracts** | Linked record | Contract link |
| **Parts** | Linked record | Part link |
| **Referral Code** | Single line text | Referral code |

**Onboarding Stage Levels:**
* `Inquiry Submitted`
* `Offer Accepted / Contract Not Sent`
* `Contract Sent / Awaiting Completion`
* `Contract Completed / Part Not Created`
* `Part Created`

### 6.2 Contracts
| Field Name | Type | Description |
| :--- | :--- | :--- |
| **Contract ID** | Single line text | Contract unique ID |
| **Contract Type** | Single select | "Individual", "Individual Partner", etc. |
| **Date Signed** | Date | Date signed |
| **Document Link** | Single line text | Document link |
| **Contract Packet Type** | Formula | Auto-calculated packet type |
| **Signed Partner Legal Name** | Single line text | Signed partner legal name |
| **Signed Partner Type** | Single line text | Signed partner type |
| **Signed Authorized Floor Price** | Currency | Signed authorized floor price |
| **SignWell Document ID** | Single line text | SignWell document ID |
| **Parts** | Linked record | Part link |
| **Partner** | Linked record | Partner link |

**Contract Packet Type:**
* `Individual Full Packet`
* `Follow-On Only`
* `Waiting for Approval`

### 6.3 Partners
| Field Name | Type | Description |
| :--- | :--- | :--- |
| **Partner ID** | Formula | Partner unique ID |
| **Partner Name** | Single line text | Partner name |
| **Partner Type** | Single select | "Individual", "Shop", "Ambassador", etc. |
| **Email** | Single line text | Email |
| **Onboarding Status** | Checkbox | Onboarding completion status |
| **Partner Onboarding Stage** | Formula | Partner onboarding stage |
| **Referral Code** | Single line text | Referral code |
| **Stripe Connected Account ID** | Single line text | Stripe account ID |
| **Payout Enabled** | Checkbox | Payout enabled status |

**Partner Onboarding Stage:**
* `Needs Review`
* `Attributed`
* `Active Referrer`

### 6.4 Parts
| Field Name | Type | Description |
| :--- | :--- | :--- |
| **Part ID** | Autonumber | Part unique ID |
| **Owner** | Linked record | Owner partner |
| **Owner Partner Type** | Lookup | Owner partner type |
| **Intake Class** | Formula | Set to **"Long Term"** for Individual |
| **Status** | Single select | Part status |
| **Contract** | Linked record | Contract link |
| **Parts Inquiry** | Linked record | Parts inquiry link |

### **Individual Onboarding Flow Summary**
`Parts Inquiries` → `Partner Type = "Individual"` → `Contracts` → `Partners` → `Parts` (Intake Class = "Long Term")
