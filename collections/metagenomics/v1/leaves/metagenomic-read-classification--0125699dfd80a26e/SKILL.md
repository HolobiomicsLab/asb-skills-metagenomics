---
name: metagenomic-read-classification
description: Use when classifying high-throughput sequencing reads (Illumina FASTQ,
  single- or paired-end) from metagenomic whole-genome sequencing to NCBI taxa, especially
  when reference genomes are incomplete or distant strains must still be classified.
when_to_use_negative:
- Reads of interest originate from non-protein-coding genomic regions — protein-level
  classification cannot classify them (Kaiju's stated disadvantage)
- The exact sequenced strains are present in the reference database — nucleotide-level
  k-mer classifiers will be more sensitive, as seen in the HiSeq and MiSeq mock-community
  data (Kraken 78.0% genus sensitivity vs Kaiju Greedy-5 73.3%)
- Input is already protein sequence without the -p option being considered, or a genus-level
  assignment is required for genera with no reference genomes — classification is
  impossible for such genera
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: Kaiju
  role: 'Primary classifier: translates reads into six frames and finds maximum (in-)exact
    protein matches via BWT backwards search in MEM or Greedy modes'
  repo: https://github.com/bioinformatics-centre/kaiju
- name: Kraken v0.10.4b
  role: k-mer nucleotide baseline classifier (default mode, k=31) used for comparison
    and merging of outputs
- name: Clark v1.1.3
  role: k-mer nucleotide baseline classifier (default mode, k=31) used in the runtime/memory
    benchmark
- name: kaiju2table
  role: Summarizes Kaiju output files at genus/phylum ranks requiring nodes.dmp and
    names.dmp
  repo: https://github.com/bioinformatics-centre/kaiju
- name: Krona
  role: Interactive visualization of classification results
  repo: https://github.com/marbl/Krona/wiki/KronaTools
- name: ART
  role: Simulates HTS reads from genomes for the genome-exclusion sensitivity benchmark
provenance:
  source_task_ids:
  - task_002
  - task_004
  - task_003
  source_papers:
  - doi: 10.1038/ncomms11257
    title: Fast and sensitive taxonomic classification for metagenomics with Kaiju
  source_content_hash: sha256:bf36d26bbc29569287b30c23af479fe37aadeb02f11e0055cdfcd9bb0cd6b138
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bioinformatics-centre/kaiju
  tools:
  - ART
  - Clark
  - Clark v1.1.3
  - Kaiju
  - Kraken
  - Kraken v0.10.4b
  - Krona
  - R
  - kaiju-mergeOutputs
  - kaiju2table
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# metagenomic-read-classification

## Summary

Taxonomic classification of metagenomic sequencing reads by translating them into six reading frames and finding maximum (in-)exact protein-level matches via a Burrows-Wheeler transform FM-index (Kaiju), benchmarked against k-mer nucleotide classifiers like Kraken and Clark.

## When to use

Use when classifying high-throughput sequencing reads (Illumina FASTQ, single- or paired-end) from metagenomic whole-genome sequencing to NCBI taxa, especially when reference genomes are incomplete or distant strains must still be classified.

## When NOT to use

- Reads of interest originate from non-protein-coding genomic regions — protein-level classification cannot classify them (Kaiju's stated disadvantage)
- The exact sequenced strains are present in the reference database — nucleotide-level k-mer classifiers will be more sensitive, as seen in the HiSeq and MiSeq mock-community data (Kraken 78.0% genus sensitivity vs Kaiju Greedy-5 73.3%)
- Input is already protein sequence without the -p option being considered, or a genus-level assignment is required for genera with no reference genomes — classification is impossible for such genera

## Inputs

- FASTQ or FASTA read files (Illumina single- or paired-end, possibly gzip-compressed)
- Reference protein database FASTA with NCBI taxon-ID headers (e.g. NCBI RefSeq complete genomes, NR microbial subset)
- NCBI taxonomy files nodes.dmp and names.dmp
- Kaiju FM-index (.fmi) built from the reference proteins

## Outputs

- Kaiju output file (kaiju.out) with per-read classification status (C/U), read name, NCBI taxon ID, and match score/length
- Merged classification file (combined.out) when combining Kaiju and Kraken outputs with kaiju-mergeOutputs
- Rank-level summary tables (e.g. metagenome_classification_summary.tsv, benchmark_suppl_table2.tsv) via kaiju2table
- Throughput (reads per second) and peak memory measurements per tool/mode

## How to apply

Build a Kaiju index (.fmi) from reference protein sequences with kaiju-makedb (or kaiju-mkbwt + kaiju-mkfmi for a custom FASTA whose headers are numeric NCBI taxon IDs), keeping nodes.dmp and names.dmp for taxonomy. Run kaiju with -t nodes.dmp -f index.fmi -i input.fastq -o kaiju.out using multiple threads (-z 25); choose the MEM mode (-a mem) for maximum exact matches or Greedy mode (default 3 mismatches, adjustable with -e up to 5) for higher sensitivity, keeping SEG low-complexity filtering enabled (-x) and Greedy thresholds at m=11, s=65, E=0.01. For paired-end data supply both files with -i and -j; for many samples use kaiju-multi with comma-separated lists. Compare performance against Kraken v0.10.4b and Clark v1.1.3 run in their default modes with k=31, and optionally merge outputs with kaiju-mergeOutputs resolving conflicts by LCA, then summarize with kaiju2table or visualize with Krona.

## Related tools

- **Kaiju** (Primary classifier: translates reads into six frames and finds maximum (in-)exact protein matches via BWT backwards search in MEM or Greedy modes) — https://github.com/bioinformatics-centre/kaiju
- **Kraken v0.10.4b** (k-mer nucleotide baseline classifier (default mode, k=31) used for comparison and merging of outputs)
- **Clark v1.1.3** (k-mer nucleotide baseline classifier (default mode, k=31) used in the runtime/memory benchmark)
- **kaiju2table** (Summarizes Kaiju output files at genus/phylum ranks requiring nodes.dmp and names.dmp) — https://github.com/bioinformatics-centre/kaiju
- **Krona** (Interactive visualization of classification results) — https://github.com/marbl/Krona/wiki/KronaTools
- **ART** (Simulates HTS reads from genomes for the genome-exclusion sensitivity benchmark)

## Examples

```
kaiju -z 25 -t nodes.dmp -f kaiju_db.fmi -i reads.fastq -a mem -x -o kaiju.out
```

## Evaluation signals

- Merged Kaiju+Kraken classification should fall within the reported 24–73% of reads across the 10 real metagenome samples (e.g. 24.7% desert soil to 73.1% bioreactor sediment)
- Kaiju should classify on average about twice as many reads as Kraken on real metagenomic data (MEM mode 13.1–48.8% more, Greedy-5 up to 56.6% more)
- Genus-level sensitivity/precision for the HiSeq mock community should match Supplementary Table 2 (Kaiju Greedy-5: 73.3%/94.4%; Kraken: 78.0%/99.2%) within rounding tolerance
- Peak memory for Kaiju should stay below 6 GB on the benchmark database, versus 72 GB for Kraken and 65–78 GB for Clark
- Output files must contain one line per read with C/U status and a valid NCBI taxon ID contained in nodes.dmp; SEG filtering (-x) should remain enabled to avoid false positives from low-complexity regions

## Limitations

- Protein-level classification cannot classify reads from non-protein-coding genomic regions
- Kaiju is less sensitive than nucleotide-level classifiers when the sequenced strains are in the reference database (HiSeq/MiSeq mock communities)
- Genus-level classification is impossible for genera with no available reference genomes
- Allowing substitutions in the backwards search causes an exponential runtime increase with the number of allowed mismatches; Greedy mode therefore restricts substitutions to match ends
- SEG filtering is always recommended to avoid false positive taxon assignments caused by spurious matches from simple repeat patterns

## Evidence

- [discussion] Protein-level classification cannot classify non-coding reads: "The obvious disadvantage of protein-level sequence classification is the inability to classify reads originating from non-protein-coding genomic regions."
- [readme] Default run modes and thresholds: "The default run mode is **Greedy** with three allowed mismatches. The number of allowed mismatches can be changed using option `-e`."
- [readme] SEG filtering rationale: "SEG filtering is always recommended in order to avoid false positive taxon assignments that are caused by spurious matches due to simple repeat patterns or other sequencing noise."
- [discussion] Real metagenome comparison result: "In our set of 10 randomly selected real metagenomic data sets, Kaiju classifies on average twice as many reads as Kraken"
- [discussion] Combined output range: "By combining Kaiju's and Kraken's output, between 24 and 73% of reads can be classified across the various samples"
