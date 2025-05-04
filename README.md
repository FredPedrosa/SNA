# SNA: Semantic Network Analysis using R and Python for Content Validity Assessment

An R Markdown script implementing a workflow for assessing the content validity and structural coherence of assessment items using semantic embeddings and network psychometric techniques. Inspired by the AI-GENIE methodology (Russell-Lasalandra, Christensen, & Golino, 2024), this script processes textual items, identifies redundancy and instability, estimates the underlying dimensional structure, and analyzes network centrality.

## Features

*   **Item Loading & Preprocessing:** Loads raw textual items from an `.RData` file.
*   **Manual Filtering:** Removes pre-identified irrelevant, problematic, or duplicate items based on provided indices.
*   **Deduplication:** Ensures only unique item phrases are processed further.
*   **Semantic Embedding Generation:** Uses the `sentence-transformers` Python library (via `reticulate`) to convert unique item phrases into high-dimensional numerical embeddings.
*   **Data Formatting for Network Analysis:** Transposes the embedding matrix so that embedding dimensions become rows (observations) and items become columns (variables), as described for network analysis of items in the AI-GENIE context.
*   **Iterative Item Reduction:** Implements a multi-step process (structured manually across iterations in the script) to refine the item pool:
    *   **Unique Variable Analysis (UVA):** Identifies and removes statistically redundant items based on network topology (using `EGAnet::UVA`).
    *   **Bootstrap Exploratory Graph Analysis (bootEGA):** Assesses the stability of each item's placement within the network structure across bootstrap samples (using `EGAnet::bootEGA`).
    *   **Stability Filtering:** Removes items that fall below a defined stability threshold (e.g., 0.75).
*   **Final Structure Estimation:** Performs a final Exploratory Graph Analysis (EGA) using the stable item set to determine the number and composition of dimensions (clusters).
*   **Item Mapping:** Maps the final stable item identifiers back to their original textual phrases.
*   **Centrality Analysis:** Estimates and plots network centrality indices (Strength, Closeness, Betweenness, Expected Influence) for the final stable network using `bootnet` and `qgraph`.
*   **Centrality Stability Assessment:** Evaluates the stability of the estimated centrality indices using bootstrapping (`bootnet`).
*   **Reporting:** Generates a PDF or HTML report containing the analysis steps, results, plots (EGA, bootEGA stability, centrality, centrality stability), and the final list of stable items grouped by estimated cluster.

## Prerequisites

To run this analysis, you need R, RStudio (recommended), and a working Python environment with specific libraries.

### 1. Install Miniconda (Recommended Python Distribution)

If you don't have Python or Conda installed, we recommend installing Miniconda:
*   Download Miniconda: [https://docs.anaconda.com/free/miniconda/](https://docs.anaconda.com/free/miniconda/)
*   Follow the installation instructions for your operating system.
*   After installation, open your terminal (Anaconda Prompt on Windows, Terminal on macOS/Linux) and verify the installation by typing `conda --version`.

### 2. Create and Activate a Conda Environment

It's best practice to create a dedicated environment to avoid package conflicts.

```bash
# Replace 'sna_env' with your preferred environment name
# Choose a Python version compatible with the required packages (e.g., 3.9, 3.10)
conda create --name sna_env python=3.9 

# Activate the environment
conda activate sna_env
```
You must activate this environment (conda activate sna_env) every time you want to run the Python-dependent parts or knit the Rmd file in a new session.

### 3. Install Required Python Packages

With the sna_env activated, install the necessary libraries using pip:
```bash
pip install sentence-transformers torch torchvision torchaudio 
# Or use 'pip install sentence-transformers tensorflow' if you prefer TensorFlow
```

### 4. Configure R/reticulate

The R script uses the reticulate package to interact with Python. You need to tell R where to find the Conda environment you created. Open the semantic_net_analysis.Rmd file and find the setup chunk. Ensure the following line points to your environment name:
```bash
# Make sure 'sna_env' matches the name you used in 'conda create'
reticulate::use_condaenv("sna_env", required = TRUE) 

# If you used venv instead of conda, use use_virtualenv:
# reticulate::use_virtualenv("/path/to/your/virtual/env", required = TRUE)
```

## How it Works

The script follows these primary steps:

1.  **Setup:** Sets the working directory and loads necessary R libraries (`reticulate`, `EGAnet`, `psych`, `qgraph`, `bootnet`). Configures options to disable ANSI color output to prevent LaTeX compilation errors.
2.  **Load & Filter:** Loads the initial item vector (`itens`) from `itens.RData`. Removes items specified in `initial_item_removal_indices`. Removes duplicate phrases.
3.  **Embed:** Initializes the Python environment and the `sentence-transformers` model. Encodes the unique phrases into an embedding matrix.
4.  **Transpose:** Transposes the embedding matrix to the format required by `EGAnet` for item network analysis (Dimensions x Items).
5.  **Iteration 1 (UVA + bootEGA):**
    *   Runs UVA on the transposed data to remove redundant items.
    *   Runs bootEGA on the UVA-reduced data to assess item stability.
    *   Identifies stable items based on the threshold.
6.  **Iteration 2 (Conditional UVA + bootEGA):**
    *   *If* unstable items were found in Iteration 1, repeats the UVA and bootEGA process on the subset of items deemed stable in Iteration 1.
    *   Identifies stable items from this iteration.
7.  **Iteration 3 (Conditional UVA + bootEGA):**
    *   *If* unstable items were found in Iteration 2, repeats the UVA and bootEGA process on the subset of items deemed stable in Iteration 2.
    *   Identifies the final set of stable items. (Handles cases where UVA might remove all remaining items).
8.  **Final EGA:** Runs a final EGA (`EGAnet::EGA`) on the dataset containing only the items identified as stable after the last completed iteration. Extracts the final cluster assignments (`wc`).
9.  **Map Back Phrases:** Extracts the names (e.g., "i3", "i7") of the final stable items. Uses a helper function (`get_index_from_item_name`) to convert these names back to the numeric indices corresponding to their position in the *original unique phrases vector*. Retrieves the original text for the stable items.
10. **Group Phrases:** Organizes the final stable phrases into groups based on their assigned cluster from the final EGA.
11. **Centrality Analysis:** Estimates the network using `bootnet::estimateNetwork` (with EBICglasso) on the final stable data. Calculates and plots standardized centrality measures using `qgraph::centralityPlot`.
12. **Centrality Stability:** Performs bootstrapping on the estimated network using `bootnet::bootnet` to assess the stability of centrality indices. Plots the stability results and calculates the Correlation Stability (CS) coefficient.
13. **Report:** Prints the grouped phrases and key centrality findings (identifying items corresponding to specific indices based on the `unique_phrases` vector) to the console/report output.

## Usage

### Prerequisites

*   **R:** A recent version of R installed.
*   **RStudio:** Recommended IDE for R.
*   **Python:** A working Python installation (compatible with `sentence-transformers`).
*   **Python Virtual Environment (Recommended):** A dedicated Python virtual environment to install Python packages without conflicts.

### Installation

1.  **R Packages:** Install the required R packages from the R console:
    ```R
    install.packages(c("reticulate", "EGAnet", "psych", "psychTools", "qgraph", "bootnet", "rmarkdown"))
    ```
2.  **Python Packages:**
    *   Activate your chosen Python virtual environment.
    *   Install the necessary Python packages using pip:
        ```bash
        pip install sentence-transformers torch # or tensorflow, depending on your backend
        ```

### Running the Analysis

1.  **Set Up Python Environment:** Ensure the `use_virtualenv("path/to/your/.venv", required = TRUE)` line in the R script points to the correct path of your Python virtual environment.
2.  **Prepare Input Data:** Place the `itens.RData` file (containing the character vector named `itens`) in the R working directory (set by `setwd()` in the script).
3.  **Run Script:** Open the `semantic_net_analysis.Rmd` file in RStudio.
4.  **Knit:** Click the "Knit" button and choose the desired output format (PDF or HTML). Ensure the necessary LaTeX distribution (like TinyTeX, `tinytex::install_tinytex()`) is installed if knitting to PDF. The options in the setup chunk should prevent common LaTeX errors related to console output.

## Input Requirements

*   **`itens.RData`**: An R data file located in the working directory. This file must contain a single object: a character vector named `itens`, where each element is a textual assessment item.

## Output Interpretation

### Console Output (During Knitting/Execution)

*   Progress messages indicating which stage is running (Embedding, UVA, bootEGA, Centrality).
*   Number of unique phrases initially.
*   Number of items kept after each UVA step.
*   Number of stable/unstable items after each bootEGA step.
*   Dimensions of the final stable dataset.
*   Number of dimensions (clusters) found by the final EGA.
*   Printed centrality results for specific items (indexing based on the original `unique_phrases` vector).

### Generated Report (PDF/HTML)

*   The knitted document containing all the code, narrative text, and results.
*   **Plots:**
    *   Final EGA Network Plot (visualizing item clusters).
    *   bootEGA Item Stability Plot (showing stability of each item across bootstraps for the relevant iterations).
    *   Centrality Plot (showing standardized scores for Strength, Closeness, Betweenness, Expected Influence).
    *   Centrality Stability Plot (showing confidence intervals for centrality estimates from bootstrapping).
*   **Textual Results:**
    *   The final list of stable item phrases, grouped by their estimated cluster.
    *   Correlation Stability (CS) coefficient values for centrality measures.

## Technology Stack

*   R
*   R Packages: `reticulate`, `EGAnet`, `psych`, `psychTools`, `qgraph`, `bootnet`, `rmarkdown`, `knitr`
*   Python
*   Python Packages: `sentence-transformers`, `torch` (or `tensorflow`)
*   LaTeX (for PDF output, e.g., via TinyTeX)
*   Pandoc (used by R Markdown)

## References

Russell-Lasalandra, L. L., Christensen, A. P., & Golino, H. (2024, September 12). Generative Psychometrics via AI-GENIE: Automatic Item Generation and Validation via Network-Integrated Evaluation. https://doi.org/10.31234/osf.io/fgbj4

Verga Pinto, A. B. (2024). Construcción de una herramienta multisistémica de evaluación en musicoterapia para el rango etario de 0 a 12 años: Manual del PREVIMMUS y su enfoque teórico [Trabajo de Fin de Máster, Universidad de Sevilla].

## How to Cite

Pedrosa, F. G. (2025). *SNA Semantic Network Analysis using R and Python for Content Validity Assessment*. [Script]. https://github.com/FredPedrosa/SNA/

## Author

*   **Prof. Dr. Frederico Pedrosa**
*   fredericopedrosa@ufmg.br

## License

This project is licensed under a modified version of the GNU General Public License v3.0.  
Commercial use is not permitted without explicit written permission from the author
