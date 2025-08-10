# Clinical Genomics Project: Whole Exome Sequence Analysis of Ewing Sarcoma Sample

## Overview
This project focuses on the analysis of Whole Exome Sequencing (WES) data from a tumor tissue sample of a 20-year-old male patient diagnosed with Ewing Sarcoma. The goal is to identify and annotate genetic variants within the protein-coding regions (exome) of the human genome to understand mutational impacts and potential clinical relevance.

Whole Exome Sequencing targets the exonic regions which harbor approximately 85% of disease-associated variants, making it an essential tool in cancer genomics and precision medicine.

---

## Project Details

### Experimental Design
- The project analyzes WES data retrieved from the Sequence Read Archive (SRA).
- Sample: BioSample ID `SAMN35991112`, SRA ID `SRS18070572`
- BioProject: PRJNA987736 titled *“Immunosurveillance shapes the emergence of neo-epitope landscapes of sarcomas, revealing prime targets for immunotherapy”*.
- Raw reads were downloaded from SRA, quality assessed, trimmed, aligned to human reference genome (hg38), and variants were called, filtered, and annotated.

### Sample Information
| Field                  | Value                                           |
|------------------------|-------------------------------------------------|
| Organism               | Homo sapiens                                    |
| Sample Source          | Tumor tissue of 20-year-old male patient       |
| Collection Date        | December 18, 2019                               |
| Location               | Pittsburgh, USA                                 |
| Sequencing Platform    | Illumina NovaSeq 6000                           |
| Read Count             | 22,414,250 paired-end reads                     |
| Base Count             | 6,724,275,000 bases                             |
| Library Strategy       | Whole Exome Sequencing (WXS)                    |
| Library Preparation    | DNA fragmentation by sonication, PCR enrichment|
| Data Package Version   | Human data package v1.0                          |

---

## Methodology

### 1. Data Collection
- Raw paired-end reads were downloaded from SRA using an automated shell script.
- Quality criteria for dataset selection: minimum file size ≥ 2GB and Phred quality score ≥ 30.

### 2. Quality Control (QC)
- **FastQC** was used to assess raw data quality, analyzing metrics such as:
  - Per base sequence quality
  - GC content
  - Adapter contamination
  - Sequence duplication levels
- QC results guide the necessity for further preprocessing.

### 3. Pre-Processing
- **Fastp** trimmed adapter sequences, removed low-quality and redundant reads.
- Post-trimming quality was verified again with FastQC.

### 4. Alignment
- Cleaned reads aligned to the human reference genome (hg38) using **BWA**.
- Reference genome was indexed with BWA prior to alignment.
- SAM files generated and converted to BAM format using **samtools**.

### 5. Post-Alignment Processing
- BAM files sorted (**GATK SortSam**) and duplicates marked/removed (**GATK MarkDuplicates**).
- Alignment metrics collected using **GATK CollectAlignmentSummaryMetrics**.
- BAM files indexed with **samtools index**.
- Base Quality Score Recalibration (BQSR) performed using **GATK**, leveraging known variant datasets to improve base call accuracy.

### 6. Variant Calling
- Variants called using **GATK HaplotypeCaller** generating VCF files.
  
### 7. Variant Filtering
- Hard filtering applied with **GATK VariantFiltration** using quality thresholds:

| Filter      | Condition          | Purpose                         |
|-------------|--------------------|--------------------------------|
| QD2         | QD < 2.0           | Low quality relative to depth   |
| QUAL30      | QUAL < 30.0        | Low variant confidence          |
| SOR3        | SOR > 3.0          | Strand bias                    |
| FS60        | FS > 60.0          | Fisher Strand bias             |
| MQ40        | MQ < 40.0          | Low mapping quality            |

- Variants passing all filters (“PASS”) were retained using **bcftools**.

### 8. Variant Quality Score Recalibration (VQSR)
- Applied machine learning based recalibration separately for INDELs and SNPs.
- Training datasets included Mills, Axiom Exome Plus, HapMap, Omni, and 1000 Genomes.
- Final high-confidence variants extracted and separated into SNPs and INDELs using **GATK SelectVariants**.

### 9. Variant Annotation
- **SnpEff** annotated filtered variants against the hg38 reference, predicting functional effects on genes.
- Further detailed biological context was obtained using **Variant Effect Predictor (VEP)** from Ensembl, providing consequence types, gene names, dbSNP IDs, and clinical significance.
- VEP results exported to Excel for convenient analysis.

---

## Tools and Software

| Step                  | Tool/Software                  |
|-----------------------|-------------------------------|
| Quality Control       | FastQC                        |
| Preprocessing         | Fastp                         |
| Alignment             | BWA, samtools                 |
| BAM Processing        | GATK (SortSam, MarkDuplicates, CollectAlignmentSummaryMetrics, BaseRecalibrator) |
| Variant Calling       | GATK HaplotypeCaller          |
| Variant Filtering     | GATK VariantFiltration, bcftools |
| Variant Recalibration | GATK VariantRecalibrator, ApplyVQSR |
| Annotation            | SnpEff, Variant Effect Predictor (VEP) |

---


## Data Structure

/data/ <br>
├── raw/ # Raw sequencing FASTQ files<br>
├── reference/ # Reference genome and annotation files (hg38)<br>
/results/<br>
├── qc_reports/ # FastQC reports<br>
├── trimmed_reads/ # Cleaned FASTQ files<br>
├── alignments/ # BAM and related files<br>
├── variants/ # VCF files (raw, filtered, recalibrated, annotated)<br>
└── annotation/ # SnpEff and VEP output files



---

#References

BioProject PRJNA987736: https://www.ncbi.nlm.nih.gov/bioproject/PRJNA987736

SRA Database: https://www.ncbi.nlm.nih.gov/sra

GATK Best Practices: https://gatk.broadinstitute.org/

SnpEff: http://snpeff.sourceforge.net/

Variant Effect Predictor (VEP): https://www.ensembl.org/info/docs/tools/vep/index.html
