# Idea 5: Notebook-to-Script (and vice-versa) Converter

## Table of Contents
- [Concept Overview](#concept-overview)
- [Target Domain(s)](#target-domains)
- [Detailed Description](#detailed-description)
  - [Core Functionality: Notebook to Script](#core-functionality-notebook-to-script)
  - [Core Functionality: Script to Notebook](#core-functionality-script-to-notebook)
  - [Example Interactions](#example-interactions)
  - [Handling Specific Elements](#handling-specific-elements)
  - [Customization Options](#customization-options)
- [How Codex CLI/Agent Features Are Leveraged](#how-codex-cliagent-features-are-leveraged)
- [Potential Benefits](#potential-benefits)
- [Considerations & Challenges](#considerations--challenges)

## Concept Overview
Codex provides utilities to convert Jupyter notebooks (`.ipynb` files) into executable Python scripts (`.py` files) for productionizing code, and conversely, to structure Python scripts into Jupyter notebooks for exploration, documentation, and interactive experimentation.

## Target Domain(s)
-   Data Science
-   Machine Learning
-   Scientific Computing
-   Any field where Jupyter notebooks are used for development and Python scripts are used for execution or vice-versa.

## Detailed Description

### Core Functionality: Notebook to Script
1.  **Parse Notebook**: Codex reads the `.ipynb` file (which is JSON formatted).
2.  **Extract Code Cells**: Identifies and extracts code from all code cells.
3.  **Concatenate Code**: Combines the extracted code in the order it appears in the notebook.
4.  **Handle Markdown/Text Cells**:
    -   Option to convert markdown cells to comments (e.g., block comments or line-by-line comments) in the Python script.
    -   Option to discard markdown cells.
5.  **Manage Magics/Shell Commands**:
    -   Option to comment out Jupyter magic commands (e.g., `%matplotlib inline`) or shell commands (`!pip install ...`).
    -   Potentially translate some common magics to equivalent Python code if possible (e.g., `%%time` could be wrapped with `time` module calls).
6.  **Parameterization (Advanced)**:
    -   Identify variables that are hardcoded in the notebook but might be better as script arguments (e.g., file paths, model parameters). Codex could suggest these for conversion to `argparse` arguments.
7.  **Output Script**: Saves the resulting Python code to a `.py` file.

### Core Functionality: Script to Notebook
1.  **Parse Python Script**: Codex reads the `.py` file.
2.  **Identify Code Blocks**:
    -   Split script into logical blocks. This could be based on function definitions, class definitions, or even blocks of code separated by multiple blank lines or specific comment patterns (e.g., `# %% [markdown]`).
3.  **Create Notebook Cells**:
    -   Each identified code block becomes a code cell in the new notebook.
    -   Comments immediately preceding a code block, or specially formatted comments (e.g., `# %% [markdown]
 # This is a title`), could be converted into markdown cells.
    -   Function/class docstrings could optionally be converted into markdown cells preceding the code cell.
4.  **Output Notebook**: Saves the resulting structure as a new `.ipynb` file.

### Example Interactions

**Example 1 (Notebook to Script):**
*User Prompt*: `"codex convert 'eda_report.ipynb' to 'run_eda.py'. Convert markdown to comments and comment out any pip installs."`

*Codex Actions*:
1.  Reads `eda_report.ipynb`.
2.  Extracts code cells.
3.  Converts markdown cells to Python block comments.
4.  Finds lines like `!pip install some_package` and converts them to `# !pip install some_package`.
5.  Saves the resulting code as `run_eda.py`.

**Example 2 (Script to Notebook):**
*User Prompt*: `"codex, transform 'training_pipeline.py' into 'training_pipeline.ipynb', creating markdown cells from triple-quoted string comments at the start of functions."`

*Codex Actions*:
1.  Reads `training_pipeline.py`.
2.  Identifies functions and other logical code blocks.
3.  For each function, if it starts with a triple-quoted string (docstring), creates a markdown cell with that content.
4.  Places the function code into a subsequent code cell.
5.  Other script parts become code cells.
6.  Saves as `training_pipeline.ipynb`.

### Handling Specific Elements
-   **Jupyter Magics**:
    -   `%matplotlib inline`: Could be commented out or replaced with a note about GUI backends if running as a script.
    -   `%%time`, `%%timeit`: Could be commented out or (more advanced) wrapped in Python's `time` module for equivalent timing.
    -   Other magics: Generally commented out.
-   **Shell Commands (`!command`)**: Usually commented out, with an option to try and convert them to `subprocess.run()` calls.
-   **`input()` calls**: In notebook-to-script, these would remain as `input()` calls. In script-to-notebook, they would work as expected in code cells.
-   **`display()` or `print()`**: These translate directly. Rich display objects from IPython might render as their text representation in a script.

### Customization Options
-   **Markdown Handling (NB to Py)**:
    -   `--md-to-comments`: Convert markdown to comments.
    -   `--md-strip`: Remove markdown cells.
    -   `--md-as-docstrings`: (If a markdown cell precedes a code cell defining a function/class) attempt to make it a docstring.
-   **Magic/Shell Command Handling (NB to Py)**:
    -   `--magics-comment`: Comment out magics (default).
    -   `--magics-strip`: Remove magics.
    -   `--shell-comment`: Comment out shell commands (default).
    -   `--shell-strip`: Remove shell commands.
    -   `--shell-to-subprocess`: Attempt to convert shell commands to `subprocess.run()`.
-   **Code Block Splitting (Py to NB)**:
    -   `--split-on-functions`: Create new cells for each function.
    -   `--split-on-classes`: Create new cells for each class.
    -   `--docstrings-to-md`: Convert function/class docstrings to markdown cells.
-   **Output File Naming**: Allow specifying the output file name.

## How Codex CLI/Agent Features Are Leveraged
-   **Natural Language Processing**: To understand the conversion direction, source/target files, and desired options (e.g., "convert markdown to comments").
-   **File System Interaction**: Reading the source `.ipynb` or `.py` file and writing the converted output file.
-   **Code Parsing & Transformation**: The agent would need to implement logic to parse the JSON structure of `.ipynb` files and the AST (Abstract Syntax Tree) of `.py` files to perform the transformations. Libraries like `nbformat` for Python could be used internally by a Python script generated by Codex, or Codex might implement its own simpler parsing if only dealing with structure.
-   **Command Generation (for complex options)**: If the conversion involves external tools like `jupytext` (though the idea is for Codex to do it natively), it could generate commands for those.
-   **Iterative Refinement**: User could ask for changes after an initial conversion, e.g., "Actually, strip the markdown instead of commenting it out."

## Potential Benefits
-   **Streamlined Workflows**: Eases the transition between exploratory work in notebooks and production-ready scripts.
-   **Improved Code Reusability**: Scripts are generally easier to import and reuse in larger projects than notebooks.
-   **Version Control Friendliness**: `.py` scripts are more diff-friendly for version control systems than `.ipynb` JSON.
-   **Documentation & Exploration**: Converting scripts to notebooks can help in documenting code or making it more accessible for interactive exploration by others.
-   **Reduced Manual Effort**: Automates a tedious and error-prone manual conversion process.

## Considerations & Challenges
-   **Fidelity of Conversion**:
    -   **NB to Py**: Rich outputs (HTML, images) in notebooks are lost. The linear execution model of a script might differ from the potentially out-of-order execution in a notebook during development. State dependency between cells needs careful handling or documentation.
    -   **Py to NB**: Heuristics for splitting scripts into meaningful cells can be imperfect. Recreating an "exploratory flow" from a linear script is non-trivial.
-   **Complexity of Notebooks/Scripts**: Very complex notebooks with intricate dependencies or scripts with advanced metaprogramming might be hard to convert perfectly.
-   **Handling of Magics and Shell Commands**: Providing robust translations for all magics or shell commands is difficult. Commenting them out is the safest default.
-   **Parameterization Logic (NB to Py)**: Automatically and correctly identifying which hardcoded values should become script parameters requires sophisticated analysis.
-   **User Expectations**: Users need to understand that the conversion is a utility to aid workflow, not a perfect one-to-one mapping that preserves all interactive nuances of a notebook.
-   **Dependency Management**: The converted script will still have the same Python package dependencies as the notebook.
-   **Execution Environment Differences**: Code that relies on specific IPython/Jupyter environment features might not work identically when run as a plain Python script.

*Timestamp: {TIMESTAMP}*
*Version: {VERSION}*
