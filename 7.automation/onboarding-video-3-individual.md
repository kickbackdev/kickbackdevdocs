# Kickback Individual Partner Onboarding Manual

## 1. Process Flow

1. The individual partner accesses the Kickback website, fills out the intake form in the dedicated menu, and submits it.
2. The admin reviews the submitted information in the Parts Inquiries table in Airtable, proposes a price, and triggers the proposal email by checking the checkbox.
3. When the partner selects 'Accept' in the received proposal email, the Individual Packet contract is sent via email.
4. The partner completes the Individual Packet contract.
5. The contract completion status, along with referral information, is reflected in the Partners and Parts tables in Airtable.
6. Two emails containing the part QR code are sent to the individual partner's email.
7. Afterwards, when the partner scans the issued individual QR code to fill out and submit the pre-filled intake form, a per-product contract is sent, and the remaining part registration procedures are executed.

## 2. Policy (P1 Core)

* **Core Policy:** Individual partners must not receive the "Referral QR email".
* **Allowed Scope:** By policy, only "Parts-related QR/emails" are allowed for Individual partners.
* **Standard for 2 Normal Emails:**
    * Mail A: SignWell document signing completion notification email
    * Mail B: Part QR label issuance email

## 3. Demonstration Timeline

<table>
  <thead>
    <tr>
      <th>Time</th>
      <th>Action</th>
      <th>Where to Look / Verify</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>00:00 – 00:10</td>
      <td>Open the Parts Inquiry Submission Form (Individual intake form).</td>
      <td>Form screen</td>
    </tr>
    <tr>
      <td>00:10 – 00:55</td>
      <td>Fill out contact and part details (partner type, email/phone, part info, etc.).</td>
      <td>Form fields</td>
    </tr>
    <tr>
      <td>00:55 – 01:05</td>
      <td>Submit the form; confirmation screen appears.</td>
      <td>Form submission completion page</td>
    </tr>
    <tr>
      <td>01:05 – 01:20</td>
      <td>Switch to email inbox to wait for automated emails.</td>
      <td>Email inbox</td>
    </tr>
    <tr>
      <td>01:20 – 01:30</td>
      <td>Open Airtable Parts Inquiries view to review the new record and process the proposal.</td>
      <td>Airtable</td>
    </tr>
    <tr>
      <td>01:30 – 01:45</td>
      <td>Open the Floor Price / Offer email.</td>
      <td>Email content</td>
    </tr>
    <tr>
      <td>01:45 – 01:55</td>
      <td>Submit the response form (Accept) and check the confirmation screen.</td>
      <td>Response form completion page</td>
    </tr>
    <tr>
      <td>01:55 – 02:10</td>
      <td>Return to email inbox and wait for the SignWell contract email.</td>
      <td>Email inbox</td>
    </tr>
    <tr>
      <td>02:10 – 02:45</td>
      <td>Open the SignWell contract and review the document.</td>
      <td>SignWell document screen</td>
    </tr>
    <tr>
      <td>02:45 – 02:55</td>
      <td>Enter required information in signature fields and complete signing.</td>
      <td>SignWell signature section</td>
    </tr>
    <tr>
      <td>02:55 – 03:05</td>
      <td>Check the signature completion confirmation screen.</td>
      <td>SignWell completion page</td>
    </tr>
    <tr>
      <td>03:05 – 03:30</td>
      <td>Go back to Airtable and confirm Partners, Contracts, and Parts records were created.</td>
      <td>Airtable</td>
    </tr>
    <tr>
      <td>03:30 – 04:20</td>
      <td>Validate linked fields (Contract ↔ Partner, Part ↔ Contract, Part ↔ Inquiry) and signed data.</td>
      <td>Airtable</td>
    </tr>
    <tr>
      <td>04:20 – 05:05</td>
      <td>Confirm the Part QR label email arrived with the QR Label image attached.</td>
      <td>Email inbox</td>
    </tr>
    <tr>
      <td>05:05 onwards</td>
      <td>Scan the received QR code, submit the pre-filled form, and proceed with follow-on steps.</td>
      <td>Form and email</td>
    </tr>
  </tbody>
</table>

## 4. Operator Checklist

### Parts Inquiries
- [ ] New row created
- [ ] Offer/Price proposal field entered
- [ ] Proposal email trigger checkbox works normally
- [ ] Individual Packet send trigger works normally after Accepted

### Contracts
- [ ] Individual Packet Contract created/updated
- [ ] Document Link (signing_url) saved
- [ ] Part/Inquiry links reflected

### Parts
- [ ] Part created/updated
- [ ] Referral attribution snapshot reflected (if applicable)
- [ ] Per-product follow-on contract loop works normally

### Email
- [ ] 2 QR-related emails arrived in the individual's email
    * Normal criteria: Only the document completion email and part QR issuance email should arrive.
    * Abnormal criteria: If a referral QR issuance email for ambassadors or shops arrives, it is considered an error.

## 5. Individual Onboarding Schema Overview

### Parts Inquiries (Onboarding Starting Point)
<table>
  <thead>
    <tr>
      <th>Field Name</th>
      <th>Type</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Partner Type</td>
      <td>Single select</td>
      <td>Select "Individual" or "Shop"</td>
    </tr>
    <tr>
      <td>Email</td>
      <td>Single line text</td>
      <td>Contact email</td>
    </tr>
    <tr>
      <td>Partner Legal Name</td>
      <td>Single line text</td>
      <td>Partner's legal name</td>
    </tr>
    <tr>
      <td>Phone Number</td>
      <td>Phone number</td>
      <td>Phone number</td>
    </tr>
    <tr>
      <td>Part Name / Description</td>
      <td>Single line text</td>
      <td>Part name and description</td>
    </tr>
    <tr>
      <td>Proposed Floor Price</td>
      <td>Currency</td>
      <td>Proposed floor price</td>
    </tr>
    <tr>
      <td>Onboarding Stage</td>
      <td>Formula</td>
      <td>Auto-calculated onboarding stage</td>
    </tr>
    <tr>
      <td>Contracts</td>
      <td>Linked record</td>
      <td>Contract link</td>
    </tr>
    <tr>
      <td>Parts</td>
      <td>Linked record</td>
      <td>Part link</td>
    </tr>
    <tr>
      <td>Referral Code</td>
      <td>Single line text</td>
      <td>Referral code</td>
    </tr>
  </tbody>
</table>

**Onboarding Stage Levels:**
* Inquiry Submitted
* Offer Accepted / Contract Not Sent
* Contract Sent / Awaiting Completion
* Contract Completed / Part Not Created
* Part Created

### Contracts
<table>
  <thead>
    <tr>
      <th>Field Name</th>
      <th>Type</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Contract ID</td>
      <td>Single line text</td>
      <td>Contract unique ID</td>
    </tr>
    <tr>
      <td>Contract Type</td>
      <td>Single select</td>
      <td>"Individual", "Individual Partner", etc.</td>
    </tr>
    <tr>
      <td>Date Signed</td>
      <td>Date</td>
      <td>Date signed</td>
    </tr>
    <tr>
      <td>Document Link</td>
      <td>Single line text</td>
      <td>Document link</td>
    </tr>
    <tr>
      <td>Contract Packet Type</td>
      <td>Formula</td>
      <td>Auto-calculated packet type</td>
    </tr>
    <tr>
      <td>Signed Partner Legal Name</td>
      <td>Single line text</td>
      <td>Signed partner legal name</td>
    </tr>
    <tr>
      <td>Signed Partner Type</td>
      <td>Single line text</td>
      <td>Signed partner type</td>
    </tr>
    <tr>
      <td>Signed Authorized Floor Price</td>
      <td>Currency</td>
      <td>Signed authorized floor price</td>
    </tr>
    <tr>
      <td>SignWell Document ID</td>
      <td>Single line text</td>
      <td>SignWell document ID</td>
    </tr>
    <tr>
      <td>Parts</td>
      <td>Linked record</td>
      <td>Part link</td>
    </tr>
    <tr>
      <td>Partner</td>
      <td>Linked record</td>
      <td>Partner link</td>
    </tr>
  </tbody>
</table>

**Contract Packet Type:**
* Individual Full Packet
* Follow-On Only
* Waiting for Approval

### Partners
<table>
  <thead>
    <tr>
      <th>Field Name</th>
      <th>Type</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Partner ID</td>
      <td>Formula</td>
      <td>Partner unique ID</td>
    </tr>
    <tr>
      <td>Partner Name</td>
      <td>Single line text</td>
      <td>Partner name</td>
    </tr>
    <tr>
      <td>Partner Type</td>
      <td>Single select</td>
      <td>"Individual", "Shop", "Ambassador", etc.</td>
    </tr>
    <tr>
      <td>Email</td>
      <td>Single line text</td>
      <td>Email</td>
    </tr>
    <tr>
      <td>Onboarding Status</td>
      <td>Checkbox</td>
      <td>Onboarding completion status</td>
    </tr>
    <tr>
      <td>Partner Onboarding Stage</td>
      <td>Formula</td>
      <td>Partner onboarding stage</td>
    </tr>
    <tr>
      <td>Referral Code</td>
      <td>Single line text</td>
      <td>Referral code</td>
    </tr>
    <tr>
      <td>Stripe Connected Account ID</td>
      <td>Single line text</td>
      <td>Stripe account ID</td>
    </tr>
    <tr>
      <td>Payout Enabled</td>
      <td>Checkbox</td>
      <td>Payout enabled status</td>
    </tr>
  </tbody>
</table>

**Partner Onboarding Stage:**
* Needs Review
* Attributed
* Active Referrer

### Parts
<table>
  <thead>
    <tr>
      <th>Field Name</th>
      <th>Type</th>
      <th>Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>Part ID</td>
      <td>Autonumber</td>
      <td>Part unique ID</td>
    </tr>
    <tr>
      <td>Owner</td>
      <td>Linked record</td>
      <td>Owner partner</td>
    </tr>
    <tr>
      <td>Owner Partner Type</td>
      <td>Lookup</td>
      <td>Owner partner type</td>
    </tr>
    <tr>
      <td>Intake Class</td>
      <td>Formula</td>
      <td>Set to "Long Term" for Individual</td>
    </tr>
    <tr>
      <td>Status</td>
      <td>Single select</td>
      <td>Part status</td>
    </tr>
    <tr>
      <td>Contract</td>
      <td>Linked record</td>
      <td>Contract link</td>
    </tr>
    <tr>
      <td>Parts Inquiry</td>
      <td>Linked record</td>
      <td>Parts inquiry link</td>
    </tr>
  </tbody>
</table>

**Individual Onboarding Flow Summary:**
Parts Inquiries -> Partner Type = "Individual" -> Contracts -> Partners -> Parts (Intake Class = "Long Term")****
