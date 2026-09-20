---
name: benchmark-figure-reproduction
description: Use when reproducing a taxonomic-classifier benchmark with genome exclusion,
  or regenerating sensitivity/precision comparison figures for reads stratified by
  number of reference genomes per genus.
when_to_use_negative:
- 'When the sequenced strain is present in the reference database: Kaiju is less sensitive
  than nucleotide-level classifiers, which can assess the entire genome (as seen in
  the HiSeq and MiSeq data sets)'
- When the analysis goal is classifying real metagenomic samples rather than measuring
  classifier sensitivity/precision in a controlled exclusion benchmark
- When target genera have fewer than 2 or more than 10 reference genomes, or no reference
  genomes at all — genus-level classification is impossible for genera with no available
  reference genomes and the 882-genome stratification only covers 242 genera with
  2-10 genomes
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: Kaiju
  role: Protein-level classifier under test; indexes excluded-genome protein references
    with kaiju-mkbwt/kaiju-mkfmi and classifies reads in Greedy (3 mismatches, m=11,
    s=65, E=0.01) and MEM modes
  repo: https://github.com/bioinformatics-centre/kaiju
- name: Kraken v0.10.4b
  role: k-mer-based comparator classifier run in default mode with k=31 for highest
    precision
- name: Clark v1.1.3
  role: k-mer-based comparator classifier run in default mode with k=31 for highest
    precision
- name: ART
  role: Simulates five sets of HTS reads (Illumina SE100/SE250, PE250, Roche/454)
    from each whole genome including plasmids
- name: ggplot2
  role: R package used to produce the benchmark figures matching Fig. 1, Fig. 2, and
    Supplementary Figures 1-3
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/ncomms11257
    title: Fast and sensitive taxonomic classification for metagenomics with Kaiju
  source_content_hash: sha256:cd505f472f190f25b66da2257640122609a5a3cf1a3505d14d3ce7a4bbce9cc6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bioinformatics-centre/kaiju
  tools:
  - ART
  - Clark
  - Clark v1.1.3
  - Kaiju
  - Kraken v0.10.4b
  - R
  - ggplot2
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# benchmark-figure-reproduction

## Summary

Reproduces the Kaiju genome exclusion benchmark on 882 genomes: ART-simulated reads (Illumina SE100/SE250, PE250, Roche/454) are classified against source-genome-excluded protein references with Kaiju (Greedy and MEM), Kraken v0.10.4b, and Clark v1.1.3, then genus/phylum sensitivity and precision are aggregated and plotted as ggplot2 figures matching Fig. 1, Fig. 2, and Supplementary Figures 1-3.

## When to use

Use when reproducing a taxonomic-classifier benchmark with genome exclusion, or regenerating sensitivity/precision comparison figures for reads stratified by number of reference genomes per genus.

## When NOT to use

- When the sequenced strain is present in the reference database: Kaiju is less sensitive than nucleotide-level classifiers, which can assess the entire genome (as seen in the HiSeq and MiSeq data sets)
- When the analysis goal is classifying real metagenomic samples rather than measuring classifier sensitivity/precision in a controlled exclusion benchmark
- When target genera have fewer than 2 or more than 10 reference genomes, or no reference genomes at all — genus-level classification is impossible for genera with no available reference genomes and the 882-genome stratification only covers 242 genera with 2-10 genomes

## Inputs

- Complete bacterial and archaeal genome assemblies with annotated proteins and plasmid sequences (NCBI RefSeq snapshot, 16 December 2014)
- Per-genome reference protein FASTA excluding the source genome (headers as NCBI taxon IDs, e.g. >1_1358)
- NCBI taxonomy files nodes.dmp and names.dmp
- Simulated high-throughput sequencing reads in FASTQ format (Illumina SE100, SE250, PE250, Roche/454)

## Outputs

- Per-read classification tables for Kaiju (Greedy, MEM), Kraken v0.10.4b, and Clark v1.1.3
- Genus- and phylum-level sensitivity and precision summary tables stratified by genomes-per-genus category
- ggplot2 figures matching the layout of Fig. 1, Fig. 2, and Supplementary Figures 1-3 (e.g., Kaiju Greedy-5 ~29% vs Clark k=31 ~16% genus sensitivity on SE100 reads from 2-genome genera)

## How to apply

For each of the 882 benchmark genomes (drawn from 242 genera with at least 2 and at most 10 genomes, filtered from 2,724 complete genomes with annotated proteins), simulate five read sets from the whole genome including plasmids using ART (Illumina single-end 100 nt, single-end 250 nt, paired-end 250 nt, and Roche/454). Build a Kaiju protein index per genome that excludes the source genome's proteins via kaiju-mkbwt followed by kaiju-mkfmi, then classify reads in Greedy mode with three allowed mismatches, minimum match length m=11, minimum score s=65, and E-value threshold E=0.01, and in MEM mode with -a mem. Run comparators Kraken v0.10.4b and Clark v1.1.3 in their default modes with k=31 for highest precision. Aggregate per-read calls into genus- and phylum-level sensitivity (fraction of reads assigned to the correct genus/phylum) and precision (fraction correct among classified), stratified by the genomes-per-genus category. Produce summary tables and ggplot2 figures in R matching the layout of Fig. 1, Fig. 2, and Supplementary Figures 1-3.

## Related tools

- **Kaiju** (Protein-level classifier under test; indexes excluded-genome protein references with kaiju-mkbwt/kaiju-mkfmi and classifies reads in Greedy (3 mismatches, m=11, s=65, E=0.01) and MEM modes) — https://github.com/bioinformatics-centre/kaiju
- **Kraken v0.10.4b** (k-mer-based comparator classifier run in default mode with k=31 for highest precision)
- **Clark v1.1.3** (k-mer-based comparator classifier run in default mode with k=31 for highest precision)
- **ART** (Simulates five sets of HTS reads (Illumina SE100/SE250, PE250, Roche/454) from each whole genome including plasmids)
- **ggplot2** (R package used to produce the benchmark figures matching Fig. 1, Fig. 2, and Supplementary Figures 1-3)

## Examples

```
kaiju-mkbwt -n 5 -a ACDEFGHIKLMNPQRSTVWY -o proteins proteins.faa; kaiju-mkfmi proteins
```

## Evaluation signals

- Computed genus-level sensitivity for SE100 reads from 2-genome genera must match the reported values: Kaiju Greedy-5 ~29% (highest) versus Clark k=31 ~16% (lowest)
- Kaiju must consistently show much higher sensitivity with only little precision loss compared with Kraken and Clark across all five simulated read types on the 882-genome benchmark
- The exclusion setup must be verified: each per-genome reference database contains no proteins from the source genome before classification
- Figures must reproduce the layout of Fig. 1, Fig. 2, and Supplementary Figures 1-3, with sensitivity/precision stratified by genomes-per-genus category over 242 genera
- The custom Kaiju index build must succeed with kaiju-mkbwt (alphabet ACDEFGHIKLMNPQRSTVWY) followed by kaiju-mkfmi, producing the .fmi file used for classification

## Limitations

- Protein-level classification cannot classify reads from non-protein-coding genomic regions
- Kaiju is less sensitive than nucleotide-level classifiers when the sequenced strain is present in the reference database
- Genus-level classification of shotgun reads is impossible for genera with no available reference genomes
- Allowing substitutions in the backwards search causes an exponential runtime increase with the number of allowed mismatches; the greedy heuristic restricts substitutions to match ends rather than all query positions
- Reproducibility gap: no public code repository or data accession is stated for the benchmark reads or Kaiju source (only web server availability is mentioned)

## Evidence

- [abstract] Kaiju classifies reads with higher sensitivity and similar precision compared with current k-mer-based classifiers: "We show in a genome exclusion benchmark that Kaiju classifies reads with higher sensitivity and similar precision compared with current k-mer-based classifiers"
- [discussion] For each of the 882 genomes, we simulated five sets of HTS reads and created a reference database not containing this genome that is then used to classify the simulated reads.: "For each of the 882 genomes, we simulated five sets of HTS reads and created a reference database not containing this genome that is then used to classify the simulated reads."
- [discussion] Figures were made with the ggplot2 package28 in the R statistical software29.: "Figures were made with the ggplot2 package28 in the R statistical software29."
- [discussion] we extracted those genera that have at least 2 and at most 10 genomes assigned. This resulted in a list of 242 genera comprising 882 genomes: "we extracted those genera that have at least 2 and at most 10 genomes assigned. This resulted in a list of 242 genera comprising 882 genomes"
- [readme] In Greedy mode, matches are filtered by a minimum length and score, but also by their E-value (similar to blastp), which can be adjusted with the option `-E`. The default value is 0.01.: "In Greedy mode, matches are filtered by a minimum length and score, but also by their E-value (similar to blastp), which can be adjusted with the option `-E`. The default value is 0.01."
