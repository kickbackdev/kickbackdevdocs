## 2. SignWell Contract Execution & System Sync

This section covers the automation flow where, upon a Partner (Ambassador, Shop, Individual) completing a signature on a SignWell contract, the document is automatically archived in the Airtable database and a notification is sent to the internal team.

### 1. Implemented Automation Flow
The functionality demonstrated in the video covers the process: **[SignWell Signature Completed → Auto-Classification by Partner Type (Path) → Airtable Contract Archiving & Mapping → Slack Notification]**.

#### SignWell Document Signed (Trigger)
* Zapier triggers immediately when a contract signature is completed in SignWell.
* The video demonstrates this using Zapier's 'Replay' feature to re-execute past test data.

#### Conditional Routing (Classification by Type)
* Using Zapier's **Paths** feature, the flow automatically branches into 3 paths based on the template name of the signed document.
    * **Path A:** Ambassador Agreement
    * **Path B:** Shop Partner Agreement
    * **Path C:** Individual Partner Agreement

#### Airtable Sync (Data Synchronization)
* **Create Contract Record:** Creates a new record in the `Contracts` table containing the signed PDF link and contract details.
* **Partner Mapping:** Links the created contract information to the corresponding partner record in the `Partners` table via **Linked Record** (updates fields like `SignWell Document` shown in the video).

#### Slack Notification (Internal Alert)
* Once processing is complete, a notification is sent to the `#ops-alerts` channel.
* **Message Format:** `New [Type] Partner Activated: [Name]` + Signed PDF file attachment.

### 2. Remaining Flow & Future Implementation
Automation for this part is **Completed**.

* **Current Status:** Tests have been completed for all 3 partner types (Ambassador, Shop, Individual), and normal operation has been confirmed.
* **Maintenance:** If contract template names change or Airtable field structures are modified in the future, Zapier's Path conditions and mapping settings must be updated.

### 3. Test Methods & Video Analysis

#### 3-1. Test Methods
The video verifies the logic quickly and accurately using the Zapier Replay feature instead of waiting for actual signatures.

1.  **Prepare Zapier Data:** Select a previously successful or pending 'Document Signed' log from Zapier History.
2.  **Execute Replay:** Click 'Replay entire Zap' to forcibly run the automation with that data.
3.  **Check Airtable:** Verify in the `Partners` table that `SignWell Document` is linked to the partner's record and that the PDF opens upon clicking.
4.  **Check Slack:** Verify that the notification message and PDF have arrived in the `#ops-alerts` channel.

#### 3-2. Video Walkthrough & Timeline Analysis

**Video (YouTube):**

<div style="max-width: 720px;">
  <iframe
    width="100%"
    height="405"
    src="https://www.youtube.com/embed/BW6cqwXRS9w"
    title="SignWell Contract Execution & System Sync - Video Walkthrough"
    frameborder="0"
    allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share"
    allowfullscreen>
  </iframe>
</div>

| Timeline | Stage | Detailed Description |
| :--- | :--- | :--- |
| **00:00 ~ 00:24** | **Logic Overview** | Shows the automation structure (Zap: `1. KB-F1-Contract-Register-dev`) branching into 3 Paths (Individual, Shop, Ambassador) on the Zapier screen. |
| **00:25 ~ 00:55** | **Detailed Logic** | Checks Path settings inside Zapier. Confirms the flow of branching by document name, updating Airtable, and sending Slack messages. |
| **00:56 ~ 01:15** | **Test 1: Ambassador** | (After Zapier Replay) Check 'Asa Gladstein' record in Airtable. <br>→ Document is linked in `Contracts` field. <br>→ Open PDF to confirm Ambassador Agreement. <br>→ Confirm notification arrival in Slack `#ops-alerts`. |
| **01:16 ~ 02:00** | **Test 2: Shop** | (Execute Zapier Replay) Re-run 'Shop Partner Agreement' data. <br>→ Contract automatically linked to Airtable 'jaewonshop' record. <br>→ Confirm PDF content (Shop Partner). <br>→ Confirm Slack notification arrival. |
| **02:01 ~ 02:45** | **Test 3: Individual** | (Execute Zapier Replay) Re-run 'Individual Partner Agreement' data. <br>→ Check Airtable 'Joe Kim' record and contract link. <br>→ Confirm PDF content (Individual Partner). <br>→ Confirm Slack notification arrival. |
