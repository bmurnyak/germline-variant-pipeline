# Germline Variant Calling Pipeline

A production-grade germline variant calling pipeline following GATK4 best practices, built with Snakemake for reproducibility and scalability.

**Author:** Balazs Murnyak, PhD  
**GitHub:** github.com/bmurnyak

## Pipeline Overview
FASTQ → FastQC → Trimmomatic → BWA-MEM → MarkDuplicates → BQSR → HaplotypeCaller → GenotypeGVCFs → VariantFiltration → MultiQC

## Tools and Versions

| Tool | Version | Purpose |
|------|---------|---------|
| Snakemake | 9.22.0 | Workflow management |
| BWA | 0.7.19 | Read alignment |
| GATK4 | 4.6.2.0 | Variant calling, BQSR, QC metrics |
| Trimmomatic | 0.40 | Adapter trimming |
| FastQC | 0.12.1 | Read quality control |
| MultiQC | 1.35 | QC report aggregation |
| samtools | 1.23.1 | BAM manipulation |
| bcftools | 1.23.1 | VCF manipulation |

## Key Results (NA12878 chr21 demo)

- **Reads:** 5,000,000 paired-end 150bp reads (~30x coverage)
- **Mapping rate:** 100%
- **Duplicate rate:** 0.035%
- **Variants called:** 213,510 total (212,644 SNPs, 866 indels)
- **PASS variants:** 209,684 (98.2% pass rate)
- **Reference:** hg38 chr21
- **Known sites:** GIAB NA12878 high-confidence variants (v3.3.2)

## Pipeline Steps

### 1. FastQC — Raw read quality control
Per-base quality scores, adapter contamination, GC content distribution.

### 2. Trimmomatic — Adapter trimming
Removes Illumina adapters, trims low-quality bases (LEADING:3, TRAILING:3, SLIDINGWINDOW:4:15, MINLEN:36).

### 3. BWA-MEM — Alignment
Aligns paired-end reads to hg38 reference with read group tags for GATK compatibility.

### 4. MarkDuplicates — PCR duplicate removal
Identifies and marks PCR duplicates to prevent variant calling inflation.

### 5. BQSR — Base Quality Score Recalibration
Corrects systematic errors in base quality scores using GIAB known variants as reference.

### 6. HaplotypeCaller — Variant calling
Calls SNPs and indels in GVCF mode for joint genotyping capability.

### 7. GenotypeGVCFs — Genotyping
Converts GVCF to final genotyped VCF.

### 8. VariantFiltration — Hard filtering
Applies GATK recommended hard filters for single-sample analysis:
- SNPs: QD<2, FS>60, MQ<40, MQRankSum<-12.5, ReadPosRankSum<-8
- Indels: QD<2, FS>200, ReadPosRankSum<-20

### 9. MultiQC — Aggregate QC report
Aggregates FastQC, Picard, and samtools metrics into a single HTML report.

## Repository Structure
germline-variant-pipeline/
|-- workflow/
|   |-- Snakefile              # Main pipeline workflow
|-- config/
|   |-- config.yaml            # Pipeline configuration
|-- data/
|   |-- raw/                   # Input FASTQ files
|   |-- reference/             # Reference genome and known sites
|-- results/
|   |-- qc/                    # Per-sample QC files
|   |-- aligned/               # BAM files
|   |-- variants/              # VCF files
|   |-- reports/               # MultiQC HTML report
|-- environment.yml            # Conda environment
|-- README.md

## Installation and Usage

### 1. Clone the repository
```bash
git clone git@github.com:bmurnyak/germline-variant-pipeline.git
cd germline-variant-pipeline
```

### 2. Create conda environment
```bash
conda env create -f environment.yml
conda activate variant-calling
```

### 3. Configure samples
Edit `config/config.yaml` to point to your FASTQ files and reference genome.

### 4. Run the pipeline
```bash
snakemake \
    --snakefile workflow/Snakefile \
    --configfile config/config.yaml \
    --cores 4
```

### 5. View results
Open `results/reports/multiqc_report.html` in a browser for the aggregated QC report.

## Clinical Relevance

This pipeline implements the GATK4 best practices workflow used in clinical genomics laboratories for germline variant calling. Key applications include:

- Hereditary cancer panel sequencing (BRCA1/2, Lynch syndrome genes)
- Whole exome sequencing for rare disease diagnosis
- Pharmacogenomics variant calling

## Notes on BQSR

Base Quality Score Recalibration requires known variant sites. This pipeline uses the GIAB NA12878 high-confidence variant set (v3.3.2) as known sites. In production with clinical samples, use dbSNP and Mills gold standard indels as known sites.
