# TCR Analysis Pipeline

A comprehensive bioinformatics pipeline for extracting and analyzing T-cell receptor (TCR) sequences from large-scale RNA sequencing data (100M+ reads). This pipeline integrates multiple state-of-the-art tools including MiXCR and TRUST4 for robust TCR repertoire analysis.

## Overview

This pipeline processes paired-end RNA-seq data to identify, assemble, and analyze TCR sequences. It's designed to handle large datasets efficiently through parallel processing, chunking strategies, and optimized I/O operations.

### Key Features

- **Multi-tool integration**: MiXCR and TRUST4 for comprehensive TCR analysis
- **Scalable processing**: Handles 100M+ read datasets through intelligent chunking
- **Parallel execution**: Multi-threading and multi-processing optimization
- **Memory efficiency**: Local data staging and streaming processing
- **Comparative analysis**: Cross-tool validation and overlap visualization
- **Caching system**: Avoids redundant computations through smart caching

## Pipeline Architecture

```
RNA-seq FASTQ files
    ↓
Data Staging & Chunking
    ↓
Parallel TCR Analysis
├── MiXCR Pipeline
│   ├── Alignment (V/D/J/C genes)
│   ├── Assembly (clonotype reconstruction)
│   └── Export (CDR3 sequences)
└── TRUST4 Pipeline
    ├── Read extraction
    ├── Contig assembly
    └── TCR annotation
    ↓
Results Integration & Visualization
    ↓
Comparative Analysis & Reports
```

## Requirements

### System Requirements
- **CPU**: 4+ cores recommended (pipeline auto-detects and uses all available cores)
- **RAM**: 16GB+ recommended for large datasets
- **Storage**: ~50GB free space for intermediate files
- **OS**: Linux (tested on Ubuntu/Google Colab)

### Software Dependencies

#### Core Tools
- **MiXCR v4.6.0+**: TCR/BCR repertoire analysis
- **TRUST4**: De novo TCR assembly from RNA-seq
- **SeqKit v2.8.2+**: FASTQ manipulation and splitting

#### Python Libraries
```bash
pandas
numpy
matplotlib
seaborn
scikit-learn
matplotlib-venn
biopython
multiprocessing
concurrent.futures
```

#### System Tools
- `wget`, `unzip`, `gzip/zcat`
- Java 8+ (for MiXCR)
- Standard Unix tools (`make`, `gcc` for TRUST4 compilation)

## Installation

### Option 1: Google Colab (Recommended)
The pipeline is optimized for Google Colab and includes automated installation:

```python
# The notebook handles all installations automatically
# Just run the setup cells sequentially
```

### Option 2: Local Installation

#### 1. Install MiXCR
```bash
wget https://github.com/milaboratory/mixcr/releases/download/v4.6.0/mixcr-4.6.0.zip
unzip mixcr-4.6.0.zip
export PATH=$PATH:/path/to/mixcr-4.6.0
```

#### 2. Install TRUST4
```bash
git clone https://github.com/liulab-dfci/TRUST4.git
cd TRUST4
make
export PATH=$PATH:/path/to/TRUST4
```

#### 3. Install SeqKit
```bash
wget https://github.com/shenwei356/seqkit/releases/download/v2.8.2/seqkit_linux_amd64.tar.gz
tar -xzf seqkit_linux_amd64.tar.gz
export PATH=$PATH:/path/to/seqkit
```

#### 4. Download Reference Files
```bash
# Human IMGT reference
wget https://github.com/liulab-dfci/TRUST4/raw/master/vdjc_db/human_IMGT+C.fa

# Human genome TCR/BCR loci
wget https://github.com/liulab-dfci/TRUST4/raw/master/ref_genome/hg38_bcrtcr.fa
```

## Usage

### Data Preparation

1. **Download RNA-seq data**: Use the provided shell scripts to download example datasets:
   ```bash
   # For Patient 1
   bash ena-file-download-read_run-SAMN03431245-fastq_ftp-20250830-1946.sh
   
   # For Patient 2  
   bash ena-file-download-read_run-SAMN03431261-fastq_ftp-20250830-2013.sh
   
   # For Patient 3
   bash ena-file-download-selected-files-20250830-2016.sh
   ```

2. **Merge technical replicates** (if applicable):
   ```bash
   zcat sample_rep1_R1.fastq.gz sample_rep2_R1.fastq.gz | gzip > sample_merged_R1.fastq.gz
   zcat sample_rep1_R2.fastq.gz sample_rep2_R2.fastq.gz | gzip > sample_merged_R2.fastq.gz
   ```

### Running the Pipeline

#### Basic Usage
```python
# Define your data structure
patients = ["Patient1", "Patient2", "Patient3"]
patient_files = {
    "Patient1": ("Patient1_merged_1.fastq.gz", "Patient1_merged_2.fastq.gz"),
    "Patient2": ("Patient2_merged_1.fastq.gz", "Patient2_merged_2.fastq.gz"),
    "Patient3": ("Patient3_merged_1.fastq.gz", "Patient3_merged_2.fastq.gz")
}

# Run the complete pipeline
run_tcr_pipeline(patients, patient_files, base_dir, results_dir)
```

#### Advanced Configuration
```python
# Custom chunking for very large files
run_mixcr_pipeline(
    fastq1="sample_R1.fastq.gz",
    fastq2="sample_R2.fastq.gz", 
    work_dir="./results",
    sample="sample_name",
    chunk_reads=2_000_000,  # 2M reads per chunk
    total_threads=16,       # Use 16 threads
    preset="rna-seq",       # MiXCR preset
    species="hsa"           # Human
)
```

## Pipeline Components

### 1. MiXCR Analysis
- **Alignment**: Maps reads to V/D/J/C gene references
- **Assembly**: Reconstructs full-length TCR sequences
- **Export**: Generates clonotype tables with CDR3 sequences

### 2. TRUST4 Analysis  
- **Extraction**: Identifies TCR-related reads
- **Assembly**: De novo assembly of TCR contigs
- **Annotation**: Maps assembled sequences to TCR genes

### 3. Comparative Analysis
- **Overlap detection**: Identifies shared clonotypes between tools
- **Visualization**: Venn diagrams showing tool agreement
- **Quality metrics**: Comparative statistics and validation

## Output Files

### MiXCR Outputs
- `{sample}_mixcr.clns`: Binary clonotype file
- `{sample}_mixcr.clones.txt`: Tab-delimited clonotype table
- `{sample}_mixcr_assemble_report.txt`: Assembly statistics

### TRUST4 Outputs
- `{sample}_trust4_report.tsv`: TCR annotation results
- `{sample}_trust4.fa`: Assembled TCR contigs
- `{sample}_trust4_merged.fa`: Merged results from all chunks

### Analysis Outputs
- Venn diagrams showing tool overlap
- Comparative statistics tables
- Quality control reports

## Performance Optimization

### Memory Management
- **Local staging**: Copies data from network drives to local disk
- **Streaming processing**: Processes compressed files directly
- **Chunk-based processing**: Splits large files for parallel processing

### Parallelization Strategy
- **Thread budgeting**: Automatically distributes CPU cores across chunks
- **Multi-level parallelism**: Patient-level and chunk-level parallel processing
- **Smart scheduling**: Balances thread allocation to avoid oversubscription

### Typical Performance
| Dataset Size | Processing Time | Memory Usage |
|-------------|----------------|--------------|
| 50M reads   | ~2-3 hours     | ~8-12GB      |
| 100M reads  | ~4-6 hours     | ~12-16GB     |
| 200M reads  | ~8-12 hours    | ~16-24GB     |

*Times are for Google Colab Pro with high-RAM runtime*

## Troubleshooting

### Common Issues

1. **Memory errors with large files**
   ```python
   # Reduce chunk size
   chunk_reads = 1_000_000  # Instead of 2_000_000
   ```

2. **MiXCR license issues**
   ```bash
   export MI_LICENSE='your-license-key'
   ```

3. **TRUST4 compilation errors**
   ```bash
   # Install build dependencies
   sudo apt-get update
   sudo apt-get install build-essential
   ```

4. **Threading issues**
   ```python
   # Reduce parallel jobs
   max_workers = min(4, os.cpu_count())
   ```

### Performance Tips

1. **Use SSD storage** for intermediate files when possible
2. **Monitor memory usage** - reduce chunk size if needed
3. **Use local drives** instead of network storage for processing
4. **Cache intermediate results** to avoid recomputation

## Data Requirements

### Input Format
- **FASTQ files**: Paired-end reads (R1/R2)
- **Compression**: gzip compressed (.fastq.gz)
- **Quality**: Illumina quality scores (Phred+33)

### Sample Types
- Bulk RNA-seq from immune cells
- Single-cell RNA-seq (with appropriate preprocessing)
- Targeted TCR sequencing data

## Validation and Quality Control

### Cross-tool Validation
The pipeline compares results between MiXCR and TRUST4 to:
- Identify high-confidence clonotypes (detected by both tools)
- Flag tool-specific artifacts
- Assess overall data quality

### Quality Metrics
- **Read alignment rates**: Percentage of reads mapping to TCR loci
- **Clonotype diversity**: Shannon entropy and other diversity indices
- **Tool concordance**: Overlap statistics between analysis methods

## Example Results

### Clonotype Table Format
```
cloneId    count    frequency    CDR3.aa              V.name    J.name
1          1500     0.15         CASSLEETQYF          TRBV5-1   TRBJ2-5
2          890      0.089        CASSLAPGATNEKLFF     TRBV6-5   TRBJ1-4
```

### Statistical Summary
```
Total unique clonotypes: 15,432
MiXCR-specific: 2,891
TRUST4-specific: 3,156
Shared (high confidence): 9,385
Tool concordance: 60.8%
```

## Citation and References

If you use this pipeline in your research, please cite:

**MiXCR**: Bolotin et al. (2015). MiXCR: software for comprehensive adaptive immunity profiling. *Nature Methods* 12, 380-381.

**TRUST4**: Song et al. (2021). TRUST4: immune repertoire reconstruction from bulk and single-cell RNA-seq data. *Nature Methods* 18, 627-630.

## License

This pipeline is provided under the MIT License. Individual tools (MiXCR, TRUST4) have their own licensing terms - please check their respective repositories.

## Contributing

Contributions are welcome! Please:
1. Fork the repository
2. Create a feature branch
3. Submit a pull request with detailed description

## Support

For issues and questions:
- Create an issue in the repository
- Check the troubleshooting section above
- Refer to the individual tool documentation:
  - [MiXCR Documentation](https://docs.milaboratories.com/)
  - [TRUST4 GitHub](https://github.com/liulab-dfci/TRUST4)

---

**Note**: This pipeline is designed for research use. For clinical applications, additional validation and regulatory compliance may be required.