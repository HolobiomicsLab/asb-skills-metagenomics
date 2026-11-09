---
name: suffix-array-taxon-lookup
description: Use when implementing or verifying protein-level read classifiers in
  the Kaiju style, where MEM hits found via BWT backwards search must be mapped to
  NCBI taxon IDs and ambiguous hits collapsed to an LCA.
when_to_use_negative:
- Reads originate from non-protein-coding genomic regions — protein-level classification
  cannot assign them (an acknowledged Kaiju limitation)
- Input sequences are already proteins and you only need match positions, not taxon
  resolution via the suffix array and taxonomy tree
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Kaiju
  role: Implements the BWT/FM-index backwards search whose suffix array yields the
    taxon identifier; determines the LCA for equally long multi-taxon matches
  repo: https://github.com/bioinformatics-centre/kaiju
- name: ART
  role: Simulates HTS reads from genomes in the reference database for verification
    of taxon assignments
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/ncomms11257
    title: Fast and sensitive taxonomic classification for metagenomics with Kaiju
  source_content_hash: sha256:45ec3d7cab3a0824f88fa8c64404d2a947f9eb1d1bbf15a7abc763d10dbdec6a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bioinformatics-centre/kaiju
  tools:
  - ART
  - Kaiju
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# suffix-array-taxon-lookup

## Summary

Recover the NCBI taxon identifier of a matched database sequence from Kaiju's suffix array after a BWT backwards search, and resolve equally long matches across multiple taxa to their least common ancestor (LCA) in the NCBI taxonomic tree (nodes.dmp). This step converts raw FM-index matches into a taxonomic assignment per read.

## When to use

Use when implementing or verifying protein-level read classifiers in the Kaiju style, where MEM hits found via BWT backwards search must be mapped to NCBI taxon IDs and ambiguous hits collapsed to an LCA.

## When NOT to use

- Reads originate from non-protein-coding genomic regions — protein-level classification cannot assign them (an acknowledged Kaiju limitation)
- Input sequences are already proteins and you only need match positions, not taxon resolution via the suffix array and taxonomy tree

## Inputs

- Backwards-search MEM results against a Kaiju FM-index (proteins.fmi) with companion suffix array (proteins.sa)
- NCBI taxonomy files nodes.dmp (and names.dmp) from taxdump.tar.gz
- FASTQ/FASTA sequencing reads translated into six-frame amino-acid fragments split at stop codons

## Outputs

- One line per read in Kaiju's default three-column tab-separated format: C/U classification flag, read name, assigned NCBI taxon identifier
- LCA taxon identifier for reads with equally long matches in multiple taxa

## How to apply

After querying the longest amino-acid fragment of a six-frame translation against the FM-index (enforcing the default minimum match length m=11), use the suffix array companion file (proteins.sa, built by kaiju-mkbwt then kaiju-mkfmi) to retrieve the taxon identifier of the database sequence containing the maximum exact match. If equally long matches occur in sequences assigned to multiple taxa, resolve them to their least common ancestor using the NCBI taxonomy nodes.dmp file. Emit the result per read as Kaiju's three-column tab-separated output: C/U flag, read name, and assigned NCBI taxon identifier. Correctness can be checked by confirming that reads simulated (e.g., with ART) from a genome in the reference database are classified (C) to that taxon or a correct ancestor, while non-matching reads are flagged U.

## Related tools

- **Kaiju** (Implements the BWT/FM-index backwards search whose suffix array yields the taxon identifier; determines the LCA for equally long multi-taxon matches) — https://github.com/bioinformatics-centre/kaiju
- **ART** (Simulates HTS reads from genomes in the reference database for verification of taxon assignments)

## Examples

```
kaiju -t nodes.dmp -f proteins.fmi -i reads.fastq -o kaiju_simulated.out
```

## Evaluation signals

- Reads simulated with ART from a genome present in the reference database are flagged C and assigned that taxon or a correct LCA ancestor
- Reads with no adequate match are flagged U in the output's first column
- Every output line has exactly three tab-separated columns (C/U flag, read name, NCBI taxon ID) in the default mode
- Assignments for ambiguous multi-taxon matches equal the least common ancestor computed from nodes.dmp
- All emitted taxon identifiers exist in the NCBI taxonomy files used to build the index

## Limitations

- Protein-level classification cannot classify reads from non-protein-coding genomic regions
- Genus-level classification is impossible for genera with no available reference genomes
- Kaiju is less sensitive than nucleotide-level classifiers when the sequenced strain is already in the reference database

## Evidence

- [discussion] the taxon identifier from the corresponding database sequence is retrieved from the suffix array and printed to the output.: "the taxon identifier from the corresponding database sequence is retrieved from the suffix array and printed to the output."
- [discussion] If equally long matches are found in multiple taxa, Kaiju determines their LCA from the taxonomic tree: "If equally long matches are found in multiple taxa, Kaiju determines their LCA from the taxonomic tree"
- [methods] The taxon identifiers must be contained in the NCBI taxonomy files nodes.dmp and names.dmp.: "The taxon identifiers must be contained in the [NCBI taxonomy files](ftp://ftp.ncbi.nlm.nih.gov/pub/taxonomy/taxdump.tar.gz) nodes.dmp and names.dmp."
- [readme] 1. either C or U, indicating whether the read is classified or unclassified. 2. name of the read 3. NCBI taxon identifier of the assigned taxon: "1. either C or U, indicating whether the read is classified or unclassified.
2. name of the read
3. NCBI taxon identifier of the assigned taxon"
