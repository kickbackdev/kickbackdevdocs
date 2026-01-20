# KKB-F1-Contract-Register-[dev] Operations Manual (Runbook)

![Kickback logo](../images/logokick.avif)

## 1. Overview

### System Purpose
The purpose of this automation system (`KKB-F1-Contract-Register-[dev]`) is to automatically process partner contracts (Individual, Shop, Ambassador) and Parts Addendums completed via **SignWell**, processing them through **Zapier** to create and update records in the **Airtable** database.

### Core Systems
* **Trigger:** SignWell (Event: Document Completed)
* **Automation Hub:** Zapier
* **Database:** Airtable (Base: Kickback Operations)

### Owner & Channels
* **System Owner:** [Operations Lead/Dev Team Lead Name]
* **Slack (Success Alerts):** `#automation-alerts` (KKB-F1-5, F1-6, F1-7, F1-8)
* **Slack (Errors/Warnings):** `#ops-alerts` (KKB-F1-8, F1-9, F1-10)

---

## 2. Failure Response Procedure

This is the response manual for alerts received in the `#ops-alerts` channel. (Tied to KKB-F1-10).

| Alert Message Type | Meaning | First Response (Operations Team) |
| :--- | :--- | :--- |
| `⚠️ F1 Input Missing` | Trigger is missing `email` or `doc_id`. | **(Ignore)** Zap stopped automatically. The trigger must be resent with the missing data. |
| `⚠️ F1 Unknown Template` | `template_name` does not match the 4 Paths. | **(Verify)** 1. Check the `template_name` from the Zap run in Zapier History.<br>2. If it's a new, valid template, request the dev team to add a new Path. |
| `⚠️ F1 Addendum Halted: Partner Not Found` | Parts Addendum was submitted, but the email does not exist in the `Partners` DB. | **(Manual Process)** 1. Contact the partner to request completion of the base contract (Individual/Shop) first.<br>2. After the base contract is complete, **Replay Task** on the failed Zap or process it manually. |
| `🚨 F1 Step Failed` | API error during Zap execution (Airtable connection failed, Code error, etc.) | **(Immediate Action)** Follow steps 1-4 below. |

### Standard Response Procedure for "F1 Step Failed"

1.  **Check Zapier History (First Step):**
    * Based on the `{{zap_name}}` and time from the Slack alert, go to **Zapier's [Task History](https://zapier.com/app/history/)** to find the **failed Zap run**.
2.  **Check Error Step and Message:**
    * Click the failed Zap to identify which **step** (`{{step_name}}`) it failed on and the specific **error** (`{{error_message}}`).
    * *(e.g., "Airtable connection failed" on the `AT Create Contract` step)*
3.  **Check Airtable Data:**
    * Go to Airtable to see if data was partially created or not created at all.
4.  **Action and Reprocessing:**
    * **(Simple Error):** If it was a temporary connection issue (Airtable/Slack), re-authenticate the app in Zapier's "My Apps" section. Then, go to the Zap History and press the **`Replay Task`** button to re-run the failed task.
    * **(Data Error):** If it's a logic error (e.g., `Code by Zapier` step), request a fix from the dev team.

---

## 3. Rollback & Maintenance

### Emergency Stop (Rollback Procedure)
If critical errors (e.g., all Zaps failing) occur repeatedly, switch to manual processing immediately.

1.  **Turn Zap OFF:**
    * Go to the Zapier dashboard and turn the **toggle switch to `OFF`** for the `KKB-F1-Contract-Register-[dev]` Zap.
2.  **Switch to Manual Processing:**
    * From the moment the Zap is off, the operations team must manually monitor SignWell completion **email notifications**.
    * Manually **copy/paste** the data into the `Partners`, `Contracts`, and `Parts` tables in Airtable.
    * Notify the relevant departments via Slack of the manual completion.

### System Change Log
*When making system changes, copy the template below and add to the top.*

| Date | Owner | Change Summary | Related Jira |
| :--- | :--- | :--- | :--- |
| 2025-11-07 | Jeong Yuhyeon | System initial build complete per KKB-F1-14 | KKB-F1-14 |
| (Date) | (Name) | (Change Summary) | (Ticket #) |

---

## 4. Onboarding Guide

A practical guide for new hires and the operations team.

### Q. How do I send a partner contract?
**A.** Simply send one of the 4 official templates (`Individual Partner Agreement`, `Shop Partner Agreement`, `Ambassador Agreement`, `Individual Parts Agreement (Addendum)`) from **SignWell** to the partner. Once the partner signs it, this Zap will run **automatically**. You don't need to worry about the next steps.

### Q. How do I check if the automation is working?
**A.** Just monitor the **Slack `#automation-alerts` channel**. You will get a notification every time a contract is processed successfully. If you don't get an alert, or you get a warning in `#ops-alerts`, follow the **'2. Failure Response Procedure'** above.

### Q. (Important) What Airtable view should I check every day?
**A.** You must check the **`Awaiting Receipt`** view (KKB-F1-13) in the Airtable **`Parts`** table every day.
* **Meaning:** This view shows a list of parts that Zapier has automatically registered, but the **physical part has not yet arrived** at the warehouse.
* **Action:** When a part on this list arrives, you must **manually change** its `Status` from `Contract Signed - Awaiting Receipt` to `In Stock` (or similar).

### Q. Who do I contact if there's an error?
**A.** 1. First, try the steps in **'2. Failure Response Procedure'** in this manual. <br> 2. If the issue can't be fixed with a `Replay Task`, contact the **System Owner**.

---

## 5. Cost Analysis (Zapier)

This automation system uses Zapier's premium features; cost analysis is as follows.

### Plan Recommendation
* `KKB-F1-3 (Paths by Zapier)`
* `KKB-F1-8 (Code by Zapier)`

The two steps above are **premium features** available only on the **Professional plan or higher**. Therefore, a **Professional plan** is mandatory to operate this system.

### Credit (Task) Calculation (Based on 100 contracts/month)

1.  **Tasks Used Per Zap Run (Estimate):**
    * Trigger (1)
    * FMT Steps (3)
    * FLT Steps (2)
    * Path Step (1)
    * Airtable Steps (Varies by path, 2-5)
    * Slack Step (1)
    * **Average ~10-12 Tasks consumed**

2.  **Estimated Monthly Usage (100 contracts):**
    * 100 contracts/month * 12 Tasks/contract = **~1,200 Tasks/month**

### Conclusion
Processing 100 contracts/month will consume approximately **1,200 Tasks**. The Zapier **Professional plan** includes **2,000 Tasks** by default, which provides a sufficient buffer for operations and testing.