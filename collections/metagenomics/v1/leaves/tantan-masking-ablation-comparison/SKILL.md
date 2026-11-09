---
name: tantan-masking-ablation-comparison
description: Use when a Patchwork/DIAMOND marker-retrieval run shows low query coverage
  and you need to decide whether disabling DIAMOND's tantan masking (--masking 0)
  recovers longer alignments, particularly with raw-read (FASTQ-derived) rather than
  assembly queries.
when_to_use_negative:
- Query material is already high-coverage, well-assembled contigs where default masking
  already yields near-complete coverage (e.g. the 95.9% identity / 92.2% coverage
  same-species benchmark)
- Percent identity is the binding constraint on downstream phylogenetic use, since
  disabling tantan slightly decreases identity even as coverage improves
- The alignment tool in use is not DIAMOND, since tantan masking is a DIAMOND-specific
  low-complexity filter toggled by --masking 0
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0091
tools:
- name: Patchwork
  role: Alignment-based retrieval and concatenation of phylogenetic markers whose
    DIAMOND invocation is toggled between tantan-masked and unmasked modes
  repo: https://github.com/fethalen/Patchwork
- name: DIAMOND
  role: BLASTX sequence aligner whose tantan masking is ablated via the --masking
    0 argument
  repo: https://github.com/bbuchfink/diamond
- name: Julia
  role: Programming language in which Patchwork is written and executed for both ablation
    runs
  repo: https://github.com/fethalen/Patchwork
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gbe/evad227
    title: 'Patchwork: Alignment-Based Retrieval and Concatenation of Phylogenetic
      Markers from Genomic Data'
  source_content_hash: sha256:5560467e8da072407445dfc24de3216368c76f173f23eaee422e862a65411871
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fethalen/Patchwork
  tools:
  - DIAMOND
  - Julia
  - Patchwork
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# tantan-masking-ablation-comparison

## Summary

An ablation protocol comparing Patchwork runs with DIAMOND's tantan low-complexity masking enabled versus disabled (--masking 0), to quantify the effect on percent identity and query coverage of recovered phylogenetic markers. Disabling tantan yields higher query coverages, especially for read-based (non-assembled) queries, at only a slight identity cost.

## When to use

Use when a Patchwork/DIAMOND marker-retrieval run shows low query coverage and you need to decide whether disabling DIAMOND's tantan masking (--masking 0) recovers longer alignments, particularly with raw-read (FASTQ-derived) rather than assembly queries.

## When NOT to use

- Query material is already high-coverage, well-assembled contigs where default masking already yields near-complete coverage (e.g. the 95.9% identity / 92.2% coverage same-species benchmark)
- Percent identity is the binding constraint on downstream phylogenetic use, since disabling tantan slightly decreases identity even as coverage improves
- The alignment tool in use is not DIAMOND, since tantan masking is a DIAMOND-specific low-complexity filter toggled by --masking 0

## Inputs

- SPAdes de novo assembly (K-mer size 55) of D. gyrociliatus Illumina short reads (PRJEB37657) as DNA query
- 815 D. gyrociliatus USCO reference sequences from GenBank accession GCA_904063045.1 (amino acid references)
- Patchwork/DIAMOND BLASTX parameter set: BLOSUM62 (gap open 11, extend 1), --ultra-sensitive --frameshift 15, with and without --masking 0

## Outputs

- Recovered marker sequences (FASTA) for the masking-enabled run
- Recovered marker sequences (FASTA) for the masking-disabled run
- Side-by-side table of percent identity (exact matches / aligned positions) and query coverage (aligned positions / reference length) for both runs

## How to apply

Run Patchwork (Julia implementation, DIAMOND BLASTX) twice on identical inputs: once with default settings (BLOSUM62, gap open 11, gap extension 1, --ultra-sensitive --frameshift 15, tantan masking on) and once with the sole change of --masking 0. Use the SPAdes-assembled D. gyrociliatus short reads (K-mer size 55) against the 815 same-species USCO reference sequences from GenBank GCA_904063045.1 as the benchmark pair. After each run, align recovered markers back to the references and compute two metrics per marker: the number of exact matches in all aligned positions (percent identity) and the total aligned positions (query coverage). Validate by confirming that the masking-disabled run shows higher query coverage than the masking-enabled run while percent identity decreases only slightly.

## Related tools

- **Patchwork** (Alignment-based retrieval and concatenation of phylogenetic markers whose DIAMOND invocation is toggled between tantan-masked and unmasked modes) — https://github.com/fethalen/Patchwork
- **DIAMOND** (BLASTX sequence aligner whose tantan masking is ablated via the --masking 0 argument) — https://github.com/bbuchfink/diamond
- **Julia** (Programming language in which Patchwork is written and executed for both ablation runs) — https://github.com/fethalen/Patchwork

## Examples

```
patchwork -q spades_k55_dgyro.fasta -r dgyro_usco815.faa --ultra-sensitive --frameshift 15 --masking 0
```

## Evaluation signals

- The masking-disabled (--masking 0) run must show strictly higher mean query coverage than the masking-enabled run, with the gain most pronounced for read-based (not assembly) queries
- Percent identity in the masking-disabled run may decrease but only slightly; a large identity drop signals a misconfiguration
- Identical markers should be recovered across both runs so the comparison is a clean single-variable ablation (only --masking differs)
- Query coverage is computed as total aligned positions and percent identity as exact matches in all aligned positions, evaluated against the same 815-reference set for both runs

## Limitations

- Disabling tantan masking trades a slight decrease in percent identity for higher query coverage
- The improvement is especially pronounced for read-based data, so assembly-based queries may benefit less
- No specific version number for Patchwork or DIAMOND is stated in the source material, so parameter behavior could differ across releases

## Evidence

- [results] Query coverage gain from disabling tantan: "query coverage improves especially for read data when tantan masking in DIAMOND is disabled (i.e. by providing --masking 0 as an argument)"
- [results] Masking-enabled baseline quality on same-species references: "We retrieved all of the initial 815 markers. On average, 95.9% of all aligned positions were identical matches, with a mean query coverage of 92.2%"
- [readme] Patchwork uses DIAMOND and Julia, framing the ablation surface: "Written in [Julia](https://julialang.org/) and utilizing
  [DIAMOND](https://github.com/bbuchfink/diamond) for maximum speed"
- [methods] Reference dataset used to evaluate recovered markers: "815 D. gyrociliatus USCOs retrieved from the published high-quality genome assembly (GenBank, accession: GCA_904063045.1)"
