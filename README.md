# TSC Pipeline — Code Documentation

Temporal Semantic Centrality (TSC) is a graph-based algorithm for personalized,
temporally-aware scientific literature discovery. This repository contains the
full pipeline for running TSC simulations, analyzing results, and empirically assessing the
theoretical properties developed in the dissertation.

---

## Repository Overview

| Code Name | Role |
|---|---|
| `TSC Master Pipeline Driver - SELF-CONTAINED` | Main simulation pipeline |
| `TSC Comparison - Focused vs Broad` | Pairwise focused vs. broad query comparison |
| `TSC Comparison Table Builder` | Multi-pairing summary table builder |
| `TSC Mechanism Analysis Tool` | Per-pairing displacement mechanism analysis |
| `TSC Mechanism Comparison Tool ` | Cross-pairing mechanism consistency tool |
| `TSC Chapter 3 Assessment` | Chapter 3 theorem empirical assessment suite |
| `Ranking Method Comparison` | TSC vs. BM25 / Dense E5 / Hybrid RRF / Degree / Eigenvector / Betweenness / Closeness comparison |
| `TSC Network Video Generator` | Animated network video (spring layout) |
| `Newman (2006): Recursive Spectral Bisection + Coreness — Community Detection (single-cell)` | Stand alone executable to produce files |
| `K-Core Topographic Maps` | Stand alone executable to produce files | 
| `TSC Histogram` | Stand alone executable to produce files | 
| `Reproduce Summary Table` | Stand alone executable to produce files |
| `TSC vs Publication Year` | Stand alone executable to produce files | 
| `Histogram Heatmap` | Stand alone executable to produce files | 
| `Ranking Comparison Heatmaps` | Stand alone executable to produce files | 
| `Replot Everything` | Stand alone executable to produce files |

---

## Dependencies

```
Data Acquisition and Parsing : biopython requests beautifulsoup4 lxml selenium
Text Processing: unidecode langdetect pycountry
Data Manipulation: pandas numpy scipy openpyxl
Embeddings and Similarity: sentence-transformers scikit-learn rank-bm25
Network Analysis: networkx pyvis
Visualization: matplotlib seaborn plotly ipywidgets
Utilities: tqdm python-dotenv

```

Install with:
```
Run all code in Tasks 1 and 2
```

All code is notebook-based and designed to run as Jupyter notebook cells and assume 
the standard scientific Python stack is already installed in the kernel environment. 
No tool imports from another tool — there are no inter-file dependencies.

---

## TF-IDF Configuration

All tools use a consistent TF-IDF vectorisation:

| Parameter | Value | Notes |
|---|---|---|
| `lowercase` | `True` | |
| `stop_words` | `"english"` | |
| `ngram_range` | `(1, 2)` | Unigrams and bigrams |
| `use_idf` | `True` | |
| `smooth_idf` | `True` | Adds 1 to document frequencies |
| `sublinear_tf` | `True` | Replaces TF with 1 + log(TF) |
| `norm` | `"l2"` | sklearn default |

---

## Set Naming Convention

Throughout the codebase, dissertation set names map to code variable names as follows:

| Dissertation Name | Code Variable | Definition |
|---|---|---|
| Set D | `set_B` | Focused docs displaced below rank threshold X |
| Set B | `set_C` | Broad query docs above rank threshold X (displace Set D) |
| Set R | `set_R` | Focused docs retained above rank threshold X |
| Set R_Q5 | `set_R_Q5` | Set R docs in top quintile of r_q score |
| Set N | `set_N` | Broad query docs below iso-TSC (background noise) |

The rank threshold X equals the size of the focused query corpus.

---

## Figure Output Conventions

Every plot function produces multiple file variants. The suffix conventions are:

| Suffix  Meaning |
|---|---|
| *(none)* | Light background (`white`), with title |
| `_dark` | Dark background (`#1a1a1a`), with title |
| `_notitle` | Light background, no title |
| `_dark_notitle` | Dark background, no title |
| `_panel_{name}` | Individual panel extracted from a multi-panel figure |

Panel files are automatically produced for every variant of any multi-panel figure,
so e.g. `Mechanism1_vocab_20260101_notitle_panel_vocab.png` is a standalone
untitled extraction of the vocabulary panel from the notitle light variant.

---

## `TSC Master Pipeline Driver - SELF-CONTAINED` — Main Simulation Pipeline
The TSC Master Pipeline Driver is a single‑file, self‑contained script that executes 
the full Temporally‑aware Scientific Corpus (TSC) workflow for a single query. 
It can either:
- Pull fresh records from APIs, or
- Load an existing combined-results CSV,

depending on user‑selected skip flags.

The driver performs all stages of the TSC methodology: API retrieval, cleaning, 
title–abstract combination, TF–IDF construction, similarity matrix generation, 
temporal relevance scoring, coarse and fine TSC grid sweeps, convexity/monotonicity 
analysis, community detection, coreness computation, and k‑core topographic mapping.

**Entry point:** Run as a Jupyter notebook cell; prompts for all required inputs 
(query terms, record limits, skip flags, and file paths).

**What the Pipeline Does:**
For a single query, the pipeline:
- Retrieves or loads records
- Cleans and deduplicates the corpus
- Builds TF–IDF vectors
- Computes pairwise cosine similarity
- Applies τ‑thresholding
- Computes temporal relevance (TR)
- Computes query relevance (QR)
- Performs Coarse-Mesh Sweep
  - Performs a coarse 2‑D sweep over (𝜏,𝜆).
  - Identifies a zoom region in (𝜏,𝜆) space containing the coarse peak.
- Performs Fine-Mesh Sweep
  - Performs a fine 2‑D sweep over all τ and λ inside the coarse peak region.
  - Produces a full fine-mesh TSC surface.
- Performs Parameter Selection
  - τ is selected from the fine-mesh results (the τ with maximum TSC‑spread).
  - λ is not selected; all λ values remain available for:
    - rankings
    - bump charts
    - stability analysis
    - convexity
    - monotonicity
- Post-processing
  - Generates rankings at the chosen τ
  - Produces λ‑sweep bump charts and stability plots
  - Computes convexity and monotonicity in λ
  - Runs Newman (2006) spectral community detection and produces community plots  
(community bar chart and community bump chart)
  - Computes k‑core coreness and produces the coreness bump chart
  - Generates 2‑D and 3‑D k‑core topographic maps

All outputs are saved to a timestamped run directory.

**Required inputs (prompted):**
API or Existing Data
You may choose either workflow:
1. Run Stage 1. (pipeline will fetch data from CrossRef, OpenAlex, arXiv, 
  Semantic Scholar, and zbMath.) 
  Provide:
  - Query terms - used to query APIs, for labeling and, if QR terms are left blank, for QR scoring.
  - Query relevance (QR) terms - used for r_q score computation (defaults to Query terms)
  - Year range for publication (optional)
  - Record limit — maximum number of records to retrieve per API.
2. Skip Stage 1 and load existing data. (This is the “API source folder” option.)
  Provide:
  - Path to an existing combined_results.csv folder.

**Output Directory:**
All outputs are saved under
- TSC_{n}_{terms}_{method}_{timestamp}/
This directory contains all AA–AH conceptual categories, plus intermediate CSVs and diagnostic plots.

**Full Output Inventory:**
- AA - Output Log
  - `AA - Output_text_*.txt` 
    - Full console log including:
      - API summaries
      - cleaning statistics
      - duplicate‑review decisions
      - coarse and fine sweep timing
      - [TSC vs Year] regression 𝑅^2
      - all warnings and diagnostics
- AB - Pass Data
  - `AB - Pass_data.csv`
    - Metadata for cross‑query comparison:
      - query terms
      - record counts
      - timing
      - chosen τ
      - fine-grid λ range
- AC - Summary Table
  - `AC - Summary_table.csv`
    - Run overview:
      - deduplication counts
       - language filtering
       - TR/QR distributions
       - final corpus size
- AD Outputs — Heatmaps, Histograms, Convexity & Monotonicity
  Heatmaps
  - `TSC_heatmap_coarse.png` — coarse TSC surface
  - `TSC_heatmap_fine.png` — fine TSC surface
  Mesh-grid Heatmaps with Embedded Histograms
  - `TSC_heatmap_coarse_histograms_{timestamp}.png` 
    Coarse (𝜏,𝜆) mesh heatmap with embedded TSC histograms in each grid cell.
  - `TSC_heatmap_fine_histograms_{timestamp}.png`
    Fine (𝜏,𝜆) mesh heatmap with embedded TSC histograms in each grid cell.
  Per‑parameter Histogram Overlays
  - `TSC_histogram_tau_{τ}_lambda_{λ}_{theme}_{timestamp}.png`  
    multiple files; one per (τ, λ) pair in the fine grid
  Convexity & Monotonicity Diagnostics
  - `Convexity_per_paper.csv` — convexity classification per document
  - `Monotonicity_per_paper.csv` — monotonicity classification per document
  - `Convexity_fraction_heatmap.png` — fraction‑convex across the grid
  - `Monotonicity_fraction_heatmap.png` — fraction‑monotone across the grid
- AE - Rankings, Bump Charts, Timing
  Rankings
  - `Ranking_FINE.xlsx` — full ranking table across λ at the chosen τ
  - `top10_ranking_by_lambda_FINE.xlsx` — top‑10 per‑λ ranking table
  Bump Charts
  - `bump_chart_FINE.png` — bump chart of documents that appear in the top‑10 for at least one value of λ.
  - `bump_chart_stable.png` — stability‑filtered bump chart
  Timing
  - `timing_summary.csv` — coarse + fine sweep timing breakdown
- AF - Community Detection
  - `DOIs_Communities_*.csv`
    Contains each document’s assigned community from Newman (2006) spectral community detection.
    Columns typically include: DOI, community ID, and (optionally) TR/QR/TSC/coreness values.
  - `Community_bar_chart.png`
    A bar chart showing the size of each detected community.
    The x‑axis lists community IDs; the y‑axis shows how many documents fall into each one.
    This is the primary overview of the community structure.
  - `Community_bump_chart.png`
    A bump chart showing the community of documents in the top‑10 for at least one value of λ. 
- AG - Coreness
  - `Coreness_values.csv` — Coreness (k‑core index) for each document in the similarity network.
  - `Coreness_bump_chart.png` — Bump chart showing the coreness of documents that appear in the top‑10 for at least one value of λ. 
- AH - K-Core Maps
  - `Kcore_contour.png` — 2-D contour maps of k-core structure across the similarity network.
  - `Kcore_community_overlay.png` — Community assignments overlaid on the k‑core contour map.
  - `Kcore_surface_DOI_spheres.png` — 3‑D k‑core surface with top10 DOIs‑labeled spheres
  - `Kcore_surface_circles.png` — 3‑D k‑core surface with DOI‑labeled circles.
- Intermediate Outputs
 - `raw_results.csv` (if Stage 1 is run)
    Produced only if Stage 1 (API querying) is run.
    Contains the raw, unprocessed records returned directly from Crossref, OpenAlex, arXiv, Semantic Scholar, and zbMATH.
    Includes fields such as DOI, title, abstract, authors, publication year, and source metadata.
    This file is the earliest snapshot of the corpus before any cleaning or filtering.
 - `cleaned.csv`
    The corpus after cleaning and deduplication.
    Includes:
      - language filtering
      - removal of malformed records
      - DOI‑based deduplication
      - abstract/title normalization
    This is the dataset used for TF–IDF construction.
 - `combined.csv`
    The corpus after title–abstract merging (Stage 2.5).
    Each row contains:
      - DOI
      - combined text field (title + abstract)
      - publication year
      - any additional metadata preserved from cleaning
    This is the input to TF–IDF vectorization.
 - `tfidf_cosine_matrix_by_doi_tau_0.00.csv`
    The full pairwise cosine similarity matrix at τ = 0.00 (i.e., no thresholding).
    Rows and columns correspond to DOIs.
    This file is used to:
      - verify TF–IDF construction
      - inspect raw similarity structure
      - debug thresholding behavior
    It is the baseline similarity matrix before τ‑thresholding.
 - `similarity_matrix_tau_*.csv`
    A family of files, one per τ value in the coarse and fine grids.
    Each file contains the τ‑thresholded similarity matrix, where entries below τ are set to zero.
    These matrices are used for:
      - TSC computation
      - convexity/monotonicity analysis
      - community detection
      - k‑core decomposition
    The filename encodes the τ value used.
 - `TR_values.csv`
    Contains temporal relevance (TR) values for each document.
    Computed from publication year using the exponential decay model described in Chapter 3.
    This file is used in:
     - TSC computation
     - histogram diagnostics
     - convexity/monotonicity validation
 - `QR_values.csv`
    Contains query relevance (QR) values for each document.
    Computed from the TF–IDF cosine similarity between each document and the query vector 𝑟_𝑞.
    Used in:
      - TSC computation
      - histogram diagnostics
      - ranking interpretation
This directory is the complete record of the run and is used for reproducibility, debugging, and cross‑query comparison.

---

## TSC Comparison Tool — Pairwise Focused vs Broad Query Comparison
Side‑by‑Side TSC Ranking Comparison
The TSC Comparison Tool compares the outputs of two separate TSC pipeline runs:
  - a focused (narrow, targeted) query run
  - a broad (wide, noisy) query run

It produces paired Excel workbooks, comparison tables, and publication‑quality PDF figures showing how the two runs differ in ranking behavior at the chosen τ.

The tool automatically locates required files inside each run folder, builds comparison tables, inserts rank‑lookup columns, computes cross‑file ranking statistics, and generates light/dark PDF figures.

**Entry point:** run as a Jupyter notebook cell; prompts for all required inputs (focused run folder, broad run folder, output directory, and optional slug/label overrides).

**What it does:**
For each pair of focused/broad runs, the tool:
  - Loads the top‑10 ranking tables
  - Loads the per‑DOI TSC score CSVs
  - Normalizes DOIs and identifies overlap
  - Builds rank‑lookup maps for both runs
  - Inserts a new “Highest Rank (other run)” column into each workbook
  - Highlights top‑10 ranks in green
  - Computes:
    - count of DOIs ranked ≤ 10
    - total records in the comparison file
    - worst top‑10 percentile
    - chosen‑τ cross‑file ranking statistics (if available)
  - Sorts rows by λ = 0.01 for readability
  - Adds hyperlinks to the source folders
  - Produces short‑slug and long‑slug Excel workbooks
  - Generates chosen‑τ ranking PDFs (light, dark, 4‑colour, 4‑colour dark)
    Only produced if the CSV contains Max_TSC_Rank_Selected_Tau.

All outputs are written to a timestamped comparison folder.

**Required inputs (prompted):**
You will be prompted for (Press Enter to accept defaults):
  - Focused query folder  
    Output directory of the focused TSC run.
  - Broad query folder  
    Output directory of the broad TSC run.
  - Output base directory  
    Where all comparison files will be written (defaults to your dissertation Compare folder).
  - Custom slugs and labels (optional)
    Used for file naming and figure titles.

Required files inside each TSC run folder (automatically located):
  -  `top10_ranking_by_lambda_FINE.xlsx`  
    The fine‑mesh bump‑chart ranking table (top‑10 across λ).
  - `DOIs_with_max_TSC_values_*.csv`  
    Per‑DOI TSC scores at optimal (τ, λ).
    Used to compute best ranks and chosen‑τ ranks.

If either file is missing, the tool raises a clear error.

All outputs are generated automatically.

**Output Directory:**
All outputs are saved under
- {output_base}/{focused_long_slug}_{broad_long_slug}_{timestamp}/
The tool automatically shortens slugs if Windows path‑length limits would be exceeded.

**Key outputs:**
- Comparison Log
  - `comparison_output_{timestamp}.txt` 
    Full console log including file detection, slug selection, rank statistics, and warnings.
- Excel Workbooks (Short Slugs)
  - `comparison_{focused}_to_{broad}.xlsx`
  - `comparison_{broad}_to_{focused}.xlsx`
    Short slugs use only the document counts (e.g., 150_to_1000).
  Each workbook contains:
    - coloured comparison tables
    - inserted rank‑lookup column
    - top‑10 counts
    - worst‑percentile row
    - chosen‑τ ranking summary (if available)
    - hyperlinks to source folders
- Excel Workbooks (Long Slugs)
  - `comparison_{focused_long}_to_{broad_long}.xlsx`
  - `comparison_{broad_long}_to_{focused_long}.xlsx`
  Long slugs include descriptive query terms (e.g., `150_RSA_Attack_Complexity_to_1000_Cryptography`).
  Same content as short‑slug workbooks, with more descriptive filenames for archival clarity.
- Chosen‑τ Ranking Figures — (Only produced if Max_TSC_Rank_Selected_Tau exists in the CSV.)
  - `ranking_chosen_tau_{focused_long}_vs_{broad_long}.pdf`
  - `ranking_chosen_tau_{focused_long}_vs_{broad_long}_dark.pdf`
  - `ranking_chosen_tau_{focused_long}_vs_{broad_long}_4colour.pdf`
  - `ranking_chosen_tau_{focused_long}_vs_{broad_long}_4colour_dark.pdf`
  Short‑slug versions are also produced.
  These figures show the chosen‑τ ranking distribution for both runs, using light/dark and 4‑colour schemes.

---

## TSC Comparison Table Builder — Multi-Pairing Summary Table
Side‑by‑Side Multi‑Pair Comparison of Focused vs Broad TSC Runs

The TSC Comparison Table Builder constructs a single Excel workbook summarizing multiple focused/broad TSC run pairs. Each pair occupies two rows (focused on top, broad on bottom), with merged cells where values match. The resulting workbook provides a consolidated view of cross‑query behavior across all selected simulations.

The tool extracts metadata from each run folder, computes cross‑comparison metrics, and produces a colour‑coded, publication‑ready summary table.

**Entry point:** Run as a Jupyter notebook cell; prompts for all required inputs (number of pairs, focused run folders, broad run folders, and optional comparison folders).

**What it does:**
For each focused/broad pair, the tool:
  - Loads run metadata from each TSC folder
  - Extracts:
    - query terms
    - QR terms
    - chosen τ
    - total DOIs
    - top‑10 DOIs
    - stable top‑10 DOIs (rank ≤ 10 across all λ)
    - number of communities represented in the top‑10
    - number of coreness values represented in the top‑10
    - TSC‑vs‑Year linear‑fit R²
  - Computes cross‑comparison metrics:
    - worst rank of any top‑10 DOI from run A when ranked in run B
    - number of top‑10 DOIs not found in the comparison run
    - number of DOIs common to both top‑10 lists
    - percentage of focused‑query DOIs that rank within the top‑N of the noisy query
      (N = total DOIs in the focused run)
  - Reads comparison metrics either from:
    - a TSC Comparison Tool output folder (preferred), or
    - the DOIs_with_max_TSC_values_.csv* files (fallback)
  - Builds a colour‑coded Excel workbook:
    - two rows per pair (focused / broad)
    - merged cells where values match
    - alternating colour schemes per pair
    - wrapped, centered headers
    - consistent formatting across all columns

All outputs are written to a timestamped Excel file.

**Required Inputs (prompted):**
You will be prompted for (Press Enter to accept defaults):
  - Number of simulation pairs to compare  
    Determines how many focused/broad folder pairs will be processed.
  - Focused query folder (Folder A) 
    Output directory of the focused TSC run.
  - Broad query folder (Folder B)  
    Output directory of the broad TSC run.
  - Comparison folder (optional)  
    A folder produced by TSC Comparison Tool containing:
      - `comparison_*chosen_tau.xlsx` files.
    If omitted, cross‑comparison metrics are computed from each run’s:
      - `DOIs_with_max_TSC_values_*.csv` file.

Default folder paths are provided for known pairs.

Required files inside each TSC run folder (automatically located):
  - `top10_ranking_by_lambda_FINE.xlsx`
    Used to extract:
      - top‑10 DOIs
      - stable DOIs
      - query terms
      - QR terms
      - total DOI count
  - `optimization_summary.csv`
    Used to extract the chosen τ.
  - `AF - DOIs_Communities_*.csv`
    Used to extract:
      - community counts
      - coreness counts
  - `AA - Output_text_*.txt`
    Used to extract:
      - stable DOI counts (fallback)
      - TSC‑vs‑Year R² (primary source)
  - `TSC_vs_Year_breusch_pagan_*.csv`  
    Used to extract R² (fallback).

If any required file is missing, the tool attempts fallbacks where possible.

**Output Directory:**
The tool produces a single Excel workbook in the current working directory:

**Key Outputs:**
- Excel workbook 
  - `Comparison_Table_{timestamp}.xlsx`
    Contains two sheets:
      - Sheet 1 - Full Summary Table
        Column order (With Worst Rank sheet):
        |Col |Header |
        |---|---|
        |1 |Query Terms |
        |2 |QR Terms |
        |3 |# DOIs in Query Results |
        |4 |Chosen τ |
        |5 |# DOIs Ranked in Top 10 |
        |6 |# DOIs Stable in Top 10 across all λ |
        |7 |# DOIs Common to both Top 10 lists |
        |8 |Worst Rank of any Top 10 DOI in Comparison Query |
        |9 |% of Focused Query DOIs in Top N of Broad Query |
        |10 |# of DOIs in Top 10 not in Comparison Query |
        |11 |TSC vs Pub Date Linear Fit R² Value |
        |12 |# of Communities Represented in Top 10 |
        |13 |# of Coreness Values Represented in Top 10 |
      - Sheet 2 - Summary Table (No Worst-Rank Column)

**Color Scheme**
Each query pair uses a two‑row colour block:
  - Pair 1: blue
  - Pair 2: green
  - Pair 3: yellow
  - Pair 4: pink
  - Pair 5+: colours repeat cyclically
Headers use a dark blue fill with white text.

---

## TSC Mechanism Analysis Tool — Displacement Mechanism Analysis
Side‑by‑Side Mechanistic Analysis of Focused vs Broad TSC Ranking Behavior

The TSC Mechanism Analysis Tool examines why broad‑query documents displace focused‑query documents in TSC rankings. It computes diagnostic statistics, constructs multiple document sets, and produces a comprehensive suite of plots and tables that reveal the mechanisms driving displacement.

The tool evaluates vocabulary match, publication recency, neighborhood structure, and tri‑factor product behavior, providing a detailed mechanistic explanation of ranking shifts between focused and broad queries.

**Entry point:** Run as a Jupyter notebook cell; prompts for all required inputs (focused run folder, broad run folder, query‑relevance terms, and comparison output folder).

**What it does:**
For a focused/broad pair of TSC runs, the tool:
1. Loads all required data
  - Combined title–abstract CSV
  - Full‑precision cosine similarity matrix
  - Fine‑mesh TSC values at chosen τ across all λ values (max‑TSC values at λ=0.01)
  - Optimization summary (τ, λ)
  - Deduplication logs (for DOI alias resolution)
  - Community and coreness CSVs
  - TSC‑vs‑Year regression diagnostics
2. Computes query‑relevance scores
  - TF‑IDF cosine similarity (r_q)
  - QR vocabulary density (unigram/bigram match)
3. Defines all document sets
  Using dissertation notation:
  |Set |Meaning |Code Variable |
  |---|---|---|
  |R |Focused documents retained above rank threshold X	 |set_R | 
  |R_Q5 |Top‑quintile QR subset of R |set_R_Q5 |
  |D |Focused documents displaced below rank threshold X |set_B |
  |B |Broad‑query documents occupying top X ranks |set_C |
  |N |Broad‑query documents below rank threshold X |set_N |
Where X = size of the focused corpus, and ranking is based on maximum TSC at chosen τ.
4. Computes mechanism‑specific diagnostics
  Mechanism 1 — Query‑relevance vocabulary match
    - Mean r_q per set
    - QR vocabulary density
    - Boxplots and distributions
  Mechanism 2 — Publication recency
    - Mean publication year per set
    - Histograms
  Mechanism 3 — Neighborhood structure
    - Degree (neighbors with cosine similarity > τ)
    - High‑r_q neighbor counts
    - Tri‑factor product scatter plots
    - Iso‑TSC boundary curves
5. Computes displacement statistics
  - Sizes of R, R_Q5, D, B, N
  - DOI alias resolution for deduplicated documents
  - TSC boundary at rank X
  - Per‑set TSC distributions
  - Quintile displacement histograms
  - Focused vs broad share of top‑X ranks
6. Produces all diagnostic outputs
All outputs are written to the comparison folder (light + dark themes).

**Required inputs (prompted):**
You will be prompted for (Press Enter to accept defaults):
  - Focused query run folder
    Output directory of the focused TSC run.
  - Broad query run folder (noisy)
    Output directory of the broad TSC run.
  - QR terms (for r_q score computation)
    Defaults to the focused run’s QR terms extracted from the AA log.
  - Comparison output folder (Defaults to the sibling Compare directory next to the broad run)
    All mechanism outputs are written here.

Required files inside each TSC run folder (automatically located):
  - `clean_results_with_combined_title_abstract*.csv`
    Used for text, DOI normalization, and QR scoring.
  - `tfidf_cosine_matrix_by_doi_tau_0.00.csv`
    Full‑precision cosine matrix; τ threshold applied in memory.
  - `TSC_Values_FINE_tau_*_lambda_*.csv`
    Fine‑mesh TSC values for chosen τ across all λ values (max-TSC occurs at λ=0.01).
  - `DOIs_with_max_TSC_values_*.csv`
    Max‑TSC values at λ=0.01; used for ranking and tri‑factor scatter.
  - `optimization_summary.csv`
    Provides chosen τ and λ (fine‑mesh peaks).
    The λ peak is always 0.01 because TSC is monotonically decreasing in λ.
  - `AF - DOIs_Communities_*.csv`
    Community and coreness assignments.
  - `AA - Output_text_*.txt`
    Used to extract QR terms and fallback diagnostics.
  - `TSC_vs_Year_breusch_pagan_*.csv`
    Fallback source for linear‑fit R².

If any required file is missing, the tool attempts fallbacks where possible.

**Output Directory:**
All outputs are written to the selected comparison folder: The comparison folder is the directory where all mechanism plots, tables, and logs are saved.

**Key Outputs:**
All outputs are written to the selected comparison folder, in both light and dark themes.
1. Tabular & Textual
  - Console Summary (printed + logged)
    A full textual summary of:
      - sizes of set R, set R_Q5, set D, set B, set N
      - QR thresholds
      - displacement counts
      - mechanism‑specific statistics
      - DOI alias resolutions
      - rank threshold X
      - iso‑TSC boundary
  This appears in the notebook and is also written to the log file.
  - Mechanism Analysis Log
    `Mechanism_analysis_log_{timestamp}.txt`
    A complete transcript of all console output:
      - set definitions
      - mechanism diagnostics
      - warnings
      - fallback resolutions
      - file detection
      - ranking summaries
      - TSC boundary
      - QR thresholds
      - displacement counts
  Used for reproducibility and archival documentation.
  - Abstract Inspection Table
    `Abstract_Inspection_{timestamp}.csv`
    A CSV or text table listing:
      - displaced focused documents (set D)
      - broad‑query documents below rank X (set B)
      - each document’s title, abstract, DOI, rank, TSC, and QR score
    Used to manually inspect semantic differences between displaced and displacing documents.
2. Distributions
  - TSC Score Histograms
    `TSC_hist_RQ5_{ts}*.png`
    `TSC_hist_D_{ts}*.png`
    `TSC_hist_B_{ts}*.png`
    Histograms of max‑TSC values (λ = 0.01) for set R_Q5, set D, set B.
    Shows how TSC magnitude differs between retained, displaced, and displacing documents.
  - Publication Year Histograms
    `PubYear_hist_R_{ts}*.png`
    `PubYear_hist_D_{ts}*.png`
    `PubYear_hist_B_{ts}*.png`
    Histograms of publication years for each set.
    Used to test Mechanism 2: Recency.
  - Vocabulary Density & r_q Boxplots
    `rq_boxplot_{ts}*.png`
    `vocab_density_boxplot_{ts}*.png`
    Boxplots comparing:
      - TF‑IDF cosine similarity (r_q)
      - QR vocabulary density
    Used to test Mechanism 1: Vocabulary match.
  - High‑r_q Neighbor Count Histograms
    `HighRQ_neighbors_hist_{ts}*.png`
    Histograms of the number of neighbors above the QR threshold.
    Used to test Mechanism 3: Neighborhood structure.
  - Tri‑Factor Product Histograms
    `TFP_hist_full_{ts}*.png`
    `TFP_hist_capped99_{ts}*.png`
    Histograms of (max_TSC)/(degree) for each set, including:
      - full distribution
      - 99%‑capped version
    Shows how tri‑factor strength differs between displaced and displacing documents.
3. Displacement & Ranking
  - Quintile Histogram
    `Quintile_Displacement_{ts}*.png`
    Shows how many focused documents in each QR quintile were:
    - retained (set R)
    - displaced (set D)
  Tests whether displacement disproportionately affects low‑QR or high‑QR documents.
  - Ranking State Plot
    `Ranking_State_{ts}.png`
    Shows the share of top‑X ranks occupied by:
    - focused documents
    - broad‑query documents
  Broken down by QR quintile.
4. Source Provenance
  - API‑Source Pie Charts
    Five pie charts showing the distribution of API sources for:
      - top‑10 focused — `API_top10_focused_{ts}*.png`
      - top‑10 broad — `API_top10_broad_{ts}*.png`
      - top‑X broad — `API_topX_broad_{ts}*.png`
      - set B — `API_setB_{ts}*.png`
      - set D — `API_setD_{ts}*.png`
    Useful for identifying whether displacement is driven by particular APIs.
  - API‑Source Stacked Bar Chart
    `API_stacked_bar_{ts}*.png`
    A single bar chart comparing API‑source composition across all five sets.
5. Tri‑factor Product Scatter Plots
  These are the core visual diagnostics.
  - Two‑Panel Scatter (Left: set D | Right: set B)
    `Scatter_two_panel_{ts}*.png`
    Both show:
      - degree (x‑axis)
      - mean tri‑factor product (y‑axis)
      - rank X iso‑TSC curve
      - set mean overlays
  - Single‑Panel Scatter
    `Scatter_single_panel_{ts}*.png`
    All sets plotted together with membership‑based colouring.
  - TSC‑Coloured Scatter
    `Scatter_TSC_colored_{ts}*.png`
    Points coloured by max‑TSC magnitude.
  - Three‑Panel Small Multiples
    `Scatter_small_multiples_{ts}*.png`
    Panels for:
      - set D
      - set B
      - set R
    Used to compare structural differences across sets.
  - Combined R_Q5 Scatter Variants
    `Scatter_RQ5_combined_{ts}*.png`
    Plots combining set R_Q5 with displaced/broad sets to highlight high‑QR retention behavior.
6. Network Export
  - Mechanism Network
    - `Mechanism_network_{ts}.gexf`
      A Gephi‑ready network containing:
       - nodes = set R_Q5 ∪ set D ∪ set B
       - edges = cosine‑similarity edges above τ
       - node sizes = max‑TSC values
       - node colours = set membership
      Used for interactive exploration of neighborhood structure.

---

## TSC Mechanism Comparison Tool — Cross-Pairing Mechanism Comparison
Side‑by‑Side Comparison of Displacement Mechanisms Across Multiple TSC Simulation Pairings

The TSC Mechanism Comparison Tool evaluates whether a consistent displacement mechanism explains why broad‑query documents outscore focused‑query documents across multiple focused/broad TSC pairings. It reads the `Mechanism_analysis_log_*.txt` files produced by the TSC Mechanism Analysis Tool, extracts mechanism‑specific metrics, computes per‑pairing differences, and determines which mechanisms are consistently positive (favor set B) across all pairings.

The tool produces a consolidated CSV, a detailed comparison log, and a multi‑panel figure summarizing mechanism behavior across all datasets.

**Entry point:** Run as a Jupyter notebook cell; prompts for the number of pairings, the log files (or folders containing them), and an output directory.

**What it does:**
For each Mechanism Analysis log, the tool:
1. Loads and parses Mechanism Analysis logs
  - Extracts:
    - focused and broad run folder names
    - query relevance terms
    - set sizes (set R, set R_Q5, set D, set B, set N)
    - chosen τ
    - DOI lists for set D and set B (for fallback degree computation)
    - TSC boundary value
    - source provenance counts
    - quintile displacement distributions
2. Extracts mechanism‑specific metrics
  - From each log, the tool reads:
    - Mechanism 1 — Query Relevance
      - mean r_q score (set B vs set D)
      - mean QR vocabulary density (set B vs set D)
    - Mechanism 2 — Publication Recency
      - mean publication year (set B vs set D)
    - Mechanism 3 — Neighborhood Structure
      - mean high‑r_q neighbor count (set B vs set D)
      - mean total degree (set B vs set D)
      - mean tri‑factor product (set B vs set D)
  All metrics are extracted as Δ = (set B − set D).
3. Computes fallback metrics when missing
  - If the Mechanism Analysis log does not contain:
    - mean total degree
    - mean high‑r_q neighbor count
  the tool loads the full‑precision cosine similarity matrix and recomputes:
    - total degree
    - high‑r_q neighbor counts
  directly from the matrix.
4. Builds a consolidated results table
  - For each pairing, the tool computes:
    - Δr_q
    - Δvocab density
    - Δpublication year
    - Δhigh‑r_q neighbor count
    - Δtotal degree
    - Δtri‑factor product
    - percentage differences where applicable
  All results are stored in a single CSV.
5. Determines mechanism consistency
  - For each mechanism group:
    - Mechanism 1: {Δr_q, Δvocab density}
    - Mechanism 2: {Δpublication year}
    - Mechanism 3: {Δhigh‑r_q neighbor count, Δtotal degree, Δtri‑factor product}
  The tool checks whether all Δ values are positive across pairings.
It reports:
  - consistent + (all positive)
  - consistent − (all negative)
  - mixed (some positive, some negative)
It then ranks mechanisms by:
  - consistency
  - mean absolute Δ magnitude
  - and identifies the primary mechanism(s).
6. Produces all diagnostic outputs
  -All outputs are written to the selected comparison folder.

**Required inputs (prompted):**
You will be prompted for (Press Enter to accept defaults):
  - Number of simulation pairings 
    Must be ≥ 2.
  - Mechanism Analysis log file or folder for each pairing  
    If a folder is provided, the latest `Mechanism_analysis_log_*.txt` is used.
  - Query terms for each pairing  
    Defaults to predefined terms for known datasets.

Required Files (automatically located)
  For each pairing:
    - `Mechanism_analysis_log_*.txt`
      The log produced by the TSC Mechanism Analysis Tool.
  Optional (fallback degree computation):
    - `tfidf_cosine_matrix_by_doi_tau_0.00.csv`
      Used only if degree metrics are missing from the log.

**Output Directory:**
All outputs are written to the selected comparison folder.
This folder will contain:
  - the mechanism comparison log
  - the consolidated CSV
  - the multi‑panel comparison figure (light + dark themes)
  - individual metric panels (PNG)

**Key Outputs:**
1. Mechanism Comparison Log
  - `Mechanism_comparison_log_{timestamp}.txt`
    Contains:
      - parsed metrics for each pairing
      - Δ values for all mechanisms
      - consistency analysis
      - mechanism ranking
      - identification of primary mechanism(s)
      - per‑pairing detailed table of all Δ values
    This log is the authoritative record of cross‑pairing mechanism behavior.
2. Consolidated Results CSV
  - `Mechanism_comparison_{timestamp}.csv`
    Contains one row per pairing with columns:
      - pairing label (𝒟₁, 𝒟₂, …)
      - query terms
      - Δr_q
      - Δvocab density
      - Δpublication year
      - Δhigh‑r_q neighbor count
      - Δtotal degree
      - Δtri‑factor product
      - percentage differences for degree and tri‑factor product
    Used for statistical analysis and inclusion in dissertation tables.
3. Multi‑Panel Comparison Figure
  - `Mechanism_comparison_panels_*.png`
    6‑panel figures summarizing:
      - Δr_q
      - Δvocab density
      - Δpublication year
      - Δhigh‑r_q neighbor count
      - Δtotal degree
      - Δtri‑factor product (% difference)
    Each panel shows:
      - dataset labels (𝒟₁, 𝒟₂, …)
      - Δ values
      - mechanism interpretation
      - color‑coded background (light or dark theme)
    A footer panel summarizes:
      - mechanism definitions
      - interpretation of positive Δ values
      - query terms used
    Individual PNG panels are also saved.
4. Per‑Pairing Δ Table (Console + Log)
  - A detailed table listing:
    - pairing label
    - query terms
    - Δr_q
    - Δvocab density
    - Δpublication year
    - Δhigh‑r_q neighbor count
    - Δtotal degree
    - Δdegree %
    - Δtri‑factor product
    - Δtri‑factor product %
  Used to inspect mechanism behavior on a per‑dataset basis.
5. Fallback Degree Computation (if needed)
  - If degree metrics are missing from the log, the tool:
    - loads the full cosine matrix
    - applies τ threshold
    - recomputes total degree for set D and set B
    - inserts these values into the comparison table
  This ensures complete metrics even when logs are incomplete.

---

## TSC Chapter 3 Assessment — Chapter 3 Theorem Empirical Assessment Suite
Numerical Assessment of Eight Core TSC Properties from Dissertation Chapter 3

The TSC Chapter 3 Assessment Tool empirically evaluates whether the eight theoretical properties of Temporal‑based Semantic Similarity (TSC), proven in Chapter 3 of the dissertation, hold on a given TSC simulation dataset. These assessments do not constitute proofs; instead, they numerically assess the behavior predicted by the theory and quantify how tight the associated bounds are.

Each assessment computes its numerical results once, then renders all figure variants (light/dark, titled/untitled, standalone panels) from the same underlying arrays. This ensures that all figure variants agree exactly.

**Entry point:** Run as a Jupyter notebook cell; prompts for the TSC simulation folder to assess and which Chapter 3 theoretical properties to evaluate.

**What it does:**
The tool evaluates the following eight theoretical results from Chapter 3:
  -  Assessment 1 — Definition 3.6.1: Lipschitz Continuity  
    Tests whether TSC satisfies the Lipschitz bound under random perturbations of edge weights.
  - Assessment 2 — Algorithm 1 / Theorem 3.10.1: Sampling‑Based Approximation 
    Evaluates the sampling‑based estimator for TSC, comparing empirical error rates to theoretical guarantees.
  - Assessment 3 — Corollary 3.2.3: Relative TSC Derivative 
    Computes the empirical derivative ratio and compares it to the theoretical bound.
  - Assessment 4 — Theorem 3.3.1 / Corollary 3.3.2: Exponential Temporal Decay  
    Fits exponential decay curves and verifies the predicted decay rate.
  - Assessment 5 — Theorem 3.5.4: Sensitivity Matrix Properties  
    Computes the sensitivity matrix and checks diagonal dominance, positivity, and structural properties.
  - Assessment 6 — Theorem 3.8.6 / Corollary 3.8.4: Eigenvalue Structure  
    Computes eigenvalues of the temporal‑relevance operator and verifies spectral bounds.
  - Assessment 7 — Theorem 3.4.2: Query‑Perturbation Sensitivity  
    Perturbs query‑relevance values and checks the predicted sensitivity behavior.
  - Proposition 3.8.7 — Joint Spectral Confirmation (A4 + A6)  
    Confirms that the exponential decay and spectral structure jointly satisfy the proposition’s constraints.
Each assessment produces numerical summaries, diagnostic plots, and standalone panel exports.

**Required inputs (prompted):**
You will be prompted for:
  - TSC simulation folder 
  - Which Chapter 3 assessments to run  
    (all, subset, or skip any)

Required Files (automatically located)
  TSC simulation folder must contain:
  - `optimization_summary.csv`
    Provides the fine‑mesh peaks for τ and λ.
  - `tfidf_cosine_matrix_by_doi_tau_0.00.csv`
    Full‑precision cosine similarity matrix.
  - `TSC_Values_FINE_tau_*_lambda_*.csv`
    Fine‑mesh TSC values for the chosen τ and λ.
If any required file is missing, the tool raises a descriptive error.

**Output Directory:** All outputs are written to a new folder automatically created inside the selected TSC simulation directory:
  - `<simulation folder>/Chapter3_Assessment_<timestamp>/`
    This folder contains:
      - all figures 
      - all CSV summaries
      - all assessment logs
    The user is not prompted for an output directory.

**Key Outputs:**
1. Assessment Logs
  Each assessment prints a detailed console summary, including:
    - parameter settings
    - bound evaluations
    - violation counts
    - fitted rates
    - empirical vs theoretical comparisons
  Tabular summaries are written to:
    - `Assessment{n}_<name>_summary_{timestamp}.csv`
2. Assessment 1 — Lipschitz Continuity
  Outputs include:\
    - Multi‑panel scatter figure — 
      `Assessment1_Lipschitz_scatter_{ts}.png` 
    - Max‑ratio bar plot  
      `Assessment1_Lipschitz_ratio_{ts}.png`
    - Summary CSV  
      `Assessment1_Lipschitz_summary_{ts}.csv`
3. Assessment 2 — Sampling‑Based Approximation
  Outputs include:
    - relative‑error histograms
      `Assessment2_relerr_hist_{ts}.png`
    - pass‑rate vs sample‑size sweep plots
      `Assessment2_passrate_sweep_{ts}.png`
    - per‑document error distributions
      `Assessment2_doc_meanerr_{ts}.png`
      `Assessment2_doc_maxerr_{ts}.png`
      `Assessment2_doc_failrate_{ts}.png`
      `Assessment2_doc_nbrcount_{ts}.png`
    - summary CSV  
      `Assessment2_Sampling_summary_{ts}.csv`
4. Assessment 3 — Relative TSC Derivative
  Produces:
    - derivative scatter plots
      `Assessment3_derivative_scatter_{ts}.png`
    - bound‑comparison plots
      `Assessment3_derivative_bound_{ts}.png`
      `Assessment3_derivative_ratio_{ts}.png`
    - summary CSV
      `Assessment3_RelativeDerivative_summary_{ts}.csv`
5. Assessment 4 — Exponential Temporal Decay
  Produces:
    - decay‑curve fits
      `Assessment4_decay_fit_{ts}.png`
    - residual plots
      `Assessment4_decay_residuals_{ts}.png`
      `Assessment4_decay_multiplot_{ts}.png`
    - summary CSV
      `Assessment4_ExponentialDecay_summary_{ts}.csv`
6. Assessment 5 — Sensitivity Matrix Properties
  Produces:
    - sensitivity‑matrix heatmaps
      `Assessment5_sensitivity_heatmap_{ts}.png`
    - diagonal‑dominance diagnostics
      `Assessment5_sensitivity_diagdom_{ts}.png`
    - eigenvalue plots
      `Assessment5_sensitivity_eigs_{ts}.png`
    - summary CSV
      `Assessment5_Sensitivity_summary_{ts}.csv`
7. Assessment 6 — Eigenvalue Structure
  Produces:
    - spectral plots
      `Assessment6_spectrum_{ts}.png`
    - eigenvalue distributions
      `Assessment6_eig_distribution_{ts}.png`
      `Assessment6_spectral_multiplot_{ts}.png`
    - summary CSV
      `Assessment6_Eigenstructure_summary_{ts}.csv`
8. Assessment 7 — Query‑Perturbation Sensitivity
  Produces:
    - perturbation‑response curves
      `Assessment7_qpert_curve_{ts}.png`
    - sensitivity scatter plots
      `Assessment7_qpert_scatter_{ts}.png`
      `Assessment7_qpert_ratio_{ts}.png`
    - summary CSV
      `Assessment7_QueryPerturbation_summary_{ts}.csv`
9. Proposition 3.8.7 — Joint Spectral Confirmation
  Produces:
    - combined spectral/decay diagnostics
      `AssessmentP_joint_spectral_{ts}.png`
      `AssessmentP_joint_decay_{ts}.png`
      `AssessmentP_joint_multiplot_{ts}.png`
    - summary CSV
      `AssessmentP_JointSpectral_summary_{ts}.csv`

---

## Ranking Method Comparison — TSC vs BM25 vs Dense E5 vs Hybrid RRF vs Degree / Eigenvector / Betweenness / Closeness
Side‑by‑Side Comparison of 20 Top‑Ranked Documents Across Semantic and Network‑Based Methods

The Ranking Method Comparison Tool compares the 20 top‑ranked DOIs produced by the TSC pipeline against a suite of alternative semantic and network‑centrality ranking methods. It evaluates how TSC’s top‑20 documents differ from rankings produced by keyword relevance, dense embeddings, hybrid fusion, and graph‑based prestige measures.

The tool produces:
  - a multi‑sheet Excel workbook containing all rankings, scores, and rank‑alignment tables
  - a bump chart visualizing rank movement across methods
  - optional dark‑theme Excel and bump‑chart variants

**Entry point:** Run as a Jupyter notebook cell; prompts for the TSC run folder to compare against alternative ranking methods.

**What it does:** 
For the selected TSC run folder, the tool computes rankings using:
  - Information‑Retrieval Methods
    - BM25 — Okapi BM25 keyword relevance
    - Dense E5 — intfloat/e5‑small‑v2 sentence embeddings (cosine similarity)
    - Hybrid RRF — Reciprocal Rank Fusion of BM25 + Dense E5
  - Network‑Centrality Methods (using the τ‑thresholded similarity network)
    - Degree — weighted degree centrality
    - Eigenvector — recursive prestige
    - Betweenness — shortest‑path bridging frequency
    - Closeness — inverse average distance
    Note: PageRank is implemented internally but not included in the comparison outputs, Excel workbook, bump chart, or Kendall‑τ tables. In a dense undirected graph, like ours, PageRank is computationally similar to Eigenvetor centrality.
  - TSC Rankings
    - Loaded from `DOIs_with_max_TSC_values_*.csv`
    - Optionally re‑ranked using fine‑mesh TSC at λ = 0.01
  - Agreement Metrics
    - Top‑20 overlap
    - Kendall τ@k between ranking pairs
  - Outputs
    - Excel workbook (light + dark themes)
    - Bump chart (light + dark themes)

**Required Inputs (prompted):**
You will be prompted for:
  - TSC run folder  
    (the directory containing the TSC simulation results)

Required Files (automatically detected)
  Inside the selected TSC run folder:
  - `optimization_summary.csv`
  - `AA - Output_text_*.txt`
  - `clean_results_with_combined_title_abstract_*.csv`
  - `DOIs_with_max_TSC_values_*.csv`
  - `tfidf_cosine_matrix_by_doi_tau_*.csv`
  If any required file is missing, the tool prints a warning and skips the corresponding ranking method.

**Output Directory:**
All outputs are written to a new folder created inside the selected TSC run directory:
  - `<run folder>/AE_Ranking_Comparison_<timestamp>/`
    This folder contains:
      - Excel workbooks
      - bump charts
      - logs and console summaries
    The user is not prompted for an output directory.

**Key Outputs:**
1. Excel Workbooks
  - `AE - Ranking_comparison_{ts}.xlsx`
    Includes two tabs:
    - Rankings
      Compares absolute ranking overlap between methods
    - Agreement
      Shows Kendall τ@20 comparisons
2. Bump Chart
  - `AE - Ranking_bump_{ts}.png`
    Bump charts for top 20 for all methods.
3. Heatmaps
  - `AE - Heatmap_overlap_{ts}.png`
    Shows heatmap of ranking overlap between methods
  - `AE - Heatmap_kendall_{ts}.png`
    Shows heatmap of Kendal τ@20 comparisons between methods
4. Console Summary
  - Includes:
    - detected τ
    - detected query terms
    - detected QR terms
    - number of documents
    - ranking method availability
    - centrality warnings
    - bump‑chart overlap and Kendall‑τ lines

---

## TSC Network Video Generator — Produces animation videos from a ranking-comparison folder
Animated Comparison of How Different Ranking Methods Identify Structural Centers in the Similarity Network

The TSC Network Video Generator creates an MP4 animation that visualizes how different ranking methods identify the “center” of the similarity network. The video shows where each method places its ranking-window region within the network’s structure.

As the ranking window scrolls from 1–100, to 51–150, and onward through rank N-99–N, the animation highlights:
  - which parts of the network each method considers central
  - where methods agree (overlapping neighborhoods)
  - where methods disagree (distinct structural centers)
  - how the induced subgraphs differ across ranking windows

This produces a visual comparison of how different methods identify structural centers across the full ranking spectrum.

**Entry point:** Run as a Jupyter notebook cell; prompts for the TSC run folder containing the similarity matrices and ranking outputs.

**What it does:**
For the selected TSC simulation folder, the tool:
  - Loads the similarity network
  - Loads ranking outputs for each method
  - Defines a scrolling ranking window (e.g., 100-document window)
  - For each window position:
    - extracts the induced subgraph for each method
    - highlights the nodes inside the current ranking window
    - overlays or juxtaposes method‑specific centers
    - draws edges and weights for the active neighborhood
  - Renders each window position as a frame
  - Stitches all frames into an MP4 animation
  - Optionally exports all frames as PNGs
The resulting video shows how the perceived “center” of the network changes depending on the ranking method, and how the varying ranking windows compare.

**Required Inputs (prompted):**
You will be prompted for:
  - TSC simulation folder
  - ranking‑window size (default: 20)
  - whether to export PNG frames
  - optional frame‑rate settings

**Required Files (auto‑detected):**
Inside the selected TSC simulation folder:
  - `DOIs_with_max_TSC_values_*.csv`
  - `clean_results_with_combined_title_abstract_*.csv`
  - `tfidf_cosine_matrix_by_doi_tau_*.csv`
  - `optimization_summary.csv`
If any required file is missing, the tool prints a warning and skips the corresponding step.

**Output Directory:**
All outputs are written to a new folder created inside the selected TSC simulation directory:
  - `<simulation folder>/TSC_Network_Video_<timestamp>/`
    This folder contains:
      - MP4 animation
      - optional PNG frames
      - console summary logs
The user is not prompted for an output directory.

**Key Outputs:**
1. MP4 Animation
  - `TSC_network_video_{ts}.mp4`
    Animation showing how different ranking methods identify different structural centers of the network as the ranking window scrolls.
2. PNG Frames (optional)
  - If frame export is enabled:
    `frame_0000.png`
    `frame_0001.png`
    …
    `frame_NNNN.png`
3. Layout Cache (optional)
  - `layout_positions.pkl`  
    Stores node positions for consistent layout across frames.
4. Console Summary
  - Runtime messages printed to the notebook, including:
    - detected τ
    - ranking‑window size
    - number of frames generated
    - number of nodes and edges per frame
    - MP4 stitching status

---

## Standalone Tools — Overview and Replication Rules
The repository includes several standalone Jupyter notebook cells that either reproduce outputs that appear in a TSC run folder (including outputs generated by separate analysis code such as the Ranking Method Comparison module), or provide new visualizations not produced by the pipeline. Each standalone cell is self‑contained, requires only a TSC run folder (or a matrix file), and regenerates the same PNGs/CSVs/logs as the corresponding module without rerunning the full workflow. 

This section summarizes what each cell corresponds to, what files it generates, and where those files are saved, without repeating the detailed descriptions found in the dedicated tool READMEs.

"Replicates” refers specifically to outputs (PNGs, CSVs, logs) that appear in a TSC run folder and that the standalone cell can reproduce without rerunning the original analysis code. Cells marked as “Replicates: Nothing” provide new visualizations not produced by previously described code.

1. Newman (2006) Community Detection + Coreness (AF + AG)
Replicates: Community‑detection and coreness modules (AF + AG) from the main pipeline.

This standalone cell runs Newman’s (2006) recursive spectral bisection on the τ‑thresholded similarity matrix and optionally computes NetworkX k‑core coreness. It mirrors the AF/AG pipeline behavior exactly, producing community assignments, community‑size plots, bump charts, and (if enabled) coreness bump charts. All outputs are written directly into the selected TSC run folder.

Produces:
  - `AF - DOIs_Communities_{ts}.csv`
  - `AF - Community_sizes_{ts}.png and _dark.png`
  - `AF - Community_bump_{ts}.png and _dark.png`
  - `AG - Coreness_bump_{ts}.png and _dark.png` (if coreness enabled)
  - `AF - Community_log_{ts}.txt`

Saves to: Directly into the TSC run folder.

Inputs:
  - TSC run folder containing:
    -  `optimization_summary.csv`
    - `tfidf_cosine_matrix_by_doi_tau_*.csv`
    - `AA - Output_text_*.txt`
    - optional: `DOIs_with_max_TSC_values_*.csv`
  - Optional: user toggle for coreness computation

2. K‑Core Topographic Maps (Standalone)
Replicates: Nothing; this is a new standalone visualization tool.
New Products: This cell generates 2‑D contour maps, 3‑D surfaces, and interactive Plotly surfaces showing the k‑core topography of any adjacency matrix. It optionally overlays nodes that appear in the TSC top‑10 for any λ — new functionality not present in the main pipeline.

Produces:
  - `Kcore_contour_{ts}.png and _dark.png`
  - `Kcore_surface_{ts}.png and _dark.png`
  - `Kcore_surface_{ts}_interactive.html` (NEW FUNCTIONALITY)
  - optional `Kcore_surface_{ts}_interactive_no_top10.html` (NEW FUNCTIONALITY)

Saves to: User‑selected output folder (default: same folder as adjacency matrix CSV).

Inputs:
  - Any square adjacency‑matrix CSV (DOIs as row/column labels)
  - User‑selected output folder (default: same folder as CSV)
  - Optional: TSC run folder for top‑10 overlay

3. TSC Histogram Generator (Standalone)
Replicates: The histogram portion of the AD – TSC Sweep Diagnostics module from the main pipeline.

This cell produces a histogram of TSC values for any chosen (τ, λ) pair. If the corresponding `TSC_Values_*.csv` does not exist, it computes TSC from scratch using the full similarity matrix and combined documents CSV.

Produces:
  - `TSC_histogram_tau_{τ}_lambda_{λ}_{ts}.png`
  - `TSC_histogram_tau_{τ}_lambda_{λ}_dark_{ts}.png`  

Saves to: Directly into the TSC run folder.

Inputs:
  - TSC run folder
  - User‑specified τ and λ
  - Uses:
    - `optimization_summary.csv`
    - `tfidf_cosine_matrix_by_doi_tau_0.00.csv`
    - combined documents CSV
    - optional existing `TSC_Values_*` files

4. Cleaning Summary Table Reproduction (AB Reconstruction)
Replicates: The AB – Cleaning Summary Table module from the main pipeline.

This cell reconstructs the cleaning summary table from a previous TSC run using the `AB - Pass_data_*.csv` file. It replays the missing‑DOI, language, missing‑abstract, and deduplication filters and prints the full summary table to the notebook.

Produces:
  - Console‑rendered summary table (no PNG)

Saves to: No files written; output is printed in the notebook.

Inputs:
  - TSC run folder containing `AB - Pass_data_*.csv`
  - Optional supplemental CSV used in the original run

5. TSC vs. Age Plot
Replicates: The TSC‑vs‑publication‑year diagnostic plot from the main pipeline.

This cell loads TSC values and publication years from a run folder and produces the scatter/regression plot showing how TSC varies with document age.

Produces:
  - `TSC_vs_Age_{ts}.png`

Saves to: Directly into the TSC run folder

Inputs:
  - TSC run folder containing:
    - `TSC_Values_*_tau_*_lambda_*.csv`
    - combined documents CSV with publication years

6. Histogram Grid (FINE/COARSE Sweep)
Replicates: Nothing; this is a new standalone visualization tool.
New Products: Uses the AD τ–λ grid from the manifest but provides a new visualization. This is a visual alternative that embeds histograms inside each cell.

This cell replaces the standard spread heatmap with a grid of actual TSC histograms for each (τ, λ) pair. It also produces a plain version and an image‑tiled version using existing histogram PNGs.

Produces:
  - `histogram_grid_FINE.png`
  - `histogram_grid_FINE_plain.png`
  - `histogram_grid_FINE_images.png`  
  (and COARSE equivalents)

Saves to: Directly into the TSC run folder.

Inputs:
  - TSC run folder containing:
    - `fine_pass1_manifest.csv` or `coarse_pass1_manifest.csv`
    - all `TSC_Values_* CSVs` referenced in the manifest
    - optional existing histogram PNGs (for the image‑tiled version)

7. Ranking Comparison Heatmaps
Replicates: The Top‑20 Overlap and Kendall‑τ heatmap outputs produced by the AE Ranking Comparison module in the main pipeline.

This standalone cell rebuilds the Top‑20 Overlap and Kendall‑τ (top‑20) heatmaps using the matrices stored in a previously generated `AE - Heatmap_data_*.json` file. It does not recompute rankings or correlations; instead, it redraws the heatmaps and applies updated title lines (τ, number of DOIs, API Query Terms, Query Relevance Terms). This is useful when retitling or regenerating AE heatmaps without rerunning the full AE module.

Produces:
  - `AE - Heatmap_overlap_{ts}_retitled.png`
  - `AE - Heatmap_kendall_{ts}_retitled.png`

Saves to: The same folder containing the `AE - Heatmap_data_*.json file`

Inputs:
  - A folder containing at least one `AE - Heatmap_data_*.json`
  - Optional overrides for API Query Terms and Query Relevance Terms

8. Replot Everything
Replicates: all plots from the main pipeline; only by re‑reading saved intermediate files.
It does not rerun any computation.

Saves to: the original simulation run folder

Inputs:
  - The folder for the original simulation run
