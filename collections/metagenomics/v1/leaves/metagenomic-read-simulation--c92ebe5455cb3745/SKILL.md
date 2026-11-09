---
name: metagenomic-read-simulation
description: Use when benchmarking a taxonomic classifier needs simulated reads with
  known origin, e.g., ART-generated Illumina/Roche-454 FASTQ sets from genomes whose
  proteins are excluded from the reference database.
when_to_use_negative:
- Input is real metagenomic FASTQ data that simply needs taxonomic classification
  — simulation is only for controlled benchmarking with known ground truth
- The sequenced strain is already present in the reference database — the genome-exclusion
  design (database must not contain the source genome) is what makes sensitivity to
  underrepresented genera measurable; nucleotide-level classifiers will dominate when
  the strain is in the database
- The goal is to assess classification of non-protein-coding regions — protein-level
  classifiers like Kaiju cannot classify reads from non-coding genomic regions, so
  a nucleotide-level simulation/benchmark is needed instead
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3169
tools:
- name: ART
  role: Simulates the five HTS read sets (Illumina SE100/SE250/PE250 and Roche/454)
    from whole genomes including plasmids, providing ground truth for the benchmark
- name: Kaiju
  role: Classifies the simulated reads against a genome-excluded protein index (Greedy
    mode with 3 mismatches, m=11, s=65, E=0.01; also MEM mode); index built with kaiju-mkbwt
    and kaiju-mkfmi
  repo: https://github.com/bioinformatics-centre/kaiju
- name: Kraken v0.10.4b
  role: k-mer-based comparator classifier run in default mode with k=31 on the same
    simulated read sets
- name: Clark v1.1.3
  role: k-mer-based comparator classifier run in default mode with k=31 on the same
    simulated read sets
- name: ggplot2
  role: R package used to produce the benchmark summary figures from aggregated sensitivity/precision
    tables
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1038/ncomms11257
    title: Fast and sensitive taxonomic classification for metagenomics with Kaiju
  source_content_hash: sha256:de689d6f988e673c4804b87afc496f910d8bb087f84960968042a165d5aa9b43
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
  - uShuffle
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# metagenomic-read-simulation

## Summary

Simulates ground-truth high-throughput sequencing reads (Illumina single-end 100 nt, single-end 250 nt, paired-end 250 nt, and Roche/454) from complete annotated genomes with ART so that taxonomic classifiers such as Kaiju, Kraken, and Clark can be benchmarked in a genome-exclusion design.

## When to use

Use when benchmarking a taxonomic classifier needs simulated reads with known origin, e.g., ART-generated Illumina/Roche-454 FASTQ sets from genomes whose proteins are excluded from the reference database.

## When NOT to use

- Input is real metagenomic FASTQ data that simply needs taxonomic classification — simulation is only for controlled benchmarking with known ground truth
- The sequenced strain is already present in the reference database — the genome-exclusion design (database must not contain the source genome) is what makes sensitivity to underrepresented genera measurable; nucleotide-level classifiers will dominate when the strain is in the database
- The goal is to assess classification of non-protein-coding regions — protein-level classifiers like Kaiju cannot classify reads from non-coding genomic regions, so a nucleotide-level simulation/benchmark is needed instead

## Inputs

- Complete annotated bacterial/archaeal genomes (e.g., NCBI complete genomes snapshot with full chromosome, plasmids, and protein annotations)
- NCBI taxonomy files (nodes.dmp, names.dmp) mapping taxon IDs for reads and proteins
- ART read simulator with Illumina (HS/MiSeq-style 100 nt and 250 nt, paired-end 250 nt) and Roche/454 simulation profiles
- Custom protein FASTA database built per genome, excluding the source genome's proteins

## Outputs

- Per-genome simulated FASTQ read sets (five types: SE100, SE250, PE250, Roche/454) from the whole genome including plasmids
- Per-read classification outputs (Kaiju classified/unclassified lines with taxon IDs and match lengths/scores; Kraken and Clark calls at k=31)
- Genus- and phylum-level sensitivity and precision tables stratified by genus category (2–10 reference genomes)
- ggplot2 figures matching the benchmark layout (e.g., Fig. 1, Fig. 2, Supplementary Figs. 1–3)

## How to apply

Retain only genomes assigned to a species with a genus that have a full chromosome with annotated proteins, then extract genera having at least 2 and at most 10 genomes (242 genera, 882 genomes in the paper). For each genome, simulate five sets of HTS reads with ART from the whole genome including plasmids, covering Illumina single-end 100 nt, single-end 250 nt, paired-end 250 nt, and Roche/454 read profiles. Build the classification reference (e.g., a Kaiju index via kaiju-mkbwt followed by kaiju-mkfmi, alphabet ACDEFGHIKLMNPQRSTVWY) from a database that excludes the source genome's proteins, so every read's true genus is known yet absent from the index. Classify each simulated read set with Kaiju (Greedy mode, three mismatches, m=11, s=65, E=0.01, plus MEM mode) and with k-mer classifiers Kraken v0.10.4b and Clark v1.1.3 in default modes using k=31. Aggregate per-read calls into genus- and phylum-level sensitivity (fraction of reads assigned the correct taxon) and precision (correct among classified) in R with ggplot2, stratified by the number of reference genomes per genus.

## Related tools

- **ART** (Simulates the five HTS read sets (Illumina SE100/SE250/PE250 and Roche/454) from whole genomes including plasmids, providing ground truth for the benchmark)
- **Kaiju** (Classifies the simulated reads against a genome-excluded protein index (Greedy mode with 3 mismatches, m=11, s=65, E=0.01; also MEM mode); index built with kaiju-mkbwt and kaiju-mkfmi) — https://github.com/bioinformatics-centre/kaiju
- **Kraken v0.10.4b** (k-mer-based comparator classifier run in default mode with k=31 on the same simulated read sets)
- **Clark v1.1.3** (k-mer-based comparator classifier run in default mode with k=31 on the same simulated read sets)
- **ggplot2** (R package used to produce the benchmark summary figures from aggregated sensitivity/precision tables)

## Evaluation signals

- Genome filtering invariants: only genomes with a full chromosome, annotated proteins, and a genus assignment are kept; the benchmark set must contain 242 genera totaling 882 genomes (each genus with 2–10 genomes)
- Exclusion check: the per-genome reference database must not contain any protein from the source genome (custom FASTA headers carry numeric NCBI taxon IDs; sequences limited to the 20 uppercase standard amino acids)
- Coverage check: for each of the 882 genomes, five distinct simulated read sets exist (Illumina SE100, SE250, PE250, Roche/454), each derived from the whole genome including plasmids
- Benchmark validation: recomputed genus-level sensitivity for 2-genome genera on Illumina SE100 reads must match reported values — Kaiju Greedy-5 ~29% (highest) versus Clark k=31 ~16% (lowest), at similar precision
- Aggregate metrics: sensitivity is the fraction of reads assigned to the correct genus/phylum and precision is the fraction of correctly assigned among classified reads; stratify all values by the number of genomes per genus

## Limitations

- Protein-level classification cannot classify reads from non-protein-coding genomic regions, so simulated sensitivity over the whole genome underestimates what nucleotide-level methods achieve on coding-complete data
- When the exact sequenced strain is present in the reference database, Kaiju is less sensitive than nucleotide-level classifiers, so the exclusion design is essential — do not interpret excluded-genome results as performance on known strains
- Genus-level classification is impossible for genera with no available reference genomes, which defines which taxa can be included in the 2–10 genome stratification
- Simulated reads (ART) approximate but do not fully reproduce real metagenomes; on 10 real metagenomic data sets Kaiju classified on average twice as many reads as Kraken, and combining both covered 24–73% of reads

## Evidence

- [discussion] Reads were simulated from the whole genome (including plasmids) using ART27.: "Reads were simulated from the whole genome (including plasmids) using ART27."
- [discussion] For each of the 882 genomes, we simulated five sets of HTS reads and created a reference database not containing this genome that is then used to classify the simulated reads.: "For each of the 882 genomes, we simulated five sets of HTS reads and created a reference database not containing this genome that is then used to classify the simulated reads."
- [discussion] Only those genomes were retained that are assigned to a species belonging to a genus and have a full chromosome with annotated proteins, resulting in a total of 2,724 genomes: "Only those genomes were retained that are assigned to a species belonging to a genus and have a full chromosome with annotated proteins, resulting in a total of 2,724 genomes"
- [discussion] Kraken (v0.10.4b) and Clark (v1.1.3) were run in their default modes using k=31 for highest precision: "Kraken (v0.10.4b) and Clark (v1.1.3) were run in their default modes using k=31 for highest precision"
- [discussion] Figures were made with the ggplot2 package28 in the R statistical software29.: "Figures were made with the ggplot2 package28 in the R statistical software29."
- [readme] Note that the protein sequences may only contain the uppercase characters of the standard 20 amino acids, all other characters need to be removed.: "Note that the protein sequences may only contain the uppercase characters of the standard 20 amino acids, all other characters need to be removed."
