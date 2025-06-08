# Idea 3: Automated Data Validation & Reporting Boilerplate

## Table of Contents
- [Concept Overview](#concept-overview)
- [Target Domain(s)](#target-domains)
- [Detailed Description](#detailed-description)
  - [Core Functionality](#core-functionality)
  - [Example Interactions](#example-interactions)
  - [Customization](#customization)
  - [Report Structure](#report-structure)
- [How Codex CLI/Agent Features Are Leveraged](#how-codex-cliagent-features-are-leveraged)
- [Potential Benefits](#potential-benefits)
- [Considerations & Challenges](#considerations--challenges)

## Concept Overview
Codex helps users generate scripts (e.g., Python with Pandas) to perform common data validation tasks on tabular datasets (like CSVs or Parquet files). It can also create a boilerplate markdown report template that can be populated by the output of the validation script.

## Target Domain(s)
-   Data Science
-   Ag-Tech
-   Health Domains
-   Any field working with structured or semi-structured data that requires quality checks.

## Detailed Description

### Core Functionality
1.  **Understand Validation Needs**: Codex interprets a user prompt specifying the dataset and the types of validation checks required.
2.  **Script Generation**:
    -   Generates a script (primarily Python using libraries like Pandas and NumPy) to perform the requested validations.
    -   Common checks could include:
        -   **Missing Value Analysis**: Count and percentage of missing values per column.
        -   **Data Type Checks**: Verify if column data types match expectations (e.g., numeric, string, boolean, datetime).
        -   **Summary Statistics**: For numerical columns (mean, median, min, max, std dev, quartiles). For categorical columns (value counts, unique values).
        -   **Range Checks**: Ensure values in specified columns fall within predefined valid ranges.
        -   **Outlier Detection**: Basic methods for flagging potential outliers (e.g., based on IQR or Z-score for specified columns).
        -   **Uniqueness Checks**: Verify if values in a column are unique (e.g., for ID columns).
        -   **Pattern Matching**: Check if string values conform to a regex pattern (e.g., for phone numbers, emails, specific codes).
3.  **Report Template Generation (Optional)**:
    -   Creates a markdown file (e.g., `data_validation_report.md`) with sections corresponding to the validation checks performed.
    -   The script can be designed to output results in a format easily integrable into this markdown (e.g., print markdown tables).
4.  **Execution and File Modification**:
    -   Codex can save the generated script and report template.
    -   With approval, it can execute the validation script.
    -   The script itself would typically print its findings to stdout or populate the markdown report.

### Example Interactions

**Example 1 (Basic CSV Validation):**
*User Prompt*: `"codex, validate 'experiment_data.csv'. Check for missing values, get summary stats for all columns, and check if 'sample_id' is unique."`

*Codex Actions*:
1.  Identifies: File `experiment_data.csv`, checks: missing values, summary stats, 'sample_id' uniqueness.
2.  Generates `validate_data.py` (simplified example):
    ```python
    import pandas as pd

    df = pd.read_csv('experiment_data.csv')

    print("## Missing Value Analysis")
    print(df.isnull().sum())
    print("\n## Summary Statistics")
    print(df.describe(include='all'))
    print("\n## Sample ID Uniqueness")
    if df['sample_id'].is_unique:
        print("sample_id is unique.")
    else:
        print(f"sample_id is NOT unique. Duplicates found: {df['sample_id'].duplicated().sum()}")
    ```
3.  Generates `validation_report.md` (basic template):
    ```markdown
    # Data Validation Report for experiment_data.csv

    ## Missing Value Analysis
    <!-- Output from script -->

    ## Summary Statistics
    <!-- Output from script -->

    ## Sample ID Uniqueness
    <!-- Output from script -->
    ```
4.  Suggests running `python validate_data.py > temp_report_output.txt` and then shows how to integrate results, or modifies the script to directly write markdown.

**Example 2 (More Specific Checks):**
*User Prompt*: `"codex, for 'patient_logs.parquet', report missing values in 'heart_rate', ensure 'age' is between 0 and 120, and list unique values in 'diagnosis_code'."`

*Codex Actions*:
1.  Identifies: File `patient_logs.parquet`, checks: missing 'heart_rate', 'age' range, unique 'diagnosis_code'.
2.  Generates a Python script using Pandas to:
    -   Load the Parquet file.
    -   Calculate missing values for `heart_rate`.
    -   Filter `age` column for values outside 0-120 range and report them.
    -   List unique values in `diagnosis_code`.
3.  Could offer to append these specific checks to an existing validation script or report.

### Customization
-   **Via Prompts**: Users can specify columns for specific checks, thresholds for outlier detection, valid ranges, regex patterns, etc.
-   **Via `AGENTS.md`**:
    -   Define standard sets of validation checks for particular dataset types within a project.
        ```markdown
        ## DataValidation: ClinicalTrialDataset
        - check_missing_values: all_columns
        - check_summary_stats: all_columns
        - column_rules:
          - name: patient_id
            unique: true
          - name: age
            range: [18, 80]
          - name: visit_date
            type: datetime
        ```
    -   Specify preferred reporting formats or libraries.

### Report Structure
A generated `validation_report.md` could include:
-   **Timestamp** of validation.
-   **Dataset Information**: File name, number of rows/columns.
-   **Sections for each validation check**:
    -   Clear headings.
    -   Summary of findings (e.g., "Column 'age' has 5 values outside the range [0, 120].").
    -   Tables or lists detailing problematic data points (e.g., a sample of rows with out-of-range values).

## How Codex CLI/Agent Features Are Leveraged
-   **Natural Language Processing**: To interpret the desired validation checks, target file, and specific parameters.
-   **Code Generation**: To create the Python (or other language) script that performs the validation.
-   **File System Interaction**: To read the data file, save the generated script, and save the markdown report template.
-   **Command Execution**: To run the generated validation script (with user approval).
-   **Context Awareness (`AGENTS.md`)**: To apply predefined validation suites or rules for known dataset types.

## Potential Benefits
-   **Time Savings**: Automates the creation of repetitive data validation scripts.
-   **Standardization**: Encourages consistent data quality checks across a project or team, especially if using `AGENTS.md` templates.
-   **Early Issue Detection**: Helps identify data quality problems early in the analysis pipeline.
-   **Improved Reproducibility**: Generated scripts document the validation steps performed.
-   **Accessibility**: Lowers the barrier for users who might not be proficient in scripting these checks from scratch.

## Considerations & Challenges
-   **Complexity of Validation Logic**: Some validation rules can be very complex or domain-specific. Codex might initially support common, generic checks.
-   **Large Datasets**: Running validation scripts on very large datasets directly in a local environment might be slow or memory-intensive. Codex might need to generate scripts that are optimized or can be run in more powerful environments (though execution itself is up to the user).
-   **Variety of Data Formats**: Supporting diverse data sources (CSV, Excel, Parquet, JSON, databases) adds complexity. Pandas helps, but connection details for databases would be needed.
-   **Sensitivity of Data**: For domains like Health, the validation script must not expose sensitive data in its output or reports. Reports should contain aggregate statistics or anonymized examples of failing data. User oversight is paramount.
-   **Integration with Reporting**: Simply generating a script and a markdown template is a good start. A more advanced version might involve Codex helping to populate the report by parsing the script's output, or generating scripts that directly output markdown.
-   **User Interface for Rules**: For very complex validation suites, prompts might become unwieldy. A more structured way to define rules (perhaps via `AGENTS.md` or a dedicated configuration file format) might be needed for advanced use.

*Timestamp: {TIMESTAMP}*
*Version: {VERSION}*
