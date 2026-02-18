# Kickback Shop Partner Onboarding Manual

This document is a guide for operators responsible for the Kickback Shop Partner Onboarding process.
Operators can follow this manual to perform and verify the entire process: **Application → Approval → Contract → Registration → QR Sending → Part Registration → Individual Contract → Final Verification.**

---

## 1. Concept and Workflow Summary

### 📌 Shop Onboarding Full Flow
1. **Shop Inquiries** (Application Received)  
   ⬇
2. **Admin Approval** (Status = `Accepted` + Check `Click to Send Agreement`)  
   ⬇
3. **Shop Master Contract Sent** (SignWell)  
   ⬇
4. **Contract Completed** → Partners & Contracts Updated  
   ⬇
5. **Shop Intake QR Sent** (Welcome Email)  
   ⬇
6. **Shop Registers Parts via QR**  
   ⬇
7. **Per Product Contract Sent & Signed**  
   ⬇
8. **Final Data Verification + QR Label Sent**

### 📌 Core Process Summary
1.  **Shop Application:** Submit Shop Application Form on the website.
2.  **Admin Approval:** Change status to `Accepted` in Airtable and check `Click to Send Agreement`.
3.  **Contract Sending:** Shop Master Contract is sent automatically.
4.  **Contract Completion:** Partner/Contract records are automatically created upon signing.
5.  **QR Sending:** Intake QR email is sent to the Shop.
6.  **Part Registration:** Shop submits parts using the Shop QR.
7.  **Individual Contract:** Per-product contract is automatically sent and signed.
8.  **Final Registration:** Parts/Contracts data linked and QR Label sent.

---

## 2. Airtable Schema & Table Description

### 2.1 Shop Inquiries (Shop Inquiry Table)
The table where the shop partner submits their initial application.

| Field Name | Type | Description |
| :--- | :--- | :--- |
| **Name** | Text | Contact Person Name |
| **Email** | Text | Email |
| **Phone Number** | Phone number | Contact Number |
| **Name of Shop** | Text | Shop Name |
| **Website** | Text | Website URL |
| **City and State of Shop Location** | Text | Shop Location |
| **Referral Code** | Text | Referral Code |
| **Status** | Single Select | New / Accepted / Rejected |
| **Click to Send Agreement** | Checkbox | **Contract Send Trigger (Required)** |
| **Agreement Sent At** | Date | Contract Sent Time |
| **SignWell Document ID** | Text | E-signature Document ID |
| **Contract Status** | Single Select | Sent / Signed / Completed / Error |
| **Date Signed** | Date | Date Signed |

### 2.2 Partners (Partner Table)
The table where the Shop is registered upon approval.

| Field Name | Type | Description |
| :--- | :--- | :--- |
| **Partner ID** | Formula | Auto-generated ID |
| **Partner Name** | Text | Partner Name |
| **Partner Type** | Single Select | Individual / Shop / Ambassador / Referral Partner |
| **Email** | Text | Email |
| **Shop Type** | Single Select | Charter / Non-Charter / N/A |
| **Subscription Status** | Single Select | Active / Past Due / Canceled / N/A |
| **Charter Status End Date** | Date | Charter End Date |
| **Stripe Connected Account ID** | Text | Stripe Account ID |
| **Stripe Account Status** | Single Select | Verified / Restricted |
| **Referral Code** | Text | Shop's own Referral Code |
| **Onboarding Status** | Checkbox | Onboarding Completion Status |
| **Partner Onboarding Stage** | Formula | Auto-calculated Onboarding Stage |
| **Welcome Email Sent** | Checkbox | QR Sent Status |
| **Payout Enabled** | Checkbox | Payout Enabled Status |
| **Parts Inquiry QR PNG** | Attachment | Intake QR Code |

**🔑 Partner Onboarding Stage Calculation Logic**
* **Shop QR Ready:** Partner Type = Shop + Referral Code exists + Stripe Connected Account ID exists
* **Attributed:** Referral attribution complete
* **Needs Review:** Other status requiring review

### 2.3 Contracts (Contract Table)
Manages Shop contracts and subsequent individual contracts.

| Field Name | Type | Description |
| :--- | :--- | :--- |
| **Contract ID** | Text | Contract Unique ID |
| **Partner** | Linked Record | Link to Partners |
| **Contract Type** | Single Select | Shop Partner / Individual Partner / Ambassador |
| **Date Signed** | Date | Date Signed |
| **Document Link** | Text | SignWell Document Link |
| **SignWell Document ID** | Text | E-signature Document ID |
| **SignWell Template ID** | Text | Template ID |
| **Contract Packet Type** | Formula | Auto-calculated Contract Type |

---

## 3. Detailed Onboarding Procedures (Shop Focus)

### 3.1 Step-by-Step Process Description

**1️⃣ Shop Application**
* Website → "Consign Your Parts" → "Apply as a Shop Partner"
* Submit Shop Info Form
* 📍 **Created Data:** New Row in `Shop Inquiries` table

**2️⃣ Admin Approval**
* Open `Shop Inquiries` view
* Check new record
* Change `Status` to **Accepted**
* **Check the `Click to Send Agreement` checkbox (Required)**
* 📍 **Automation Trigger:** Shop Master Contract Sent

**3️⃣ Shop Master Contract Signing**
* Shop receives contract email
* Click SignWell link
* Complete contract signing
* 📍 **Webhook Action:** Partner Enroll automation runs

**4️⃣ Airtable Auto-Reflection Verification**
* **Partners:**
    * Partner Type = `Shop`
    * Referral Code created
    * Onboarding Status checked
    * Enters `Shop QR Ready` stage if Stripe account is linked
* **Contracts:**
    * Shop Master Contract created
    * SignWell Document ID saved
    * **Document Link saved** (stores the `signing_url`)

**5️⃣ Shop Intake QR Sending**
* `Welcome Email Sent` = false → Automation runs
* **Intake QR** sent to Shop email

**6️⃣ Shop Registers Parts via QR**
* Click Intake Form link in email
* Input part information and submit
* 📍 **Created Data:** `Parts Inquiries` created

**7️⃣ Floor Price Offer → Per Product Contract Sending**
* Input `Floor Price`
* **Check `Click to Send Offer Email` (Required)**
* Shop selects **Accept**
* **Per Product Contract** automatically sent and signing completed

**8️⃣ Final Data Verification**
* Parts created
* Contract ↔ Part connected
* Inquiry ↔ Contract connected
* QR Label created and email sent

---

## 4. Demo Timeline (Video Based)

| Time | Action | Where to Verify |
| :--- | :--- | :--- |
| **00:00 – 00:05** | Start Shop Partner Application | Website |
| **00:05 – 00:43** | Submit Shop Info | Shop Info Submission Form |
| **00:43 – 01:05** | **Admin Approval (Status=Accepted & Checkbox On)** | Airtable |
| **01:05 – 01:40** | Receive Master Contract Email | Gmail |
| **01:40 – 02:05** | Sign Master Contract | SignWell |
| **02:05 – 02:50** | Verify Partner & Contract Creation | Airtable |
| **02:50 – 03:05** | Check Intake QR Email | Gmail |
| **03:05 – 04:13** | Register Part | Intake Form |
| **04:13 – 04:45** | Offer Floor Price | Airtable |
| **04:45 – 05:05** | Accept Offer | Gmail |
| **05:05 – 05:33** | Sign Per Product Contract | SignWell |
| **05:33 – 08:50** | Final Data Integrity Check | Airtable |
| **08:50 – 09:01** | Check QR Label Email | Gmail |

---

## 5. Operator Checklist

### Shop Inquiries
- [ ] Verify new application Row creation
- [ ] Verify Status = Accepted change
- [ ] **Verify `Click to Send Agreement` is checked**
- [ ] Verify SignWell Document ID is saved

### Partners
- [ ] Verify Partner Type = `Shop`
- [ ] Verify Referral Code creation
- [ ] Verify Welcome Email Sent trigger worked normally
- [ ] Verify Stripe account connection status

### Contracts
- [ ] Verify Shop Master Contract creation
- [ ] **Verify Document Link is saved** (stores `signing_url`)
- [ ] Verify Date Signed is recorded

### Parts
- [ ] Verify Part creation
- [ ] **Verify `Click to Send Offer Email` is checked** (Offer email sent)
- [ ] Verify Contract/Inquiry connection
- [ ] Verify QR Label creation

### Email Verification
- [ ] Verify Shop Intake QR email receipt
- [ ] Verify Part QR Label email receipt
- [ ] **Verify NO unnecessary Referral QR email receipt**

---

### 🚨 Core Policy Summary (P1 / P2 Applied)
* **Shop / Ambassador / Referral Partner:** Receive Intake QR Email **(YES)**
* **Individual:** Receive Intake QR Email **(NO, Prohibited)**
* The **Document Link** field must store the `signing_url`.
* **Contract ↔ Partner ↔ Inquiry ↔ Part** links must be connected.
