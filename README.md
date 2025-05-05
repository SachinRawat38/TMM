# CECL Calculation using Transition Matrix Multiplication (TMM)

This Jupyter Notebook (`TMM1.ipynb`) implements a process to calculate the Current Expected Credit Losses (CECL) for a loan portfolio using a Transition Matrix Multiplication (TMM) approach based on historical loan performance data.

## Overview

The notebook performs the following key steps:

1.  **Data Loading:** Reads loan-level data from a pipe-delimited CSV file without a header.
2.  **Data Preprocessing:**
    *   Assigns predefined headers to the DataFrame.
    *   Converts specific date columns from `MMYYYY` integer format to datetime objects.
    *   Handles duplicate loan entries.
    *   Performs missing data imputation (specifically for `Loan_Age`) and drops records missing critical information (`Loan_Identifier`, `Current_Loan_Delinquency_Status`, `Current_Actual_UPB`, `Monthly_Reporting_Period`).
    *   Maps numerical delinquency status codes to descriptive strings (`current`, `ingrace`, `30dpd`, `60dpd`, `90dpd`, `chargedoff`).
3.  **Feature Engineering:** Creates a `Next_Loan_Delinquency_Status` column to track status changes month-over-month for each loan.
4.  **Transition Matrix Calculation:** Computes a probability transition matrix based on observed changes in loan delinquency status.
5.  **Weighted Average Age (WAVG):** Calculates the weighted average age of the loans in the portfolio, weighted by the current unpaid principal balance (UPB).
6.  **Cumulative Gross Loss (CGL) Curve:** Projects the probability of loans being in each delinquency state over a future period (`WAVG + 24` months) using the transition matrix.
7.  **Allowance for Loan and Lease Losses (ALLL):** Calculates a proxy for ALLL based on the projected increase in the 'chargedoff' state probability over a specific future 12-month window derived from the CGL curve and WAVG.
8.  **CECL Calculation:** Computes the final CECL value as `1.5 * ALLL`. *Note: The 1.5 multiplier might be a specific assumption or placeholder.*

## Dependencies

The script requires the following Python libraries:

*   `numpy`
*   `pandas`
*   `matplotlib` (imported but not used in the provided core logic)
*   `seaborn` (imported but not used in the provided core logic)

You can install them using pip:
```bash
pip install numpy pandas matplotlib seaborn

Input Data Format (data.csv)

The notebook expects an input CSV file (named data.csv by default) with the following characteristics:

Delimiter: Pipe (|) separated values.

Header: The file must not contain a header row. Headers are hardcoded in the add_header_to_dataframe function.

Columns: The file must contain exactly 108 columns in the specific order expected by the hardcoded header list in the add_header_to_dataframe function. Key columns used in the calculation include (but are not limited to):

Monthly_Reporting_Period

Loan_Identifier

Original_Interest_Rate

Current_Interest_Rate

Original_UPB

Current_Actual_UPB

Original_Loan_Term

Origination_Date

First_Payment_Date

Loan_Age

Remaining_Months_To_Maturity

Maturity_Date

Current_Loan_Delinquency_Status (Numerical codes expected: '00', '01', '02', '03', '04', etc.)

And others as listed in the function.

Date Format: Columns specified in the convert_mmYYYY_to_datetime function (e.g., Monthly_Reporting_Period, Origination_Date) must be integers in MMYYYY format.

Workflow Functions

read_csv_to_dataframe: Loads the pipe-delimited CSV.

add_header_to_dataframe: Adds the predefined column names.

convert_mmYYYY_to_datetime: Converts specific integer date columns to datetime.

handle_duplicates: Removes duplicate rows.

handling_missing_data: Imputes Loan_Age and drops rows with critical missing values.

outlier_detection: Placeholder (currently does nothing, assumes no outlier removal for critical metrics).

add_next_delinquency_status: Determines the delinquency status in the subsequent month for each loan record.

update_status_column: Maps numerical delinquency codes to string representations ('current', 'ingrace', '30dpd', etc.) and handles missing next statuses by mapping them to 'chargedoff'.

analyze_loan_transitions: Calculates the state transition probability matrix.

weighted_average_age: Computes the UPB-weighted average loan age.

compute_cgl_curve: Projects future state probabilities using the transition matrix.

compute_ALLL: Calculates the ALLL based on the CGL curve.

compute_cecl: Orchestrates the entire workflow from data loading to final CECL calculation.

How to Use

Prepare Data: Ensure your loan data is in a file named data.csv (or update the path in the final cell) and strictly adheres to the specified format (pipe-delimited, no header, correct columns and order, MMYYYY dates).

Install Dependencies: Make sure all required libraries are installed.

Run Notebook: Execute the cells in the TMM1.ipynb notebook sequentially. The final cell will call the compute_cecl function and print the calculated CECL value. Intermediate results like the Transition Matrix (TMM), CGL curve (cgl), and the processed DataFrame (df) are also available.

Output

The primary output is the calculated CECL value, printed to the console by the last cell. The compute_cecl function also returns:

CECL: The final calculated CECL score.

TM: The calculated Transition Matrix (pandas DataFrame).

Cgl_Curve: The computed Cumulative Gross Loss curve (list of numpy arrays).

df: The processed pandas DataFrame after cleaning and feature engineering.

Important Notes

The script relies heavily on hardcoded column names and a specific input data structure. Deviations will likely cause errors.

The definition of delinquency states ('current', 'ingrace', '30dpd', '60dpd', '90dpd', 'chargedoff') is based on the mapping in update_status_column. Any value not explicitly mapped or missing in the 'Next_Loan_Delinquency_Status' is treated as 'chargedoff'.

The final CECL calculation uses a 1.5 * ALLL formula. The justification for the 1.5 multiplier is not provided in the code and may need business context.

The outlier detection step is intentionally skipped for critical financial metrics.
