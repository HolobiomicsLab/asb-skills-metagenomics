---
name: read-retention-benchmarking-across-tools
description: Use when comparing host read removal tools on the same human sequencing
  data (Illumina paired-end or ONT FASTQ) and you need a quantitative per-tool retained/removed
  read proportion table to benchmark decontamination stringency.
when_to_use_negative:
- The goal is microbial (non-human) read retention accuracy — use Hostile masked indexes
  (human-t2t-hla-argos985 or virus/phage-masked) rather than a human-removal benchmark.
- The input data is non-human host contamination (e.g. mouse) — use the mouse-mm39
  index workflow instead of the human-tool comparison.
- You need functional preservation of microbial content after decontamination rather
  than a measure of how much human sequence is removed — this benchmark only reports
  read counts, not downstream analysis quality.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Hostile
  role: Primary decontamination tool being benchmarked; removes reads aligning to
    the human-t2t-hla index using Bowtie2 for short reads and Minimap2 for long reads,
    emitting JSON logs with reads_in/reads_out
  repo: https://github.com/bede/hostile
- name: HRRT (sra-human-scrubber)
  role: Comparison baseline; k-mer-based tool that identifies human reads against
    an Eukaryota-derived k-mer database and masks them with N
  repo: https://github.com/ncbi/sra-human-scrubber
- name: Bowtie2
  role: Alignment engine used by Hostile for short-read decontamination (implicit
    --sensitive preset, -k 1, no secondary alignments)
- name: Minimap2
  role: Alignment engine used by Hostile for ONT long-read decontamination (-x map-ont
    preset, --secondary no)
- name: conda
  role: Installation route for Hostile and its non-Python dependencies (Bowtie2, Minimap2,
    Samtools, Bedtools)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad728
    title: 'Hostile: accurate decontamination of microbial host sequences'
  source_content_hash: sha256:ffa2fc82df3d142a39cca74c239f528642d601de7ccd5a5e81d454ec24c411d7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bede/hostile
  tools:
  - Bowtie2
  - HRRT
  - HRRT (sra-human-scrubber)
  - Hostile
  - Minimap2
  - conda
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# read-retention-benchmarking-across-tools

## Summary

A benchmarking procedure that compares how many human reads two host-decontamination tools retain on identical real human FASTQ datasets, e.g. Hostile versus the NCBI Human Read Removal Tool (HRRT), to quantify relative removal performance.

## When to use

Use when comparing host read removal tools on the same human sequencing data (Illumina paired-end or ONT FASTQ) and you need a quantitative per-tool retained/removed read proportion table to benchmark decontamination stringency.

## When NOT to use

- The goal is microbial (non-human) read retention accuracy — use Hostile masked indexes (human-t2t-hla-argos985 or virus/phage-masked) rather than a human-removal benchmark.
- The input data is non-human host contamination (e.g. mouse) — use the mouse-mm39 index workflow instead of the human-tool comparison.
- You need functional preservation of microbial content after decontamination rather than a measure of how much human sequence is removed — this benchmark only reports read counts, not downstream analysis quality.

## Inputs

- Real human paired-end Illumina FASTQ(.gz) files (e.g. 27 expanded 1000 Genomes Project 2x150bp genomes at 30x; NA12878 2x100bp at 10x)
- Real human ONT long-read FASTQ(.gz) file (e.g. NA12878)
- Pre-built Hostile human-t2t-hla index (fetched via hostile index fetch)
- HRRT v2.1.0 pre-built k-mer database (retrieved via init_db.sh from NCBI ftp)

## Outputs

- Per-tool, per-dataset reads_in and reads_out counts from Hostile JSON decontamination logs
- HRRT spot/read masked-or-removed counts from aligns_to run summaries
- A comparison table of retained/removed read proportions per tool and dataset (e.g. Hostile 0.132% vs HRRT 0.160% at 10x Illumina; 0.038% vs 0.037% on ONT)

## How to apply

Install Hostile from bioconda (`conda create -y -n hostile -c conda-forge -c bioconda hostile`) alongside HRRT v2.1.0, then cache the default human reference with `hostile index fetch` so the human-t2t-hla index (T2T-CHM13v2.0 + IPD-IMGT/HLA v3.51) is available. Run `hostile clean` on paired Illumina reads (Bowtie2 short-read mode, implicit --sensitive preset, -k 1, no secondary alignments) and `hostile clean --fastq1` on ONT long reads (Minimap2, -x map-ont, --secondary no), capturing reads_in/reads_out counts from the JSON decontamination logs. Run HRRT on the identical FASTQ files (its scrub.sh masks identified human reads with N by default). Compute the retained read proportion per tool per dataset from the logs and assemble a single comparison table; validate against the paper's benchmarks (e.g. Hostile 0.594% vs HRRT 0.729% retained on 30x Illumina 1000 Genomes samples, corresponding to Hostile removing ~23% more human reads).

## Related tools

- **Hostile** (Primary decontamination tool being benchmarked; removes reads aligning to the human-t2t-hla index using Bowtie2 for short reads and Minimap2 for long reads, emitting JSON logs with reads_in/reads_out) — https://github.com/bede/hostile
- **HRRT (sra-human-scrubber)** (Comparison baseline; k-mer-based tool that identifies human reads against an Eukaryota-derived k-mer database and masks them with N) — https://github.com/ncbi/sra-human-scrubber
- **Bowtie2** (Alignment engine used by Hostile for short-read decontamination (implicit --sensitive preset, -k 1, no secondary alignments))
- **Minimap2** (Alignment engine used by Hostile for ONT long-read decontamination (-x map-ont preset, --secondary no))
- **conda** (Installation route for Hostile and its non-Python dependencies (Bowtie2, Minimap2, Samtools, Bedtools))

## Examples

```
hostile clean --index human-t2t-hla --fastq1 sample_R1.fastq.gz --fastq2 sample_R2.fastq.gz --output cleaned/
```

## Evaluation signals

- Retention proportions reproduce the paper's benchmark values: Hostile 0.132% vs HRRT 0.160% retained on NA12878 10x Illumina; Hostile 0.594% vs HRRT 0.729% on 30x Illumina 1000 Genomes; Hostile 0.038% vs HRRT 0.037% on ONT.
- Mean Hostile human-read removal exceeds HRRT by ~23% on the 30x Illumina dataset (ratio of removed reads consistent with 0.594% vs 0.729% retention).
- Hostile removes ≥99.6% of real human reads across datasets (retention ≤0.4%).
- Hostile JSON logs report reads_out < reads_in for every human sample, and counts sum consistently with the retained-percentage table.
- HRRT run summaries report total spot/read counts and 'spot(s) masked or removed' matching the input file's read count.

## Limitations

- Hostile requires 4GB RAM for short reads (Bowtie2) and 13GB for long reads (Minimap2); HRRT by default scales to all available threads.
- HRRT masks identified human reads with N by default rather than removing them, so its output reads can be misleadingly 'retained' unless the -x (remove spots) option is used or masking is accounted for.
- HRRT's Eukaryota-derived k-mer database makes it aggressive about identifying human reads (includes k-mers common to primates and other taxa), which affects the comparison.
- Prior Hostile versions did not verify downloaded index checksums (issue #20) and could corrupt empty fastq.gz files when no reads remained after decontamination (issue #24) — use the latest release.

## Evidence

- [abstract] removes at least 99.6% of real human reads and retains at least 99.989% of simulated bacterial reads: "our approach removes at least 99.6% of real human reads and retains at least 99.989% of simulated bacterial reads"
- [abstract] Hostile removes 21%-23% more human short reads than existing tool: "Compared with an existing tool, Hostile removes 21%-23% more human short reads and 21-43 times fewer bacterial reads, typically in less time"
- [methods] hostile index fetch caches the default human-t2t-hla index: "To optionally download and cache the default index (`human-t2t-hla`) ahead of time, run `hostile index fetch`."
- [methods] conda installation command for Hostile: "conda create -y -n hostile -c conda-forge -c bioconda hostile"
- [methods] default index composition T2T-CHM13v2.0 plus HLA: "The default index `human-t2t-hla` comprises [T2T-CHM13v2.0](https://www.ncbi.nlm.nih.gov/assembly/11828891) and [IPD-IMGT/HLA](https://www.ebi.ac.uk/ipd/imgt/hla/) v3.51"
- [readme] HRRT masks human reads with N in fastq.clean output: [evidence span withheld — anchor status: external_doc]
- [readme] Hostile RAM requirements for Bowtie2 and Minimap2: "Hostile requires 4GB of RAM for decontaminating short reads (Bowtie2) and 13GB for long reads (Minimap2)"
