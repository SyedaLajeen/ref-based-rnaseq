# Reference-Based RNA-Seq Data Analysis — Galaxy

**Organism:** *Drosophila melanogaster*  
**Experiment:** *pasilla* gene knockdown (RNAi) vs untreated — differential gene expression  
**Samples:** GSM461177 (untreated, paired-end) vs GSM461180 (treated, paired-end)  
**Platform:** [Galaxy Europe](https://usegalaxy.eu)  
**Tutorial:** [GTN — Reference-based RNA-Seq data analysis](https://training.galaxyproject.org/training-material/topics/transcriptomics/tutorials/ref-based/tutorial.html)  
**Raw Data DOI:** [10.5281/zenodo.1185122](https://doi.org/10.5281/zenodo.1185122)

---

## Biological Background

This project reproduces the analysis from Brooks et al. (2011), which investigated
the role of the *pasilla* (*ps*) gene — an RNA-binding splicing factor — in
*Drosophila melanogaster*. RNAi was used to deplete *pasilla*, and paired-end
RNA-seq was performed on treated (depleted) and untreated cells. The goal was to
identify genes whose expression is regulated by *pasilla* at the transcriptomic level.

---

## Experimental Design

| GEO Accession | Condition       | Read Type  | Role            |
|---------------|-----------------|------------|-----------------|
| GSM461177     | Untreated       | Paired-end | Control sample  |
| GSM461180     | Treated (RNAi)  | Paired-end | Knockdown sample|

Full dataset includes 7 samples (GSM461176–GSM461182).
This tutorial uses 2 samples for demonstration of the complete pipeline.

---

## Raw Input Data

FASTQ files are **not committed** to this repository (each ~1.5 GB).  
Download them directly from Zenodo:

| File | Sample | Read |
|---|---|---|
| [GSM461177_1.fastqsanger](https://zenodo.org/record/1185122/files/GSM461177_1.fastqsanger) | Untreated | Forward (R1) |
| [GSM461177_2.fastqsanger](https://zenodo.org/record/1185122/files/GSM461177_2.fastqsanger) | Untreated | Reverse (R2) |
| [GSM461180_1.fastqsanger](https://zenodo.org/record/1185122/files/GSM461180_1.fastqsanger) | Treated   | Forward (R1) |
| [GSM461180_2.fastqsanger](https://zenodo.org/record/1185122/files/GSM461180_2.fastqsanger) | Treated   | Reverse (R2) |

**Genome annotation used:**  
`Drosophila_melanogaster.BDGP6.32.109_UCSC.gtf.gz`  
Available from Galaxy Shared Data Library:  
`Shared Data → Data Libraries → GTN-Material → Transcriptomics → ref-based → zenodo.1185122`

---

## Analysis Pipeline
Raw FASTQ Reads (GSM461177 + GSM461180)
│
▼
┌─────────────────────┐
│  Step 1: QC         │  Falco + MultiQC
└─────────────────────┘
│
▼
┌─────────────────────┐
│  Step 2: Trimming   │  Cutadapt + MultiQC
└─────────────────────┘
│
▼
┌─────────────────────┐
│  Step 3: Mapping    │  RNA STAR (dm6) + MultiQC
└─────────────────────┘
│
▼
┌─────────────────────┐
│  Step 4: Post-map   │  RSeQC + Samtools idxstats + MultiQC
│         QC          │
└─────────────────────┘
│
▼
┌─────────────────────┐
│  Step 5: Counting   │  featureCounts
└─────────────────────┘
│
▼
┌─────────────────────┐
│  Step 6: DESeq2     │  Differential Expression Analysis
└─────────────────────┘
│
▼
┌─────────────────────┐
│  Step 7: Visualize  │  Heatmap2 + Volcano Plot
└─────────────────────┘
│
▼
┌─────────────────────┐
│  Step 8: Enrichment │  goseq (GO) + Pathview (KEGG)
└─────────────────────┘

---

## Methods Summary

### Step 1 — Quality Control
**Tool:** Falco + MultiQC  
**Output:** `results/01_fastqc/`

- GSM461177: 10.6 million paired sequences, good quality throughout
- GSM461180: 12.3 million paired sequences; reverse reads show quality drop at 3' end
- High duplication rates in both samples (expected and normal for RNA-seq)

### Step 2 — Adapter & Quality Trimming
**Tool:** Cutadapt + MultiQC  
**Output:** `results/02_trimming/`

- Removed low-quality bases from 3' read ends
- Discarded reads below minimum length threshold
- GSM461177: 5,072,810 bp trimmed (forward) / 8,648,619 bp (reverse)
- GSM461180: 10,224,537 bp (forward) / 51,746,850 bp (reverse)
- Large trimming in GSM461180 reverse reads was expected given poor QC at 3' end

### Step 3 — Splice-Aware Read Mapping
**Tool:** RNA STAR v2.7.11b + MultiQC  
**Reference:** *D. melanogaster* dm6 genome (built-in Galaxy index)  
**Output:** `results/03_mapping/`

- Paired-end mapping with GTF splice junction annotation
- Junction overhang: 36 bp (read length − 1)
- ~80% of reads mapped uniquely per sample (above the 70% quality threshold)
- Low multimapper rate (<10%) — good mapping specificity

### Step 4 — Post-Mapping Quality Control
**Tools:** RSeQC (Gene Body Coverage + Read Distribution) + Samtools idxstats + MultiQC  
**Output:** `results/03_mapping/`

- Even 5' to 3' gene body coverage — no RNA degradation or library prep bias
- >80% reads mapped to exonic regions (expected for mRNA-seq)
- ~2% intronic reads, ~5% intergenic reads — confirms RNA-seq data quality

### Step 5 — Gene-Level Read Counting
**Tool:** featureCounts  
**Output:** `results/04_featurecounts/`

- Counts reads overlapping annotated genes using the GTF file
- Top expressed gene: FBgn0284245 (~128,740 counts in GSM461177 / ~127,400 in GSM461180)

### Step 6 — Differential Expression Analysis
**Tool:** DESeq2  
**Output:** `results/05_deseq2/`

- Normalizes for sequencing depth and library composition
- Applies negative binomial model with dispersion estimation
- Benjamini-Hochberg correction for multiple testing
- Significance threshold: adjusted p-value (padj) < 0.05, |log2FC| > 1
- Outputs: full results table, annotated results table, diagnostic plots (PCA, MA, dispersion)

### Step 7 — Visualization
**Tools:** Heatmap2, Volcano Plot  
**Output:** `results/06_visualization/`

- Heatmap of top 130 differentially expressed genes (z-score normalized)
  - Blue = relatively low expression, Red = relatively high expression
- Volcano plot: log2FC on X-axis vs −log10(padj) on Y-axis
  - Grey = not significant, Red = significantly upregulated, Blue = significantly downregulated
- KEGG pathway diagrams showing affected molecular pathways

### Step 8 — Functional Enrichment Analysis
**Tools:** goseq + Pathview  
**Output:** `results/07_goseq/` and `results/06_visualization/`

- goseq corrects for gene-length bias in DE gene lists
- Identifies enriched GO biological processes, molecular functions, cellular components
- Pathview maps DE genes onto KEGG pathway diagrams

---

## Repository Structure
galaxy-ref-based-rnaseq/
│
├── README.md
│
├── data/
│   ├── raw_reads/              ← FASTQ files not committed (see Zenodo URLs above)
│   └── annotation/             ← GTF file not committed (see Zenodo)
│
└── results/
│
├── 01_fastqc/
│   └── multiqc_fastqc_falco_report/     ← MultiQC HTML + data (Falco QC)
│
├── 02_trimming/
│   └── multiqc_cutadapt_report/         ← MultiQC HTML + data (Cutadapt)
│
├── 03_mapping/
│   ├── multiqc_star_mapping_report/     ← MultiQC HTML + data (STAR)
│   └── multiqc_rseqc_report/            ← MultiQC HTML + data (RSeQC)
│
├── 04_featurecounts/
│   └── featureCounts_all_samples/       ← Count tables per sample
│
├── 05_deseq2/
│   ├── DESeq2_results_treated_vs_untreated.tabular
│   ├── DESeq2_annotated_results.tabular
│   └── DESeq2_plots_PCA_MA_dispersion.pdf
│
├── 06_visualization/
│   ├── heatmap2_top130_DE_genes.pdf
│   ├── heatmap2_top130_DE_genes_v2.pdf
│   └── KEGG_pathview_pathways/          ← KEGG pathway diagrams
│
└── 07_goseq/
└── goseq_top_DE_genes.pdf           ← GO enrichment results

---

## How to Reproduce This Analysis

1. Go to [usegalaxy.eu](https://usegalaxy.eu) and log in (create account if needed)
2. Click **"+"** in the History panel → name your history `Reference-based RNA-seq`
3. Click the **upload icon** (top right of tool panel) → **"Paste/Fetch Data"**
4. Paste each Zenodo FASTQ URL one by one → click **Start** → **Close**
5. Import GTF annotation from Galaxy Shared Data Library:
   `Shared Data → Data Libraries → GTN-Material → Transcriptomics → ref-based → zenodo.1185122`
6. Follow every step of the tutorial at:
   https://training.galaxyproject.org/training-material/topics/transcriptomics/tutorials/ref-based/tutorial.html
7. Download results from Galaxy History and place in the `results/` subfolders as shown above

---

## Tools Used

| Tool | Version | Step | Purpose |
|---|---|---|---|
| Falco | - | QC | FastQC-compatible quality assessment |
| MultiQC | 1.27+galaxy4 | All steps | Aggregate QC reports |
| Cutadapt | - | Trimming | Adapter and quality trimming |
| RNA STAR | 2.7.11b+galaxy0 | Mapping | Splice-aware read alignment |
| Samtools idxstats | 2.0.7 | Post-map QC | Per-chromosome read counts |
| RSeQC | 2.6.x | Post-map QC | Gene body coverage + read distribution |
| featureCounts | - | Counting | Gene-level read counting |
| DESeq2 | - | DE analysis | Differential expression |
| Heatmap2 | - | Visualization | Heatmap of top DE genes |
| Volcano Plot | - | Visualization | Statistical significance plot |
| goseq | - | Enrichment | GO term enrichment (length-corrected) |
| Pathview | - | Enrichment | KEGG pathway visualization |

---

