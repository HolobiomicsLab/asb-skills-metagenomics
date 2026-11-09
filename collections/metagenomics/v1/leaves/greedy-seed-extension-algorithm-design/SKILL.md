---
name: greedy-seed-extension-algorithm-design
description: Use when implementing or modifying inexact-match search in a BWT/FM-index-based
  metagenomic classifier, specifically seeding with exact MEMs and extending with
  BLOSUM62-scored substitutions at match ends under a score threshold.
when_to_use_negative:
- Reads originate from non-protein-coding genomic regions, where protein-level classification
  is inherently unable to classify them
- The sequenced strains are already present in the reference database, where nucleotide-level
  classifiers are more sensitive than this protein-level inexact matching
- A large number of allowed mismatches is required, since permitting substitutions
  in the backwards search causes an exponential runtime increase
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Kaiju
  role: The C/C++ program whose backwards-search routines in kaiju/src implement the
    Greedy seed-extension mode; run in Greedy mode (default) and MEM mode (-a mem)
    for comparison
  repo: https://github.com/bioinformatics-centre/kaiju
- name: ART
  role: Simulates HTS reads used as FASTQ input to benchmark Greedy-mode sensitivity
    against MEM mode
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/ncomms11257
    title: Fast and sensitive taxonomic classification for metagenomics with Kaiju
  source_content_hash: sha256:0eac25f634d0ddc8429a7c8c0249569eaf7d4089c683c36e8ff14491b303a9f8
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
yes_rate: 0.7142857142857143
n_claims: 7
---

# greedy-seed-extension-algorithm-design

## Summary

A skill for implementing Kaiju's Greedy search mode, which extends exact-match seeds (MEMs of minimum length 7) found via FM-index backwards search by allowing a limited number of amino acid substitutions scored with BLOSUM62, enabling inexact protein-level read classification beyond exact matches.

## When to use

Use when implementing or modifying inexact-match search in a BWT/FM-index-based metagenomic classifier, specifically seeding with exact MEMs and extending with BLOSUM62-scored substitutions at match ends under a score threshold.

## When NOT to use

- Reads originate from non-protein-coding genomic regions, where protein-level classification is inherently unable to classify them
- The sequenced strains are already present in the reference database, where nucleotide-level classifiers are more sensitive than this protein-level inexact matching
- A large number of allowed mismatches is required, since permitting substitutions in the backwards search causes an exponential runtime increase

## Inputs

- Simulated or real Illumina/454 metagenomic sequencing reads in FASTQ (or FASTA, optionally gzip-compressed)
- Kaiju FM-index file (kaiju_db.fmi) built from a reference protein database (e.g., kaiju-mkbwt -a ACDEFGHIKLMNPQRSTVWY then kaiju-mkfmi)
- NCBI taxonomy files nodes.dmp and names.dmp containing valid taxon identifiers

## Outputs

- Tab-separated classification file (C/U status, read name, NCBI taxon identifier; verbose mode adds match score, taxon and accession lists, and matching fragment sequences)
- Per-mode classification-rate and agreement summary (e.g., greedy_vs_mem_comparison.tsv)

## How to apply

First enumerate all MEMs of minimum seed length 7 by backwards search over the BWT of the protein reference index. Extend each seed at its left end by allowing substitutions (default 3 allowed mismatches, adjustable via -e, up to ~5), scoring the extension with the BLOSUM62 substitution matrix; this greedy restriction of substitutions to match ends is required because allowing them at all query positions causes an exponential runtime increase. Translate reads into six reading frames, split at stop codons, rank the resulting amino acid fragments by cumulative BLOSUM62 score, and query the highest-scoring fragment first, terminating once a match exceeds the minimum score threshold s (default 65, adjustable via -s) and passes the E-value filter (default 0.01 via -E). Assign the taxon from the suffix array, resolving equally long matches across taxa by LCA in the NCBI taxonomy tree. Compile with 'make' in kaiju/src and validate against the MEM mode (kaiju -a mem) by comparing classified read names and NCBI taxon IDs on the same simulated FASTQ input.

## Related tools

- **Kaiju** (The C/C++ program whose backwards-search routines in kaiju/src implement the Greedy seed-extension mode; run in Greedy mode (default) and MEM mode (-a mem) for comparison) — https://github.com/bioinformatics-centre/kaiju
- **ART** (Simulates HTS reads used as FASTQ input to benchmark Greedy-mode sensitivity against MEM mode)

## Examples

```
cd kaiju/src && make && kaiju -a greedy -t nodes.dmp -f kaiju_db.fmi -i reads.fastq -s 65 -e 3 -E 0.01 -z 25 -o kaiju_greedy.out
```

## Evaluation signals

- Greedy mode classifies at least as many reads as MEM mode at comparable precision on the same simulated-read FASTQ and kaiju_db.fmi index, consistent with 'Greedy run mode yields a higher sensitivity compared with MEM mode'
- All reported classifications resolve to taxon identifiers present in the reference taxonomy in nodes.dmp
- Defaults are in force: minimum seed length 7, score threshold s=65 (-s), 3 allowed mismatches (-e), E-value filter 0.01 (-E), minimum match length m=11 (-m)
- Reads with equally long matches in multiple taxa are assigned the LCA of those taxa rather than a single leaf taxon
- The modified binary compiles cleanly with 'cd kaiju/src; make' and produces executables in kaiju/bin

## Limitations

- Allowing substitutions during the backwards search causes an exponential runtime increase in the number of allowed mismatched positions, motivating the greedy restriction of substitutions to match ends
- The greedy heuristic restricts substitutions to the ends of matches rather than all query positions, so mismatches elsewhere in the read are missed
- Protein-level classification cannot classify reads originating from non-protein-coding genomic regions
- Kaiju is less sensitive than nucleotide-level classifiers when the sequenced strains are in the reference database

## Evidence

- [discussion] Greedy mode seeds then extend with substitutions at left ends: "which first locates all MEMs of a minimum seed length (default 7) and then extends them by allowing substitutions at the left ends of each seed match"
- [discussion] Fragments ranked by BLOSUM62 score and searched highest-scoring first: "we rank the fragments by their BLOSUM62 score and start the database search with the highest scoring fragment"
- [discussion] Runtime exponentially increases with allowed mismatches: "extend the match at the cost of an exponential increase of runtime depending on the number of allowed mismatched positions"
- [readme] Default Greedy mode with three allowed mismatches; -s and -E options: "The default run mode is **Greedy** with three allowed mismatches. The number of allowed mismatches can be changed using option `-e`."
- [readme] Greedy mode defaults for score and E-value cutoffs: "The cutoffs for minimum required match length and match score can be changed using the options `-m` (default: 11) and `-s` (default: 65)."
