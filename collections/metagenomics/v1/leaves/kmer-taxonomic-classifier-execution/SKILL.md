---
name: kmer-taxonomic-classifier-execution
description: Use when classifying 10k-read HiSeq or MiSeq mock community FASTQ datasets
  with Kraken (k=31) and Kaiju Greedy-5 to reproduce genus- and phylum-level sensitivity/precision
  benchmarks, or when deciding between k-mer and protein-level classifiers for short
  Illumina reads.
when_to_use_negative:
- 'Sequenced strains are present in the reference database: nucleotide-level classifiers
  outperform Kaiju''s protein-level search on such reads, as the paper observed for
  the HiSeq and MiSeq data sets.'
- Input is already a classification output or abundance table rather than raw FASTQ
  reads — this skill executes classifiers on reads, it does not re-analyze downstream
  tables.
- Genus-level assignment is required for genera with no available reference genomes
  — the paper notes such classification is impossible regardless of classifier.
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0080
tools:
- name: Kaiju
  role: Classifies reads in Greedy mode with -e 5 (Greedy-5); its kaiju2table utility
    summarizes output at genus and phylum ranks using nodes.dmp and names.dmp
  repo: https://github.com/bioinformatics-centre/kaiju
- name: Kraken v0.10.4b
  role: Comparison k-mer-based classifier run in default mode with k=31 for highest
    precision on the same mock community reads
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/ncomms11257
    title: Fast and sensitive taxonomic classification for metagenomics with Kaiju
  source_content_hash: sha256:255843ca03ccecb0ea097a6a78eee2f375e4da135b2b5676ae7cadfe2fa3baee
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
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# kmer-taxonomic-classifier-execution

## Summary

Executes and benchmarks k-mer-based (Kraken v0.10.4b, k=31) versus protein-level (Kaiju Greedy-5, -e 5) taxonomic classifiers on Illumina HiSeq/MiSeq mock community FASTQ reads, summarizing genus- and phylum-level sensitivity and precision for comparison against the paper's Supplementary Table 2.

## When to use

Use when classifying 10k-read HiSeq or MiSeq mock community FASTQ datasets with Kraken (k=31) and Kaiju Greedy-5 to reproduce genus- and phylum-level sensitivity/precision benchmarks, or when deciding between k-mer and protein-level classifiers for short Illumina reads.

## When NOT to use

- Sequenced strains are present in the reference database: nucleotide-level classifiers outperform Kaiju's protein-level search on such reads, as the paper observed for the HiSeq and MiSeq data sets.
- Input is already a classification output or abundance table rather than raw FASTQ reads — this skill executes classifiers on reads, it does not re-analyze downstream tables.
- Genus-level assignment is required for genera with no available reference genomes — the paper notes such classification is impossible regardless of classifier.

## Inputs

- 10k-read HiSeq mock community sequencing FASTQ file
- 10k-read MiSeq mock community sequencing FASTQ file
- Kaiju FM-index reference database (.fmi, e.g. kaiju_db_refseq.fmi)
- NCBI taxonomy files nodes.dmp and names.dmp
- Kraken v0.10.4b database (default k=31)

## Outputs

- Kaiju classification output files (C/U flag, read name, NCBI taxon identifier per read)
- Kraken classification output files per dataset
- kaiju2table genus- and phylum-rank summary tables
- benchmark_suppl_table2.tsv containing eight sensitivity/precision values (2 datasets x 2 classifiers x 2 ranks)

## How to apply

Obtain the 10k-read HiSeq and MiSeq mock community FASTQ files, a Kaiju reference index (.fmi, e.g. built with kaiju-makedb or downloaded pre-built), and the NCBI taxonomy files nodes.dmp and names.dmp. Classify each dataset with Kaiju in its default Greedy mode, raising the allowed mismatches to 5 via -e (Greedy-5) while keeping defaults m=11 (minimum match length), s=65 (minimum score), E=0.01 (E-value filter), and SEG low-complexity filtering enabled, writing output with -o and parallel threads with -z. Classify the identical reads with Kraken v0.10.4b in default mode with k=31, the setting the paper used for highest precision. Summarize Kaiju output with kaiju2table at the genus and phylum ranks (both require nodes.dmp and names.dmp) and tally Kraken assignments to the same two ranks. Compute sensitivity as the fraction of reads from known mock-community taxa that receive a classification and precision as the fraction of classified reads assigned to the correct taxon, per classifier per rank per dataset, and assemble the eight values into benchmark_suppl_table2.tsv.

## Related tools

- **Kaiju** (Classifies reads in Greedy mode with -e 5 (Greedy-5); its kaiju2table utility summarizes output at genus and phylum ranks using nodes.dmp and names.dmp) — https://github.com/bioinformatics-centre/kaiju
- **Kraken v0.10.4b** (Comparison k-mer-based classifier run in default mode with k=31 for highest precision on the same mock community reads)

## Examples

```
kaiju -z 25 -t nodes.dmp -f kaiju_db.fmi -i hiseq_mock_10k.fastq -e 5 -o kaiju_hiseq.out
```

## Evaluation signals

- HiSeq genus-level results match Supplementary Table 2 within rounding tolerance: Kaiju Greedy-5 73.3% sensitivity / 94.4% precision; Kraken 78.0% sensitivity / 99.2% precision.
- The final deliverable contains exactly eight sensitivity/precision values (HiSeq and MiSeq x Kaiju and Kraken x genus and phylum) assembled into benchmark_suppl_table2.tsv.
- Sensitivity is computed only over reads from known mock-community taxa that receive a classification; precision is computed over all classified reads assigned to the correct taxon.
- Kaiju Greedy-5 run keeps defaults m=11, s=65, E=0.01 and SEG filtering enabled, changing only the mismatch allowance to 5 via -e; Kraken runs in default mode with k=31.
- Kaiju output lines begin with C or U and carry an NCBI taxon identifier resolvable against nodes.dmp and names.dmp.

## Limitations

- Raising allowed mismatches to 5 in Greedy mode causes an exponential increase of runtime depending on the number of mismatched positions.
- Protein-level classification cannot classify reads originating from non-protein-coding genomic regions.
- Kaiju is less sensitive than nucleotide-level classifiers when the sequenced strains are in the reference database, as seen in the HiSeq and MiSeq mock community data sets.
- The greedy heuristic restricts substitutions to the ends of seed matches rather than all positions in the query sequence.

## Evidence

- [discussion] Kraken run mode and k-mer setting: "Kraken (v0.10.4b) and Clark (v1.1.3) were run in their default modes using k=31 for highest precision"
- [discussion] Kaiju less sensitive on HiSeq/MiSeq: "Kaiju would be less sensitive than nucleotide-level classifiers, which can assess the entire genome, as seen in the HiSeq and MiSeq data sets"
- [readme] Greedy mode mismatch option: "The default run mode is **Greedy** with three allowed mismatches.
The number of allowed mismatches can be changed using option `-e`."
- [methods] SEG low-complexity filtering: "Option `-x` enables filtering of query sequences containing low-complexity regions by using the SEG algorithm from the blast+ package."
- [methods] Greedy E-value filtering: "matches are filtered by a minimum length and score, but also by their E-value (similar to blastp), which can be adjusted with the option `-E`"
