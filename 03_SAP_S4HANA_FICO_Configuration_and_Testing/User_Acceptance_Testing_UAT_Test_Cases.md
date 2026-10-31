# User Acceptance Testing (UAT) Execution Register
## Project Apex: Financial Processes in SAP S/4HANA

```text
Document: APEX-UAT-REG-003
Client: Apex Global Dynamics Pvt. Ltd. (Company Code IN10)
Test Environment: Quality Assurance Client (QAS 300)
Tested By: Junior SAP FICO Analyst & Client Finance Team
Status: 100% Passed (Ready for Go-Live)
```

---

## Master UAT Test Execution Summary

| Test Case ID | Business Scenario / Module | T-Code / Fiori App | Test Data | Expected Result | Actual Result | Status |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **TC-GL-001** | Post General Ledger Journal Voucher | `FB50` | Dr. Factory Rent (`510200`) INR 2,50,000<br>Cr. Bank (`100100`) INR 2,50,000 | Document generated with document type `SA`. Balance sheet and P&L updated in `ACDOCA`. | Document `1000000015` posted. Cost center validated. | **PASS** |
| **TC-GL-002** | Test Document Splitting on Profit Center | `FB50` | Dr. Pune Office Expense (PC `PC_AUTO_PART`)<br>Dr. Chennai Expense (PC `PC_AFTER_MKT`)<br>Cr. Bank | System automatically generates zero-balance clearing entries by profit center. | Verified in `ACDOCA`. Split lines generated accurately. | **PASS** |
| **TC-GL-003** | Test Period Lock Enforcement | `FB50` | Post entry in closed period (Period 12 of prior FY) | System blocks transaction with error `OB52: Period closed`. | System displayed error message `F5 286`. | **PASS** |
| **TC-AP-001** | Create Domestic Vendor Business Partner | `BP` | Vendor: *Tata Steel BSL Ltd*, Group `VEND`, Role `FLVN00` | BP number `500001` created. Linked to Recon Account `200100` and Payment Terms `NT30`. | BP `500001` generated. Tables `BUT000` & `LFA1` updated. | **PASS** |
| **TC-AP-002** | Post Vendor Invoice with GST & TDS | `FB60` | Vendor `500001`, Gross: INR 45,00,000, Tax `I1` (CGST 9% + SGST 9%), TDS `C1` (2%) | Base: INR 38,13,559.32<br>CGST: INR 3,43,220.34<br>SGST: INR 3,43,220.34<br>TDS: INR 76,271.19<br>Vendor Net: INR 44,23,728.81 | Document `1900000045` posted. All tax line items calculated exactly. | **PASS** |
| **TC-AP-003** | Vendor Line Item Display | `FBL1N` | Vendor `500001` | Displays open item with red icon showing net payable and due date `14.04.2024`. | Open line item visible with correct payment baseline date. | **PASS** |
| **TC-AP-004** | Execute Automatic Payment Run (F110) | `F110` | Run Date `30.04.2024`, ID `AP001`, Payment Method `T` | Due vendor open items cleared. Clearing payment doc `1500000021` created (`Doc Type KZ`). | Payment run completed with 0 errors. Bank sub-account credited. | **PASS** |
| **TC-AR-001** | Create Domestic Customer Business Partner | `BP` | Customer: *Bajaj Auto Ltd*, Group `CUST`, Role `FLCU00` | BP number `100001` created. Recon Account `100500`, Payment Terms `NT45`. | BP `100001` created. Tables `BUT000` & `KNA1` updated. | **PASS** |
| **TC-AR-002** | Post Customer Direct Sales Invoice | `FB70` | Customer `100001`, Gross: INR 84,00,000, Tax `O1` (Output CGST/SGST 18%) | Dr. Customer Recon (`100500`) INR 84,00,000<br>Cr. Revenue (`400100`) INR 71,18,644.07<br>Cr. Output GST Payable INR 12,81,355.93 | Document `1800000030` posted. Customer ledger updated. | **PASS** |
| **TC-AR-003** | Post Incoming Customer Receipt & Clear Invoice | `F-28` | Customer `100001`, Amount INR 84,00,000 | Payment matched against invoice `1800000030`. Both items marked cleared (Green status). | Document `1400000012` posted. Open items cleared in `FBL5N`. | **PASS** |
| **TC-AA-001** | Create Asset Master Record | `AS01` | Asset Class `2000` (Plant & Machinery), Description: CNC Milling Machine #04 | Asset Master `200001-0` created under Cost Center `CC_PUN_PROD`. | Asset number `200001-0` generated with depreciation key `WDV1`. | **PASS** |
| **TC-AA-002** | Asset Acquisition via Vendor | `F-90` | Asset `200001-0`, Vendor `500002`, Amount INR 35,00,000 | Capitalizes asset in real-time. Dr. Asset Cost (`110100`), Cr. Vendor (`200100`). | Asset Explorer (`AW01N`) displays acquisition value of INR 35,00,000. | **PASS** |
| **TC-AA-003** | Execute Periodic Asset Depreciation | `AFAB` | Company Code `IN10`, Period 01, Year 2024 (Test Run & Live Run) | System calculates depreciation and posts: Dr. Depr Expense (`530100`), Cr. Acc Depr (`110110`). | Depreciation run posted successfully. Log verified. | **PASS** |
| **TC-FIN-001**| Generate Financial Statements (Balance Sheet & P&L) | `F.01` / Fiori | Financial Statement Version `AP01`, Period 01 to 01, CoCode `IN10` | Real-time Balance Sheet and Profit & Loss report generated matching Trial Balance. | Report generated in 1.4 seconds from table `ACDOCA`. | **PASS** |

---

## Key Sign-Off Notes
* All 14 core business transactions executed in the QAS client without functional deviations.
* Tax compliance verified against Indian GST and TDS rates.
* Financial statement figures reconciled against baseline Excel models.
