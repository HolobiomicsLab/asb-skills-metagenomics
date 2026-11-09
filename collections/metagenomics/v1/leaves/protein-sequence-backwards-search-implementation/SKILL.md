---
name: protein-sequence-backwards-search-implementation
description: Use when metagenomic reads must be classified at the protein level with
  inexact matching — e.g., when genera are underrepresented in the reference database
  and exact-match (MEM) classifiers miss too many reads.
when_to_use_negative:
- Reads originate from non-protein-coding genomic regions — protein-level classification
  cannot classify these reads
- The sequenced strains are present in the reference database and maximum nucleotide-level
  sensitivity is required — nucleotide-level classifiers outperform Kaiju in that
  setting
- The goal is exact-match-only classification speed — MEM mode (-a mem) avoids the
  exponential runtime increase from allowing mismatches in the backwards search
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: Kaiju
  role: Primary classifier whose Greedy mode is implemented/modified; performs FM-index
    backwards search over the BWT of a protein reference database and assigns NCBI
    taxa
  repo: https://github.com/bioinformatics-centre/kaiju
- name: ART
  role: Simulates HTS reads from reference genomes to generate benchmark FASTQ inputs
    for comparing Greedy vs. MEM classification
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/ncomms11257
    title: Fast and sensitive taxonomic classification for metagenomics with Kaiju
  source_content_hash: sha256:b0b7d6e424afca7fb4abde4107788d03a526ba8908b37b6a5066ae61237c6d90
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bioinformatics-centre/kaiju
  tools:
  - ART
  - Kaiju
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# protein-sequence-backwards-search-implementation

## Summary

Implements and runs Kaiju's Greedy search mode, which locates minimum-length seed matches (default 7 aa) in a protein-level FM-index via backwards search over the BWT and extends them with up to a specified number of amino acid substitutions scored by the BLOSUM62 matrix, enabling inexact protein-level taxonomic classification beyond exact MEM matches.

## When to use

Use when metagenomic reads must be classified at the protein level with inexact matching — e.g., when genera are underrepresented in the reference database and exact-match (MEM) classifiers miss too many reads.

## When NOT to use

- Reads originate from non-protein-coding genomic regions — protein-level classification cannot classify these reads
- The sequenced strains are present in the reference database and maximum nucleotide-level sensitivity is required — nucleotide-level classifiers outperform Kaiju in that setting
- The goal is exact-match-only classification speed — MEM mode (-a mem) avoids the exponential runtime increase from allowing mismatches in the backwards search

## Inputs

- Simulated or real HTS reads in FASTQ (or FASTA, optionally gzip-compressed), e.g., Illumina reads simulated with ART
- Kaiju protein reference database index file kaiju_db.fmi built by kaiju-mkbwt and kaiju-mkfmi (or kaiju-makedb)
- NCBI taxonomy files nodes.dmp and names.dmp

## Outputs

- Kaiju output file with per-read lines: C/U classification flag, read name, NCBI taxon identifier (plus score, matching taxa, accession numbers, and fragment sequences with -v)
- Comparison table (e.g., greedy_vs_mem_comparison.tsv) of classified read names, assigned taxon IDs, per-mode classification rates, and Greedy-vs-MEM agreement/sensitivity

## How to apply

Locate the backwards-search routines in the Kaiju C/C++ source (kaiju/src) and implement Greedy mode: enumerate all MEMs of minimum seed length 7 via the FM-index backwards search over the BWT of the translated six-frame query fragments, then extend each seed at its left end allowing substitutions (default 3, adjustable via -e), scoring the extension with BLOSUM62. Compile with 'make' in kaiju/src to produce executables in kaiju/bin. Run the Greedy binary with -t nodes.dmp -f kaiju_db.fmi -i reads.fastq -o out, ranking translated fragments by cumulative BLOSUM62 score, querying the highest-scoring fragment first, and terminating once a match exceeds the minimum score s (default 65); filter matches by E-value (default 0.01 via -E) and minimum match length m (default 11). Assign LCA taxon IDs for equally long matches in multiple taxa, and validate by comparing per-mode classification rates and read-level taxon agreement against the MEM mode (kaiju -a mem) on the same simulated reads — Greedy should classify at least as many reads at comparable precision, consistent with the paper's finding that Greedy yields higher sensitivity.

## Related tools

- **Kaiju** (Primary classifier whose Greedy mode is implemented/modified; performs FM-index backwards search over the BWT of a protein reference database and assigns NCBI taxa) — https://github.com/bioinformatics-centre/kaiju
- **ART** (Simulates HTS reads from reference genomes to generate benchmark FASTQ inputs for comparing Greedy vs. MEM classification)

## Examples

```
kaiju -t nodes.dmp -f kaiju_db.fmi -i reads.fastq -o kaiju.out -z 25 -e 3 -s 65 -E 0.01 -m 11
```

## Evaluation signals

- All classifications resolve to valid taxa present in nodes.dmp (no taxon IDs outside the reference taxonomy)
- Greedy-mode classification rate is at least as high as MEM mode (-a mem) at comparable precision, matching the paper's 'higher sensitivity' finding
- Matches pass filters: minimum length m=11, minimum BLOSUM62 score s=65, and E-value ≤ 0.01
- Output lines follow the Kaiju 3-column (or 7-column with -v) tab-separated schema with C/U flags and NCBI taxon IDs
- Runtime scales acceptably: allowing more mismatches (-e > 3) should show the documented exponential runtime increase, confirming the substitution extension is active

## Limitations

- Allowing substitutions in the backwards search causes an exponential runtime increase depending on the number of allowed mismatched positions; hence substitutions are restricted to match ends via the greedy heuristic
- Protein-level classification cannot classify reads from non-protein-coding genomic regions
- Genus-level classification is impossible for genera with no available reference genomes; Kaiju is less sensitive than nucleotide-level classifiers when sequenced strains are in the database

## Evidence

- [discussion] Greedy seed extension with BLOSUM62: "which first locates all MEMs of a minimum seed length (default 7) and then extends them by allowing substitutions at the left ends of each seed match"
- [discussion] Fragment ranking by BLOSUM62 score: "we rank the fragments by their BLOSUM62 score and start the database search with the highest scoring fragment"
- [readme] Default Greedy parameters: "The default run mode is **Greedy** with three allowed mismatches. The number of allowed mismatches can be changed using option `-e`."
- [readme] E-value and score cutoffs: "matches are filtered by a minimum length and score, but also by their E-value (similar to blastp), which can be adjusted with the option `-E`. The default value is 0.01."
- [discussion] Runtime limitation of inexact backwards search: "extend the match at the cost of an exponential increase of runtime depending on the number of allowed mismatched positions"
