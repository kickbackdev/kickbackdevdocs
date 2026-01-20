# Airtable Warehouse & Automation Design (Kickback Operations)

<img src="../images/logokick.avif" alt="Kickback logo" width="120" />

This document describes the Airtable schema, warehouse codes, automations, and QR label design used in the **Kickback Operations** base.  
It complements the KKB-F1 Contract Register runbook and focuses on warehouse routing, SLAs, and parts-level status tracking.

---

## 1. Overview

### 1.1 Purpose

- Standardize **warehouse codes** and **capability profiles**.
- Track parts through a **Short Term** or **Long Term (Met → Sub)** path.
- Enforce core **SLAs** using Airtable formulas and automations.
- Drive operations via simple **checkbox-based updates** on the `Parts` table.
- Generate **URL-based QR labels** that open the **record detail view** in Airtable.

### 1.2 Key assumptions

- Intake classification:
  - `Owner Partner Type = Shop` → **Short Term**
  - `Owner Partner Type = Individual` → **Long Term**
- Long Term items follow a default path: **Metropolitan (Met) → Suburban (Sub)**.
- Core SLAs:
  1. `Receipt → Permanent Shelving` ≤ **24 hours**
  2. `Permanent Shelving → Photo Batch` ≤ **24 hours**
  3. `Receipt → Listing` ≤ **48 hours**
  4. Selling cycle: **7–14 days** from listing to sale

Operators interact primarily with the **Parts** table by checking boxes to mark step completion.

---

## 2. Warehouse Codes & Capabilities

### 2.1 Warehouse codes

Warehouse codes are standardized as:

```text
{STATE2}-{COUNTY}-{CAT}-{NN}
e.g. MD-PG-MET-01, MD-PG-SUB-03
```

- **State Code**: 2-letter state code (e.g. `MD`)
- **County Code**: short county code (e.g. `PG`)
- **CAT**:

  - `MET` = Metropolitan hub
  - `SUB` = Suburban hub

- **NN**: sequence number (2 digits, zero-padded)

### 2.2 Capability profiles

#### Short Term (all functions, single warehouse)

Flow:

```text
Drop-off
→ Parts intake processing (scan QR)
→ Condition photos
→ Temporary Shelving (≤ 24h)
→ Cleaning & Authentication
→ Temporary Shelving
→ Photo batch processing (daily)
→ Permanent shelving for sale
→ Listing
```

All steps are performed in one warehouse.

#### Long Term (split by Met / Sub)

- **Metropolitan (Met)**

  ```text
  Drop-off
  → Parts intake processing (scan QR)
  → Condition photos
  → Temporary Shelving (≤ 24h)
  → Cleaning & Authentication
  ```

- **Suburban (Sub)**

  ```text
  Parts intake processing (scan QR)
  → Temporary Shelving
  → Photo batch processing (daily)
  → Permanent shelving for sale
  → Listing
  ```

---

## 3. New Tables

### 3.1 `Warehouses` (warehouse master)

Represents each physical warehouse and its capabilities.

| Field                  | Type                 | Description                                                                                            |
| ---------------------- | -------------------- | ------------------------------------------------------------------------------------------------------ |
| `Warehouse Code`       | Primary (Formula)    | `CONCATENATE({State Code},"-",{County Code},"-",{Category},"-",LPAD({Seq},2,"0"))` e.g. `MD-PG-MET-01` |
| `State Code`           | Single line text     | 2-letter state code (e.g. `MD`)                                                                        |
| `County Code`          | Single line text     | County short code (e.g. `PG`)                                                                          |
| `Category`             | Single select        | `Met`, `Sub`                                                                                           |
| `Capability Profile`   | Single select        | `Short Term`, `Long Term-Met`, `Long Term-Sub`                                                         |
| `Step Coverage`        | Multi select         | Steps performed: `DROP`, `INTAKE`, `PHOTOS`, `TSHELF`, `CLEAN_AUTH`, `PBATCH`, `PSHELF`, `LIST`        |
| `Default Outbound Sub` | Link to `Warehouses` | Default Sub destination for a Met hub                                                                  |
| `Seq`                  | Autonumber           | Sequence for code generation                                                                           |

Capability mapping:

- **Short Term**: covers all steps
  `DROP, INTAKE, PHOTOS, TSHELF, CLEAN_AUTH, PBATCH, PSHELF, LIST`
- **Long Term-Met**:
  `DROP, INTAKE, PHOTOS, TSHELF, CLEAN_AUTH`
- **Long Term-Sub**:
  `INTAKE, TSHELF, PBATCH, PSHELF, LIST`

### 3.2 `Warehouse Transfers` (transfer ledger)

Tracks Met → Sub (or any) warehouse transfer.

| Field            | Type                 | Description                                                                                    |
| ---------------- | -------------------- | ---------------------------------------------------------------------------------------------- |
| `Transfer ID`    | Primary (Autonumber) | Transfer identifier                                                                            |
| `Part`           | Link to `Parts`      | Part being transferred                                                                         |
| `From Warehouse` | Link to `Warehouses` | Origin warehouse                                                                               |
| `To Warehouse`   | Link to `Warehouses` | Destination warehouse                                                                          |
| `Requested At`   | DateTime             | Transfer requested time                                                                        |
| `Dispatched At`  | DateTime             | Actual departure time                                                                          |
| `Received At`    | DateTime             | Arrival time                                                                                   |
| `Status`         | Single select        | `Planned`, `In Transit`, `Received`, `Canceled`                                                |
| `Transit Hours`  | Formula              | `IF(AND({Dispatched At},{Received At}), DATETIME_DIFF({Received At},{Dispatched At},"hours"))` |

---

## 4. Existing Tables – Patches

### 4.1 `Parts` table

The existing high-level lifecycle `Status` is preserved.
We add routing context, stage checkboxes + timestamps, helper fields, and SLA fields.

#### 4.1.A Routing & ownership context

- `Owner Partner Type`

  - **Type**: Lookup (`Partners → Partner Type`)

- `Intake Class`

  - **Type**: Formula
  - **Logic**:

    ```text
    IF({Owner Partner Type}="Shop","Short Term",
       IF({Owner Partner Type}="Individual","Long Term","N/A"))
    ```

- `Current Warehouse`

  - **Type**: Link to `Warehouses`

- `Current Warehouse Category`

  - **Type**: Lookup from `Current Warehouse → Category`

- `Planned Next Warehouse`

  - **Type**: Link to `Warehouses`

- `Transfer Needed?`

  - **Type**: Formula
  - **Logic** (using lookup field `Current Warehouse Category`):

    ```text
    IF(
      {Intake Class}="Long Term"
      AND {Current Warehouse Category}="Met",
      1,
      0
    )
    ```

- `Routing Plan`

  - **Type**: Formula
  - **Logic**:

    ```text
    IF({Intake Class}="Long Term","Met → Sub","Met only")
    ```

#### 4.1.B Stage checkboxes + timestamps

Each operational step is tracked by a checkbox and corresponding timestamp.

| Step                      | Checkbox field              | Timestamp field         |
| ------------------------- | --------------------------- | ----------------------- |
| Intake scan / receipt     | `Intake Scan`               | `Intake Scanned At`     |
| Condition photos          | `Condition Photos`          | `Condition Photos At`   |
| Temp shelving (pre/post)  | `Temp Shelving`             | `Temp Shelving At`      |
| Cleaning & authentication | `Cleaning & Authentication` | `Clean&Auth At`         |
| Photo batch processing    | `Photo Batch Processing`    | `Photo Batch At`        |
| Permanent shelving        | `Permanent Shelving`        | `Permanent Shelving At` |
| Listing published         | `Listing Published`         | `Listing At`            |

Airtable Automations are responsible for:

- When a checkbox becomes checked → write `NOW()` into the paired `… At` field.
- Optionally adjust lifecycle `Status` if needed.

#### 4.1.C Lifecycle helper fields

- `Processing Stage`

  - **Type**: Formula
  - **Logic** (from latest completed step):

    ```text
    IF({Listing Published}, "LISTED",
    IF({Permanent Shelving}, "PERM_SHELF",
    IF({Photo Batch Processing}, "PHOTO_BATCH",
    IF({Cleaning & Authentication}, "CLEAN_AUTH",
    IF({Temp Shelving}, "TEMP_SHELF",
    IF({Condition Photos}, "PHOTOS",
    IF({Intake Scan}, "INTAKE","PENDING")))))))
    ```

- `Stage Started At`

  - **Type**: Formula
  - **Logic** (timestamp of latest completed step):

    ```text
    IF({Listing At},{Listing At},
    IF({Permanent Shelving At},{Permanent Shelving At},
    IF({Photo Batch At},{Photo Batch At},
    IF({Clean&Auth At},{Clean&Auth At},
    IF({Temp Shelving At},{Temp Shelving At},
    IF({Condition Photos At},{Condition Photos At},
    {Intake Scanned At}))))))
    ```

- `Hours in Stage`

  - **Type**: Formula
  - **Logic**:

    ```text
    DATETIME_DIFF(NOW(),{Stage Started At},"hours")
    ```

- `Next Action`

  - **Type**: Formula (text)
  - Uses `Processing Stage` + `Step Coverage` (from `Current Warehouse`) to suggest the next step to the operator (e.g. `"Next: CLEAN_AUTH at MD-PG-MET-01"`).

- `Blocker Reason`

  - **Type**: Single line text
  - Manual reason for delay or exception.

#### 4.1.D SLA & wait time fields

- `Received At`

  - **Type**: DateTime
  - Populated by automation when:

    - `Status` becomes `"Received - Pending Inspection"`, **or**
    - `Intake Scan` is checked (QR scan).

- `SLA_24h_Receipt_to_PermShelf_OK`

  - **Type**: Formula
  - **Logic**:

    ```text
    IF(
      {Permanent Shelving At},
      DATETIME_DIFF({Permanent Shelving At},{Received At},"hours")<=24,
      DATETIME_DIFF(NOW(),{Received At},"hours")<=24
    )
    ```

- `SLA_24h_PermShelf_to_Photo_OK`

  - **Type**: Formula
  - **Logic**:

    ```text
    IF(
      {Photo Batch At},
      DATETIME_DIFF({Photo Batch At},{Permanent Shelving At},"hours")<=24,
      IF(
        {Permanent Shelving At},
        DATETIME_DIFF(NOW(),{Permanent Shelving At},"hours")<=24
      )
    )
    ```

- `SLA_48h_Receipt_to_Listing_OK`

  - **Type**: Formula
  - **Logic**:

    ```text
    IF(
      {Listing At},
      DATETIME_DIFF({Listing At},{Received At},"hours")<=48,
      DATETIME_DIFF(NOW(),{Received At},"hours")<=48
    )
    ```

- `Hours Waiting to Listing`

  - **Type**: Formula
  - **Logic**:

    ```text
    IF(
      {Listing At},
      0,
      DATETIME_DIFF(NOW(),{Received At},"hours")
    )
    ```

- `Selling Days`

  - **Type**: Formula
  - Assumes `Sale Record` is a link to `Sales` and `Sale Date` is either a lookup or rollup:

    ```text
    IF(
      {Sale Record},
      DATETIME_DIFF({Sale Date},{Listing At},"days")
    )
    ```

- `Selling 7–14d?`

  - **Type**: Checkbox / Formula
  - Indicates whether `Selling Days` is within the 7–14 day target range.

#### 4.1.E Operator interaction

- All checkboxes live on the **Parts** record.
- Operators update status by simply checking the relevant box as each step is completed.
- Automations:

  - Populate timestamps.
  - Maintain `Processing Stage`, `SLA_*` flags, and optional `Status` updates.

### 4.2 `Sales` table auxiliary fields

- Use existing `Delivered Date` and related fields from prior spec.
- Link back to `Parts` for selling-cycle SLAs (`Selling Days`, `Selling 7–14d?`).

---

## 5. Recommended Views

Create saved views in `Parts` for operations and SLA monitoring.

1. **`_SLA Breach – 24h Receipt→Perm`**

   - Filter:

     - `SLA_24h_Receipt_to_PermShelf_OK = 0`
     - `Status` not in `{Listed for Sale, Sold, Returned}`

2. **`_SLA Breach – 48h Receipt→Listing`**

   - Filter:

     - `SLA_48h_Receipt_to_Listing_OK = 0`

3. **`_LongTerm – Needs Transfer`**

   - Filter:

     - `Transfer Needed? = 1`
     - No open `Warehouse Transfers` record for this Part.

4. **`_Stuck > 12h in Stage`**

   - Filter:

     - `Hours in Stage >= 12`
     - `Listing Published = 0`

These views are used by Automations (e.g. SLA alerts) and by operators to prioritize work.

---

## 6. Airtable Automations

The **Kickback Operations → Automations** panel should contain automations aligned with the screenshot:

- Generate QR Code
- Intake Scan
- Status → Received At
- Long term route
- Transfer
- ALERT: SLA Breach
- Condition Photos
- Temp Shelving
- Cleaning & Authentication
- Photo Batch At
- Permanent Shelving
- Listing Published

### 6.1 Checkbox → timestamp (per step)

**Automations (examples):**

1. `Intake Scan`

   - Trigger: When record matches conditions

     - `Intake Scan = checked`
     - `Intake Scanned At` is empty

   - Action: Update record

     - `Intake Scanned At = NOW()`
     - Optionally set `Received At = NOW()` if not set.

2. `Condition Photos`

   - Trigger: `Condition Photos = checked` and `Condition Photos At` is empty
   - Action: `Condition Photos At = NOW()`

3. `Temp Shelving`

   - Trigger: `Temp Shelving = checked` and `Temp Shelving At` is empty
   - Action: `Temp Shelving At = NOW()`

4. `Cleaning & Authentication`

   - Trigger: `Cleaning & Authentication = checked` and `Clean&Auth At` is empty
   - Action: `Clean&Auth At = NOW()`

5. `Photo Batch At`

   - Trigger: `Photo Batch Processing = checked` and `Photo Batch At` is empty
   - Action: `Photo Batch At = NOW()`

6. `Permanent Shelving`

   - Trigger: `Permanent Shelving = checked` and `Permanent Shelving At` is empty
   - Action: `Permanent Shelving At = NOW()`

7. `Listing Published`

   - Trigger: `Listing Published = checked` and `Listing At` is empty
   - Action: `Listing At = NOW()`

### 6.2 Status → Received At

- Automation: `Status → Received At`
- Trigger: When `Status` changes to `"Received - Pending Inspection"`.
- Action: set `Received At = NOW()` (if empty).

### 6.3 Long Term routing creation

- Automation: `Long term route`
- Trigger: `Intake Scan = checked` AND `Transfer Needed? = 1`
- Action: Create a new record in `Warehouse Transfers`:

  - `Part = this Part`
  - `From Warehouse = Current Warehouse`
  - `To Warehouse = Current Warehouse.Default Outbound Sub`
  - `Status = Planned`
  - `Requested At = NOW()`

### 6.4 Transfer completion

- Automation: `Transfer`
- Trigger: In `Warehouse Transfers`, when `Status` changes to `Received`.
- Action: Update linked `Part`:

  - `Current Warehouse = To Warehouse`

### 6.5 SLA alerts (Slack)

- Automation: `ALERT: SLA Breach`
- Trigger (any of):

  - `SLA_48h_Receipt_to_Listing_OK = 0`
  - `SLA_24h_PermShelf_to_Photo_OK = 0`

- Action: Send Slack message:

  - Include: `Part ID`, `Current Warehouse`, `Processing Stage`, `Hours in Stage`, `Next Action`, and a direct record URL (see QR section).

---

## 7. QR Labels – URL-based design

### 7.1 Requirement

- **Current**: QR is generated as embedded text (e.g. only part number) inside a text field. Scanning leads to a plain value / line item.
- **Goal**: QR should open the **Airtable record detail view** (interface screen) so operators can see and update the full record.
- **Constraint**: Changes to other fields must **not invalidate existing QR codes**.

### 7.2 Record URL formula field

1. Open the desired **Interface** record view and copy its URL, e.g.:

   ```text
   https://airtable.com/appXXXX/pagYYYY/recZZZZ...
   ```

2. In `Parts`, create a formula field:

   - Name: `Internal Record URL`

   - Type: Formula

   - Formula:

     ```text
     "https://airtable.com/appXXXX/pagYYYY/" & RECORD_ID()
     ```

   - Replace `appXXXX` and `pagYYYY` with the actual **base ID** and **page ID**.

   - When scanned on a signed-in device, this URL opens the **record detail view**.

### 7.3 QR attachment field

- Add an attachment field:

  - Name: `QR Label PNG`
  - Type: Attachment

- Optionally:

  - Add `Label Printed` (Checkbox) to track if the physical label has been printed.
  - Add `Status` option `"QR Generated"` as a checkpoint.

### 7.4 QR generation automation (`Generate QR Code`)

- Trigger: When record is created or updated.

- Conditions:

  - `QR Label PNG` is empty
  - `Internal Record URL` is not empty

- Action: `Run script`.
  Input variables:

  - `recId` = `RECORD_ID()`
  - `url` = `Internal Record URL`

- Script example:

  ```js
  const { recId, url } = input.config();
  const table = base.getTable("Parts");
  const qr =
    "https://quickchart.io/qr?text=" +
    encodeURIComponent(url) +
    "&size=512&margin=2";

  await table.updateRecordAsync(recId, {
    "QR Label PNG": [{ url: qr }],
    Status: { name: "QR Generated" },
  });
  ```

- Optional second action: send a Slack message like
  `"QR generated: {Part Number} / {Current Warehouse}"`.

### 7.5 Backfill for existing data

- Create a view `QR Missing` on `Parts`:

  - Filter: `QR Label PNG is empty`.

- Temporarily change the `Generate QR Code` trigger to:

  - `When record enters view (QR Missing)`

- Allow the automation to run and populate QR codes for all existing rows.
- Turn off or revert the trigger after backfill is complete.

### 7.6 Label printing

- Use **Page Designer** to create a label template:

  - Fields: `Item Name`, `Asset ID` or `Part Number`, `Warehouse Code`, `QR Label PNG`.

- After printing, mark `Label Printed = checked`.

### 7.7 Access & security notes

- URLs require Airtable login; scanning on a non-logged-in device will lead to the login screen.
- `appXXXX/pagYYYY` are constants; do not change after deployment without updating the formula.
- The old “text-embedded QR” field can be hidden or deprecated after migration.

---

## 8. Operational checklist

1. **At receipt / intake**

   - Scan QR or check `Intake Scan`.
   - Confirm `Received At` is populated.

2. **During processing**

   - As each step completes, check the relevant box:

     - `Condition Photos`
     - `Temp Shelving`
     - `Cleaning & Authentication`
     - `Photo Batch Processing`
     - `Permanent Shelving`
     - `Listing Published`

   - Automations handle timestamps and SLA fields.

3. **Long Term routing**

   - For `Intake Class = Long Term` at a Met warehouse:

     - `Transfer Needed?` becomes `1`.
     - `Long term route` automation creates a `Warehouse Transfers` record.

   - When `Warehouse Transfers.Status = Received`, `Transfer` automation updates `Current Warehouse` to the Sub.

4. **Monitoring**

   - Check SLA views:

     - `_SLA Breach – 24h Receipt→Perm`
     - `_SLA Breach – 48h Receipt→Listing`
     - `_Stuck > 12h in Stage`
     - `_LongTerm – Needs Transfer`

   - Respond to `ALERT: SLA Breach` Slack notifications.

5. **Listing & sale**

   - When listing is live: check `Listing Published`.
   - When sold: link to `Sale Record` and ensure selling-cycle metrics are populated.

---

## 9. Rollout order (minimal viable deployment)

1. Create **`Warehouses`** and **`Warehouse Transfers`** tables.
2. Add routing, checkbox, SLA, and helper fields to **`Parts`**.
3. Implement automations:

   - Checkbox → timestamp
   - Status → `Received At`
   - Long Term routing and Transfer update

4. Configure **QR label** fields and `Generate QR Code` automation.
5. Set up views and **SLA Slack alerts**.
6. Migrate/hide legacy text-based QR fields and roll out new labels.

With these elements in place, warehouse codes, categories, wait times, state tracking, Met → Sub transfers, and manual-step operations are fully integrated into Airtable and surfaced via the Parts table as the single source of truth.

```

```
