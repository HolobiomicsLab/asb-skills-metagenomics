---
name: taxon-rank-abundance-summarization
description: Use when you have per-read classification output (Kaiju tabular output
  with C/U status and NCBI taxon IDs, or Kraken assignments) and need genus- or phylum-level
  counts, sensitivity, and precision summaries.
when_to_use_negative:
- Input is already a rank-level abundance table rather than per-read classifier output
- Reads come from taxa absent from the reference database — genus-level classification
  is impossible for genera with no available reference genomes
- Reads originate from non-protein-coding genomic regions, which protein-level classifiers
  like Kaiju cannot classify in the first place
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: Kaiju
  role: Produces the per-read classified output (Greedy mode, -e 5, -z threads, -o
    output file) whose taxon IDs are then summarized per rank
  repo: https://github.com/bioinformatics-centre/kaiju
- name: kaiju2table
  role: Summarizes Kaiju output files at genus and phylum ranks, requiring nodes.dmp
    and names.dmp
  repo: https://github.com/bioinformatics-centre/kaiju
- name: Kraken v0.10.4b
  role: Comparison classifier run with k=31 whose assignments are tallied to the same
    genus/phylum ranks for sensitivity/precision comparison
- name: Krona
  role: Optional interactive visualization of the summarized taxonomic composition
  repo: https://github.com/marbl/Krona/wiki/KronaTools
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/ncomms11257
    title: Fast and sensitive taxonomic classification for metagenomics with Kaiju
  source_content_hash: sha256:6215d87b1276b1bb9105c9a1ad27150268ce6f2712c3e62d18ece8175450d07c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bioinformatics-centre/kaiju
  tools:
  - Kaiju
  - Kraken v0.10.4b
  - Krona
  - kaiju2table
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# taxon-rank-abundance-summarization

## Summary

Summarizes raw per-read taxonomic classifier output (e.g. Kaiju output files, Kraken assignments) into rank-specific abundance tables at chosen NCBI taxonomy ranks such as genus and phylum, so sensitivity and precision can be computed per rank per classifier.

## When to use

Use when you have per-read classification output (Kaiju tabular output with C/U status and NCBI taxon IDs, or Kraken assignments) and need genus- or phylum-level counts, sensitivity, and precision summaries.

## When NOT to use

- Input is already a rank-level abundance table rather than per-read classifier output
- Reads come from taxa absent from the reference database — genus-level classification is impossible for genera with no available reference genomes
- Reads originate from non-protein-coding genomic regions, which protein-level classifiers like Kaiju cannot classify in the first place

## Inputs

- Kaiju output file (tab-separated, one line per read: C/U flag, read name, NCBI taxon ID)
- Kraken v0.10.4b per-read classification output
- NCBI taxonomy files nodes.dmp and names.dmp
- Kaiju FM-index (.fmi) reference database used for classification

## Outputs

- Genus-rank abundance table from kaiju2table (taxon, read counts per sample)
- Phylum-rank abundance table from kaiju2table
- Sensitivity/precision matrix per classifier per rank per dataset (e.g. benchmark_suppl_table2.tsv)

## How to apply

Run the classifier first (e.g. Kaiju in Greedy mode with m=11, s=65, E=0.01, SEG filtering enabled, five allowed mismatches via -e), producing a tabular output with one line per read where column 1 is C or U and column 3 is the NCBI taxon identifier. Then summarize each output file with kaiju2table at the genus and phylum ranks; kaiju2table requires the NCBI taxonomy files nodes.dmp and names.dmp to resolve each taxon ID to its ancestors at the requested rank. For Kraken output, tally assignments to the same two ranks so the classifiers are comparable. Compute sensitivity as the fraction of reads from known mock-community taxa receiving a classification and precision as the fraction of classified reads assigned to the correct taxon, per classifier, per rank, and per dataset, then assemble the values (e.g. the eight sensitivity/precision values) into a TSV such as benchmark_suppl_table2.tsv.

## Related tools

- **Kaiju** (Produces the per-read classified output (Greedy mode, -e 5, -z threads, -o output file) whose taxon IDs are then summarized per rank) — https://github.com/bioinformatics-centre/kaiju
- **kaiju2table** (Summarizes Kaiju output files at genus and phylum ranks, requiring nodes.dmp and names.dmp) — https://github.com/bioinformatics-centre/kaiju
- **Kraken v0.10.4b** (Comparison classifier run with k=31 whose assignments are tallied to the same genus/phylum ranks for sensitivity/precision comparison)
- **Krona** (Optional interactive visualization of the summarized taxonomic composition) — https://github.com/marbl/Krona/wiki/KronaTools

## Examples

```
kaiju2table -t nodes.dmp -n names.dmp -r genus -o genus_table.tsv kaiju.out
```

## Evaluation signals

- Reproduced genus- and phylum-level sensitivity/precision values match Supplementary Table 2 within rounding tolerance (e.g. HiSeq genus: Kaiju 73.3%/94.4%, Kraken 78.0%/99.2%)
- Every taxon ID in the Kaiju output resolves through nodes.dmp/names.dmp to the requested rank without unmapped IDs
- Sum of classified + unclassified reads in the summary equals the total input read count (e.g. 10k reads per HiSeq/MiSeq mock FASTQ)
- The same rank set (genus, phylum) is tallied identically for Kaiju and Kraken outputs so metrics are comparable
- Output is a well-formed TSV with one sensitivity and one precision value per classifier per rank per dataset (eight values for two classifiers x two ranks x two datasets)

## Limitations

- Protein-level classification cannot classify reads from non-protein-coding genomic regions, capping the sensitivity computed from the summary table
- Kaiju is less sensitive than nucleotide-level classifiers such as Kraken when the sequenced strains are present in the reference database, as seen on the HiSeq and MiSeq mock datasets
- Genus-level classification is impossible for genera with no available reference genomes
- Taxon identifiers must be contained in the NCBI taxonomy files nodes.dmp and names.dmp; out-of-date taxonomy files will break rank resolution

## Evidence

- [intro] Reads are directly assigned to taxa using the NCBI taxonomy and a reference database of protein sequences from microbial and viral genomes.: "Reads are directly assigned to taxa using the NCBI taxonomy and a reference database of protein sequences from microbial and viral genomes."
- [methods] The taxon identifiers must be contained in the NCBI taxonomy files nodes.dmp and names.dmp.: "The taxon identifiers must be contained in the [NCBI taxonomy files](ftp://ftp.ncbi.nlm.nih.gov/pub/taxonomy/taxdump.tar.gz) nodes.dmp and names.dmp."
- [readme] Kaiju will print one line for each read or read pair.: "Kaiju will print one line for each read or read pair."
- [readme] either C or U, indicating whether the read is classified or unclassified.: "1. either C or U, indicating whether the read is classified or unclassified."
- [discussion] Kaiju would be less sensitive than nucleotide-level classifiers, which can assess the entire genome, as seen in the HiSeq and MiSeq data sets: "Kaiju would be less sensitive than nucleotide-level classifiers, which can assess the entire genome, as seen in the HiSeq and MiSeq data sets"
