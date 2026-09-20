---
name: greedy-inexact-match-classification-tuning
description: Use when classifying FASTQ/FASTA metagenomic reads with Kaiju's Greedy
  mode and you must tune mismatch allowance, match length/score cutoffs, or E-value
  filtering to balance sensitivity and precision against k-mer classifiers like Kraken
  (k=31).
when_to_use_negative:
- Reads originate largely from non-protein-coding genomic regions — Kaiju's protein-level
  classification cannot assign them, so a nucleotide-level classifier is the wrong-tool
  alternative
- The sequenced strains are already in a nucleotide reference database — Kraken-like
  nucleotide classifiers assess the entire genome and will be more sensitive than
  Kaiju on the HiSeq and MiSeq data sets
- You need to allow many substitutions across all query positions — the Greedy backwards
  search restricts substitutions to match ends and runtime grows exponentially with
  the number of allowed mismatches
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3168
tools:
- name: Kaiju
  role: Greedy-mode classifier whose -e, -m, -s, -E, -x, -z and -o options are tuned
    by this skill; finds maximum (in-)exact protein-level matches using the Burrows-Wheeler
    transform
  repo: https://github.com/bioinformatics-centre/kaiju
- name: Kraken v0.10.4b
  role: k-mer-based comparison baseline, run in default mode with k=31 for highest
    precision to benchmark Greedy-mode sensitivity/precision
- name: kaiju2table
  role: Summarizes Kaiju output into genus- and phylum-level tables (requires nodes.dmp
    and names.dmp) for computing sensitivity and precision
  repo: https://github.com/bioinformatics-centre/kaiju
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/ncomms11257
    title: Fast and sensitive taxonomic classification for metagenomics with Kaiju
  source_content_hash: sha256:d06ab50f979249fec0ef25b1620817703d91a03b7833307130a0446065bf7b1e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bioinformatics-centre/kaiju
  tools:
  - Kaiju
  - Kraken v0.10.4b
  - kaiju2table
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# greedy-inexact-match-classification-tuning

## Summary

Tunes Kaiju's Greedy run mode for taxonomic classification of metagenomic sequencing reads, adjusting the allowed mismatches (-e), minimum match length (-m, default 11), match score (-s, default 65), and E-value cutoff (-E, default 0.01) to trade sensitivity against precision on protein-level matches found via the BWT.

## When to use

Use when classifying FASTQ/FASTA metagenomic reads with Kaiju's Greedy mode and you must tune mismatch allowance, match length/score cutoffs, or E-value filtering to balance sensitivity and precision against k-mer classifiers like Kraken (k=31).

## When NOT to use

- Reads originate largely from non-protein-coding genomic regions — Kaiju's protein-level classification cannot assign them, so a nucleotide-level classifier is the wrong-tool alternative
- The sequenced strains are already in a nucleotide reference database — Kraken-like nucleotide classifiers assess the entire genome and will be more sensitive than Kaiju on the HiSeq and MiSeq data sets
- You need to allow many substitutions across all query positions — the Greedy backwards search restricts substitutions to match ends and runtime grows exponentially with the number of allowed mismatches

## Inputs

- HiSeq or MiSeq mock-community sequencing reads in FASTQ (or FASTA, optionally gzip-compressed)
- Kaiju reference index file (.fmi), e.g. built by kaiju-makedb or kaiju-mkbwt/kaiju-mkfmi
- NCBI taxonomy files nodes.dmp and names.dmp

## Outputs

- Kaiju classification output file (columns: C/U status, read name, NCBI taxon identifier, plus match score/taxon/accession columns with -v)
- Genus- and phylum-level sensitivity and precision tables (e.g. benchmark_suppl_table2.tsv) via kaiju2table

## How to apply

Run Kaiju in its default Greedy mode, which first locates all MEMs of a minimum seed length (default 7) and extends them by allowing substitutions at the left ends of each seed match, searching BLOSUM62-ranked amino-acid fragments from the six-frame translation of each read. Keep the default filters unless the benchmark calls for loosening them: m=11 minimum match length, s=65 minimum BLOSUM62 score, E=0.01 E-value threshold, and SEG low-complexity filtering enabled by default via -x. Raise the allowed mismatches with -e (e.g. from the default 3 to 5) when reproducing the HiSeq/MiSeq mock-community comparison, and parallelize with -z while writing results with -o. Always build or download a Kaiju .fmi index and keep the NCBI taxonomy files nodes.dmp and names.dmp at hand for taxon assignment. For validation, summarize output with kaiju2table at genus and phylum ranks and compute sensitivity/precision on known mock-community taxa, benchmarking against Kraken v0.10.4b run in default mode with k=31 for highest precision.

## Related tools

- **Kaiju** (Greedy-mode classifier whose -e, -m, -s, -E, -x, -z and -o options are tuned by this skill; finds maximum (in-)exact protein-level matches using the Burrows-Wheeler transform) — https://github.com/bioinformatics-centre/kaiju
- **Kraken v0.10.4b** (k-mer-based comparison baseline, run in default mode with k=31 for highest precision to benchmark Greedy-mode sensitivity/precision)
- **kaiju2table** (Summarizes Kaiju output into genus- and phylum-level tables (requires nodes.dmp and names.dmp) for computing sensitivity and precision) — https://github.com/bioinformatics-centre/kaiju

## Examples

```
kaiju -z 25 -t nodes.dmp -f kaiju_db.fmi -i inputfile.fastq -e 5 -o kaiju.out
```

## Evaluation signals

- Reproduced HiSeq genus-level values match Supplementary Table 2 within rounding tolerance: Kaiju Greedy-5 73.3% sensitivity / 94.4% precision and Kraken k=31 78.0% sensitivity / 99.2% precision
- Greedy run uses the default filters unless deliberately changed: m=11 minimum match length, s=65 minimum score, E=0.01 E-value, SEG filtering enabled via -x
- Output file contains one line per read/pair with C or U classification status, read name, and an NCBI taxon identifier in the specified column order
- Number of allowed mismatches (-e) is explicitly recorded; changing it from the default 3 (e.g. to 5) should raise genus-level sensitivity at some precision cost
- Precision computed only over classified reads, sensitivity only over reads from known mock-community taxa, per rank (genus, phylum) per dataset

## Limitations

- Extending matches by allowing substitutions causes an exponential runtime increase with the number of allowed mismatched positions
- The greedy heuristic only introduces substitutions at the left ends of seed matches rather than at all query positions
- Protein-level classification cannot classify reads from non-protein-coding genomic regions
- Kaiju is less sensitive than nucleotide-level classifiers when the sequenced strains are present in the reference database, as seen in the HiSeq and MiSeq data sets

## Evidence

- [readme] The default run mode is Greedy with three allowed mismatches. The number of allowed mismatches can be changed using option -e.: "The default run mode is **Greedy** with three allowed mismatches.
The number of allowed mismatches can be changed using option `-e`."
- [readme] The cutoffs for minimum required match length and match score can be changed using the options -m (default: 11) and -s (default: 65).: "The cutoffs for minimum required match length and match score can be changed using the options `-m` (default: 11) and `-s` (default: 65)."
- [methods] Option -x enables filtering of query sequences containing low-complexity regions by using the SEG algorithm from the blast+ package.: "Option `-x` enables filtering of query sequences containing low-complexity regions by using the SEG algorithm from the blast+ package."
- [discussion] we chose s=65 as default value for Kaiju's Greedy mode: "we chose s=65 as default value for Kaiju's Greedy mode"
- [discussion] Greedy mode first locates all MEMs of a minimum seed length (default 7) and then extends them by allowing substitutions at the left ends of each seed match.: "which first locates all MEMs of a minimum seed length (default 7) and then extends them by allowing substitutions at the left ends of each seed match"
- [discussion] Kraken (v0.10.4b) and Clark (v1.1.3) were run in their default modes using k=31 for highest precision: "Kraken (v0.10.4b) and Clark (v1.1.3) were run in their default modes using k=31 for highest precision"
- [discussion] Kaiju would be less sensitive than nucleotide-level classifiers, which can assess the entire genome, as seen in the HiSeq and MiSeq data sets: "Kaiju would be less sensitive than nucleotide-level classifiers, which can assess the entire genome, as seen in the HiSeq and MiSeq data sets"
- [discussion] extend the match at the cost of an exponential increase of runtime depending on the number of allowed mismatched positions: "extend the match at the cost of an exponential increase of runtime depending on the number of allowed mismatched positions"
