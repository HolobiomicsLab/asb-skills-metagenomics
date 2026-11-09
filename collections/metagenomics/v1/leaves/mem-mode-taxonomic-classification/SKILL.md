---
name: mem-mode-taxonomic-classification
description: Use when classifying Illumina or Roche/454 metagenomic sequencing reads
  against a protein reference database (e.g. NCBI nr microbial subset) and maximum
  exact match sensitivity is preferred over k-mer-based classifiers like Kraken or
  Clark.
when_to_use_negative:
- Sequenced strains are present in the reference database and nucleotide-level classifiers
  can assess the entire genome — Kaiju is then less sensitive than nucleotide-level
  approaches
- Reads originate from non-protein-coding genomic regions, which protein-level classification
  cannot assign
- The target genus has no available reference genomes — genus-level classification
  of the reads is impossible
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0080
tools:
- name: Kaiju
  role: Performs MEM-mode classification via backwards search on the BWT/FM-index
    of the protein reference database
  repo: https://github.com/bioinformatics-centre/kaiju
- name: kaiju-mkbwt
  role: Constructs the BWT of the protein database with alphabet ACDEFGHIKLMNPQRSTVWY
    prior to indexing
  repo: https://github.com/bioinformatics-centre/kaiju
- name: kaiju-mkfmi
  role: Converts the BWT into the FM-index (.fmi) consumed by the classifier
  repo: https://github.com/bioinformatics-centre/kaiju
- name: uShuffle
  role: Generates a shuffled null-model database (window 100 aa) to validate the m=11
    cutoff
- name: Krona
  role: Interactive visualization of Kaiju classification output
  repo: https://github.com/marbl/Krona/wiki/KronaTools
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/ncomms11257
    title: Fast and sensitive taxonomic classification for metagenomics with Kaiju
  source_content_hash: sha256:bc7acc4e319b92be6e2c97c769d38723eb60fd8547f3d702293d88b9c4fe6d6e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bioinformatics-centre/kaiju
  tools:
  - ART
  - Kaiju
  - Krona
  - R
  - ggplot2
  - kaiju-mkbwt
  - kaiju-mkfmi
  - uShuffle
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# mem-mode-taxonomic-classification

## Summary

Classify metagenomic reads taxonomically using Kaiju's MEM run mode, which finds maximum (in-)exact matches on the protein level via a Burrows-Wheeler transform index, with default minimum match length m=11.

## When to use

Use when classifying Illumina or Roche/454 metagenomic sequencing reads against a protein reference database (e.g. NCBI nr microbial subset) and maximum exact match sensitivity is preferred over k-mer-based classifiers like Kraken or Clark.

## When NOT to use

- Sequenced strains are present in the reference database and nucleotide-level classifiers can assess the entire genome — Kaiju is then less sensitive than nucleotide-level approaches
- Reads originate from non-protein-coding genomic regions, which protein-level classification cannot assign
- The target genus has no available reference genomes — genus-level classification of the reads is impossible

## Inputs

- FASTQ or FASTA metagenomic read files (optionally gzip-compressed; paired-end via -i and -j)
- Kaiju FM-index (.fmi) built with kaiju-mkbwt/kaiju-mkfmi from a protein FASTA with NCBI taxon-ID headers
- NCBI taxonomy files nodes.dmp and names.dmp

## Outputs

- Tab-separated classification file with one line per read: C/U status, read name, NCBI taxon ID; with -v, additional columns including best-match length (column 4), matching taxon IDs, accession numbers, and fragment sequences
- Optionally an interactive Krona visualization of the taxonomic assignments

## How to apply

Build a Kaiju index from a protein FASTA (headers are NCBI taxon IDs) using kaiju-mkbwt with alphabet ACDEFGHIKLMNPQRSTVWY followed by kaiju-mkfmi. Run Kaiju with -a mem to activate MEM mode; each read is translated into six reading frames, split at stop codons, and fragments are queried longest-first by backwards search in the BWT, with equally long matches in multiple taxa resolved to their LCA. Keep the default minimum match length m=11: shuffling the microbial NR protein database with uShuffle (window 100 aa) showed ~95% of spurious matches are ≤11 amino acids, while raising m to 12 loses more precision than the sensitivity gained on genus and phylum levels. Use SEG low-complexity filtering (default on) to avoid false positives from repeats, and run with -v to extract the best-match length from column 4 of the verbose output for quality checks.

## Related tools

- **Kaiju** (Performs MEM-mode classification via backwards search on the BWT/FM-index of the protein reference database) — https://github.com/bioinformatics-centre/kaiju
- **kaiju-mkbwt** (Constructs the BWT of the protein database with alphabet ACDEFGHIKLMNPQRSTVWY prior to indexing) — https://github.com/bioinformatics-centre/kaiju
- **kaiju-mkfmi** (Converts the BWT into the FM-index (.fmi) consumed by the classifier) — https://github.com/bioinformatics-centre/kaiju
- **uShuffle** (Generates a shuffled null-model database (window 100 aa) to validate the m=11 cutoff)
- **Krona** (Interactive visualization of Kaiju classification output) — https://github.com/marbl/Krona/wiki/KronaTools

## Examples

```
kaiju -t nodes.dmp -f kaiju_db.fmi -i inputfile.fastq -a mem -v -o kaiju.out
```

## Evaluation signals

- ~95% of matches against a uShuffle-shuffled database (window 100 aa) should be ≤11 amino acids long
- In the null-model comparison, >75% of wrong classifications but only ~2% of correct ones have match lengths ≤11 aa (Supplementary Fig. 5)
- Raising m from 11 to 12 should lose more precision than the sensitivity gained on both genus and phylum levels
- Output is one line per read/pair; classified lines start with 'C' and carry a valid NCBI taxon ID resolvable via nodes.dmp
- Classification throughput should reach millions of reads per minute with memory footprint below 6 GB on the benchmark database

## Limitations

- Protein-level classification cannot classify reads from non-protein-coding genomic regions
- Allowing substitutions in the backwards search causes an exponential runtime increase with the number of allowed mismatches
- Kaiju is less sensitive than nucleotide-level classifiers when the sequenced strain is in the reference database
- Genus-level classification is impossible for genera with no available reference genomes

## Evidence

- [discussion] Kaiju translates each read into the six possible reading frames, which are then split at stop codons into amino acid fragments.: "Kaiju translates each read into the six possible reading frames, which are then split at stop codons into amino acid fragments."
- [discussion] one can observe that ∼95% of them are ≤11 amino acids long: "one can observe that ∼95% of them are ≤11 amino acids long"
- [discussion] we created a shuffled version of the microbial subset of NCBI's NR protein database, using uShuffle25 with a window length of 100 amino acids: "we created a shuffled version of the microbial subset of NCBI's NR protein database, using uShuffle25 with a window length of 100 amino acids"
- [readme] The run mode can be changed to MEM using option -a:: "The run mode can be changed to **MEM** using option `-a`:"
- [readme] kaiju-mkbwt -n 5 -a ACDEFGHIKLMNPQRSTVWY -o proteins proteins.faa: "kaiju-mkbwt -n 5 -a ACDEFGHIKLMNPQRSTVWY -o proteins proteins.faa
kaiju-mkfmi proteins"
- [discussion] If equally long matches are found in multiple taxa, Kaiju determines their LCA from the taxonomic tree: "If equally long matches are found in multiple taxa, Kaiju determines their LCA from the taxonomic tree"
