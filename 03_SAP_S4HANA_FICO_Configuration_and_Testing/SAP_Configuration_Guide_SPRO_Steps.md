# SAP S/4HANA Finance (FICO) SPRO Configuration Runbook
## Project Apex: Greenfield Enterprise Implementation

This document serves as the comprehensive configuration manual. It details the exact **SPRO (IMG) Menu Paths**, **Transaction Codes (T-Codes)**, and configuration values implemented for **Apex Global Dynamics Pvt. Ltd. (Company Code `IN10`)**.

---

## Section 1: Enterprise Structure Configuration

### 1.1 Define Company (Trading Partner / Group Level)
* **T-Code**: `OX15`
* **IMG Path**: *Enterprise Structure > Definition > Financial Accounting > Define Company*
* **Field Values**:
  - Company: `APEX01`
  - Company Name: `Apex Global Holdings Ltd`
  - City / Country / Currency / Language: `Mumbai` / `IN` / `INR` / `EN`

### 1.2 Define Company Code (Legal Entity)
* **T-Code**: `OX02`
* **IMG Path**: *Enterprise Structure > Definition > Financial Accounting > Edit, Copy, Delete, Check Company Code > Edit Company Code Data*
* **Field Values**:
  - Company Code: `IN10`
  - Company Name: `Apex Global Dynamics Pvt. Ltd.`
  - City / Country / Currency / Language: `Pune` / `IN` / `INR` / `EN`

### 1.3 Assign Company Code to Company
* **T-Code**: `OX16`
* **IMG Path**: *Enterprise Structure > Assignment > Financial Accounting > Assign Company Code to Company*
* **Assignment**: `IN10` -> `APEX01`

### 1.4 Maintain Controlling Area
* **T-Code**: `OX06` / `OKKP`
* **IMG Path**: *Enterprise Structure > Definition > Controlling > Maintain Controlling Area*
* **Field Values**:
  - Controlling Area: `1000`
  - Name: `Apex Controlling Area`
  - CoCd -> CO Area: `Cross-Company-Code Controlling`
  - Currency Type: `10` (Company Code Currency - `INR`)
  - Chart of Accounts: `COA1`
  - Fiscal Year Variant: `V3`
  - Cost Center Standard Hierarchy: `APEX_H1`

### 1.5 Assign Company Code to Controlling Area
* **T-Code**: `OX19`
* **IMG Path**: *Enterprise Structure > Assignment > Controlling > Assign Company Code to Controlling Area*
* **Assignment**: `IN10` -> `1000`

---

## Section 2: Financial Accounting Global Settings

### 2.1 Maintain Fiscal Year Variant
* **T-Code**: `OB29`
* **IMG Path**: *Financial Accounting > Financial Accounting Global Settings > Ledgers > Fiscal Year and Posting Periods > Maintain Fiscal Year Variant*
* **Configuration**: Variant **`V3`** (April to March, 12 Normal Posting Periods + 4 Special Closing Periods). Special periods used for year-end audit adjustments.

### 2.2 Assign Company Code to Fiscal Year Variant
* **T-Code**: `OB37`
* **Assignment**: `IN10` -> `V3`

### 2.3 Define Posting Period Variant & Assign to Company Code
* **T-Code**: `OBBO` (Define Variant: `PP10` for Apex India)
* **T-Code**: `OBBP` (Assign `IN10` -> `PP10`)
* **T-Code**: `OB52` (Open and Close Posting Periods):
  - Account Type `+` (Valid for all accounts): Period `1` to `12` open for Year `2024`.
  - Account Types `A` (Assets), `D` (Customers), `K` (Vendors), `M` (Materials), `S` (General Ledger).

### 2.4 Define Field Status Variant & Field Status Groups
* **T-Code**: `OBC4` (Copied from standard `0001` to `FS10`)
* **T-Code**: `OBC5` (Assign `IN10` -> `FS10`)
* **Key Field Status Groups Configured**:
  - `G001`: General with text (Cost center optional)
  - `G005`: Bank accounts (Value date mandatory)
  - `G006`: Material stock accounts
  - `G029`: Revenue accounts (Profit center mandatory)
  - `G067`: Reconciliation accounts (Vendors/Customers - Reference number required)

### 2.5 Document Types and Number Ranges
* **T-Code**: `OBA7` (Document Types) & `FBN1` (Number Ranges for Company Code `IN10`)
  - `SA`: G/L Account Document (Range `01`: `1000000000` - `1999999999`)
  - `KR`: Vendor Invoice (Range `19`: `1900000000` - `1999999999`)
  - `KZ`: Vendor Payment (Range `15`: `1500000000` - `1599999999`)
  - `DR`: Customer Invoice (Range `18`: `1800000000` - `1899999999`)
  - `DZ`: Customer Payment (Range `14`: `1400000000` - `1499999999`)
  - `AA`: Asset Posting (Range `01`)

---

## Section 3: General Ledger Accounting (S/4HANA Specifics)

### 3.1 S/4HANA Innovation: GL Account Types
Unlike legacy ECC, in S/4HANA GL Master records (`FS00`) are created with one of 4 Account Types:
1. **X - Balance Sheet Account**: Cash, Bank, Assets, Liabilities, Inventory.
2. **N - Non-operating Expense/Income**: Bank interest charges, gains on asset sales.
3. **P - Primary Costs or Revenue**: Direct raw materials, factory wages, sales revenues (integrated with Controlling cost elements).
4. **S - Secondary Costs**: Internal activity allocation, assessment cycles, overhead distribution (no longer maintained in CO via KA06; now part of unified FS00!).

### 3.2 S/4HANA Innovation: Universal Journal (`ACDOCA`)
* Single table `ACDOCA` acts as the single source of truth.
* Merges data from legacy tables: `BSEG`, `BSIS`, `BSAS`, `BSID`, `BSAD`, `BSIK`, `BSAK`, `COEP`, `FAGLFLEXA`, and `ANEP`.
* Provides instantaneous reporting across company codes, profit centers, and cost centers without batch settlement.

### 3.3 Configure Document Splitting
* **IMG Path**: *Financial Accounting > General Ledger Accounting > Business Transactions > Document Splitting*
* **Active**: Yes
* **Splitting Characteristic**: **Profit Center** (Check `Zero Balance` checkbox).
* *Business Impact*: When an invoice with multiple profit centers is booked, the system automatically balances balance sheet lines (e.g. Accounts Payable and GST lines) across profit centers, enabling independent Balance Sheets by Plant!

---

## Section 4: Business Partner (BP) & Customer-Vendor Integration (CVI)

In SAP S/4HANA, legacy transactions `XK01`/`XD01` are obsolete and redirect to **`BP`**.

### 4.1 Define BP Grouping and Number Ranges
* **IMG Path**: *Cross-Application Components > SAP Business Partner > Business Partner > Basic Settings > Number Ranges and Groupings*
* Define Grouping `VEND` -> Range `500001` - `599999` (Internal Numbering).
* Define Grouping `CUST` -> Range `100001` - `199999` (Internal Numbering).

### 4.2 Customer-Vendor Integration (CVI) Synchronization
* **IMG Path**: *Master Data Synchronization > Customer/Vendor Integration > Business Partner Settings > Settings for Vendor Integration > Field Assignment for Vendor Integration > Assign Keys*
* Linked BP Grouping `VEND` to Vendor Account Group `VEND` with same number range.
* Linked BP Grouping `CUST` to Customer Account Group `CUST` with same number range.

### 4.3 Mandatory BP Roles Assigned
* **`000000` (Business Partner General)**: Legal name, address, PAN, search terms.
* **`FLVN00` (FI Vendor - Company Code Level)**: Reconciliation Account (`200100`), Payment Terms (`NT30`), Withholding Tax (TDS).
* **`FLVN01` (Purchasing Vendor)**: Order currency (`INR`), Incoterms, GR-based invoice verification.
* **`FLCU00` (FI Customer - Company Code Level)**: Reconciliation Account (`100500`), Payment Terms (`NT45`).
* **`FLCU01` (Sales Customer)**: Sales Org `1000`, Distribution Channel `10`, Pricing Procedure.

---

## Section 5: Accounts Payable - Automatic Payment Program (`FBZP`)

Configured all 6 sub-steps of `FBZP`:
1. **All Company Codes**: Payer Company Code `IN10`, Sending Company Code `IN10`, Outgoing payment tolerance days = `3`.
2. **Paying Company Codes**: Minimum amount for outgoing payment = `INR 100.00`.
3. **Payment Methods in Country**: Country `IN`, Payment Method `T` (Bank Transfer / NEFT / RTGS).
4. **Payment Methods in Company Code**: Assigned Method `T` to Company Code `IN10`.
5. **Bank Determination**: Ranked Bank `HDFC` as Rank `1`. Assigned Bank Sub-Account `100120` (Outward Clearing Account).
6. **House Banks (`FI12`)**: House Bank `HDFC1`, Account ID `ACT01`, Bank Account `5020001`, Currency `INR`.

---

## Section 6: Asset Accounting (New Asset Accounting in S/4HANA)

### 6.1 Chart of Depreciation
* **T-Code**: `EC08`
* Copied reference chart of depreciation `0IN` (India) to `APEX`.

### 6.2 Asset Classes Defined
* `1000`: Factory Buildings (Useful life 30 years, Depreciation Key `SLM1` Straight Line)
* `2000`: Plant & Machinery (Useful life 15 years, Depreciation Key `WDV1` Written Down Value)
* `3000`: Office & IT Equipment (Useful life 3 years, Depreciation Key `SLM1`)

### 6.3 Real-Time Integration with General Ledger
* In S/4HANA New Asset Accounting, depreciation and acquisitions post directly into ledger `0L` (Local GAAP) and `2L` (IFRS) simultaneously in table `ACDOCA` without requiring delta depreciation runs (`ASKB` is obsolete in S/4HANA!).
