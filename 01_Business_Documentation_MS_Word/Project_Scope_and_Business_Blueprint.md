# Business Blueprint (BBP): Financial Accounting & Controlling
## Project Apex: SAP S/4HANA Greenfield Implementation

```text
Document Reference: APEX-BBP-FI-001
Client: Apex Global Dynamics Pvt. Ltd.
Author: Associate SAP FICO / Financial Systems Consultant
Version: 1.0 (Final Approved)
Tool / Application: Authored in Microsoft Word
```

---

## 1. Executive Summary & Project Objectives
Apex Global Dynamics Pvt. Ltd. is an automotive component manufacturer operating plants in Pune and Chennai, with a corporate sales office in Mumbai. Prior to this project, financial records were maintained on **Tally.ERP 9**. 

### 1.1 Objectives of the Implementation
1. **Unified Financial Data Model**: Implement SAP S/4HANA **Universal Journal (`ACDOCA`)** to eliminate reconciliation between Financial Accounting (FI) and Management Accounting (CO).
2. **Standardized Chart of Accounts**: Replace decentralized Tally ledgers with a unified corporate Chart of Accounts (`COA1`).
3. **Statutory Compliance**: Ensure seamless integration with Indian GST (Input/Output CGST, SGST, IGST, RCM) and Withholding Tax (TDS u/s 194C, 194J, 194I, 194Q).
4. **Automated Sub-ledger Integration**: Deploy the S/4HANA **Business Partner (BP)** model for Vendors and Customers with automated Reconciliation Accounts.
5. **Real-Time Cost Controlling**: Implement Cost Center Accounting (CCA) and Profit Center Accounting (PCA) to track plant-level profitability.

---

## 2. Enterprise Structure Design (Organizational Units)

| Org Unit | Code | Description | Currency | Mapping to Tally |
| :--- | :--- | :--- | :--- | :--- |
| **Client** | `400` | SAP Production Client | N/A | Apex Company File |
| **Company Code** | `IN10` | Apex Global Dynamics Pvt. Ltd. | INR | Tally Primary Company |
| **Chart of Accounts** | `COA1` | Apex Global Standard COA | INR | Tally Groups & Ledgers |
| **Fiscal Year Variant**| `V3` | April to March (12 periods + 4 special) | N/A | Tally FY 2023-24 |
| **Posting Period Var**| `PP10` | Posting Period Variant for IN10 | N/A | Tally Period Lock |
| **Field Status Variant**| `FS10` | Field Status Variant for IN10 | N/A | Tally Voucher Config |
| **Controlling Area** | `1000` | Apex Controlling Area (Cross-CoCode ready) | INR | Tally Cost Categories |
| **Credit Control Area**| `CR10` | Apex Centralized Credit Control | INR | Tally Credit Limits |

---

## 3. As-Is (Tally.ERP 9) vs. To-Be (SAP S/4HANA) Process Matrix

| Business Process | As-Is (Tally.ERP 9) | To-Be (SAP S/4HANA FICO) | Strategic Benefit |
| :--- | :--- | :--- | :--- |
| **General Ledger Posting** | Standalone voucher entries (`F7: Journal`). No field status enforcement. | Dual-entry with Field Status Groups (`G001`, `G005`), Universal Journal `ACDOCA` real-time posting. | 100% audit trail, mandatory cost center assignment on expense lines. |
| **Vendor Management** | Created as Sundry Creditors under single group. No separate tax codes. | Unified **Business Partner (BP)** with Roles `000000` (General), `FLVN00` (FI Vendor), `FLVN01` (Purchasing). | Single master record across finance and supply chain. No duplicate vendors. |
| **Vendor Invoicing (P2P)**| Manual entry in Purchase Voucher (`F9`). No automatic 3-way match. | Standard invoice verification via `FB60` (Direct FI) and `MIRO` (PO-based 3-way match against GRN). | Eliminates over-billing and duplicate payments. |
| **Payment Processing** | Manual cheque writing or individual portal banking uploads. | **Automatic Payment Program (F110)** generating automated DMEE payment files and bank advice. | Processes 200+ vendor disbursements in a single run with bank integration. |
| **Customer Billing (O2C)**| Sales Voucher (`F8`) with basic bill-by-bill reference. | Customer Invoicing via `FB70` (Direct FI) or `VF01` (SD Billing), linked to BP Customer Roles (`FLCU00`/`FLCU01`). | Credit limit check (`OVA8`), automated Dunning (`F150`) for overdue accounts. |
| **Asset Accounting** | Manual depreciation vouchers calculated in Excel at year-end. | **New Asset Accounting** in S/4HANA. Real-time online posting to GL for acquisition, retirement, and periodic `AFAB` depreciation. | Audit-compliant depreciation under Companies Act 2013 and Income Tax Act. |
| **Cost Allocation** | Basic Cost Centers in Tally with manual journal distribution. | Cost Center Accounting (CCA) with internal activity allocation, assessment cycles, and Profit Centers (PCA). | Real-time plant-wise and product-line EBITDA reporting. |

---

## 4. Sub-Module Functional Specifications

### 4.1 General Ledger (FI-GL)
* **Leading Ledger (`0L`)**: Configured for Indian Local GAAP and Companies Act 2013 compliance.
* **Non-Leading Ledger (`2L`)**: Configured for IFRS / Group reporting (calendar year January to December).
* **Document Splitting**: Active with zero-balance indicator on **Profit Center**, enabling automatic balance sheets by operating plant (Pune vs. Chennai).

### 4.2 Accounts Payable (FI-AP)
* **Reconciliation GL Accounts**: 
  - `200100` (Trade Creditors - Domestic)
  - `200110` (Trade Creditors - Services)
  - `200120` (Trade Creditors - Imports)
* **Payment Terms**:
  - `0001`: Immediate payment
  - `NT15`: Net 15 days
  - `NT30`: Net 30 days
  - `NT45`: Net 45 days
* **TDS Configuration**: Extended Withholding Tax configured for Section 194C (1% Individual, 2% Company), Section 194J (10% / 2% Technical), and Section 194I (10% Rent).

### 4.3 Accounts Receivable (FI-AR)
* **Reconciliation GL Accounts**:
  - `100500` (Trade Debtors - Domestic)
  - `100510` (Trade Debtors - Foreign)
* **Dunning Procedure (`AP01`)**:
  - Level 1: Polite reminder (10 days past due).
  - Level 2: Formal demand letter + phone call (20 days past due).
  - Level 3: Legal notice + credit hold (35 days past due).

### 4.4 Controlling (CO-CCA & CO-PCA)
* **Controlling Area**: `1000` (Cost Center standard hierarchy `APEX_H1`).
* **Cost Centers**:
  - `CC_PUN_PROD` (Pune Production Plant)
  - `CC_CHN_PROD` (Chennai Production Plant)
  - `CC_CORP_ADM` (Corporate Administration)
  - `CC_SALES_MKT` (Sales & Marketing)
* **Profit Centers**:
  - `PC_AUTO_PART` (Automotive Component Business)
  - `PC_AFTER_MKT` (Aftermarket Spare Parts)

---

## 5. Cutover Strategy & Sign-Off Criteria
1. **Freeze Legacy Postings in Tally**: Books closed on March 31, 2024 at 18:00 IST.
2. **Master Data Migration**: Uploaded via SAP Migration Cockpit (LTMC).
3. **Open Item Cutover**: Posted via technical cutover clearing account (`999999`).
4. **Sign-off**: Requires joint verification between Implementation Lead and Client CFO.
