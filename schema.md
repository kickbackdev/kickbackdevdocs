# Airtable-based Central Data Architecture

<img src="images/logokick.avif" alt="Kickback logo" width="120" />

Automation quality depends on schema quality. Airtable serves as the **single source of truth**, so design a robust relational schema that mirrors all business rules.

The **“Kickback Operations”** base links partners, parts, sales, settlements, and subscriptions to eliminate duplication and enable reliable finance reporting and automation logic.

### 1.1. Airtable Base Schema

**Table 1: Partners (Unified partner registry)**

Manage every party that receives payouts or is charged subscriptions in one table.

| **Field Name**              | **Field Type**          | **Description**                                                           |
| --------------------------- | ----------------------- | ------------------------------------------------------------------------- |
| Partner ID                  | Primary Field (Formula) | Unique key. `CONCATENATE({Partner Name}, "-", RECORD_ID())`               |
| Partner Name                | Single Line Text        | Partner or business name.                                                 |
| Partner Type                | Single Select           | `Individual`, `Shop`, `Ambassador`                                        |
| Email                       | Email                   | Used as onboarding key.                                                   |
| Stripe Connected Account ID | Single Line Text        | For payouts to the partner.                                               |
| Stripe Customer ID          | Single Line Text        | For charging subscriptions. Separate from connected account.              |
| SignWell Document ID        | Single Line Text        | Signed contract document ID.                                              |
| Referred By                 | Link to Partners        | Self-reference for multi-level commissions.                               |
| Referrals                   | Link to Partners        | Reverse link of “Referred By.”                                            |
| Owned Parts                 | Link to Parts           | Parts owned by this partner.                                              |
| Sales                       | Link to Sales           | Sales involving this partner.                                             |
| Payout Line Items           | Link to Payouts         | All payout line items related to the partner.                             |
| Shop Type                   | Single Select           | For `Shop`: `Charter` or `Non-Charter`.                                   |
| Subscription Status         | Single Select           | For Non-Charter shops: `Active`, `Past Due`, `Canceled`, `N/A`.           |
| Charter Status End Date     | Date                    | End of 1-year fee waiver for Charter shops. Drives auto-conversion.       |
| Stripe Account Status       | Single Select           | `Verified`, `Restricted`, etc. Enables proactive ops on failing accounts. |

**Table 2: Parts (Inventory control)**

Keep physical and digital flows aligned.

| **Field Name**        | **Field Type**             | **Description**                                                                                                                                                                                                       |
| --------------------- | -------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Part ID               | Primary Field (Autonumber) | Unique identifier.                                                                                                                                                                                                    |
| QR Code ID            | Single Line Text           | Binds physical part to digital record for scan-based updates.                                                                                                                                                         |
| Item Name             | Single Line Text           | Part name.                                                                                                                                                                                                            |
| Owner                 | Link to Partners           | Owner of the part.                                                                                                                                                                                                    |
| Status                | Single Select              | Lifecycle:¹ `Awaiting Contract` → `Contract Signed - Awaiting Receipt` → `QR Generated` → `Picked Up` → `Received - Pending Inspection` → `Warehouse Verified` → `In Stock` → `Listed for Sale` → `Sold` → `Returned` |
| Part Description      | Long Text                  | Details.                                                                                                                                                                                                              |
| Vehicle Compatibility | Long Text                  | Make/model/years.                                                                                                                                                                                                     |
| Condition             | Single Select              | `New`, `Like New`, `Used - Good`, `As-Is`.                                                                                                                                                                            |
| Location Code         | Single Line Text           | Warehouse slot.                                                                                                                                                                                                       |
| Date Added            | Date                       | Registration date.                                                                                                                                                                                                    |
| Sale Record           | Link to Sales              | Linked sale when sold.                                                                                                                                                                                                |

¹ `Status` is granular to bridge contract vs physical intake. Only after `Received - Pending Inspection` passes does a part become `In Stock`.

**Table 3: Sales (Transactional ledger)**

Each sale is a standalone financial event, with refund fields.

> Added field: Delivered Date for payout-window logic and SLA reporting.

| **Field Name**              | **Field Type**             | **Description**                                                                                                  |
| --------------------------- | -------------------------- | ---------------------------------------------------------------------------------------------------------------- |
| Sale ID                     | Primary Field (Autonumber) | Unique identifier.                                                                                               |
| Part Sold                   | Link to Parts              | The sold part.                                                                                                   |
| Marketplace Source          | Single Select              | Channel, e.g., `Garage App`, `Facebook Marketplace`, etc.                                                        |
| Sale Date                   | Date                       | Transaction date.                                                                                                |
| **Delivered Date**          | **Date**                   | Buyer hand-off date. Optional. Same as Sale Date for same-day pickup.                                            |
| Gross Sale Price            | Currency                   | Total price including taxes/fees.                                                                                |
| **Payout Eligibility Date** | **Formula**                | **Recommended**: `IF({Delivered Date}, DATEADD({Delivered Date}, 14, 'days'), DATEADD({Sale Date}, 14, 'days'))` |
| Payout Status               | Single Select              | `Pending Return Window`, `Payable`, `Processing`, `Paid`.                                                        |
| Refund Status               | Single Select              | `None`, `Requested`, `Processed`.                                                                                |
| Refund Amount               | Currency                   | Actual refunded amount if any.                                                                                   |
| Payout Batch                | Link to Settlement Batches | Settlement batch linkage.                                                                                        |

**Table 4: Payouts (Line-item payouts and clawbacks)**

Granular ledger of all positive and negative payouts per sale.

| **Field Name**     | **Field Type**             | **Description**                                                                                     |
| ------------------ | -------------------------- | --------------------------------------------------------------------------------------------------- |
| Payout ID          | Primary Field (Autonumber) | Unique identifier.                                                                                  |
| Sale               | Link to Sales              | Related sale.                                                                                       |
| Payee              | Link to Partners           | Recipient or clawback target.                                                                       |
| Payout Amount      | Currency                   | Positive for payout, negative for clawback.                                                         |
| Commission Type    | Single Select              | `Owner Share`, `Shop Referral Fee`, `Ambassador L1`, `Ambassador L2`, `Ambassador L3+`, `Clawback`. |
| Stripe Transfer ID | Single Line Text           | For reconciliation.                                                                                 |
| Status             | Single Select              | `Payable`, `Paid`, `Error`.                                                                         |
| Error Message      | Long Text                  | Stripe API error detail on failure.                                                                 |

**Table 5: Contracts (Agreement tracking)**

| **Field Name** | **Field Type**                   | **Description**                                     |
| -------------- | -------------------------------- | --------------------------------------------------- |
| Contract ID    | Primary Field (Single Line Text) | SignWell document ID as primary key.                |
| Partner        | Link to Partners                 | Counterparty.                                       |
| Contract Type  | Single Select                    | `Individual Partner`, `Shop Partner`, `Ambassador`. |
| Date Signed    | Date                             | Signature completion date.                          |
| Document Link  | URL                              | Link to the signed document.                        |

**Table 6: Settlement Batches (Run-level grouping)**

| **Field Name**          | **Field Type**          | **Description**                      |
| ----------------------- | ----------------------- | ------------------------------------ |
| Batch ID                | Primary Field (Formula) | `CONCATENATE("SETTLE-", {Run Date})` |
| Run Date                | Date                    | Settlement execution date.           |
| Total Payouts Processed | Number                  | Count of processed payouts.          |
| Total Amount Paid       | Currency                | Sum of paid amounts.                 |
| Status                  | Single Select           | `Running`, `Completed`, `Failed`.    |
| Sales Processed         | Link to Sales           | Sales included in the batch.         |

---

# Kickback mockdata set (for automation case)

## Partners

| Partner ID | Partner Name | Partner Type | Email                                                 | Stripe Connected Account ID | Stripe Customer ID | SignWell Document ID | Referred By | Shop Type   | Subscription Status | Charter Status End Date | Stripe Account Status |
| ---------- | ------------ | ------------ | ----------------------------------------------------- | --------------------------- | ------------------ | -------------------- | ----------- | ----------- | ------------------- | ----------------------- | --------------------- |
| AMB-ALICE  | Amb Alice    | Ambassador   | [alice@amb.example.com](mailto:alice@amb.example.com) | acct_8001                   | cus_9101           | doc_5001             |             |             | N/A                 |                         | Verified              |
| AMB-BOB    | Amb Bob      | Ambassador   | [bob@amb.example.com](mailto:bob@amb.example.com)     | acct_8002                   | cus_9102           | doc_5002             | Amb Alice   |             | N/A                 |                         | Verified              |
| AMB-CARA   | Amb Cara     | Ambassador   | [cara@amb.example.com](mailto:cara@amb.example.com)   | acct_8003                   | cus_9103           | doc_5003             | Amb Bob     |             | N/A                 |                         | Verified              |
| AMB-DAN    | Amb Dan      | Ambassador   | [dan@amb.example.com](mailto:dan@amb.example.com)     | acct_8004                   | cus_9104           | doc_5004             | Amb Cara    |             | N/A                 |                         | Verified              |
| SHOP-TITAN | Titan Motors | Shop         | [ops@titan.example.com](mailto:ops@titan.example.com) | acct_7001                   | cus_9201           | doc_5101             | Amb Alice   | Charter     | N/A                 | 2025-11-27              | Verified              |
| SHOP-NOVA  | Nova Auto    | Shop         | [ops@nova.example.com](mailto:ops@nova.example.com)   | acct_7002                   | cus_9202           | doc_5102             | Amb Bob     | Non-Charter | Active              |                         | Verified              |
| SHOP-ATLAS | Atlas Auto   | Shop         | [ops@atlas.example.com](mailto:ops@atlas.example.com) | acct_7003                   | cus_9203           | doc_5103             | Amb Cara    | Non-Charter | Past Due            |                         | Verified              |
| SHOP-PRIME | Prime Auto   | Shop         | [ops@prime.example.com](mailto:ops@prime.example.com) |                             | cus_9204           | doc_5104             | Amb Dan     | Non-Charter | Canceled            |                         | Restricted            |
| OWN-LEE    | Jin Lee      | Individual   | [jin.lee@example.com](mailto:jin.lee@example.com)     | acct_6001                   |                    | doc_5201             | Amb Bob     |             | N/A                 |                         | Verified              |
| OWN-PARK   | Min Park     | Individual   | [min.park@example.com](mailto:min.park@example.com)   | acct_6002                   |                    | doc_5202             | Amb Cara    |             | N/A                 |                         | Verified              |
| OWN-KIM    | Ara Kim      | Individual   | [ara.kim@example.com](mailto:ara.kim@example.com)     | acct_6003                   |                    | doc_5203             | Amb Dan     |             | N/A                 |                         | Verified              |
| OWN-CHOI   | Hyun Choi    | Individual   | [hyun.choi@example.com](mailto:hyun.choi@example.com) | acct_6004                   |                    | doc_5204             |             |             | N/A                 |                         | Verified              |

---

## Parts

| Part ID | QR Code ID | Item Name                  | Owner (Link) | Status                             | Part Description   | Vehicle Compatibility      | Condition   | Location Code | Date Added | Sale Record |
| ------- | ---------- | -------------------------- | ------------ | ---------------------------------- | ------------------ | -------------------------- | ----------- | ------------- | ---------- | ----------- |
| 1       | QR-100001  | BMW 330i G40 Headlight RH  | Jin Lee      | Awaiting Contract                  | Legacy intake item | BMW 3 Series 2019-2022     | Like New    |               | 2025-08-29 |             |
| 2       | QR-100002  | Audi A4 B9 Turbocharger    | Nova Auto    | Contract Signed - Awaiting Receipt | Unit boxed         | Audi A4 2017-2022          | New         |               | 2025-09-13 |             |
| 3       | QR-100003  | Kicker CVR10 Subwoofer x3  | Prime Auto   | QR Generated                       | Set of three       | Universal                  | Used - Good |               | 2025-09-18 |             |
| 4       | QR-100004  | CheatAmp 2CH-001           | Atlas Auto   | Picked Up                          | 2CH amplifier      | Universal                  | As-Is       |               | 2025-09-23 |             |
| 5       | QR-100005  | Tesla Model 3 MCU          | Ara Kim      | Received - Pending Inspection      | MCU pulled         | Tesla Model 3 2019-2020    | Used - Good | R1-01-A-1     | 2025-09-28 |             |
| 6       | QR-100006  | Ford F-150 Tailgate        | Titan Motors | Warehouse Verified                 | No dents           | Ford F-150 2018-2022       | Like New    | R2-03-B-2     | 2025-09-30 |             |
| 7       | QR-100007  | Hyundai Elantra Alternator | Min Park     | In Stock                           | Rebuilt            | Hyundai Elantra 2016-2019  | Used - Good | R2-05-A-1     | 2025-10-08 |             |
| 8       | QR-100008  | Kia Sorento Radiator       | Nova Auto    | Listed for Sale                    | OEM                | Kia Sorento 2016-2018      | New         | R2-01-A-3     | 2025-10-13 |             |
| 9       | QR-100009  | BMW G20 Taillight LH       | Jin Lee      | Sold                               | Minor scratch      | BMW 3 Series 2019-2022     | Used - Good | R3-02-A-2     | 2025-10-14 | 9           |
| 10      | QR-100010  | Honda Civic ECU            | Hyun Choi    | In Stock                           | Tested             | Honda Civic 2016-2018      | Like New    | R4-01-C-1     | 2025-10-18 |             |
| 11      | QR-100011  | Toyota Camry Door LH       | Titan Motors | Sold                               | Blue               | Toyota Camry 2015-2017     | Used - Good | R4-02-C-2     | 2025-10-20 | 11          |
| 12      | QR-100012  | Subaru Outback Heater Core | Prime Auto   | In Stock                           | Aftermarket        | Subaru Outback 2010-2014   | As-Is       | R5-01-D-1     | 2025-10-23 |             |
| 13      | QR-100013  | Mercedes C-Class Grille    | Nova Auto    | Sold                               | AMG style          | Mercedes C-Class 2015-2018 | New         | R5-02-D-2     | 2025-10-25 | 13          |
| 14      | QR-100014  | Jeep Wrangler Roof Rack    | Ara Kim      | Returned                           | Buyer remorse      | Jeep Wrangler 2012-2018    | Used - Good | R1-02-A-2     | 2025-10-27 | 14          |

---

## Sales

| Sale ID | Part Sold (Link) | Marketplace Source   | Sale Date  | Delivered Date | Gross Sale Price | Payout Eligibility Date | Payout Status         | Refund Status | Refund Amount | Payout Batch      |
| ------- | ---------------- | -------------------- | ---------- | -------------- | ---------------- | ----------------------- | --------------------- | ------------- | ------------- | ----------------- |
| 9       | 9                | Garage App           | 2025-10-15 |                | 199.99           | 2025-10-27              | Payable               | None          | 0.0           |                   |
| 10      | 10               | Facebook Marketplace | 2025-10-19 |                | 299.99           | 2025-10-30              | Pending Return Window | None          | 0.0           |                   |
| 11      | 11               | Garage App           | 2025-10-21 |                | 499.99           | 2025-10-29              | Processing            | Requested     | 0.0           | SETTLE-2025-10-25 |
| 12      | 12               | eBay                 | 2025-10-22 | **2025-10-22** | 1200.00          | 2025-11-05              | Pending Return Window | None          | 0.0           |                   |
| 13      | 13               | Garage App           | 2025-10-26 | **2025-10-26** | 650.00           | 2025-11-09              | Paid                  | None          | 0.0           | SETTLE-2025-10-25 |
| 14      | 14               | Garage App           | 2025-10-27 | **2025-10-27** | 350.00           | 2025-11-10              | Paid                  | Processed     | 350.00        | SETTLE-2025-10-25 |

---

## Payouts

| Payout ID | Sale (Link) | Payee (Link)             | Payout Amount | Commission Type   | Stripe Transfer ID | Status    | Error Message                              |
| --------- | ----------- | ------------------------ | ------------- | ----------------- | ------------------ | --------- | ------------------------------------------ |
| 1         | 9           | Jin Lee                  | 129.99        | Owner Share       |                    | Payable   |                                            |
| 2         | 9           | Amb Bob                  | 10.0          | Ambassador L1     |                    | Payable   |                                            |
| 3         | 9           | Amb Alice                | 3.0           | Ambassador L2     |                    | Payable   |                                            |
| 4         | 10          | Hyun Choi                | 194.99        | Owner Share       |                    | Payable   |                                            |
| 5         | 10          | Nova Auto                | 15.0          | Shop Referral Fee |                    | Payable   |                                            |
| 6         | 10          | Amb Alice                | 15.0          | Ambassador L1     |                    | Payable   |                                            |
| 7         | 11          | Titan Motors             | 324.99        | Owner Share       |                    | Payable   |                                            |
| 8         | 11          | Amb Cara                 | 25.0          | Ambassador L1     |                    | Payable   |                                            |
| 9         | 11          | Amb Bob                  | 7.5           | Ambassador L2     |                    | Payable   |                                            |
| 10        | 11          | Amb Alice                | 2.5           | Ambassador L3+    |                    | Payable   |                                            |
| 11        | 12          | Prime Auto               | 780.0         | Owner Share       |                    | **Error** | account_invalid: missing connected account |
| 12        | 12          | Amb Dan                  | 60.0          | Ambassador L1     |                    | Payable   |                                            |
| 13        | 12          | Amb Cara                 | 18.0          | Ambassador L2     |                    | Payable   |                                            |
| 14        | 12          | Amb Bob                  | 6.0           | Ambassador L3+    |                    | Payable   |                                            |
| 15        | 12          | Referral Partner OneTime | 36.0          | Referral Partner  |                    | Payable   |                                            |
| 16        | 13          | Nova Auto                | 422.5         | Owner Share       |                    | **Error** | balance_insufficient                       |
| 17        | 13          | Amb Bob                  | 32.5          | Ambassador L1     |                    | Payable   |                                            |
| 18        | 14          | Ara Kim                  | 227.5         | Owner Share       |                    | Payable   |                                            |
| 19        | 14          | Ara Kim                  | -227.5        | Clawback          |                    | Payable   |                                            |

---

## Contracts

| Contract ID   | Partner (Link) | Contract Type      | Date Signed | Document Link                           |
| ------------- | -------------- | ------------------ | ----------- | --------------------------------------- |
| SW-AMB-ALICE  | Amb Alice      | Ambassador         | 2025-08-19  | https://signwell.example.com/AMB-ALICE  |
| SW-AMB-BOB    | Amb Bob        | Ambassador         | 2025-08-19  | https://signwell.example.com/AMB-BOB    |
| SW-AMB-CARA   | Amb Cara       | Ambassador         | 2025-08-19  | https://signwell.example.com/AMB-CARA   |
| SW-AMB-DAN    | Amb Dan        | Ambassador         | 2025-08-19  | https://signwell.example.com/AMB-DAN    |
| SW-SHOP-TITAN | Titan Motors   | Shop Partner       | 2025-08-19  | https://signwell.example.com/SHOP-TITAN |
| SW-SHOP-NOVA  | Nova Auto      | Shop Partner       | 2025-08-19  | https://signwell.example.com/SHOP-NOVA  |
| SW-SHOP-ATLAS | Atlas Auto     | Shop Partner       | 2025-08-19  | https://signwell.example.com/SHOP-ATLAS |
| SW-SHOP-PRIME | Prime Auto     | Shop Partner       | 2025-08-19  | https://signwell.example.com/SHOP-PRIME |
| SW-OWN-LEE    | Jin Lee        | Individual Partner | 2025-08-19  | https://signwell.example.com/OWN-LEE    |
| SW-OWN-PARK   | Min Park       | Individual Partner | 2025-08-19  | https://signwell.example.com/OWN-PARK   |
| SW-OWN-KIM    | Ara Kim        | Individual Partner | 2025-08-19  | https://signwell.example.com/OWN-KIM    |
| SW-OWN-CHOI   | Hyun Choi      | Individual Partner | 2025-08-19  | https://signwell.example.com/OWN-CHOI   |

---

## Settlement Batches

| Batch ID          | Run Date   | Total Payouts Processed | Total Amount Paid | Status    | Sales Processed (Links) |
| ----------------- | ---------- | ----------------------- | ----------------- | --------- | ----------------------- |
| SETTLE-2025-10-25 | 2025-10-25 | 17                      | 1107.97           | Completed | 9,11,13,14              |
| SETTLE-2025-10-28 | 2025-10-28 | 0                       | 0.0               | Running   |                         |
| SETTLE-2025-10-20 | 2025-10-20 | 0                       | 0.0               | Failed    |                         |
