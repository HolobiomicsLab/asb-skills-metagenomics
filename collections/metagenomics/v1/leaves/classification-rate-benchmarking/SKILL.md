---
name: classification-rate-benchmarking
description: Use when comparing how many reads different metagenomic classifiers (e.g.
  Kaiju MEM/Greedy-5 vs Kraken k=31) assign on the same real FASTQ metagenomes, or
  when quantifying overlap and merged classification totals.
when_to_use_negative:
- Input reads are from organisms whose exact sequenced strains are in the reference
  database — nucleotide-level classifiers can assess the entire genome and would be
  more sensitive, so a protein-level comparison underestimates the nucleotide classifier's
  advantage
- The goal is classifying reads from non-protein-coding genomic regions — protein-level
  Kaiju cannot classify these, skewing rate comparisons
- Reads are already protein sequences or the analysis targets a single simulated benchmark
  rather than real metagenome classification rates
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: Kaiju
  role: Protein-level classifier run in MEM mode (-a mem) and Greedy mode with 5 mismatches
    (-e 5, -s 70) with SEG filtering (-x) to classify reads
  repo: https://github.com/bioinformatics-centre/kaiju
- name: Kraken
  role: Nucleotide k-mer baseline classifier run in default mode with k=31 against
    a database built from the same genomes
- name: kaiju-mergeOutputs
  role: Merges sorted Kaiju and Kraken outputs by read name, resolving conflicting
    taxon identifiers with the LCA rule (-c lca)
  repo: https://github.com/bioinformatics-centre/kaiju
- name: kaiju2table
  role: Optionally summarizes merged per-read classifications at a chosen taxonomic
    rank for reporting
  repo: https://github.com/bioinformatics-centre/kaiju
- name: R
  role: Used with ggplot2 to assemble classification-rate figures and summaries
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/ncomms11257
    title: Fast and sensitive taxonomic classification for metagenomics with Kaiju
  source_content_hash: sha256:ee2b2fc9ffb798775879d47cb292f52676c93575c63f9c003e24f67855215da1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bioinformatics-centre/kaiju
  tools:
  - Kaiju
  - Kraken
  - R
  - kaiju-mergeOutputs
  - kaiju2table
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# classification-rate-benchmarking

## Summary

Benchmarking protocol for comparing protein-level (Kaiju MEM/Greedy) and nucleotide k-mer (Kraken) taxonomic classification rates on real metagenomic FASTQ datasets, including merging outputs with LCA conflict resolution to quantify combined sensitivity.

## When to use

Use when comparing how many reads different metagenomic classifiers (e.g. Kaiju MEM/Greedy-5 vs Kraken k=31) assign on the same real FASTQ metagenomes, or when quantifying overlap and merged classification totals.

## When NOT to use

- Input reads are from organisms whose exact sequenced strains are in the reference database — nucleotide-level classifiers can assess the entire genome and would be more sensitive, so a protein-level comparison underestimates the nucleotide classifier's advantage
- The goal is classifying reads from non-protein-coding genomic regions — protein-level Kaiju cannot classify these, skewing rate comparisons
- Reads are already protein sequences or the analysis targets a single simulated benchmark rather than real metagenome classification rates

## Inputs

- Ten real metagenomic FASTQ datasets (saliva, vagina, gut, lake, Amazon plume, Baltic sea, desert soil, bioreactor samples)
- Kaiju FM-index (.fmi) built from protein sequences of 2,724 complete bacterial and archaeal genomes
- Kraken nucleotide database built from the same 2,724 genomes
- NCBI taxonomy files nodes.dmp and names.dmp

## Outputs

- Per-dataset Kaiju output files (kaiju.out) for MEM and Greedy-5 modes
- Per-dataset Kraken output files
- Merged classification files (combined.out) from kaiju-mergeOutputs with LCA-resolved taxa
- metagenome_classification_summary.tsv with classified percentages, overlaps, and merged totals per dataset

## How to apply

Build both indexes from the identical genome set: the Kaiju .fmi from protein sequences of the 2,724 retained complete bacterial and archaeal genomes (snapshot 16 December 2014) and a Kraken nucleotide database from the same genomes. Run Kaiju in MEM mode with a minimum match length of 11 amino acids (-a mem -m 12 in the Fig. 3 setup) and separately in Greedy mode with 5 allowed mismatches (-e 5) and minimum score 70 (-s 70), enabling SEG low-complexity filtering (-x) on all 10 FASTQ datasets (saliva, vagina, gut, lake, Amazon plume, Baltic sea, desert soil, bioreactors), using kaiju-multi with parallel threads (-z). Run Kraken v0.10.4b in its default mode with k=31 against the nucleotide database. Sort Kaiju and Kraken outputs by read name (sort -k2,2), merge with kaiju-mergeOutputs resolving conflicting taxon identifiers by LCA, then compute per-dataset classified-read percentages and pairwise overlaps (optionally summarized at rank level with kaiju2table). Validate by checking that Kaiju classifies on average about twice as many reads as Kraken and that the merged output falls in the reported 24-73% range across samples.

## Related tools

- **Kaiju** (Protein-level classifier run in MEM mode (-a mem) and Greedy mode with 5 mismatches (-e 5, -s 70) with SEG filtering (-x) to classify reads) — https://github.com/bioinformatics-centre/kaiju
- **Kraken** (Nucleotide k-mer baseline classifier run in default mode with k=31 against a database built from the same genomes)
- **kaiju-mergeOutputs** (Merges sorted Kaiju and Kraken outputs by read name, resolving conflicting taxon identifiers with the LCA rule (-c lca)) — https://github.com/bioinformatics-centre/kaiju
- **kaiju2table** (Optionally summarizes merged per-read classifications at a chosen taxonomic rank for reporting) — https://github.com/bioinformatics-centre/kaiju
- **R** (Used with ggplot2 to assemble classification-rate figures and summaries)

## Examples

```
kaiju-multi -z 25 -t nodes.dmp -f kaiju_db.fmi -a mem -m 12 -x -i sample1.fastq,sample2.fastq -o sample1.out,sample2.out
```

## Evaluation signals

- Kaiju MEM classifies between 13.1% (Human vagina) and 48.8% (Bioreactor sediment) more reads than Kraken; 17.8-56.6% more in Greedy-5 mode
- Merged Kaiju Greedy-5 + Kraken classification totals fall between 24.7% (Desert soil) and 73.1% (Bioreactor sediment) of reads across the 10 samples
- Kaiju classifies on average about twice as many reads as Kraken across the 10 real metagenomic datasets
- All classifier outputs sort cleanly by read name (sort -k2,2) before merging; merged file contains exactly one line per input read with C/U status
- Percentages computed as classified reads divided by total reads per dataset match across the four conditions (Kaiju MEM, Kaiju Greedy, Kraken, merged)

## Limitations

- Protein-level classification cannot classify reads originating from non-protein-coding genomic regions
- Kaiju is less sensitive than nucleotide-level classifiers when the sequenced strains are present in the reference database (e.g. HiSeq and MiSeq data sets)
- Genus-level classification is impossible for genera with no available reference genomes
- Allowing substitutions in the backwards search causes an exponential runtime increase with the number of allowed mismatches; the greedy heuristic restricts substitutions to match ends

## Evidence

- [discussion] Kaiju classifies on average twice as many reads as Kraken in 10 real metagenomic data sets: "In our set of 10 randomly selected real metagenomic data sets, Kaiju classifies on average twice as many reads as Kraken"
- [discussion] Combined Kaiju+Kraken classification between 24 and 73% of reads: "By combining Kaiju's and Kraken's output, between 24 and 73% of reads can be classified across the various samples"
- [discussion] Kraken and Clark run in default mode with k=31: "Kraken (v0.10.4b) and Clark (v1.1.3) were run in their default modes using k=31 for highest precision"
- [discussion] Genome filtering to 2,724 genomes: "Only those genomes were retained that are assigned to a species belonging to a genus and have a full chromosome with annotated proteins, resulting in a total of 2,724 genomes"
- [readme] SEG low-complexity filtering recommended: "SEG filtering
is always recommended in order to avoid false positive taxon assignments that
are caused by spurious matches due to simple repeat patterns or other
sequencing noise."
- [readme] Greedy mode mismatch options: "The default run mode is **Greedy** with three allowed mismatches.
The number of allowed mismatches can be changed using option `-e`."
