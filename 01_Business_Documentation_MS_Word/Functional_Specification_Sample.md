# Functional Specification (FS): Custom Enhancement
## Automated Electronic Vendor Payment Advice & GST E-Way Reference

```text
Document Reference: APEX-FS-FI-005
RICEFW Object Type: Enhancement / Form (SmartForm/Adobe Form & Output Control)
Module: SAP S/4HANA Finance (FI-AP)
Author: Associate SAP FICO Functional Analyst
Application: Authored in Microsoft Word
```

---

## 1. Business Requirement & Problem Statement
During legacy operations on **Tally.ERP 9**, the accounts team had to manually print vendor payment vouchers, scan them, and email individual remittance notes to suppliers. This resulted in:
* Over 35 vendor inquiries daily regarding unallocated payments.
* Delays in vendors reconciling TDS deducted at source.
* Inability to automatically attach the bank's UTR (Unique Transaction Reference) number.

### Proposed Solution in SAP S/4HANA
Develop a custom Adobe Form / SmartForm program (`ZFI_PAYMENT_ADVICE`) triggered automatically when the **Automatic Payment Program (`F110`)** posts payment documents (`Document Type KZ`). The program extracts payment details, formats an audit-compliant PDF advice letter, and emails it directly to the vendor's registered email address stored in the **Business Partner** master.

---

## 2. Technical & Data Mapping Specifications

### 2.1 Trigger Event
* **Event**: Successful execution of `F110` Payment Run.
* **Output Medium**: External Send (Email PDF attachment) via SAP Business Workplace (SOST).

### 2.2 Source Tables & Fields

| Output Field Name | SAP S/4HANA Source Table | Field Name | Transformation / Logic |
| :--- | :--- | :--- | :--- |
| **Payer Company Name** | `T001` | `BUTXT` | Where `BUKRS = IN10` |
| **Payer GSTIN** | `J_1BBRANCH` | `GSTIN` | Branch tax registration |
| **Vendor Code** | `LFA1` / `BUT000` | `LIFNR` / `PARTNER` | Vendor Business Partner ID |
| **Vendor Name** | `BUT000` | `NAME_ORG1` | Primary Business Partner Name |
| **Vendor Email ID** | `ADR6` | `SMTP_ADDR` | Linked via `LFA1-ADRNR` |
| **Payment Document No**| `BKPF` | `BELNR` | Cleared payment doc (Doc Type `KZ`) |
| **Payment Date** | `BKPF` | `BUDAT` | Posting date of payment |
| **Bank UTR Number** | `REGUH` | `CHCTN` / `ZUTR` | Electronic bank reference from DMEE |
| **Cleared Invoice No** | `BSEG` | `REBZT` / `XBLNR` | Original vendor invoice reference |
| **Gross Invoice Amount**| `BSEG` | `WRBTR` | Original amount before deductions |
| **TDS Withheld Amount** | `WITH_ITEM` | `WT_QBSHH` | Section 194C / 194J deduction |
| **Net Paid Amount** | `REGUH` | `RBETR` | Actual bank transfer amount |

---

## 3. Form Layout & Design Mockup

```text
+--------------------------------------------------------------------------+
|  APEX GLOBAL DYNAMICS PVT. LTD.                                          |
|  Plant 1: Chakan Industrial Area, Pune, Maharashtra - 410501            |
|  GSTIN: 27AAACA9999P1Z1 | PAN: AAACA9999P | Email: accounts@apexgroup.in |
+--------------------------------------------------------------------------+
|                        REMITTANCE PAYMENT ADVICE                         |
+--------------------------------------------------------------------------+
|  To: Tata Steel BSL Limited                    Date: 30.04.2024          |
|  Vendor Code: 500001                           Payment Doc: 1500000045   |
|  GSTIN: 27AAACT2727Q1ZT                        Bank UTR: HDFCN241209981  |
+--------------------------------------------------------------------------+
| Invoice Ref      Date        Gross Amt (INR)   TDS Ded (INR)   Net Paid  |
| ------------------------------------------------------------------------ |
| TS-MUM-99120   15.03.2024      4,500,000.00       90,000.00   4,410,000  |
| TS-MUM-99450   25.03.2024      3,800,000.00       76,000.00   3,724,000  |
| ------------------------------------------------------------------------ |
| TOTAL PAID (INR):                                          8,134,000.00  |
+--------------------------------------------------------------------------+
| Amount in Words: Rupees Eighty One Lakh Thirty Four Thousand Only        |
| Bank: HDFC Bank Main Account (5020001)                                   |
| *This is a system generated advice and does not require signature.       |
+--------------------------------------------------------------------------+
```

---

## 4. Test Scenarios & Acceptance Criteria
1. **Single Invoice Payment**: Verify math: Net = Gross - TDS. Confirm email delivered to vendor.
2. **Multiple Invoices Consolidated**: Verify multiple lines appear under a single payment advice document.
3. **Invalid Email Exception**: If `ADR6-SMTP_ADDR` is blank, system routes exception notification to `ap_team@apexgroup.in`.
