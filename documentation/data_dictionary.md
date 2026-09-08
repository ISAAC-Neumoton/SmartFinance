# Data Dictionary: SmartFinance Bank Credit Analytics Model

This data dictionary documents the analytical schema, transformation rules, business definitions, and data types used in the **SmartFinance Bank: Loan Application Performance & Credit Decision Analytics** model[cite: 1].

---

## 1. Fact Table: `FACT_LOAN_APPLICATION`

Granular transactional records representing individual loan applications, their monetary parameters, operational velocity, and foreign keys linked to dimensions[cite: 1].

| Field Name | Source Column | Data Type | Example Value | Business Definition | Transformation / Business Logic |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Loan_ID** | `Loan_ID` | Whole Number | `271877` | Unique identifier assigned to each application record[cite: 1]. | Formatted as Whole Number; validated for uniqueness[cite: 1]. |
| **Previous_Application_ID** | `an_ID_PREVIOUS` | Whole Number | `1038818` | Identifier associated with an applicant's previous loan record[cite: 1]. | Renamed for business readability; set as Whole Number[cite: 1]. |
| **Contract_Type_Key** | *Derived* | Whole Number | `1` | Foreign Key referencing `DIM_CONTRACT`[cite: 1]. | Surrogate integer key generated during dimensional extraction[cite: 1]. |
| **Status_Key** | *Derived* | Whole Number | `1` | Foreign Key referencing `DIM_STATUS`[cite: 1]. | Surrogate integer key generated during dimensional extraction[cite: 1]. |
| **Loan_Purpose_Key** | *Derived* | Whole Number | `12` | Foreign Key referencing `DIM_LOAN_PURPOSE`[cite: 1]. | Surrogate integer key linking to standardized loan purpose[cite: 1]. |
| **Payment_Type_Key** | *Derived* | Whole Number | `2` | Foreign Key referencing `DIM_PAYMENT_TYPE`[cite: 1]. | Surrogate integer key linking to disbursement/settlement channel[cite: 1]. |
| **Reject_Reason_Key** | *Derived* | Whole Number | `4` | Foreign Key referencing `DIM_REJECT_REASON`[cite: 1]. | Surrogate integer key linking to underwriting denial codes[cite: 1]. |
| **Weekday_Key** | *Derived* | Whole Number | `1` | Foreign Key referencing `DIM_WEEKDAY`[cite: 1]. | Surrogate integer key representing the calendar day of application[cite: 1]. |
| **Application_Hour** | `HOUR_APPR_PROCESS_START` | Whole Number | `14` | 24-hour timestamp indicating when the application was initiated[cite: 1]. | Standardized name; typed as Whole Number (0–23)[cite: 1]. |
| **Hour_Time_Band** | *Derived* | Text | `Afternoon` | Operational day-part bucket based on `Application_Hour`[cite: 1]. | Conditional Logic: `0–5` = Early Morning, `6–11` = Morning, `12–17` = Afternoon, `18–23` = Evening[cite: 1]. |
| **Application_Amount** | `AMT_APPLICATION` | Decimal Number | `607500.00` | Principal amount requested by the credit applicant[cite: 1]. | Renamed from raw code; cast to Currency/Decimal[cite: 1]. |
| **Application_Amount_Band** | *Derived* | Text | `Very High` | Grouping of the requested loan size[cite: 1]. | Conditional Logic: `0–100,000` = Low, `100,001–300,000` = Medium, `300,001–600,000` = High, `>600,000` = Very High[cite: 1]. |
| **Credit_Amount** | `AMT_CREDIT` | Decimal Number | `679671.00` | Final total credit facility approved/recorded by the bank[cite: 1]. | Renamed; cast to Decimal Number[cite: 1]. |
| **Credit_Amount_Band** | *Derived* | Text | `Very High` | Classification tier for granted/recorded credit amount[cite: 1]. | Conditional Logic: `0–100,000` = Low, `100,001–300,000` = Medium, `300,001–600,000` = High, `>600,000` = Very High[cite: 1]. |
| **Down_Payment** | `AMT_DOWN_PAYMENT` | Decimal Number | `0.00` | Initial equity/cash contribution made by borrower[cite: 1]. | Missing values preserved or flagged; not automatically converted to 0 unless verified[cite: 1]. |
| **Down_Payment_Status** | *Derived* | Text | `No Down Payment` | Categorical indicator of borrower down payment contribution[cite: 1]. | Conditional Logic: `Down_Payment > 0` = "Paid", else "No Down Payment"[cite: 1]. |
| **Goods_Price** | `AMT_GOODS_PRICE` | Decimal Number | `607500.00` | Total valuation price of goods financed by the loan[cite: 1]. | Renamed; formatted as Decimal Number[cite: 1]. |
| **Annuity** | `AMT_ANNUITY` | Decimal Number | `22550.00` | Scheduled regular installment repayment amount[cite: 1]. | Renamed; missing/blank values cleaned and standardized[cite: 1]. |
| **Credit_Gap** | *Derived* | Decimal Number | `72171.00` | Net difference between approved credit and requested amount[cite: 1]. | Formula: `[Credit_Amount] - [Application_Amount]`[cite: 1]. |
| **Decision_Days** | `DAYS_DECISION` | Whole Number | `164` | Elapsed duration (in days) required to reach a decision[cite: 1]. | Converted raw negative values to absolute positive integers via `Number.Abs`[cite: 1]. |
| **Decision_Speed_Category**| *Derived* | Text | `Moderate` | Underwriting velocity bracket[cite: 1]. | Conditional Logic: `0–30` = Very Fast, `31–90` = Fast, `91–180` = Moderate, `181–365` = Slow, `>365` = Very Slow[cite: 1]. |
| **Approval_Flag** | *Derived* | Whole Number | `1` | Binary flag identifying approved applications[cite: 1]. | Conditional Logic: `1` if status is "Approved", otherwise `0`[cite: 1]. |
| **Refusal_Flag** | *Derived* | Whole Number | `0` | Binary flag identifying declined applications[cite: 1]. | Conditional Logic: `1` if status is "Refused", otherwise `0`[cite: 1]. |
| **Cancellation_Flag** | *Derived* | Whole Number | `0` | Binary flag identifying canceled application requests[cite: 1]. | Conditional Logic: `1` if status is "Canceled", otherwise `0`[cite: 1]. |

---

## 2. Dimension Tables

### `DIM_CONTRACT`
Master lookup containing credit contract products offered by SmartFinance Bank[cite: 1].

| Field Name | Data Type | Example Value | Description |
| :--- | :--- | :--- | :--- |
| **Contract_Key** | Whole Number | `1` | Primary Key identifying the credit contract type[cite: 1]. |
| **Contract_Type** | Text | `Cash loans` | Descriptive product name (`Cash loans`, `Consumer loans`, `Revolving loans`)[cite: 1]. |

---

### `DIM_STATUS`
Master lookup table defining the operational disposition/outcome of credit files[cite: 1].

| Field Name | Data Type | Example Value | Description |
| :--- | :--- | :--- | :--- |
| **Status_Key** | Whole Number | `1` | Primary Key identifying application status[cite: 1]. |
| **Contract_Status** | Text | `Approved` | Final decision outcome (`Approved`, `Refused`, `Canceled`, `Unused`)[cite: 1]. |

---

### `DIM_LOAN_PURPOSE`
Lookup table standardizing borrower-stated financing intent[cite: 1].

| Field Name | Data Type | Example Value | Description |
| :--- | :--- | :--- | :--- |
| **Purpose_Key** | Whole Number | `12` | Primary Key for loan purpose[cite: 1]. |
| **Loan_Purpose** | Text | `Everyday expenses` | Cleaned descriptive purpose; placeholder codes like `XNA` mapped to `"Unspecified / XNA"`[cite: 1]. |

---

### `DIM_PAYMENT_TYPE`
Master table cataloging loan disbursement and collection rails[cite: 1].

| Field Name | Data Type | Example Value | Description |
| :--- | :--- | :--- | :--- |
| **Payment_Key** | Whole Number | `1` | Primary Key for payment rail[cite: 1]. |
| **Payment_Type** | Text | `Cash through the bank` | Name of settlement channel (e.g., cash, bank account transfer)[cite: 1]. |

---

### `DIM_REJECT_REASON`
Master classification table for credit rejection and underwriting decline codes[cite: 1].

| Field Name | Data Type | Example Value | Description |
| :--- | :--- | :--- | :--- |
| **Reject_Reason_Key** | Whole Number | `3` | Primary Key for rejection driver[cite: 1]. |
| **Reject_Reason** | Text | `HC` | Underwriting decline code (e.g., `HC`, `LIMIT`, `SCO`, `XAP`)[cite: 1]. |

---

### `DIM_WEEKDAY`
Calendar dimension capturing day-of-week attributes for application origination[cite: 1].

| Field Name | Data Type | Example Value | Description |
| :--- | :--- | :--- | :--- |
| **Weekday_Key** | Whole Number | `1` | Primary Key for day sequence (e.g., `1` to `7`)[cite: 1]. |
| **Application_Weekday** | Text | `MONDAY` | Full day name of application origination[cite: 1]. |