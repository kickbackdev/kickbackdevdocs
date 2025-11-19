# Kickback Dev Docs

Central documentation hub for Kickback operations.  
This index is the top level for multiple processes (e.g., Contract Automation, Settlement) and points to the detailed docs inside each process.

---

## Kickback Platform Automation Operations Manual

This manual is the umbrella guide for each automation stream (contract, settlement, etc.).

- [Kickback Platform Automation Operations Manual](1.runbook/platform_operations_manual.md)
  - **Contract Register Automation (KKB-F1)**  
    - [KKB-F1-Contract-Register-[dev] Operations Manual (Runbook)](1.runbook/README.md)
    - **Mappings**  
      - [Field Mapping Specification](2.mappings/field_mapping.md)  
      - Raw spreadsheet: `2.mappings/field_mapping.csv.xlsx`
    - **Automation Paths**  
      - [Automation Paths Overview](3.paths/paths_overview.md)
    - **Tests**  
      - [E2E Test Results Summary](4.tests/e2e_result.md)  
      - Raw spreadsheet: `4.tests/E2E_results.xlsx`
    - **Screenshots**  
      - Common utilities and filters: `5.screenshots/common_utility_and_filters.png`  
      - Path A – Shop partner flow: `5.screenshots/path A - shop partner.png`  
      - Path B – Individual partner flow: `5.screenshots/path B - individual partner.png`  
      - Part C – Ambassador partner flow: `5.screenshots/part C - Ambassador partner.png`  
      - Part C – Parts Addendum flow: `5.screenshots/part C - part Addendum.png`
    - **Slack Evidence**  
      - Ambassador contract test: `6.slack/Ambassador Test ScreenShot.png`  
      - Individual contract test: `6.slack/Individual Test Screenshot.png`  
      - Parts error test: `6.slack/Parts Error Test Screenshot .png`  
      - Parts success test: `6.slack/Parts Test Screenshot.png`  
      - Shop test: `6.slack/Shop Test ScreenShot.png`
    - **Airtable Automations & Schema**  
      - [Warehouse & Airtable Automation Design](7.automation/warehouse_automation.md)  
      - [Kickback Operations Data Schema](schema.md)
  - **Settlement Operations**  
    - Settlement process documentation will be added to this branch. When content is ready, new directories and links will appear here.

