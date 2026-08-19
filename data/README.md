# Data guide

This directory contains the raw and processed data used by the STAR Protocols GBM computational workflow.

Large sequencing datasets and generated R objects are intentionally **not stored directly in the GitHub repository**. Instead, users should obtain the required public datasets from their original sources and place them in the directory structure described below.

The workflow uses three main data components:

1. Mouse single-cell RNA-seq
2. Mouse spatial transcriptomics
3. Human TCGA-GBM bulk RNA-seq and clinical data

Raw count data should be retained whenever possible because downstream methods such as inferCNV and CARD require untransformed counts.

---

# Directory structure

```text
data/
│
├── README.md
│
├── raw/
│   ├── single_cell_raw/
│   ├── spatial_raw/
│   └── bulk/
│
└── processed/
    ├── single_cell/
    ├── spatial/
    └── bulk/
```

Most processed objects and intermediate files are generated automatically by the analysis modules.

---

# 1. Single-cell RNA-seq data

## Source

The primary mouse GBM single-cell RNA-seq dataset used in this workflow is available through NCBI GEO.

**GEO accession:** `GSE298688`

The study generated Chromium Next GEM Single Cell 3′ v3.1 libraries and processed the sequencing data against the mouse genome before downstream Seurat analysis.

The STAR Protocols workflow begins from the publicly available 10x-style count matrices rather than from FASTQ files.

## Samples

The workflow uses three samples:

| Sample | GEO sample accession | Experimental group |
| ------ | -------------------- | ------------------ |
| C2_S2  | GSM9021009           | Control            |
| N1_S3  | GSM9021010           | Niacin             |
| N2_S4  | GSM9021011           | Niacin             |

Place the files in:

```text
data/raw/single_cell_raw/
```

Module 01 expects the following filenames:

```text
GSM9021009_C2_S2_matrix.mtx.gz
GSM9021009_C2_S2_features.tsv.gz
GSM9021009_C2_S2_barcodes.tsv.gz

GSM9021010_N1_S3_matrix.mtx.gz
GSM9021010_N1_S3_features.tsv.gz
GSM9021010_N1_S3_barcodes.tsv.gz

GSM9021011_N2_S4_matrix.mtx.gz
GSM9021011_N2_S4_features.tsv.gz
GSM9021011_N2_S4_barcodes.tsv.gz
```

The directory should therefore look like:

```text
data/raw/single_cell_raw/
├── GSM9021009_C2_S2_matrix.mtx.gz
├── GSM9021009_C2_S2_features.tsv.gz
├── GSM9021009_C2_S2_barcodes.tsv.gz
├── GSM9021010_N1_S3_matrix.mtx.gz
├── GSM9021010_N1_S3_features.tsv.gz
├── GSM9021010_N1_S3_barcodes.tsv.gz
├── GSM9021011_N2_S4_matrix.mtx.gz
├── GSM9021011_N2_S4_features.tsv.gz
└── GSM9021011_N2_S4_barcodes.tsv.gz
```

## Modules using these data

The single-cell data enter the workflow in:

```text
Module 01 — Assemble single-cell RNA-seq Seurat object
```

The resulting object is subsequently used by Modules 02–05 and as the reference dataset for CARD in Module 13.

## Generated processed data

Module 01 generates sample-level and merged Seurat objects under:

```text
data/processed/single_cell/
```

These processed objects are workflow outputs and do not need to be downloaded separately when the analysis is run from raw data.

---

# 2. Spatial transcriptomics data

## Source

The mouse GBM spatial transcriptomics dataset associated with the study is available through NCBI GEO.

**GEO accession:** `GSE298689`

The associated mouse GBM spatial dataset is also linked to:

**NCBI BioProject:** `PRJNA914489`

The workflow analyzes four spatial tissue sections representing normal tissue, vehicle-treated tumors, and niacin-treated tumor tissue. The protocol preserves sample identity throughout integration and downstream analysis.

## Samples

The spatial workflow uses the following sample identities:

| Analysis label   | Spatial image/sample identifier | Biological group |
| ---------------- | ------------------------------- | ---------------- |
| Normal_Tissue    | Naive_A9                        | Normal tissue    |
| Vehicle_1        | Vehicle1_C5                     | Vehicle          |
| Vehicle_2        | Vehicle2_D5                     | Vehicle          |
| Niacin_Treatment | Niacin2_B5                      | Niacin           |

Place the raw Visium datasets under:

```text
data/raw/spatial_raw/
```

Each spatial sample should retain the standard 10x Genomics Visium directory components required to reconstruct a spatial Seurat object, including:

```text
filtered_feature_bc_matrix/
spatial/
```

or the equivalent files required by `Seurat::Load10X_Spatial()`.

The required spatial inputs include the feature-barcode matrix together with tissue images and spot-coordinate information.

A representative structure is:

```text
data/raw/spatial_raw/
│
├── Naive_A9/
│   ├── filtered_feature_bc_matrix/
│   └── spatial/
│
├── Vehicle1_C5/
│   ├── filtered_feature_bc_matrix/
│   └── spatial/
│
├── Vehicle2_D5/
│   ├── filtered_feature_bc_matrix/
│   └── spatial/
│
└── Niacin2_B5/
    ├── filtered_feature_bc_matrix/
    └── spatial/
```

Users should preserve the original 10x spatial image and barcode-coordinate files because the workflow validates that image objects remain correctly associated with their spatial barcodes after merging and integration.

## Modules using these data

The spatial datasets enter the workflow in:

```text
Module 06 — Spatial transcriptomics assembly and quality control
```

They are subsequently used by Modules 07–13.

The spatial workflow includes:

```text
06 — Assembly and QC
07 — Normalization and clustering
08 — Cluster differential expression
09 — inferCNV
10 — Cancer-spot classification
11 — SingleR annotation
12 — CellChat
13 — CARD deconvolution
```

## Raw counts

Do not discard the raw spatial count matrices after normalization.

Raw counts are required for downstream analyses including:

```text
inferCNV
CARD
```

The accompanying STAR Protocols workflow explicitly retains raw count information for these analyses.

---

# 3. SingleR reference data

SingleR annotation is performed using reference expression datasets provided through the Bioconductor `celldex` ecosystem.

The original study used a mouse bulk RNA-seq reference through CellDex/SingleR for cell-type assignment.

These reference datasets are obtained through R and therefore do not need to be manually stored in:

```text
data/raw/
```

The required versions of:

```text
SingleR
celldex
SingleCellExperiment
SummarizedExperiment
```

are captured in the repository `renv.lock`.

Users adapting the workflow to another species or biological system should select an appropriate SingleR reference rather than assuming that the mouse reference used here is universally applicable.

---

# 4. inferCNV supporting data

inferCNV requires:

* raw spatial expression counts
* spot/group annotations
* chromosome and genomic-position information for genes
* biologically appropriate reference groups

The workflow constructs the required inferCNV inputs during Modules 09 and 10.

Gene-position information should correspond to the species and gene identifiers used in the spatial dataset. The STAR Protocols manuscript identifies gene-position information as a required inferCNV input.

Generated inferCNV files and objects are workflow outputs and should not normally be committed to GitHub.

---

# 5. CARD single-cell reference

CARD spatial deconvolution uses the processed and SingleR-annotated mouse GBM scRNA-seq dataset generated by the single-cell workflow.

The reference is therefore produced internally by:

```text
Module 04 — Single-cell RNA-seq cell-type annotation with SingleR
```

and consumed by:

```text
Module 13 — Spatial cell-type deconvolution with CARD
```

The original study similarly used the mouse GBM single-cell dataset as a reference for CARD spatial deconvolution.

Users running the complete workflow from raw data do not need to manually supply a separate CARD reference object.

---

# 6. TCGA-GBM data

## Source

Module 14 performs independent human GBM validation using TCGA-GBM bulk RNA-seq and clinical data.

The original study used TCGA GBM mRNA-seq data obtained through UCSC Xena.

The current STAR Protocols workflow retrieves these datasets programmatically using:

```r
UCSCXenaTools
```

Therefore, users generally do **not** need to manually download TCGA files before running Module 14.

## Data retrieved

The workflow accesses TCGA-GBM gene-expression and clinical information from the UCSC Xena TCGA hub.

Representative datasets include:

```text
TCGA.GBM.sampleMap/HiSeqV2
TCGA.GBM.sampleMap/GBM_clinicalMatrix
```

The UCSC Xena workflow identified the HiSeqV2 expression matrix and GBM clinical matrix as the corresponding expression and phenotype datasets.

Additional survival data may also be retrieved when required by Module 14.

## Storage location

Downloaded TCGA files are written under the repository data structure, for example:

```text
data/raw/bulk/GBM/
```

or the specific repository-relative TCGA directory defined in Module 14.

Processed TCGA data are written under:

```text
data/processed/bulk/
```

Results are written under:

```text
results/bulk/
```

and figures under:

```text
figures/bulk/
```

Because TCGA data are downloaded programmatically, the downloaded files themselves should generally remain excluded from Git tracking.

---

# Raw versus processed data

The repository distinguishes between source data and generated intermediate objects.

## Raw data

```text
data/raw/
```

Contains externally acquired datasets required to begin the workflow.

These files are not committed to the GitHub repository because they can be large and are available from public repositories.

## Processed data

```text
data/processed/
```

Contains Seurat objects, intermediate matrices, annotation objects, and other generated workflow checkpoints.

These files can be regenerated from the raw inputs and are generally excluded from GitHub.

---

# GitHub data policy

Large biological datasets should not be committed directly to this repository.

The repository `.gitignore` excludes generated or downloaded contents under:

```text
data/raw/
data/processed/
results/
figures/
```

while allowing documentation files and placeholder files to remain tracked.

Users should acquire the required public data independently and follow the accession-specific data-use terms associated with the original repositories.

---

# Expected starting data layout

Before running the complete workflow, the repository should approximately contain:

```text
data/
│
├── README.md
│
└── raw/
    │
    ├── single_cell_raw/
    │   ├── GSM9021009_C2_S2_matrix.mtx.gz
    │   ├── GSM9021009_C2_S2_features.tsv.gz
    │   ├── GSM9021009_C2_S2_barcodes.tsv.gz
    │   ├── GSM9021010_N1_S3_matrix.mtx.gz
    │   ├── GSM9021010_N1_S3_features.tsv.gz
    │   ├── GSM9021010_N1_S3_barcodes.tsv.gz
    │   ├── GSM9021011_N2_S4_matrix.mtx.gz
    │   ├── GSM9021011_N2_S4_features.tsv.gz
    │   └── GSM9021011_N2_S4_barcodes.tsv.gz
    │
    └── spatial_raw/
        ├── Naive_A9/
        ├── Vehicle1_C5/
        ├── Vehicle2_D5/
        └── Niacin2_B5/
```

TCGA-GBM data are obtained automatically by Module 14 and therefore do not need to be present before beginning the single-cell or spatial workflows.

---

# Data provenance

The experimental single-cell and spatial datasets used in this workflow originate from the study:

**Mirzaei et al.**
*Spatial single-cell profiling identifies protein kinase Cδ-expressing microglia with anti-tumor function in glioblastoma.*
*iScience* 29, 114281 (2026).

Relevant public accessions include:

```text
Single-cell RNA-seq:
GSE298688

Spatial transcriptomics:
GSE298689

Spatial mouse GBM BioProject:
PRJNA914489

Human validation:
TCGA-GBM via UCSC Xena
```

---

# Reproducibility note

The analysis scripts use project-relative paths through the R package:

```r
here
```

Users should therefore preserve the repository directory structure rather than editing the scripts to contain machine-specific absolute paths.

The required R package versions are recorded in:

```text
renv.lock
```

and additional environment information is provided in:

```text
environment/sessionInfo.txt
environment/system_requirements.txt
```

