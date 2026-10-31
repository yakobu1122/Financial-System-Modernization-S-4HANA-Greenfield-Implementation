# Data Migration Cockpit (LTMC / Migrate Your Data) Guide
## Legacy Tally.ERP 9 Extraction to SAP S/4HANA Staging

This guide documents the technical migration methodology utilized in **Project Apex**. It bridges the raw data extracted from **Tally.ERP 9** to the structured XML/Excel upload files ingested by the **SAP S/4HANA Migration Cockpit (LTMC)**.

---

## 1. Migration Object Architecture

| Migration Sequence | Migration Object Name | Source in Tally | Target SAP S/4HANA Object | Key Excel Worksheets Used |
| :--- | :--- | :--- | :--- | :--- |
| **Step 1** | **G/L Account Master** | List of Accounts / Ledgers | `SKA1` / `SKB1` (Universal Journal) | `Tally_to_SAP_Chart_of_Accounts_Mapping.csv` |
| **Step 2** | **Supplier / Vendor** | Sundry Creditors Ledgers | `BUT000` / `LFA1` (BP Role `FLVN00`) | `Vendor_and_Customer_Business_Partner_Dataset.csv` |
| **Step 3** | **Customer** | Sundry Debtors Ledgers | `BUT000` / `KNA1` (BP Role `FLCU00`) | `Vendor_and_Customer_Business_Partner_Dataset.csv` |
| **Step 4** | **Fixed Asset Master** | Fixed Asset Register | `ANLA` / `ANLB` (Asset Classes `1000`, `2000`) | `Cutover_Trial_Balance_Reconciliation.csv` |
| **Step 5** | **FI Open Items - Vendor** | Bill-by-Bill Payables | `BSEG` (Doc Type `KR`, Offset `999999`) | `Open_Items_Cutover_Upload_Template.csv` |
| **Step 6** | **FI Open Items - Customer**| Bill-by-Bill Receivables | `BSEG` (Doc Type `DR`, Offset `999999`) | `Open_Items_Cutover_Upload_Template.csv` |
| **Step 7** | **FI Historical Balance** | Closing Trial Balance | `ACDOCA` (Offset `999999`) | `Cutover_Trial_Balance_Reconciliation.csv` |

---

## 2. Step-by-Step Execution in SAP Migration Cockpit (`LTMC`)

1. **Launch Migration Cockpit**:
   - In SAP S/4HANA GUI, execute transaction **`LTMC`** (or access Fiori app **Migrate Your Data**).
   - Create Migration Project: `MIG_APEX_FIN` with target database connection.

2. **Download Standard XML Migration Templates**:
   - Select object **Supplier - create with new internal numbering**.
   - Download template spreadsheet (`.xml` format).

3. **Populate Cleansed Data from MS Excel**:
   - Open template in Excel. Copy data from cleansed staging sheet:
     - Sheet `General Data`: Legal Name, Street Address, Postal Code, Country (`IN`), Region (`27` for Maharashtra), Tax Number 3 (`GSTIN`), Tax Number 4 (`PAN`).
     - Sheet `Company Data`: Company Code (`IN10`), Reconciliation Account (`200100`), Payment Terms (`NT30`), Payment Methods (`T`).
     - Sheet `Bank Details`: Bank Country (`IN`), Bank Key (IFSC Code), Account Number.

4. **Upload & Validate**:
   - Upload file back into LTMC web console.
   - Click **Validate Data**: Checks field lengths, mandatory inputs, and data types.

5. **Convert Values (Value Mapping)**:
   - Map legacy values to SAP internal keys:
     - Country: `India` -> `IN`
     - Currency: `Rupees` -> `INR`
     - Reconciliation Account: Automatically mapped to GL `200100`.

6. **Simulate & Execute Import**:
   - Click **Simulate Import**: Executes test commit in memory without saving to database.
   - Review log for zero error status.
   - Click **Execute Import**: Writes records permanently to database tables `BUT000`, `LFA1`, and `ACDOCA`.
