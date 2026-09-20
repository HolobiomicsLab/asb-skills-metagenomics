---
name: mock-community-benchmark-reproduction
description: Use when benchmarking taxonomic classifiers on mock community Illumina
  reads and you need to reproduce genus/phylum sensitivity and precision comparisons
  between protein-level (Kaiju) and k-mer-based (Kraken) classifiers.
when_to_use_negative:
- Reads derive from simulated data where the sequenced strain genomes are present
  in the reference database — nucleotide-level classifiers like Kraken will dominate
  and the protein-level comparison is not informative
- The goal is strain- or species-level resolution for genera with no reference genomes
  — genus-level classification is impossible in that case for any classifier
- Input is already a genus/phylum abundance table; this skill operates on raw FASTQ
  reads and classifier outputs
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: Kaiju
  role: Protein-level classifier run in Greedy mode with 5 allowed mismatches (-e
    5) to classify mock community reads; summarized with kaiju2table at genus and
    phylum ranks
  repo: https://github.com/bioinformatics-centre/kaiju
- name: Kraken v0.10.4b
  role: k-mer-based comparator classifier run in default mode with k=31 for highest
    precision on the same mock community reads
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/ncomms11257
    title: Fast and sensitive taxonomic classification for metagenomics with Kaiju
  source_content_hash: sha256:997e187d43833851edf69d09acf58397238a4a67b3ceabc7ecf44a28db4adb7d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bioinformatics-centre/kaiju
  tools:
  - Kaiju
  - Kraken v0.10.4b
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# mock-community-benchmark-reproduction

## Summary

Reproduces the genus- and phylum-level sensitivity/precision benchmark of Kaiju (Greedy-5 mode) versus Kraken (k=31) on the HiSeq and MiSeq mock community FASTQ datasets, yielding the eight sensitivity/precision values of Supplementary Table 2.

## When to use

Use when benchmarking taxonomic classifiers on mock community Illumina reads and you need to reproduce genus/phylum sensitivity and precision comparisons between protein-level (Kaiju) and k-mer-based (Kraken) classifiers.

## When NOT to use

- Reads derive from simulated data where the sequenced strain genomes are present in the reference database — nucleotide-level classifiers like Kraken will dominate and the protein-level comparison is not informative
- The goal is strain- or species-level resolution for genera with no reference genomes — genus-level classification is impossible in that case for any classifier
- Input is already a genus/phylum abundance table; this skill operates on raw FASTQ reads and classifier outputs

## Inputs

- 10k-read HiSeq mock community FASTQ dataset
- 10k-read MiSeq mock community FASTQ dataset
- Kaiju reference index (.fmi file, e.g. kaiju_db_refseq.fmi)
- NCBI taxonomy files nodes.dmp and names.dmp
- Mock community ground-truth taxon list for sensitivity/precision computation

## Outputs

- Kaiju classification output files (C/U, read name, NCBI taxon id) per dataset
- Kraken classification output files per dataset
- Genus- and phylum-level summary tables from kaiju2table
- benchmark_suppl_table2.tsv with eight sensitivity/precision values (2 classifiers x 2 ranks x 2 datasets)

## How to apply

Obtain the 10k-read HiSeq and MiSeq mock community FASTQ files plus a Kaiju reference index (.fmi) and the NCBI taxonomy files nodes.dmp and names.dmp. Classify each dataset with Kaiju in its default Greedy mode but raise the allowed mismatches to 5 via -e, keeping defaults (m=11, s=65, E=0.01, SEG filtering enabled) and writing output with -o using multiple threads (-z). Run Kraken v0.10.4b in its default mode with k=31, which the authors chose for highest precision. Summarize Kaiju output with kaiju2table at genus and phylum ranks (requiring nodes.dmp and names.dmp) and tally Kraken assignments at the same two ranks. Compute sensitivity as the fraction of reads from known mock-community taxa that receive a classification, and precision as the fraction of classified reads assigned to the correct taxon; the target values include HiSeq genus-level Kaiju 73.3%/94.4% and Kraken 78.0%/99.2%.

## Related tools

- **Kaiju** (Protein-level classifier run in Greedy mode with 5 allowed mismatches (-e 5) to classify mock community reads; summarized with kaiju2table at genus and phylum ranks) — https://github.com/bioinformatics-centre/kaiju
- **Kraken v0.10.4b** (k-mer-based comparator classifier run in default mode with k=31 for highest precision on the same mock community reads)

## Examples

```
kaiju -z 25 -t nodes.dmp -f kaiju_db_refseq.fmi -i hisseq_mock_10k.fastq -e 5 -o kaiju_greedy5_hisseq.out
```

## Evaluation signals

- Reproduced genus-level values match Supplementary Table 2 within rounding tolerance, e.g. HiSeq genus: Kaiju 73.3% sensitivity / 94.4% precision and Kraken 78.0% sensitivity / 99.2% precision
- All eight sensitivity/precision values (2 classifiers x 2 ranks x 2 datasets) are populated in benchmark_suppl_table2.tsv
- Kaiju runs used Greedy mode with 5 allowed mismatches while retaining defaults m=11, s=65, E=0.01 and SEG filtering enabled
- Kraken was run in default mode with k=31, matching the article's stated configuration
- Every read in the 10k-read FASTQ files appears exactly once in each classifier output with a C or U classification flag

## Limitations

- Protein-level classification cannot classify reads originating from non-protein-coding genomic regions, lowering Kaiju's sensitivity on short-read mock communities
- Kaiju is less sensitive than nucleotide-level classifiers when sequenced strains are in the reference database, as observed for the HiSeq and MiSeq data sets
- Allowing substitutions in the backwards search causes an exponential runtime increase with the number of allowed mismatches (Greedy-5 is markedly slower than the default 3)

## Evidence

- [discussion] Kraken (v0.10.4b) and Clark (v1.1.3) were run in their default modes using k=31 for highest precision: "Kraken (v0.10.4b) and Clark (v1.1.3) were run in their default modes using k=31 for highest precision"
- [discussion] Kaiju would be less sensitive than nucleotide-level classifiers when strains are in the database: "Kaiju would be less sensitive than nucleotide-level classifiers, which can assess the entire genome, as seen in the HiSeq and MiSeq data sets"
- [methods] The taxon identifiers must be contained in the NCBI taxonomy files nodes.dmp and names.dmp.: "The taxon identifiers must be contained in the [NCBI taxonomy files](ftp://ftp.ncbi.nlm.nih.gov/pub/taxonomy/taxdump.tar.gz) nodes.dmp and names.dmp."
- [readme] Default run mode is Greedy with three allowed mismatches; -e changes it: "The default run mode is **Greedy** with three allowed mismatches. The number of allowed mismatches can be changed using option `-e`."
- [readme] Greedy mode matches filtered by length, score, and E-value with defaults m=11, s=65, E=0.01: "matches are filtered by a minimum length and score, but also by their E-value (similar to blastp), which can be adjusted with the option `-E`. The default value is 0.01."
