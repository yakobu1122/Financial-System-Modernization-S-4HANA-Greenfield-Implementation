# Excel Data Engineering, Cleansing & Reconciliation Guide
### Project Apex: Tally.ERP 9 to SAP S/4HANA Migration

As a Junior Financial Systems / SAP FICO Analyst, **Microsoft Excel** is your most critical tool during an ERP implementation. Recruiters look for specific, proven Excel capabilities applied to financial data. This document outlines the exact methodology and formulas used to extract, cleanse, map, and reconcile data from Tally.ERP 9 into SAP S/4HANA.

---

## 1. Legacy Data Extraction from Tally.ERP 9
1. **Chart of Accounts & Ledgers**: Exported from `Tally Gateway > Display > List of Accounts > Export (Ctrl + E) > Format: Excel`.
2. **Closing Trial Balance**: Exported from `Tally Gateway > Display > Trial Balance > F12 (Detailed) > Export`.
3. **Open Customer/Vendor Bills**: Exported from `Tally Gateway > Display > Statements of Accounts > Outstandings > Receivables / Payables > F12 (Bill-wise)`.

---

## 2. Data Cleansing Formulas Used in Excel

### A. Removing Leading/Trailing Spaces & Non-Printable Characters
Tally text exports often contain hidden spaces and carriage returns that cause SAP upload errors (e.g., maximum field length overflow in `NAME1` or `STCD1`):
```excel
=TRIM(CLEAN(A2))
```
*Applied to: Vendor Names, Customer Names, City, Street Address.*

### B. Standardizing Case & Legal Entity Titles
```excel
=PROPER(TRIM(A2))
```
*Ensured "tata steel bsl limited" becomes "Tata Steel Bsl Limited".*

### C. Validating Indian GSTIN Format (15 Characters)
Indian Goods and Services Tax Identification Number (GSTIN) follows a strict statutory structure:
- 2 digits (State Code)
- 10 alphanumeric (PAN)
- 1 entity code + 'Z' + 1 checksum digit.
In Excel, length validation was applied:
```excel
=IF(LEN(TRIM(F2))=15, "VALID", "INVALID_LENGTH")
```
And state-code matching against the master state table:
```excel
=IF(LEFT(F2, 2)=TEXT(VLOOKUP(D2, State_Code_Table, 2, FALSE), "00"), "STATE_MATCH", "STATE_MISMATCH")
```

### D. Duplicate Detection (Vendor & Customer PAN Check)
Preventing duplicate Business Partner records in SAP:
```excel
=IF(COUNTIF($G$2:$G$100, G2)>1, "DUPLICATE_PAN", "UNIQUE")
```
*Identified vendors with identical PAN numbers and consolidated them prior to upload.*

---

## 3. Data Transformation & Lookup Mapping Matrix

### A. Dynamic Tally Ledger to SAP GL Account Lookup
When mapping 500+ operational ledgers into standard SAP 6-digit GL accounts:
```excel
=XLOOKUP(B2, Tally_Mapping_Table[Tally_Ledger_Name], Tally_Mapping_Table[SAP_GL_Account], "UNMAPPED_GL", 0)
```
*(Or classic VLOOKUP: `=VLOOKUP(B2, Mapping_Range, 4, FALSE)`)*

### B. Determining SAP Account Types Dynamically
```excel
=IFS(
    ISNUMBER(SEARCH("Recon", G2)), "X (Balance Sheet)",
    ISNUMBER(SEARCH("Revenue", G2)), "P (Primary Cost/Revenue)",
    ISNUMBER(SEARCH("Expense", G2)), "P (Primary Cost/Revenue)",
    TRUE, "X (Balance Sheet)"
)
```

---

## 4. Cutover Trial Balance Reconciliation & Variance Modeling

Recruiters will ask: *"How did you prove that SAP balances matched Tally balances to the penny?"*

The reconciliation sheet was structured as follows:
| Column | Header | Formula |
| :--- | :--- | :--- |
| **A** | SAP GL Code | Direct input from Chart of Accounts |
| **B** | Account Description | `=XLOOKUP(A2, COA_Table[GL], COA_Table[Desc])` |
| **C** | Tally Debit (INR) | `=SUMIFS(Tally_TB[Debit], Tally_TB[SAP_GL], A2)` |
| **D** | Tally Credit (INR) | `=SUMIFS(Tally_TB[Credit], Tally_TB[SAP_GL], A2)` |
| **E** | SAP Loaded Debit (INR) | Pulled from SAP `FAGLL03` or `F.01` Trial Balance report |
| **F** | SAP Loaded Credit (INR)| Pulled from SAP `FAGLL03` or `F.01` Trial Balance report |
| **G** | Debit Variance | `=ROUND(C2 - E2, 2)` |
| **H** | Credit Variance | `=ROUND(D2 - F2, 2)` |
| **I** | Reconciliation Status | `=IF(AND(G2=0, H2=0), "MATCHED", "INVESTIGATE_VARIANCE")` |

### Key Variance Handling During Project Apex:
* **Foreign Currency Open Items**: Tally recorded customer *Detroit Heavy Gear* at book rate ₹78/USD. S/4HANA required cutover revaluation at March 31 RBI spot rate (₹83.12/USD). The ₹7.88/USD variance was routed to the Foreign Exchange Fluctuation Clearing GL, resolving the variance to 0.00.

---

## 5. Pivot Tables & Executive Reporting in Excel
1. **Subledger vs. General Ledger Verification**:
   - Built a Pivot Table summarizing all open vendor items by Vendor Group.
   - Compared against the SAP Reconciliation Account (`200100` Domestic RM & `200110` Services).
2. **Aging Analysis Matrix**:
   - Calculated outstanding aging buckets using `=DATEDIF(Due_Date, Cutover_Date, "D")`.
   - Bucketed into: `0-30 Days`, `31-60 Days`, `61-90 Days`, `>90 Days`.
