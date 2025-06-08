# Idea 9: Bioinformatics Tool Automation & Scripting

## Table of Contents
- [Concept Overview](#concept-overview)
- [Target Domain(s)](#target-domains)
- [Detailed Description](#detailed-description)
  - [Core Functionality](#core-functionality)
  - [Example Workflow: Germline Variant Calling](#example-workflow-germline-variant-calling)
  - [Supported Tools (Examples)](#supported-tools-examples)
  - [Data Format Integration](#data-format-integration)
  - [Managing Intermediate Files & Dependencies](#managing-intermediate-files--dependencies)
- [How Codex CLI/Agent Features Are Leveraged](#how-codex-cliagent-features-are-leveraged)
- [Potential Benefits](#potential-benefits)
- [Considerations & Challenges](#considerations--challenges)

## Concept Overview
Codex assists bioinformaticians by generating scripts and automating command-line calls to common bioinformatics tools. It can help construct complex pipelines, manage intermediate files, and ensure correct parameter usage for tools involved in tasks like sequence alignment, variant calling, and genomic data analysis.

## Target Domain(s)
-   Bioinformatics
-   Genomics
-   Computational Biology
-   Medical Genomics

## Detailed Description

### Core Functionality
1.  **Interpret Bioinformatic Task**: Codex parses user prompts that describe a bioinformatic objective or a specific tool they want to use.
    -   Examples: "Align FASTQ reads to a reference genome using BWA-MEM," "Sort and index a BAM file with Samtools," "Call variants using GATK HaplotypeCaller."
2.  **Tool Identification and Parameterization**:
    -   Identifies the specific bioinformatics tool(s) required (e.g., BWA, Samtools, GATK, Picard, BLAST).
    -   Determines necessary input files (FASTQ, FASTA, BAM, VCF, reference genomes, index files).
    -   Extracts or infers parameters for the tools (e.g., thread count, memory allocation, specific algorithm options).
3.  **Script/Command Generation**:
    -   Generates shell scripts or individual commands to execute the bioinformatic tools.
    -   Can construct multi-step pipelines by chaining commands.
    -   Ensures correct syntax and common parameter usage for the specified tools.
4.  **File Management**:
    -   Helps define input and output file names and paths.
    -   Can suggest conventional naming for intermediate and final output files.
5.  **Dependency Awareness (Basic)**:
    -   Recognizes common dependencies between steps (e.g., a BAM file must be sorted before variant calling).
    -   Can suggest or include commands for creating necessary index files (e.g., `samtools index`, `bwa index`).

### Example Workflow: Germline Variant Calling (Simplified)

*User Prompt*: `"codex, create a script to call germline variants from 'tumor_sample.bam' using hg38 reference. First, mark duplicates with Picard, then use GATK HaplotypeCaller. Output VCF as 'tumor_variants.vcf'."`
(Assumes reference genome files like `hg38.fa`, `.dict`, `.fai` are available and paths known or provided).

*Codex Actions*:
1.  Identifies tools: Picard MarkDuplicates, GATK HaplotypeCaller.
2.  Identifies inputs: `tumor_sample.bam`, `hg38.fa` (and its indices).
3.  Identifies output: `tumor_variants.vcf`.
4.  Generates `variant_calling_pipeline.sh` (conceptual example):
    ```bash
    #!/bin/bash
    set -e # Exit on error

    # Input parameters (Codex might prompt for these or use defaults)
    INPUT_BAM="tumor_sample.bam"
    REFERENCE_GENOME="path/to/hg38.fa" # User needs to provide this path
    OUTPUT_VCF="tumor_variants.vcf"
    PICARD_JAR="path/to/picard.jar" # User needs to provide this path
    GATK_JAR="path/to/gatk.jar"     # User needs to provide this path

    DEDUP_BAM="${INPUT_BAM%.bam}.dedup.bam"
    METRICS_FILE="${INPUT_BAM%.bam}.dedup_metrics.txt"

    echo "Step 1: Mark Duplicates with Picard"
    java -jar $PICARD_JAR MarkDuplicates \
        INPUT=$INPUT_BAM \
        OUTPUT=$DEDUP_BAM \
        METRICS_FILE=$METRICS_FILE \
        ASSUME_SORT_ORDER=coordinate

    echo "Step 2: Index the deduplicated BAM"
    samtools index $DEDUP_BAM

    echo "Step 3: Call Variants with GATK HaplotypeCaller"
    java -jar $GATK_JAR HaplotypeCaller \
        -R $REFERENCE_GENOME \
        -I $DEDUP_BAM \
        -O $OUTPUT_VCF

    echo "Pipeline complete. Variants written to $OUTPUT_VCF"
    ```
5.  Codex would highlight that paths to JARs and reference files need to be correctly set by the user. It might also suggest `samtools index` if not explicitly mentioned for the input BAM if needed by Picard.

### Supported Tools (Examples)
-   **Aligners**: BWA, Bowtie2, STAR
-   **SAM/BAM/CRAM Manipulation**: Samtools, Picard
-   **Variant Calling**: GATK, FreeBayes, bcftools
-   **Sequence Analysis**: BLAST suite, Bedtools, MEME suite
-   **Quality Control**: FastQC, MultiQC

### Data Format Integration
Codex should be aware of common bioinformatics file formats and how they are used by different tools:
-   **FASTA/FASTQ**: Input for aligners, sequence databases.
-   **SAM/BAM/CRAM**: Output of aligners, input for variant callers and manipulation tools.
-   **VCF/BCF**: Output of variant callers.
-   **BED/GFF/GTF**: Genomic interval files.
-   **Index Files**: `.fai` (FASTA index), `.dict` (sequence dictionary), `.bai`/`.csi` (BAM/CRAM index), BWA indexes, etc. Codex could suggest commands to generate these if they are missing.

### Managing Intermediate Files & Dependencies
-   **Naming Conventions**: Suggest logical names for intermediate files (e.g., `sample.sorted.bam`, `sample.dedup.bam`).
-   **Pipeline Structure**: For multi-step workflows, ensure the output of one step is correctly fed as input to the next.
-   **Error Checking**: Generated scripts should ideally include basic error checking (`set -e` in shell scripts) or use Python's `subprocess` with error checking.

## How Codex CLI/Agent Features Are Leveraged
-   **Natural Language Processing**: To understand complex bioinformatics tasks, tool names, file types, and parameters.
-   **Code/Script Generation**: Creating shell scripts or Python scripts that orchestrate CLI tool execution.
-   **File System Interaction**: Managing input/output file paths, potentially creating directories for outputs.
-   **Command Execution**: Running the generated scripts or individual tool commands (with user approval).
-   **`AGENTS.md` Customization**:
    -   Store paths to commonly used executables (e.g., `PICARD_JAR`, `GATK_PATH`).
    -   Define standard parameters for specific tools or analyses (e.g., default adapter sequences for trimming, preferred settings for an aligner).
    -   Store paths to reference genomes and annotation files.
    -   Define shortcodes for common pipelines (e.g., `"codex run my_variant_pipeline on sampleX.fastq"`).
-   **Iterative Refinement**: Users could ask Codex to modify a generated script, add a step, or change parameters.

## Potential Benefits
-   **Reduces Tedium**: Automates the generation of often lengthy and repetitive command lines for bioinformatics tools.
-   **Lowers Error Rate**: Helps avoid typos and incorrect parameter usage, which are common in complex CLI tools.
-   **Improves Reproducibility**: Generated scripts serve as a precise record of the analysis performed.
-   **Accessibility**: Makes complex tools more accessible to researchers who may not be CLI experts.
-   **Pipeline Construction**: Simplifies the process of building and connecting multi-step bioinformatics workflows.
-   **Best Practice Reinforcement**: Can be guided by `AGENTS.md` to suggest or use parameters aligned with best practices for certain analyses.

## Considerations & Challenges
-   **Tool Versioning and Compatibility**: Bioinformatics tools are frequently updated, and parameters can change. Codex's knowledge would need to be kept current, or it would need to rely heavily on parsing `--help` output (Idea 2).
-   **Environment Management**: Many bioinformatics tools have complex dependencies. Users are typically expected to manage these with Conda, Docker, or environment modules. Codex would primarily script tool execution, assuming the tools are callable in the environment.
-   **Reference Data Management**: Paths to large reference genomes, index files, and annotation databases are critical and user-specific. Codex would need a robust way for users to specify these (e.g., via prompts, `AGENTS.md`, or environment variables).
-   **Computational Resources**: Bioinformatics tasks can be computationally intensive (CPU, memory, disk space). Codex generates scripts; the user is responsible for running them in an appropriate environment (e.g., HPC cluster, cloud). Codex could potentially generate job submission scripts for common schedulers (e.g., Slurm, SGE) as an advanced feature.
-   **Complexity of Biological Questions**: While Codex can automate tool usage, the interpretation of results and the design of biologically meaningful analyses remain firmly with the user.
-   **Error Interpretation**: Errors from bioinformatics tools can be cryptic. While Codex can display them, providing intelligent suggestions for fixes would be an advanced capability.

*Timestamp: {TIMESTAMP}*
*Version: {VERSION}*
