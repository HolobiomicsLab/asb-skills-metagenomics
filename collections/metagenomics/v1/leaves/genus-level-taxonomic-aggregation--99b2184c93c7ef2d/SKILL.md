---
name: genus-level-taxonomic-aggregation
description: Use when benchmarking a metagenomic read classifier on genome-exclusion
  simulations where the true source genome is known, and per-read calls must be summarized
  into genus-/phylum-level sensitivity and precision stratified by reference-genome
  availability.
when_to_use_negative:
- Classifying real metagenomes without ground-truth source genomes — there is no known
  correct genus per read, so sensitivity cannot be computed and only fraction-classified
  metrics (e.g. Kaiju classifying up to 10x more reads) apply
- Genera with no available reference genomes — the paper states genus-level classification
  of shotgun reads is impossible there, so stratified sensitivity is undefined
- Evaluating on data where the sequenced strain is already in the reference database
  — nucleotide-level classifiers assess the entire genome and Kaiju is less sensitive
  in that regime, inverting the benchmark's conclusion
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: Kaiju
  role: Protein-level classifier whose per-read taxon assignments (Greedy mode, m=11,
    s=65, E=0.01; also MEM mode) are aggregated into genus-level sensitivity/precision
  repo: https://github.com/bioinformatics-centre/kaiju
- name: Kraken
  role: k-mer-based comparator classifier (v0.10.4b, k=31) whose per-read outputs
    feed the same genus-level sensitivity/precision aggregation
- name: Clark
  role: k-mer-based comparator classifier (v1.1.3, k=31) whose per-read outputs feed
    the same genus-level sensitivity/precision aggregation
- name: ART
  role: Simulates the five HTS read sets from each of the 882 genomes (whole genome
    including plasmids), providing reads with known correct genus for the aggregation
- name: ggplot2
  role: R package used to render the stratified sensitivity/precision benchmark figures
- name: R
  role: Statistical software in which per-read classifications are aggregated into
    summary tables and figures
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/ncomms11257
    title: Fast and sensitive taxonomic classification for metagenomics with Kaiju
  source_content_hash: sha256:2df5ce287ee966192a0b69347240f4acb12d168952d1967965014a06fa2d522d
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
  - ggplot2
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# genus-level-taxonomic-aggregation

## Summary

Aggregates per-read taxonomic classifications from metagenomic classifiers (Kaiju, Kraken, Clark) into genus- and phylum-level sensitivity and precision, stratified by how many reference genomes each genus has, to quantify classifier behavior in underrepresented genera.

## When to use

Use when benchmarking a metagenomic read classifier on genome-exclusion simulations where the true source genome is known, and per-read calls must be summarized into genus-/phylum-level sensitivity and precision stratified by reference-genome availability.

## When NOT to use

- Classifying real metagenomes without ground-truth source genomes — there is no known correct genus per read, so sensitivity cannot be computed and only fraction-classified metrics (e.g. Kaiju classifying up to 10x more reads) apply
- Genera with no available reference genomes — the paper states genus-level classification of shotgun reads is impossible there, so stratified sensitivity is undefined
- Evaluating on data where the sequenced strain is already in the reference database — nucleotide-level classifiers assess the entire genome and Kaiju is less sensitive in that regime, inverting the benchmark's conclusion

## Inputs

- Simulated HTS read sets in FASTQ format (Illumina SE100/SE250, PE250, Roche/454) generated with ART from each of the 882 benchmark genomes including plasmids
- Per-read classification output files from Kaiju (tab-separated C/U status, read name, NCBI taxon ID), Kraken v0.10.4b, and Clark v1.1.3
- NCBI taxonomy files nodes.dmp and names.dmp for resolving taxon IDs to genera/phyla
- Genome-to-genus mapping for the 242 benchmark genera (2-10 genomes each, 882 genomes total) defining the exclusion and stratification categories

## Outputs

- Per-classifier genus-level sensitivity (fraction of reads assigned to the correct genus) and phylum-level sensitivity tables
- Per-classifier precision values (fraction correctly assigned among classified reads)
- Results stratified by genus category (number of reference genomes per genus)
- ggplot2 figures in R reproducing the layout of Fig. 1, Fig. 2, and Supplementary Figures 1-3

## How to apply

Simulate read sets per benchmark genome (ART; e.g. Illumina SE 100 nt, SE 250 nt, PE 250 nt, Roche/454) and classify them against a reference database that excludes the source genome, so each read has a known correct genus. Run Kaiju in Greedy mode (3 allowed mismatches, minimum match length m=11, minimum score s=65, E-value threshold E=0.01) and optionally MEM mode (-a mem), plus comparator classifiers Kraken v0.10.4b and Clark v1.1.3 with k=31. Then aggregate per-read output lines: sensitivity is the fraction of all reads assigned to the correct genus (or phylum), precision is the fraction of correctly assigned reads among classified reads. Stratify results by genus category defined by the number of reference genomes per genus (in the paper, 242 genera with at least 2 and at most 10 genomes, comprising 882 genomes), and summarize in R/ggplot2 tables and figures mirroring the paper's Fig. 1, Fig. 2, and Supplementary Figures 1-3.

## Related tools

- **Kaiju** (Protein-level classifier whose per-read taxon assignments (Greedy mode, m=11, s=65, E=0.01; also MEM mode) are aggregated into genus-level sensitivity/precision) — https://github.com/bioinformatics-centre/kaiju
- **Kraken** (k-mer-based comparator classifier (v0.10.4b, k=31) whose per-read outputs feed the same genus-level sensitivity/precision aggregation)
- **Clark** (k-mer-based comparator classifier (v1.1.3, k=31) whose per-read outputs feed the same genus-level sensitivity/precision aggregation)
- **ART** (Simulates the five HTS read sets from each of the 882 genomes (whole genome including plasmids), providing reads with known correct genus for the aggregation)
- **ggplot2** (R package used to render the stratified sensitivity/precision benchmark figures)
- **R** (Statistical software in which per-read classifications are aggregated into summary tables and figures)

## Examples

```
kaiju -t nodes.dmp -f kaiju_db.fmi -i SE100.fastq -a greedy -e 3 -m 11 -s 65 -E 0.01 -o kaiju_greedy.out
```

## Evaluation signals

- For Illumina SE 100 nt reads from genera with only two reference genomes, aggregated genus-level sensitivity must match the reported values: Kaiju Greedy-5 ~29% (highest) versus Clark k=31 ~16% (lowest)
- Across all 882-genome benchmark conditions, Kaiju must show consistently higher sensitivity with only little precision loss relative to Kraken and Clark
- Sensitivity and precision must be reported as fractions in [0,1], with precision computed only over classified reads and sensitivity over all reads
- Stratification counts must reconcile: 242 genera with at least 2 and at most 10 genomes, totaling 882 genomes
- Per-read output lines must reconcile with aggregated tables: each read contributes one C (classified, with NCBI taxon ID) or U (unclassified) entry per classifier

## Limitations

- Protein-level classification cannot classify reads originating from non-protein-coding genomic regions, capping achievable sensitivity
- Genus-level classification of shotgun reads is impossible for genera with no available reference genomes
- Kaiju is less sensitive than nucleotide-level classifiers when the sequenced strain is present in the reference database, as seen in the HiSeq and MiSeq data sets
- Allowing substitutions in the backwards search causes an exponential runtime increase with the number of allowed mismatches, constraining Greedy-mode settings

## Evidence

- [discussion] Stratification basis: "we extracted those genera that have at least 2 and at most 10 genomes assigned. This resulted in a list of 242 genera comprising 882 genomes"
- [discussion] Exclusion benchmark design: "For each of the 882 genomes, we simulated five sets of HTS reads and created a reference database not containing this genome that is then used to classify the simulated reads."
- [discussion] Headline benchmark result to validate against: "Our benchmark on 882 genomes and five types of simulated reads shows that Kaiju consistently achieves a much higher sensitivity with only little loss of precision compared with Kraken and Clark"
- [readme] Kaiju Greedy-mode thresholds used before aggregation: "The cutoffs for minimum required match length and match score can be changed using the options `-m` (default: 11) and `-s` (default: 65)."
- [readme] Taxon assignment basis for per-read output lines: "Reads are directly assigned to taxa using the NCBI taxonomy and a reference database of protein sequences from microbial and viral genomes."
