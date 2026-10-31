# Standard Operating Procedure (SOP) & End-User Training Guide
## Finance & Accounting Operations in SAP S/4HANA

```text
Document: APEX-SOP-FIN-002
Intended Audience: Finance Executives, AP/AR Accountants, Plant Billing Clerks
Transitioning From: Tally.ERP 9 to SAP S/4HANA
Author: Associate SAP FICO / Business Systems Analyst
Platform: Authored in Microsoft Word
```

---

## 1. Quick Navigation: Tally Shortcuts vs. SAP T-Codes & Fiori Apps

| Financial Action | Tally.ERP 9 Method | SAP GUI Transaction | SAP S/4HANA Fiori App Tile |
| :--- | :--- | :--- | :--- |
| **Post General Ledger Journal** | `Gateway > Accounting Vouchers > F7 (Journal)` | `FB50` | *Manage Journal Entries / Post General Journal Entry* |
| **Vendor Invoice Booking (Direct FI)** | `Gateway > Accounting Vouchers > F9 (Purchase)` | `FB60` | *Create Incoming Invoices* |
| **Vendor Payment (Manual)** | `Gateway > Accounting Vouchers > F5 (Payment)` | `F-53` | *Post Outgoing Payments* |
| **Automatic Vendor Bulk Payment** | Not Available (Manual Banking Portal Upload) | `F110` | *Manage Automatic Payments* |
| **Customer Billing (Direct FI)** | `Gateway > Accounting Vouchers > F8 (Sales)` | `FB70` | *Create Outgoing Invoices* |
| **Customer Payment Receipt** | `Gateway > Accounting Vouchers > F6 (Receipt)` | `F-28` | *Post Incoming Payments* |
| **Display GL Account Balances** | `Display > Account Books > Ledger` | `FS10N` / `FAGLB03` | *Display GL Account Balances* |
| **Display Vendor / Customer Line Items** | `Display > Statement of Accounts > Outstandings` | `FBL1N` / `FBL5N` | *Manage Vendor / Customer Line Items* |
| **Execute Depreciation Run** | Manual calculation in Excel | `AFAB` | *Schedule Asset Accounting Jobs* |

---

## 2. Procedure 1: Booking a Vendor Invoice (`FB60` / Fiori)

### Purpose
To record an incoming non-PO supplier or service invoice with applicable GST and TDS deduction.

### Step-by-Step Instructions
1. Enter transaction code **`FB60`** or open Fiori app **Create Incoming Invoices**.
2. **Header Screen Fields**:
   * **Vendor**: Enter SAP Business Partner number (e.g., `500001` for Tata Steel BSL Ltd).
   * **Invoice Date**: Enter vendor's physical invoice date.
   * **Posting Date**: Enter current open posting period date.
   * **Amount**: Enter total gross invoice value (e.g., `₹45,00,000`).
   * **Tax Code**: Select appropriate tax code:
     - `I1`: Input CGST 9% + SGST 9% (Intra-state Maharashtra)
     - `I2`: Input IGST 18% (Inter-state supply)
   * **Calculate Tax**: Check the checkbox `[X] Calculate Tax`.
3. **Item Details (GL Line Items)**:
   * **G/L Account**: Enter expense account (e.g., `500100` Raw Material Consumption).
   * **D/C**: Select `Debit`.
   * **Amount**: Leave blank or enter net base amount (system auto-calculates if left blank).
   * **Cost Center**: Enter cost center (e.g., `CC_PUN_PROD` for Pune Plant).
4. **Withholding Tax (TDS) Tab**:
   * Verify TDS code automatically populated (e.g., `C1` for Section 194C Contractor 2%).
5. **Simulate & Post**:
   * Click **Simulate** (`F9`). Verify the generated accounting entries:
     - *Dr. Raw Material Consumption (`500100`)*: Base Amount
     - *Dr. Input CGST Receivable (`100800`)*: 9% Tax
     - *Dr. Input SGST Receivable (`100810`)*: 9% Tax
     - *Cr. Vendor Account (`500001`)*: Net Payable Amount
     - *Cr. TDS Payable u/s 194C (`200600`)*: Withheld TDS Amount
   * Verify the balance traffic light is **Green** (Balance = 0.00).
   * Click **Post** (`Ctrl + S`). Document number generated (e.g., `1900000045`).

---

## 3. Procedure 2: Executing Automatic Payment Program (`F110`)

### Purpose
To execute batch disbursements for due vendor open invoices, generate bank payment advice, and produce payment media files.

### Step-by-Step Instructions
1. Enter transaction code **`F110`**.
2. **Parameters Tab**:
   * **Run Date**: Enter today's date (e.g., `30.04.2024`).
   * **Identification**: Enter a 5-character ID (e.g., `AP001`).
   * **Posting Date**: Payment date for bank credit.
   * **Docs Entered Up To**: Enter cut-off date for invoices.
   * **Company Code**: `IN10`.
   * **Payment Method**: `T` (Bank Electronic Transfer).
   * **Next Posting Date**: Set date of the subsequent scheduled payment run (e.g., `07.05.2024`).
   * **Vendor Accounts**: Enter range `500001` to `500050`.
3. **Additional Log Tab**:
   * Select `[X] Due date check`, `[X] Payment method selection in all cases`, and `[X] Line items of the payment documents`.
4. **Proposal Run**:
   * Click **Proposal** (`F6`) > Select `Start Immediately`.
   * Review proposal log (`F7` or click glasses icon).
   * Ensure no vendors appear in the exception list (red status).
5. **Payment Run**:
   * Click **Payment Run** (`F8`). Confirm execution.
   * Output: SAP posts payment documents (`KZ`), clears vendor line items from `FBL1N`, and credits the Outward Clearing Bank GL (`100120`).

---

## 4. Procedure 3: Customer Receipt & Open Item Clearing (`F-28`)

### Purpose
To record payment received from a customer against outstanding sales invoices.

### Step-by-Step Instructions
1. Enter transaction code **`F-28`**.
2. **Header Information**:
   * **Document Date & Posting Date**: Date of bank credit advice.
   * **Company Code**: `IN10`.
   * **Currency**: `INR`.
   * **Account (Bank)**: `100110` (ICICI Inward Clearing).
   * **Amount**: Net amount credited by bank (e.g., `₹84,00,000`).
3. **Open Item Selection**:
   * **Account**: Customer BP number (e.g., `100001` Bajaj Auto Limited).
   * **Account Type**: `D` (Customer).
4. **Process Open Items**:
   * Click **Process Open Items**.
   * Double-click the matching invoice number (e.g., `INV-2324-1120`).
   * Verify "Not Assigned" balance equals **`0.00`**.
5. **Post**:
   * Click **Post** (`Ctrl + S`). Document number `1400000012` is generated.
   * Both invoice and payment are now cleared in Customer Line Item Display (`FBL5N`).

---

## 5. Troubleshooting Common End-User Errors

| Error Message in SAP | Root Cause | Exact Resolution |
| :--- | :--- | :--- |
| **"Posting period 001 2024 is not open for variant PP10"** | The accounting period is locked. | Request Finance Lead to open period via transaction **`OB52`** for account type `+` and `S`. |
| **"No authorization for Company Code IN10"** | User role missing security profile. | Contact SAP Security team to assign role `Z_FI_AP_USER` to user ID. |
| **"Account 500100 requires an assignment to a CO object"** | Expense GL configured as Cost Element (Type 1) without a cost center. | Enter a valid Cost Center (e.g., `CC_PUN_PROD`) in the line item field or configure default via `OKB9`. |
| **"Tax code I1 does not exist for country IN"** | Configuration missing in FTXP. | Verify tax code setup under Country Key `IN` via `FTXP` or assign correct tax procedure. |
