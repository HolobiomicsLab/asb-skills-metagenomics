---
name: bwt-backwards-search-mem
description: Use when classifying high-throughput sequencing reads (Illumina or Roche/454)
  against a protein reference database at taxon level, especially when nucleotide-level
  k-mer classifiers miss divergent organisms.
when_to_use_negative:
- Reads originate predominantly from non-protein-coding genomic regions, where protein-level
  classification cannot assign them
- The sequenced strains are already in the reference database with nucleotide sequences,
  where nucleotide-level classifiers assessing the entire genome are more sensitive
- Input is already protein sequence and no six-frame translation is wanted (still
  usable via -p, but the translation-centric workflow does not apply)
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: Kaiju
  role: Implements the BWT/FM-index backwards search for protein-level MEMs and LCA
    assignment; kaiju-mkbwt and kaiju-mkfmi build the index
  repo: https://github.com/bioinformatics-centre/kaiju
- name: ART
  role: Simulates HTS reads from reference genomes (whole genome including plasmids)
    to benchmark classification sensitivity and precision
- name: Krona
  role: Interactive visualization of Kaiju classification output
  repo: https://github.com/marbl/Krona
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/ncomms11257
    title: Fast and sensitive taxonomic classification for metagenomics with Kaiju
  source_content_hash: sha256:f36a99fc15a13dbbd13481a50dd83249efb47bfca205a1fa33978f4867b80b6c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bioinformatics-centre/kaiju
  tools:
  - ART
  - Kaiju
  - Krona
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# bwt-backwards-search-mem

## Summary

Kaiju's protein-level taxonomic classification via maximum exact matches (MEMs) found with a backwards search on a Burrows-Wheeler transform/FM-index of a reference protein database, assigning reads to NCBI taxa or their least common ancestor.

## When to use

Use when classifying high-throughput sequencing reads (Illumina or Roche/454) against a protein reference database at taxon level, especially when nucleotide-level k-mer classifiers miss divergent organisms.

## When NOT to use

- Reads originate predominantly from non-protein-coding genomic regions, where protein-level classification cannot assign them
- The sequenced strains are already in the reference database with nucleotide sequences, where nucleotide-level classifiers assessing the entire genome are more sensitive
- Input is already protein sequence and no six-frame translation is wanted (still usable via -p, but the translation-centric workflow does not apply)

## Inputs

- FASTQ or FASTA sequencing reads (single- or paired-end, optionally gzip-compressed)
- Custom protein reference database FASTA with numeric NCBI taxon ID headers
- NCBI taxonomy files nodes.dmp and names.dmp
- Kaiju FM-index file (.fmi) built with kaiju-mkbwt and kaiju-mkfmi

## Outputs

- Tab-separated classification file with three columns: C/U flag, read name, assigned NCBI taxon identifier (7 additional columns with -v)
- Intermediate index files: proteins.bwt, proteins.sa, proteins.fmi

## How to apply

Build the index from a FASTA of protein sequences whose headers carry numeric NCBI taxon IDs (optionally counter-prefixed) using kaiju-mkbwt (-n threads, alphabet ACDEFGHIKLMNPQRSTVWY) followed by kaiju-mkfmi, yielding proteins.bwt, proteins.sa, and proteins.fmi. Translate each read into all six reading frames and split the translations at stop codons into amino acid fragments, then sort fragments by length and query the longest fragment first using the backwards search in the BWT, enforcing the default minimum match length of 11 amino acids (chosen because ~95% of matches to a shuffled NR subset are ≤11 aa). Retrieve the taxon identifier from the suffix array; when equally long matches occur in multiple taxa, assign their least common ancestor from the NCBI taxonomy tree (nodes.dmp). Emit one tab-separated line per read with the C/U flag, read name, and assigned taxon ID; runs simulated with ART from genomes in the database should be flagged C with the correct taxon or a correct LCA ancestor.

## Related tools

- **Kaiju** (Implements the BWT/FM-index backwards search for protein-level MEMs and LCA assignment; kaiju-mkbwt and kaiju-mkfmi build the index) — https://github.com/bioinformatics-centre/kaiju
- **ART** (Simulates HTS reads from reference genomes (whole genome including plasmids) to benchmark classification sensitivity and precision)
- **Krona** (Interactive visualization of Kaiju classification output) — https://github.com/marbl/Krona

## Examples

```
kaiju -t nodes.dmp -f kaiju_db.fmi -i inputfile.fastq -a mem -o kaiju.out
```

## Evaluation signals

- Reads simulated with ART from a genome present in the database are flagged C and assigned that taxon or a correct LCA ancestor
- Reads with no adequate match are flagged U in the first output column
- All assigned taxon identifiers resolve in nodes.dmp and match the numeric IDs encoded in the database FASTA headers
- Every output line contains exactly three tab-separated fields (C/U, read name, taxon ID) in the default output layout
- Matched fragments satisfy the minimum match length of 11 amino acids in MEM mode

## Limitations

- Protein-level classification cannot classify reads from non-protein-coding genomic regions
- Kaiju is less sensitive than nucleotide-level classifiers when the sequenced strains are in the reference database
- Allowing substitutions in the backwards search causes an exponential runtime increase with the number of allowed mismatches, restricting the greedy heuristic to match ends
- Protein sequences in a custom database may only contain the uppercase 20 standard amino acids; all other characters must be removed

## Evidence

- [discussion] Translate reads into six frames and split at stop codons: "Kaiju translates each read into the six possible reading frames, which are then split at stop codons into amino acid fragments."
- [discussion] Backwards search ordering and MEM: "These fragments are sorted by length, and, beginning with the longest fragment, queried against the reference database using the backwards search in the BWT."
- [discussion] Suffix array taxon retrieval and LCA: "If equally long matches are found in multiple taxa, Kaiju determines their LCA from the taxonomic tree"
- [discussion] Default m=11 rationale: "one can observe that ∼95% of them are ≤11 amino acids long"
- [readme] Custom database index construction: "Kaiju's index is created using the programs `kaiju-mkbwt` and `kaiju-mkfmi`"
- [readme] Custom FASTA header format: "the headers are the numeric NCBI taxon identifiers of the protein sequences, which can optionally be prefixed by another identifier (e.g. a counter) followed by an underscore"
- [readme] Output format: "1. either C or U, indicating whether the read is classified or unclassified.
2. name of the read
3. NCBI taxon identifier of the assigned taxon"
