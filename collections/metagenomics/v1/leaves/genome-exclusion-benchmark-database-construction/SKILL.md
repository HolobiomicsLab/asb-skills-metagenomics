---
name: genome-exclusion-benchmark-database-construction
description: Use when benchmarking taxonomic classifiers on genera underrepresented
  in reference databases via leave-one-genome-out protein/k-mer databases and simulated
  reads (Illumina/Roche-454).
when_to_use_negative:
- When the sequenced strain is itself in the reference database — nucleotide-level
  classifiers will be more sensitive and the exclusion design does not apply
- When reads originate from non-protein-coding genomic regions — Kaiju's protein-level
  classification cannot assign them
- When target genera have no available reference genomes — genus-level classification
  (and thus this benchmark stratification) is impossible
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3174
tools:
- name: Kaiju
  role: Protein-level classifier under test; custom genome-excluded indexes built
    with kaiju-mkbwt and kaiju-mkfmi, run in Greedy (3 mismatches, m=11, s=65, E=0.01)
    and MEM modes
  repo: https://github.com/bioinformatics-centre/kaiju
- name: ART
  role: Simulates the five HTS read sets from each whole genome including plasmids
- name: Kraken
  role: k-mer-based baseline classifier (v0.10.4b, default mode, k=31)
- name: Clark
  role: k-mer-based baseline classifier (v1.1.3, default mode, k=31)
- name: R / ggplot2
  role: Aggregates per-read classifications into sensitivity/precision statistics
    and renders the benchmark figures
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/ncomms11257
    title: Fast and sensitive taxonomic classification for metagenomics with Kaiju
  source_content_hash: sha256:7003e0933a1a45f40c408c8e2c070e09e9126a6438a52a7b3ee620637fb229a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bioinformatics-centre/kaiju
  tools:
  - ART
  - Clark
  - Kaiju
  - Kraken
  - Kraken v0.10.4b
  - R
  - R / ggplot2
  - ggplot2
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# genome-exclusion-benchmark-database-construction

## Summary

Reproduces the Kaiju genome-exclusion benchmark: for each of 882 complete bacterial/archaeal genomes (242 genera with 2–10 genomes), a Kaiju protein reference database excluding the source genome is built with kaiju-mkbwt/kaiju-mkfmi, ART-simulated reads are classified, and genus/phylum sensitivity and precision are computed against Kraken and Clark.

## When to use

Use when benchmarking taxonomic classifiers on genera underrepresented in reference databases via leave-one-genome-out protein/k-mer databases and simulated reads (Illumina/Roche-454).

## When NOT to use

- When the sequenced strain is itself in the reference database — nucleotide-level classifiers will be more sensitive and the exclusion design does not apply
- When reads originate from non-protein-coding genomic regions — Kaiju's protein-level classification cannot assign them
- When target genera have no available reference genomes — genus-level classification (and thus this benchmark stratification) is impossible

## Inputs

- Complete bacterial/archaeal genome assemblies with annotated proteins and plasmid sequences (NCBI snapshot, e.g. 16 December 2014)
- Per-genome protein FASTA files with headers carrying numeric NCBI taxon identifiers
- NCBI taxonomy files nodes.dmp and names.dmp
- ART simulation parameter sets (Illumina SE100, SE250, PE250; Roche/454)

## Outputs

- Per-genome Kaiju index files (proteins.bwt/.fmi) built from genome-excluded protein sets
- Simulated FASTQ read sets (five read types per genome)
- Kaiju classification outputs with C/U flags and NCBI taxon IDs; Kraken and Clark classification outputs
- Genus- and phylum-level sensitivity/precision tables stratified by genomes-per-genus category (242 genera, 882 genomes)
- ggplot2 benchmark figures matching Fig. 1, Fig. 2 and Supplementary Figures 1–3

## How to apply

Start from a snapshot of all complete NCBI bacterial and archaeal genomes, retain only genomes with a full chromosome and annotated proteins, and extract genera with at least 2 and at most 10 genomes, yielding 242 genera comprising 882 genomes. For each genome, simulate five sets of HTS reads with ART from the whole genome including plasmids (Illumina SE 100 nt, SE 250 nt, PE 250 nt, and Roche/454). Build a custom Kaiju protein FASTA whose headers are numeric NCBI taxon identifiers, excluding the source genome's proteins, then construct the index with kaiju-mkbwt (-n 5, alphabet ACDEFGHIKLMNPQRSTVWY) followed by kaiju-mkfmi. Classify the simulated reads against each genome-excluded database with Kaiju in Greedy mode (3 allowed mismatches, m=11, s=65, E=0.01) and in MEM mode (-a mem), and with Kraken v0.10.4b and Clark v1.1.3 in default modes using k=31 for highest precision. Aggregate per-read assignments into genus- and phylum-level sensitivity (correct assignments among all reads) and precision (correct among classified), stratified by the number of genomes per genus, and produce summary tables and ggplot2 figures in R.

## Related tools

- **Kaiju** (Protein-level classifier under test; custom genome-excluded indexes built with kaiju-mkbwt and kaiju-mkfmi, run in Greedy (3 mismatches, m=11, s=65, E=0.01) and MEM modes) — https://github.com/bioinformatics-centre/kaiju
- **ART** (Simulates the five HTS read sets from each whole genome including plasmids)
- **Kraken** (k-mer-based baseline classifier (v0.10.4b, default mode, k=31))
- **Clark** (k-mer-based baseline classifier (v1.1.3, default mode, k=31))
- **R / ggplot2** (Aggregates per-read classifications into sensitivity/precision statistics and renders the benchmark figures)

## Examples

```
kaiju-mkbwt -n 5 -a ACDEFGHIKLMNPQRSTVWY -o proteins proteins.faa; kaiju-mkfmi proteins; kaiju -t nodes.dmp -f proteins.fmi -i reads.fastq -a greedy -e 3 -m 11 -s 65 -E 0.01
```

## Evaluation signals

- Leave-one-genome-out invariant: the Kaiju protein FASTA for each benchmark run contains no proteins from the source genome
- Every taxon identifier used in the custom FASTA headers resolves in nodes.dmp/names.dmp
- kaiju-mkbwt followed by kaiju-mkfmi yields the .fmi index consumed by kaiju for all 882 leave-one-out databases
- Computed genus-level sensitivity for 2-genome genera on Illumina SE 100 nt reads matches the reported values: Kaiju Greedy-5 ~29% versus Clark k=31 ~16%, with Kraken/Clark precision comparable to Kaiju
- Sensitivity/precision are reported separately at genus and phylum levels for all five read types (SE100, SE250, PE250, Roche/454) and stratified by genomes-per-genus category

## Limitations

- Protein-level classification cannot classify reads from non-protein-coding genomic regions, lowering sensitivity relative to nucleotide classifiers on those fractions
- Kaiju is less sensitive than nucleotide-level classifiers when the sequenced strain is present in the reference database
- Allowing substitutions in the backwards search causes an exponential runtime increase with the number of allowed mismatches; the greedy heuristic restricts substitutions to match ends rather than all query positions
- Reproducibility gap: no public code repository or data accession for the benchmark read sets is stated in the section, only Kaiju's command-line and web server availability

## Evidence

- [discussion] For each of the 882 genomes, we simulated five sets of HTS reads and created a reference database not containing this genome that is then used to classify the simulated reads.: "For each of the 882 genomes, we simulated five sets of HTS reads and created a reference database not containing this genome that is then used to classify the simulated reads."
- [discussion] 242 genera comprising 882 genomes: "we extracted those genera that have at least 2 and at most 10 genomes assigned. This resulted in a list of 242 genera comprising 882 genomes"
- [discussion] Kraken and Clark run modes: "Kraken (v0.10.4b) and Clark (v1.1.3) were run in their default modes using k=31 for highest precision"
- [methods] Greedy default mode: "The default run mode is **Greedy** with three allowed mismatches."
- [readme] Custom database FASTA format: "The format needs to be a FASTA file in which the headers are the numeric NCBI taxon identifiers of the protein sequences"
- [readme] Index construction programs: "Then, Kaiju's index is created using the programs `kaiju-mkbwt` and `kaiju-mkfmi`."
