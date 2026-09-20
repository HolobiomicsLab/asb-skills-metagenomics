---
name: taxonomic-classifier-sensitivity-precision-computation
description: Use when benchmarking metagenomic taxonomic classifiers via genome exclusion
  — i.e., simulating reads (ART, Illumina/Roche-454 read models), classifying them
  against databases that omit the source genome, and computing genus/phylum sensitivity
  and precision.
when_to_use_negative:
- Evaluating nucleotide-level classifier performance when the sequenced strain is
  already in the reference database — the article notes Kaiju would be less sensitive
  than nucleotide-level classifiers in that setting
- Classifying reads originating from non-protein-coding genomic regions — protein-level
  classification (Kaiju) cannot assign these
- Genus-level benchmarking of genera with only one or no available reference genomes
  — genus-level classification is impossible and the stratification requires 2-10
  genomes per genus
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0196
tools:
- name: Kaiju
  role: Classifier under test; protein-level classification of simulated reads against
    genome-exclusion databases, run in Greedy and MEM modes
  repo: https://github.com/bioinformatics-centre/kaiju
- name: Kraken v0.10.4b
  role: k-mer-based comparator classifier run in default mode with k=31 for highest
    precision
- name: Clark v1.1.3
  role: k-mer-based comparator classifier run in default mode with k=31 for highest
    precision
- name: ART
  role: Simulates the five HTS read sets from the whole genome including plasmids
    for each benchmark genome
- name: R with ggplot2
  role: Aggregates per-read classifications into sensitivity/precision tables and
    produces the benchmark figures
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/ncomms11257
    title: Fast and sensitive taxonomic classification for metagenomics with Kaiju
  source_content_hash: sha256:6e5b3b4690fdac0c110c83643c6f1fac09b834d0cd5c9f1feceb35ddf650c551
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
  - R with ggplot2
  - ggplot2
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# taxonomic-classifier-sensitivity-precision-computation

## Summary

Genome-exclusion benchmarking of taxonomic classifiers: simulate HTS reads from whole genomes, classify them against a reference database lacking the source genome, and compute genus/phylum-level sensitivity and precision. Used to expose sensitivity differences between protein-level classifiers (Kaiju) and k-mer classifiers (Kraken, Clark) in underrepresented genera.

## When to use

Use when benchmarking metagenomic taxonomic classifiers via genome exclusion — i.e., simulating reads (ART, Illumina/Roche-454 read models), classifying them against databases that omit the source genome, and computing genus/phylum sensitivity and precision.

## When NOT to use

- Evaluating nucleotide-level classifier performance when the sequenced strain is already in the reference database — the article notes Kaiju would be less sensitive than nucleotide-level classifiers in that setting
- Classifying reads originating from non-protein-coding genomic regions — protein-level classification (Kaiju) cannot assign these
- Genus-level benchmarking of genera with only one or no available reference genomes — genus-level classification is impossible and the stratification requires 2-10 genomes per genus

## Inputs

- FASTQ read sets simulated with ART from whole genomes including plasmids (Illumina SE100, SE250, PE250, Roche/454)
- Reference protein FASTA files per genome-exclusion database (source genome's proteins removed)
- NCBI taxonomy files nodes.dmp and names.dmp
- Per-read classification outputs from Kaiju (Greedy and MEM modes), Kraken v0.10.4b, and Clark v1.1.3
- Genus-to-genome mapping for stratification (242 genera, 882 genomes)

## Outputs

- Per-genus and per-phylum sensitivity and precision tables, stratified by number of reference genomes per genus
- ggplot2 figures in R reproducing Fig. 1, Fig. 2, and Supplementary Figures 1-3 layouts
- Comparison summary (e.g., Kaiju Greedy-5 ~29% vs Clark k=31 ~16% genus sensitivity on SE100 reads from 2-genome genera)

## How to apply

Simulate five sets of HTS reads (Illumina single-end 100 nt, single-end 250 nt, paired-end 250 nt, and Roche/454) from the whole genome including plasmids of each benchmark genome using ART. For each of the 882 genomes, build a Kaiju protein reference database excluding the source genome's proteins with kaiju-mkbwt followed by kaiju-mkfmi, then classify the reads with Kaiju in Greedy mode (three allowed mismatches, minimum match length m=11, minimum score s=65, E-value threshold E=0.01) and in MEM mode via '-a mem'. Classify the identical read sets with Kraken v0.10.4b and Clark v1.1.3 in their default modes using k=31 for highest precision. Aggregate per-read assignments into genus- and phylum-level sensitivity (fraction of reads assigned to the correct genus/phylum) and precision (correctly assigned among classified), stratified by genus category based on the number of genomes per genus — the benchmark uses 242 genera with at least 2 and at most 10 genomes, totaling 882 genomes. Produce summary tables and ggplot2 figures in R matching the layout of Fig. 1, Fig. 2, and Supplementary Figures 1-3.

## Related tools

- **Kaiju** (Classifier under test; protein-level classification of simulated reads against genome-exclusion databases, run in Greedy and MEM modes) — https://github.com/bioinformatics-centre/kaiju
- **Kraken v0.10.4b** (k-mer-based comparator classifier run in default mode with k=31 for highest precision)
- **Clark v1.1.3** (k-mer-based comparator classifier run in default mode with k=31 for highest precision)
- **ART** (Simulates the five HTS read sets from the whole genome including plasmids for each benchmark genome)
- **R with ggplot2** (Aggregates per-read classifications into sensitivity/precision tables and produces the benchmark figures)

## Examples

```
kaiju -t nodes.dmp -f kaiju_db.fmi -i reads.fastq -m 11 -s 65 -E 0.01 -o kaiju.out
```

## Evaluation signals

- Per-genome reference database must provably exclude the source genome's proteins before classification
- Computed genus-level sensitivity must match reported benchmark values, e.g. Kaiju Greedy-5 ~29% versus Clark k=31 ~16% for SE100 reads from 2-genome genera
- Kaiju should show higher sensitivity at similar precision relative to Kraken and Clark across the five read types on the 882-genome benchmark
- Stratification must cover the 242 genera comprising 882 genomes (at least 2 and at most 10 genomes per genus)
- All classifier runs use the same simulated read sets, with Kraken and Clark run in default modes at k=31

## Limitations

- Protein-level classification cannot classify reads from non-protein-coding genomic regions
- Kaiju is less sensitive than nucleotide-level classifiers when sequenced strains are in the reference database
- Allowing substitutions in the backwards search causes an exponential runtime increase with the number of allowed mismatches; the greedy heuristic restricts substitutions to match ends rather than all query positions
- No public code repository or data accession is stated in the section for the benchmark reads or Kaiju source (only a web server availability is mentioned)

## Evidence

- [discussion] For each of the 882 genomes, we simulated five sets of HTS reads and created a reference database not containing this genome that is then used to classify the simulated reads.: "For each of the 882 genomes, we simulated five sets of HTS reads and created a reference database not containing this genome that is then used to classify the simulated reads."
- [discussion] Kraken and Clark run settings: "Kraken (v0.10.4b) and Clark (v1.1.3) were run in their default modes using k=31 for highest precision"
- [discussion] Genus stratification basis: "we extracted those genera that have at least 2 and at most 10 genomes assigned. This resulted in a list of 242 genera comprising 882 genomes"
- [discussion] Benchmark headline result: "Our benchmark on 882 genomes and five types of simulated reads shows that Kaiju consistently achieves a much higher sensitivity with only little loss of precision compared with Kraken and Clark"
- [discussion] Read simulation source: "Reads were simulated from the whole genome (including plasmids) using ART27."
- [readme] Kaiju Greedy parameters: "The cutoffs for minimum required match length and match score can be changed using the options `-m` (default: 11) and `-s` (default: 65)."
