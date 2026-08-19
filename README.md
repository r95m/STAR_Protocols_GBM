# Reproducible single-cell and spatial transcriptomic analysis of glioblastoma

This repository contains the analysis workflow accompanying a **STAR Protocols** manuscript describing reproducible single-cell RNA-sequencing (scRNA-seq), spatial transcriptomics, cell-type annotation, cell-cell communication, copy-number inference, spatial deconvolution, and external TCGA validation in glioblastoma (GBM).

The workflow was developed to support the computational analyses associated with the study of **PRKCD-associated biology and niacin treatment in glioblastoma**, with an emphasis on reproducibility, modular analysis, and transparent intermediate outputs.

The repository is organized as 14 numbered R Markdown modules comprising sequential and branching analysis workflows.. Individual modules generate checkpoint objects, result tables, and figures that can be inspected independently or used as inputs for subsequent analyses.

---

## Workflow overview

The analysis is divided into three major components:

1. **Single-cell RNA-seq analysis**
2. **Spatial transcriptomics analysis**
3. **TCGA-GBM external validation**

### Single-cell RNA-seq workflow

```text
01 — scRNA-seq assembly
        │
        ▼
02 — Quality control and filtering
        │
        ▼
03 — Normalization, clustering, and differential expression
        │
        ▼
04 — SingleR cell-type annotation
        │
        ├──────────────► 05 — CellChat
        │
        └──────────────► scRNA-seq reference for CARD
```

### Spatial transcriptomics workflow

```text
06 — Spatial assembly and quality control
        │
        ▼
07 — Normalization and clustering
        │
        ├──────────────► 08 — Spatial cluster differential expression
        │
        ▼
09 — inferCNV
        │
        ▼
10 — Cancer-spot classification
        │
        ▼
11 — Spatial SingleR annotation
        │
        ├──────────────► 12 — Spatial CellChat
        │
        └──────────────► 13 — CARD deconvolution
                                  ▲
                                  │
                        Module 04 scRNA-seq
                            reference
```

### External validation

```text
14 — TCGA-GBM validation
```

---

# Analysis modules

| Module | Analysis                             | Primary input          | Primary output                        |
| ------ | ------------------------------------ | ---------------------- | ------------------------------------- |
| 01     | scRNA-seq assembly                   | Raw 10x count matrices | Merged raw scRNA-seq Seurat object    |
| 02     | scRNA-seq QC and filtering           | Module 01              | QC-filtered Seurat object             |
| 03     | Normalization, clustering, and DEA   | Module 02              | Processed and clustered Seurat object |
| 04     | SingleR cell-type annotation         | Module 03              | Annotated scRNA-seq reference         |
| 05     | scRNA-seq CellChat                   | Module 04              | CellChat communication results        |
| 06     | Spatial assembly and QC              | Raw Visium data        | QC-filtered spatial Seurat object     |
| 07     | Spatial normalization and clustering | Module 06              | Processed spatial Seurat object       |
| 08     | Spatial cluster DEA                  | Module 07              | Spatial cluster marker results        |
| 09     | Spatial inferCNV                     | Module 07              | inferCNV results                      |
| 10     | Cancer-spot classification           | Module 09              | CNV-annotated spatial Seurat object   |
| 11     | Spatial SingleR                      | Module 10              | Cell-type-annotated spatial object    |
| 12     | Spatial CellChat                     | Module 11              | Spatial communication results         |
| 13     | CARD deconvolution                   | Modules 04 and 11      | Module 04 scRNA-seq reference + Module 11 spatial object  |
| 14     | TCGA-GBM validation                  | TCGA-GBM               | External validation results           |

---

# Repository structure

The repository uses project-relative paths through the R package `here`. Analyses should therefore be run from within the cloned repository without modifying absolute filesystem paths.

```text
STAR_Protocols_GBM/
│
├── README.md
├── LICENSE
├── .gitignore
├── renv.lock
│
├── scripts/
│   ├── 01_seurat_assembly.Rmd
│   ├── 02_seurat_qc_filtering.Rmd
│   ├── 03_seurat_normalization_clustering_dea.Rmd
│   ├── 04_seurat_singleR.Rmd
│   ├── 05_seurat_cellchat.Rmd
│   ├── 06_spatial_assembly_qc.Rmd
│   ├── 07_spatial_normalization_clustering.Rmd
│   ├── 08_spatial_cluster_dea.Rmd
│   ├── 09_spatial_infercnv.Rmd
│   ├── 10_spatial_cancer_spots.Rmd
│   ├── 11_spatial_singleR.Rmd
│   ├── 12_spatial_cellchat.Rmd
│   ├── 13_spatial_card_deconv.Rmd
│   └── 14_tcga_validation.Rmd
│
├── data/
│   ├── README.md
│   ├── raw/
│   │   ├── single_cell_raw/
│   │   ├── spatial_raw/
│   │   └── bulk/
│   │
│   └── processed/
│       ├── single_cell/
│       ├── spatial/
│       └── bulk/
│
├── results/
│   ├── single_cell/
│   ├── spatial/
│   └── bulk/
│
├── figures/
│   ├── single_cell/
│   ├── spatial/
│   └── bulk/
│
├── STAR_Protocols_GBM.Rproj
```

Some directories are created automatically by individual analysis modules when required.

---

# Data availability

## Single-cell RNA-seq

The scRNA-seq workflow uses publicly available data from:

**GEO accession: GSE298688**

The workflow analyzes three samples:

| Sample | GEO accession | Experimental group |
| ------ | ------------- | ------------------ |
| C2_S2  | GSM9021009    | Control            |
| N1_S3  | GSM9021010    | Niacin             |
| N2_S4  | GSM9021011    | Niacin             |

Raw 10x Genomics matrices should be placed in:

```text
data/raw/single_cell_raw/
```

Module 01 expects the corresponding matrix, feature, and barcode files for each sample.

---

## Spatial transcriptomics

The spatial workflow analyzes four Visium datasets representing normal tissue, vehicle-treated tumors, and niacin-treated tumors.

Raw Visium data should be placed within:

```text
data/raw/spatial_raw/
```

The expected directory organization and filenames are documented within Module 06 and the accompanying `data/README.md`.

---

## TCGA-GBM

Human glioblastoma validation is performed using **TCGA-GBM** bulk transcriptomic and clinical data.

Module 14 retrieves the required TCGA datasets through `UCSCXenaTools` and stores downloaded data within the repository data structure.

TCGA-derived outputs are organized under:

```text
data/processed/bulk/
results/bulk/
figures/bulk/
```

---

# Software requirements

The workflow is implemented primarily in **R**.

Major packages used across the analysis include:

* Seurat
* SeuratObject
* SingleR
* celldex
* Harmony
* CellChat
* inferCNV
* CARD
* MuSiC
* Matrix
* dplyr
* tidyr
* tibble
* ggplot2
* patchwork
* pheatmap
* UCSCXenaTools
* limma
* fgsea
* msigdbr
* immunedeconv
* survival
* survminer
* here

Additional package dependencies are documented within the individual modules.

---

# Reproducible environment

Package versions can affect the behavior of several components of this workflow, particularly Seurat, CellChat, inferCNV, SingleR, and CARD.

The repository therefore uses an `renv` environment to record the R package versions used for the analysis.

After cloning the repository, install `renv` if required:

```r
install.packages("renv")
```

Then restore the project environment:

```r
renv::restore()
```

```r
renv::status()
```

Package restoration may take several minutes because the workflow includes packages from CRAN, Bioconductor, and GitHub. 
Users should run renv::status() after restoration to confirm that the project library is synchronized with renv.lock.

# Running the workflow

Clone the repository and open STAR_Protocols_GBM.Rproj in RStudio. This ensures that the repository root is used as the working project directory.

Because paths are constructed using `here::here()`, users should not need to modify machine-specific paths.

The complete workflow can be reproduced by running the R Markdown modules in numerical order:

```text
01 → 02 → 03 → 04 → 05
06 → 07 → 08 → 09 → 10 → 11 → 12 → 13
14
```

Not every module is strictly dependent on the immediately preceding module. For example:

* Module 05 branches from the Module 04 scRNA-seq object.
* Module 08 performs cluster-level spatial differential expression from the processed spatial object.
* Module 09 begins the inferCNV branch from the processed spatial dataset.
* Module 13 combines the annotated scRNA-seq reference with the spatial dataset for CARD deconvolution.
* Module 14 represents an independent TCGA-GBM validation workflow.

Each R Markdown file contains a detailed header specifying its purpose, required input files, generated outputs, and important analysis notes.

---

# Output organization

Intermediate datasets are written primarily to:

```text
data/processed/
```

Statistical and analytical results are written to:

```text
results/
```

Figures are written to:

```text
figures/
```

This separation allows processed data objects, quantitative results, and visual outputs to be managed independently.

Large generated files may be excluded from the GitHub repository and regenerated by running the corresponding analysis module.

---

# Analysis overview

## scRNA-seq

The single-cell workflow includes:

* raw count-matrix assembly
* quality-control assessment and filtering
* normalization
* dimensionality reduction
* graph-based clustering
* cluster differential expression
* SingleR reference-based annotation
* marker-based annotation assessment
* CellChat communication inference

The annotated scRNA-seq dataset also serves as the reference for CARD spatial deconvolution.

---

## Spatial transcriptomics

The spatial workflow includes:

* Visium data assembly
* quality-control assessment and filtering
* normalization
* dimensionality reduction
* Harmony-based integration
* graph-based clustering
* spatial cluster differential expression
* inferCNV analysis
* CNV-burden estimation
* malignant-enriched spot classification
* SingleR spot annotation
* CellChat communication inference
* CARD cell-type deconvolution

Raw count data are retained where required by downstream methods.

---

## TCGA validation

The TCGA-GBM workflow provides independent human validation of PRKCD-associated molecular patterns.

Analyses include:

* TCGA-GBM expression-data acquisition and processing
* clinical-data integration
* PRKCD-associated differential expression
* pathway enrichment analysis
* immune deconvolution
* immune-signature correlation analysis
* overall survival analysis
* progression-free survival analysis

---

# Interpretation of computational predictions

Several analyses in this repository generate computational estimates rather than direct experimental measurements.

### SingleR

SingleR annotations in scRNA-seq data represent reference-based predicted cell identities.

For spatial transcriptomics, Visium spots may contain multiple cells. Spatial SingleR labels should therefore be interpreted as the predominant transcriptional identity associated with a spot rather than definitive single-cell classifications.

### inferCNV

inferCNV detects large-scale expression patterns consistent with copy-number alterations.

The resulting CNV burden and malignant-enriched classifications should not be interpreted as direct DNA-based confirmation of genomic copy-number state.

### CARD

CARD estimates cell-type proportions within spatial transcriptomic spots using a single-cell reference. These proportions are model-based deconvolution estimates rather than direct measurements of cellular abundance.

### CellChat

CellChat identifies candidate ligand-receptor communication relationships based on gene-expression patterns.

Predicted interactions should not be interpreted as direct evidence of physical interaction or causal signaling between cell populations.

---

# Reproducibility

The workflow was designed around several reproducibility principles:

* project-relative paths using `here`
* explicit input and output directories
* numbered analysis modules
* saved intermediate checkpoint objects
* fixed random seeds where applicable
* exported result tables
* saved publication-quality figures
* documented analysis parameters
* recorded software versions
* separation of raw data, processed data, results, and figures

Users adapting the workflow to independent datasets should document any modifications to filtering thresholds, reference datasets, clustering parameters, or other analysis-specific settings.

---

# Citation

If you use this workflow, please cite the accompanying STAR Protocols article.

**STAR Protocols citation:**
*To be added upon publication.*

The workflow builds upon the associated glioblastoma study:

**Associated publication:**
*Full citation to be added.*

Users should additionally cite the original publications for the computational tools used in their analyses, including Seurat, SingleR, Harmony, CellChat, inferCNV, CARD, and other relevant software.

---

# License

Licensing information for the code in this repository is provided in the `LICENSE` file.

---

# Contact

Questions regarding the workflow, implementation, or reproducibility of the analyses can be submitted through the GitHub repository's issue tracker.

---

## Repository status

This repository accompanies an analysis workflow prepared for **STAR Protocols**.

The code is organized into modular, reproducible analysis steps to facilitate inspection, reuse, and adaptation to related single-cell and spatial transcriptomic studies.

