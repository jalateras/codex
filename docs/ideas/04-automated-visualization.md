# Idea 4: Automated Data Visualization Scripting

## Table of Contents
- [Concept Overview](#concept-overview)
- [Target Domain(s)](#target-domains)
- [Detailed Description](#detailed-description)
  - [Core Functionality](#core-functionality)
  - [Example Interactions](#example-interactions)
  - [Supported Plot Types (Examples)](#supported-plot-types-examples)
  - [Customization via Prompts](#customization-via-prompts)
  - [Handling of Output](#handling-of-output)
- [How Codex CLI/Agent Features Are Leveraged](#how-codex-cliagent-features-are-leveraged)
- [Potential Benefits](#potential-benefits)
- [Considerations & Challenges](#considerations--challenges)

## Concept Overview
Codex assists users in quickly generating scripts (primarily Python using libraries like Matplotlib, Seaborn, and Plotly) for common data visualizations based on natural language prompts. This allows for rapid exploration and presentation of data without manually writing plotting code.

## Target Domain(s)
-   Data Science
-   Ag-Tech (e.g., visualizing sensor data, crop yields)
-   Health Domains (e.g., plotting patient statistics, epidemiological trends)
-   Any field requiring quick visualization of datasets.

## Detailed Description

### Core Functionality
1.  **Interpret Visualization Request**: Codex parses a user's prompt to understand:
    -   The data source (e.g., CSV file, Parquet file, potentially a dataframe already in an interactive session if supported).
    -   The columns to be plotted.
    -   The desired type of plot (histogram, scatter plot, line plot, bar chart, box plot, etc.).
    -   Any specific customizations (titles, labels, colors, grouping variables).
2.  **Script Generation**:
    -   Generates a Python script using appropriate libraries:
        -   **Matplotlib**: For fundamental plotting.
        -   **Seaborn**: For more statistically sophisticated and aesthetically pleasing plots.
        -   **Plotly**: For interactive plots (if the environment supports it or for HTML export).
    -   The script will include code for:
        -   Loading the data (e.g., `pd.read_csv()`, `pd.read_parquet()`).
        -   Performing any necessary data selection or simple transformations (e.g., filtering, grouping for the plot).
        -   Generating the plot with specified customizations.
        -   Saving the plot to a file (e.g., PNG, SVG, PDF) or displaying it.
3.  **Execution and Output**:
    -   Codex saves the generated script.
    -   With user approval, executes the script.
    -   Informs the user where the plot was saved or attempts to display it if in a capable terminal/environment.

### Example Interactions

**Example 1 (Simple Histogram):**
*User Prompt*: `"codex, plot a histogram of the 'age' column from 'survey_data.csv'"`

*Codex Actions*:
1.  Identifies: Plot type (histogram), column ('age'), data file ('survey_data.csv').
2.  Generates `plot_age_histogram.py`:
    ```python
    import pandas as pd
    import matplotlib.pyplot as plt
    import seaborn as sns

    df = pd.read_csv('survey_data.csv')
    plt.figure(figsize=(10, 6))
    sns.histplot(df['age'], kde=True)
    plt.title('Histogram of Age')
    plt.xlabel('Age')
    plt.ylabel('Frequency')
    plt.grid(True)
    plt.savefig('age_histogram.png')
    print("Histogram saved to age_histogram.png")
    # plt.show() # Could be added if interactive display is desired/possible
    ```
3.  Suggests running `python plot_age_histogram.py`.

**Example 2 (Scatter Plot with Grouping):**
*User Prompt*: `"codex, create a scatter plot of 'temperature' vs 'yield' from 'farm_data.xlsx', colored by 'fertilizer_type'. Save as svg."`

*Codex Actions*:
1.  Identifies: Plot type (scatter), x-column ('temperature'), y-column ('yield'), color-grouping ('fertilizer_type'), file ('farm_data.xlsx'), output format (SVG).
2.  Generates a Python script using Seaborn:
    ```python
    import pandas as pd
    import matplotlib.pyplot as plt
    import seaborn as sns

    df = pd.read_excel('farm_data.xlsx') # Assumes openpyxl is installed or guides user
    plt.figure(figsize=(10, 6))
    sns.scatterplot(data=df, x='temperature', y='yield', hue='fertilizer_type')
    plt.title('Temperature vs. Yield by Fertilizer Type')
    plt.xlabel('Temperature')
    plt.ylabel('Yield')
    plt.grid(True)
    plt.savefig('temp_yield_scatter.svg')
    print("Scatter plot saved to temp_yield_scatter.svg")
    ```

### Supported Plot Types (Examples)
-   **Basic**: Histogram, Bar Chart, Line Plot, Scatter Plot.
-   **Statistical**: Box Plot, Violin Plot, KDE Plot, Pair Plot (for multiple variables), Heatmap.
-   **Time-Series**: Line plots with proper datetime axis formatting.
-   **Interactive (with Plotly)**: If specified, generate plots that can be saved as HTML or potentially rendered in environments that support Plotly's interactive features.

### Customization via Prompts
Users should be able to specify:
-   Plot titles, axis labels, legends.
-   Colors, markers, line styles.
-   Figure size, resolution (DPI for saved images).
-   Logarithmic scales.
-   Number of bins for histograms.
-   Subplots for comparing multiple visualizations.
-   Basic data transformations before plotting (e.g., "plot log of 'income'").

### Handling of Output
-   **Default**: Save plots to a file (e.g., `plot_name.png`, `.svg`, `.pdf`) in the current working directory or a specified output directory.
-   **Interactive Display**: If Codex can detect a suitable environment (e.g., a terminal that supports image display like iTerm2 with imgcat, or if running within a Jupyter-like context), it could attempt to display the plot directly after generation. This is more advanced.
-   **HTML for Interactive Plots**: Plotly graphs would typically be saved as HTML files which can be opened in a browser.

## How Codex CLI/Agent Features Are Leveraged
-   **Natural Language Processing**: To understand the type of plot, data columns, file names, and desired customizations from the user's request.
-   **Code Generation**: To create Python scripts tailored to the specific visualization task using appropriate libraries.
-   **File System Interaction**: To read the input data file and save the generated plot image or HTML file.
-   **Command Execution**: To run the generated Python script (with user approval).
-   **Context Awareness (`AGENTS.md`)**:
    -   Could store preferred plotting libraries or styles (e.g., "default to seaborn style 'whitegrid'").
    -   Define common plot types for specific datasets frequently used by the user/team.
-   **Iterative Refinement**: If a plot isn't quite right, the user could provide follow-up prompts like `"codex, change the title to '...'"` or `"codex, use a log scale for the y-axis"`, and Codex would modify the existing script or generate a new one.

## Potential Benefits
-   **Rapid Exploration**: Quickly visualize data without writing boilerplate plotting code, speeding up exploratory data analysis (EDA).
-   **Accessibility**: Makes plotting accessible to users less familiar with the intricacies of plotting libraries.
-   **Reproducibility**: The generated scripts serve as a record of how the plot was created.
-   **Customization**: Allows for a good degree of customization through natural language.
-   **Focus on Interpretation**: Users can spend more time interpreting plots rather than generating them.

## Considerations & Challenges
-   **Complexity of Requests**: Highly complex or very specific, niche plot types might be difficult for Codex to interpret and generate correctly.
-   **Data Loading & Cleaning**: The feature would likely assume reasonably clean, tabular data. Complex data loading or significant preprocessing before plotting would be a separate task (though Codex could also assist with that).
-   **Library Dependencies**: The generated scripts will require specific Python libraries (Pandas, Matplotlib, Seaborn, Plotly, openpyxl for Excel) to be installed in the user's environment. Codex might need to check for these or suggest installation.
-   **Interactivity Limitations**: True interactive plotting (pan, zoom, hover) in a standard terminal is challenging. Relying on saving to HTML for Plotly or image files for static plots is more robust.
-   **Ambiguity in Prompts**: "Plot X and Y" could mean many things. Codex will need to make sensible defaults or ask clarifying questions (e.g., "Do you want a scatter plot or a line plot for X and Y?").
-   **Aesthetic Control**: While basic customizations are possible, achieving highly polished, publication-quality aesthetics through prompts alone might be difficult. The goal is more rapid exploration and good-enough visualizations.
-   **Large Datasets**: Plotting very large datasets can be resource-intensive. Codex might generate scripts that include data sampling or aggregation techniques if it detects large input files.

*Timestamp: {TIMESTAMP}*
*Version: {VERSION}*
