### **Test Video Link**

<img src="../images/logokick.avif" alt="Kickback logo" width="120" />

https://drive.google.com/file/d/1RyKxpfXDUvHXmZNHFH4R2efCxi_I2Z_0/view?usp=share_link

### **Visual Walkthrough: Automated Settlement & Payout System**

This guide explains the automated workflow shown in the video, which handles the daily financial settlements for the Kickback platform.

**1. Initiating the Settlement Run (Video 00:00 – 00:23)**

- **What is happening:** The system wakes up on a daily schedule (e.g., 9:00 AM). Its first job is to create a "container" to organize all of today's payments.
- **Visual Cue:** In the **Airtable** screen, you can see a new row appear in the **`Settlement Batches`** table.
- **Why it matters:** This generates a unique Batch ID (e.g., `SETTLE-2025-11-20...`) that groups every payment made today, ensuring auditability and organized reporting.

**2. Calculating Commissions & Creating Records (Video 00:24 – 02:28)**

- **What is happening:** The system identifies every item sold that is eligible for payout. It then acts as a calculator. It uses a custom script (JavaScript) to figure out exactly how much to pay the **Part Owner**, the **Shop**, and any **Ambassadors** involved in the sale.
- **Visual Cue:**
  - In **Zapier**, you see a **"Code by Zapier"** step running complex math.
  - In **Airtable**, you watch the **`Payouts`** table rapidly fill up with individual line items (e.g., "Owner Share," "Ambassador Commission").
- **Why it matters:** This replaces manual spreadsheets. It handles complex multi-level commissions instantly and creates a precise financial ledger before any money moves.

**3. Executing the Transfer via Stripe (Video 02:29 – 03:18)**

- **What is happening:** Now that the amounts are calculated, the system prepares to send the actual funds using Stripe.
- **Visual Cue:** In **Zapier**, you see a step labeled **"Webhooks / Custom Request."**
- **Why it matters:** Standard automation tools often can't handle direct transfers to partner accounts. This custom step speaks directly to Stripe's core engine to securely move funds from the platform to the partner's bank account.

**4. Safety Checks & Error Handling (Video 02:52 & 03:22)**

- **What is happening:** The system includes built-in safety measures.
  - **Speed Control:** You see a **"Delay"** step. This ensures we don't overwhelm the banking system by sending too many requests at once.
  - **Safety Net:** At the end, you see **"Paths"** (Branching logic). This checks the result of every transfer. If a payment fails (e.g., insufficient funds or invalid account), it routes the error to a separate path to alert the team immediately.
- **Why it matters:** This ensures the automation is robust. It prevents system crashes and guarantees that no failed payment goes unnoticed.

**Things to note**
Stripe-related features are disabled.
