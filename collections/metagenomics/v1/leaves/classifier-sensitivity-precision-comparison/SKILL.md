---
name: classifier-sensitivity-precision-comparison
description: Use when comparing read classifiers on datasets of known composition
  (mock communities or simulated reads) where genus- or phylum-level sensitivity and
  precision must be quantified per tool.
when_to_use_negative:
- Input reads come from real metagenomes of unknown composition, so true sensitivity
  and precision cannot be computed — report classification rates instead
- Reads derive from strains present in the reference database, where nucleotide-level
  classifiers are expected to outperform Kaiju's protein-level matching
- The reads originate largely from non-protein-coding regions, which Kaiju's protein-level
  classification cannot classify by design
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: Kaiju
  role: Classifies reads in Greedy mode (-e 5) or MEM mode (kaiju -a mem) against
    a .fmi index; outputs summarized with kaiju2table at genus and phylum ranks
  repo: https://github.com/bioinformatics-centre/kaiju
- name: Kraken v0.10.4b
  role: k-mer-based comparator run in default mode with k=31 for highest precision
    on the same mock community FASTQ files
- name: ART
  role: Simulates HTS reads (optionally with source-genome exclusion) to create ground-truth
    benchmarks for the sensitivity/precision comparison
- name: Krona
  role: Interactive visualization of the classification results
  repo: https://github.com/marbl/Krona/wiki/KronaTools
provenance:
  source_task_ids:
  - task_007
  - task_003
  source_papers:
  - doi: 10.1038/ncomms11257
    title: Fast and sensitive taxonomic classification for metagenomics with Kaiju
  source_content_hash: sha256:94738aea9637c046d8c540f49227f110d47c1169c277a064709cabffd2b28cf1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bioinformatics-centre/kaiju
  tools:
  - ART
  - Kaiju
  - Kraken v0.10.4b
  - Krona
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# classifier-sensitivity-precision-comparison

## Summary

Benchmark taxonomic classifiers by classifying the same FASTQ reads (e.g. Kaiju Greedy-5 vs Kraken k=31) and computing genus- and phylum-level sensitivity and precision against known mock-community ground truth, as done in the Kaiju HiSeq/MiSeq mock community evaluation.

## When to use

Use when comparing read classifiers on datasets of known composition (mock communities or simulated reads) where genus- or phylum-level sensitivity and precision must be quantified per tool.

## When NOT to use

- Input reads come from real metagenomes of unknown composition, so true sensitivity and precision cannot be computed — report classification rates instead
- Reads derive from strains present in the reference database, where nucleotide-level classifiers are expected to outperform Kaiju's protein-level matching
- The reads originate largely from non-protein-coding regions, which Kaiju's protein-level classification cannot classify by design

## Inputs

- FASTQ read files with known taxonomic composition (e.g. 10k-read HiSeq and MiSeq mock community datasets)
- Kaiju reference index (.fmi, e.g. kaiju_db.fmi)
- NCBI taxonomy files nodes.dmp and names.dmp
- Kraken database for k=31 classification

## Outputs

- Per-classifier, per-rank (genus, phylum) sensitivity and precision values (e.g. benchmark_suppl_table2.tsv)
- Summary of classified read names and assigned NCBI taxon identifiers per classifier
- Optional mode-comparison table (e.g. greedy_vs_mem_comparison.tsv) with per-mode classification rates and agreement

## How to apply

Classify each input FASTQ with every classifier under its intended high-precision/high-sensitivity configuration: Kaiju in Greedy mode with mismatches raised to 5 via -e while keeping defaults (m=11, s=65, E=0.01, SEG filtering enabled), and Kraken v0.10.4b in default mode with k=31 for highest precision. Summarize Kaiju outputs with kaiju2table at the genus and phylum ranks using nodes.dmp and names.dmp, and tally Kraken assignments to the same two ranks. For each classifier, rank, and dataset, compute sensitivity as the fraction of reads from known mock-community taxa that receive a classification and precision as the fraction of classified reads assigned to the correct taxon. Assemble the per-classifier, per-rank, per-dataset metrics into a comparison table (e.g. benchmark_suppl_table2.tsv) and validate against published values within rounding tolerance. Optionally repeat the design across run modes (Greedy vs MEM via kaiju -a mem) to compare classification rates and agreement on identical reads and index.

## Related tools

- **Kaiju** (Classifies reads in Greedy mode (-e 5) or MEM mode (kaiju -a mem) against a .fmi index; outputs summarized with kaiju2table at genus and phylum ranks) — https://github.com/bioinformatics-centre/kaiju
- **Kraken v0.10.4b** (k-mer-based comparator run in default mode with k=31 for highest precision on the same mock community FASTQ files)
- **ART** (Simulates HTS reads (optionally with source-genome exclusion) to create ground-truth benchmarks for the sensitivity/precision comparison)
- **Krona** (Interactive visualization of the classification results) — https://github.com/marbl/Krona/wiki/KronaTools

## Examples

```
kaiju -z 25 -t nodes.dmp -f kaiju_db.fmi -i hiseq_mock.fastq -e 5 -o kaiju_greedy5.out; kaiju2table -t nodes.dmp -n names.dmp -r genus -o kaiju_genus.tsv kaiju_greedy5.out
```

## Evaluation signals

- Reproduced HiSeq genus-level values match Supplementary Table 2 within rounding tolerance: Kaiju Greedy-5 73.3% sensitivity / 94.4% precision vs Kraken k=31 78.0% / 99.2%
- Every classified read's NCBI taxon identifier resolves within the reference taxonomy in nodes.dmp
- Per-rank metrics are computed identically for both classifiers (same genus and phylum rank cuts via kaiju2table and Kraken tallying)
- In mode comparisons, Greedy mode classifies at least as many reads as MEM mode at comparable precision, consistent with 'Greedy run mode yields a higher sensitivity compared with MEM mode'
- Classification of the same input FASTQ with both tools yields comparable classified-read name sets, enabling direct agreement computation

## Limitations

- Kraken achieved higher genus-level sensitivity and precision (78.0%/99.2%) than Kaiju Greedy-5 (73.3%/94.4%) on the short HiSeq mock community reads; Kaiju's advantage appears in genome-exclusion and real metagenome settings
- Allowing more substitutions in the Greedy backwards search causes an exponential runtime increase with the number of allowed mismatches
- Kaiju's protein-level classification cannot classify reads from non-protein-coding genomic regions
- Genus-level classification is impossible for genera with no available reference genomes

## Evidence

- [discussion] Kraken (v0.10.4b) and Clark (v1.1.3) were run in their default modes using k=31 for highest precision: "Kraken (v0.10.4b) and Clark (v1.1.3) were run in their default modes using k=31 for highest precision"
- [discussion] Greedy mode locates MEMs then extends with substitutions, ranked by BLOSUM62 score: "we rank the fragments by their BLOSUM62 score and start the database search with the highest scoring fragment"
- [readme] Default Greedy parameters: three allowed mismatches, s=65, E-value 0.01, m=11: "The default run mode is **Greedy** with three allowed mismatches. The number of allowed mismatches can be changed using option `-e`."
- [readme] SEG filtering avoids false positive taxon assignments: "SEG filtering is always recommended in order to avoid false positive taxon assignments"
- [discussion] Kaiju achieves higher sensitivity with little precision loss versus Kraken and Clark in the genome exclusion benchmark: "Kaiju consistently achieves a much higher sensitivity with only little loss of precision compared with Kraken and Clark"
- [discussion] Kaiju would be less sensitive than nucleotide-level classifiers when sequenced strains are in the database: "Kaiju would be less sensitive than nucleotide-level classifiers, which can assess the entire genome, as seen in the HiSeq and MiSeq data sets"
