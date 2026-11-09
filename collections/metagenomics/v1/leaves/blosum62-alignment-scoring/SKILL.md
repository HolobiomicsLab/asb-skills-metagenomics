---
name: blosum62-alignment-scoring
description: Use when implementing or tuning inexact protein-level search that extends
  exact seed matches with amino acid substitutions and needs a substitution-matrix
  score (BLOSUM62) to rank and filter candidate matches.
when_to_use_negative:
- Input reads originate from non-protein-coding genomic regions — protein-level classification
  cannot classify them
- The sequenced strains are already in the reference database — nucleotide-level classifiers
  will be more sensitive than this protein-level substitution-matrix approach
- Only exact maximum-length matches are needed — MEM mode (kaiju -a mem) is faster
  since it avoids substitution extensions and their runtime cost
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0194
tools:
- name: Kaiju
  role: 'Implements the Greedy run mode: BLOSUM62-scored substitution extensions of
    MEM seeds during FM-index backwards search, with -s (min score, default 65), -e
    (mismatches, default 3), -m (min length, default 11), and -E (E-value, default
    0.01) options'
  repo: https://github.com/bioinformatics-centre/kaiju
- name: ART
  role: Simulates HTS reads from reference genomes to benchmark Greedy-mode sensitivity
    against MEM mode in a genome-exclusion setup
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/ncomms11257
    title: Fast and sensitive taxonomic classification for metagenomics with Kaiju
  source_content_hash: sha256:846ee64ad54628a99d1ca0a90724ad739893c63b36f380c60d29f3b0fcc73a4d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bioinformatics-centre/kaiju
  tools:
  - ART
  - Kaiju
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0194
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# blosum62-alignment-scoring

## Summary

Scores protein-level alignments for Kaiju's Greedy run mode using the BLOSUM62 substitution matrix, ranking translated amino acid fragments by cumulative score so the highest-scoring fragment is searched first and a read is classified once a match exceeds the minimum score threshold s (default 65).

## When to use

Use when implementing or tuning inexact protein-level search that extends exact seed matches with amino acid substitutions and needs a substitution-matrix score (BLOSUM62) to rank and filter candidate matches.

## When NOT to use

- Input reads originate from non-protein-coding genomic regions — protein-level classification cannot classify them
- The sequenced strains are already in the reference database — nucleotide-level classifiers will be more sensitive than this protein-level substitution-matrix approach
- Only exact maximum-length matches are needed — MEM mode (kaiju -a mem) is faster since it avoids substitution extensions and their runtime cost

## Inputs

- Simulated or real metagenomic sequencing reads in FASTQ (or gzip-compressed FASTQ/FASTA) format
- Kaiju FM-index database file (kaiju_db.fmi, e.g. built by kaiju-mkbwt and kaiju-mkfmi or kaiju-makedb)
- NCBI taxonomy files nodes.dmp (and names.dmp) containing valid taxon identifiers

## Outputs

- Per-read classification output (C/U flag, read name, NCBI taxon identifier, match score, and optional verbose columns via -v)
- Comparison table of classified read names and taxon IDs between Greedy and MEM modes (e.g. greedy_vs_mem_comparison.tsv) with per-mode classification rates

## How to apply

In Kaiju's Greedy mode, first locate all maximum exact matches (MEMs) of minimum seed length (default 7) via the FM-index backwards search over the BWT, then extend each seed at its left end allowing a configurable number of substitutions (default 3, adjustable via -e up to ~5), scoring the extended alignment with the BLOSUM62 substitution matrix. Rather than sorting fragments by length as in MEM mode, rank the translated amino acid fragments by their cumulative BLOSUM62 score and query the highest-scoring fragment first. Terminate the search for a read once a match exceeds the minimum score threshold s (default 65, adjustable via -s); also filter matches by minimum length (default m=11, via -m) and by E-value (default 0.01, via -E). The greedy restriction of substitutions to match ends is deliberate: allowing substitutions at all query positions in the backwards search causes an exponential runtime increase with the number of allowed mismatches. After modifying the search routines in kaiju/src, recompile with 'make' and compare Greedy-mode classifications against MEM mode (kaiju -a mem) on the same simulated-read FASTQ and kaiju_db.fmi index.

## Related tools

- **Kaiju** (Implements the Greedy run mode: BLOSUM62-scored substitution extensions of MEM seeds during FM-index backwards search, with -s (min score, default 65), -e (mismatches, default 3), -m (min length, default 11), and -E (E-value, default 0.01) options) — https://github.com/bioinformatics-centre/kaiju
- **ART** (Simulates HTS reads from reference genomes to benchmark Greedy-mode sensitivity against MEM mode in a genome-exclusion setup)

## Examples

```
kaiju -t nodes.dmp -f kaiju_db.fmi -i reads.fastq -o kaiju.out -e 3 -s 65 -E 0.01 -z 25
```

## Evaluation signals

- Greedy-mode sensitivity relative to MEM mode should be at least as high at comparable precision, consistent with the paper's finding that 'Greedy run mode yields a higher sensitivity compared with MEM mode'
- All emitted taxon identifiers must resolve within the reference taxonomy in nodes.dmp
- Match scores in the output must be at or above the minimum score threshold s (default 65), with matches also passing the minimum length (m=11) and E-value (0.01) filters
- Classified read names from a Greedy run should overlap substantially with MEM-mode classifications on the same FASTQ and kaiju_db.fmi; disagreements should be dominated by Greedy classifying additional reads
- Runtime should scale acceptably with the number of allowed mismatches (-e); large mismatch settings should show the expected exponential slowdown rather than silently completing

## Limitations

- Allowing substitutions in the backwards search causes an exponential increase of runtime depending on the number of allowed mismatched positions
- The greedy heuristic restricts substitutions to the ends of matches instead of all query positions due to search-space expansion
- Protein-level classification cannot classify reads from non-protein-coding genomic regions
- Kaiju is less sensitive than nucleotide-level classifiers when the sequenced strains are present in the reference database

## Evidence

- [discussion] which first locates all MEMs of a minimum seed length (default 7) and then extends them by allowing substitutions at the left ends of each seed match: "which first locates all MEMs of a minimum seed length (default 7) and then extends them by allowing substitutions at the left ends of each seed match"
- [discussion] we rank the fragments by their BLOSUM62 score and start the database search with the highest scoring fragment: "we rank the fragments by their BLOSUM62 score and start the database search with the highest scoring fragment"
- [discussion] we chose s=65 as default value for Kaiju's Greedy mode: "we chose s=65 as default value for Kaiju's Greedy mode"
- [readme] The default run mode is **Greedy** with three allowed mismatches. The number of allowed mismatches can be changed using option `-e`.: "The default run mode is **Greedy** with three allowed mismatches. The number of allowed mismatches can be changed using option `-e`."
- [methods] matches are filtered by a minimum length and score, but also by their E-value (similar to blastp), which can be adjusted with the option `-E`: "matches are filtered by a minimum length and score, but also by their E-value (similar to blastp), which can be adjusted with the option `-E`"
