---
name: query-coverage-accuracy-evaluation
description: Use when evaluating how completely Patchwork (or a DIAMOND-based pipeline)
  recovered reference markers, e.g. quantifying whether disabling tantan masking improves
  query coverage for read-based inputs.
when_to_use_negative:
- The goal is de novo marker discovery without reference sequences to align recovered
  markers against — the metrics require same-species (or homologous) references.
- Input is already a curated alignment of high-identity markers where coverage differences
  are not the question (e.g. markers already above the 90% aligned-position trimming
  threshold).
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: Patchwork
  role: Alignment-based retrieval and concatenation of phylogenetic markers whose
    outputs (recovered markers) are evaluated for identity and query coverage
  repo: https://github.com/fethalen/Patchwork
- name: DIAMOND
  role: Sequence aligner performing the BLASTX search underlying Patchwork; its tantan
    masking is toggled via --masking 0 in the A/B comparison
  repo: https://github.com/bbuchfink/diamond
- name: Julia
  role: Programming language in which Patchwork is written and executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gbe/evad227
    title: 'Patchwork: Alignment-Based Retrieval and Concatenation of Phylogenetic
      Markers from Genomic Data'
  source_content_hash: sha256:2b3c6d7cb1d6e0fc1e2c498c6cd2c50acc34029f7784337dce0ae7d943f91648
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# query-coverage-accuracy-evaluation

## Summary

Benchmark a marker-retrieval run by computing percent identity (exact matches over all aligned positions) and query coverage against reference sequences, and test DIAMOND settings such as --masking 0 that trade small identity losses for higher query coverage.

## When to use

Use when evaluating how completely Patchwork (or a DIAMOND-based pipeline) recovered reference markers, e.g. quantifying whether disabling tantan masking improves query coverage for read-based inputs.

## When NOT to use

- The goal is de novo marker discovery without reference sequences to align recovered markers against — the metrics require same-species (or homologous) references.
- Input is already a curated alignment of high-identity markers where coverage differences are not the question (e.g. markers already above the 90% aligned-position trimming threshold).

## Inputs

- Query assembly (e.g. SPAdes de novo assembly of D. gyrociliatus Illumina short reads, K-mer size 55) or raw short reads
- Reference USCO amino acid sequences (e.g. 815 D. gyrociliatus USCOs from GenBank accession GCA_904063045.1)
- Patchwork run outputs from paired conditions (default --masking vs --masking 0)

## Outputs

- Per-marker percent identity (exact matches / all aligned positions)
- Per-marker and mean query coverage (total aligned positions)
- Side-by-side table comparing masking-enabled vs masking-disabled runs

## How to apply

Run Patchwork (Julia implementation using DIAMOND BLASTX with default BLOSUM62 scoring, gap open 11, gap extension 1, --ultra-sensitive --frameshift 15) with tantan masking enabled, then repeat the identical run with the sole change of --masking 0. For each run, align the recovered markers back to the reference (e.g. the 815 D. gyrociliatus USCOs from GenBank GCA_904063045.1) and compute two metrics: percent identity, defined as the number of exact matches in all aligned positions, and query coverage, defined as the total aligned positions. Tabulate percent identity and query coverage side by side for the masking-enabled and masking-disabled runs. The rationale is that tantan masking suppresses low-complexity regions, so disabling it can recover more aligned positions while percent identity decreases only slightly — an effect especially pronounced when using raw read data rather than SPAdes assemblies as the query.

## Related tools

- **Patchwork** (Alignment-based retrieval and concatenation of phylogenetic markers whose outputs (recovered markers) are evaluated for identity and query coverage) — https://github.com/fethalen/Patchwork
- **DIAMOND** (Sequence aligner performing the BLASTX search underlying Patchwork; its tantan masking is toggled via --masking 0 in the A/B comparison) — https://github.com/bbuchfink/diamond
- **Julia** (Programming language in which Patchwork is written and executed)

## Examples

```
julia patchwork.jl --query spades_k55_assembly.fasta --reference dgyrociliatus_usco.faa --ultra-sensitive --frameshift 15 --masking 0
```

## Evaluation signals

- The masking-disabled run (--masking 0) shows higher query coverage than the masking-enabled run, especially for read-based queries.
- Percent identity decreases only slightly when tantan masking is disabled; a large identity drop invalidates the coverage gain.
- For the high-quality same-species assembly baseline, all 815 reference markers are retrieved with ~95.9% average identical aligned positions and ~92.2% mean query coverage, matching the reported benchmark.
- Both compared runs are identical except for the masking setting (same BLOSUM62 scoring, gap open 11, gap extension 1, --ultra-sensitive --frameshift 15), so metric differences are attributable to masking alone.

## Limitations

- Disabling tantan masking can slightly reduce percent identity, so coverage gains must be weighed against identity losses.
- The improvement is most pronounced for read-based data; assembled queries may show smaller or negligible coverage differences.
- Metrics depend on having appropriate reference sequences; using references that are too distant can produce chimeric or non-homologous alignments (the authors recommend limiting references to near-USCOs).

## Evidence

- [results] query coverage improves especially for read data when tantan masking in DIAMOND is disabled (i.e. by providing --masking 0 as an argument): "query coverage improves especially for read data when tantan masking in DIAMOND is disabled (i.e. by providing --masking 0 as an argument)"
- [results] We retrieved all of the initial 815 markers. On average, 95.9% of all aligned positions were identical matches, with a mean query coverage of 92.2%: "We retrieved all of the initial 815 markers. On average, 95.9% of all aligned positions were identical matches, with a mean query coverage of 92.2%"
- [methods] 815 D. gyrociliatus USCOs retrieved from the published high-quality genome assembly (GenBank, accession: GCA_904063045.1): "815 D. gyrociliatus USCOs retrieved from the published high-quality genome assembly (GenBank, accession: GCA_904063045.1)"
- [readme] Patchwork is an alignment-based program for retrieving and concatenating phylogenetic markers from whole-genome sequencing (WGS) data.: "Patchwork is an alignment-based program for retrieving and concatenating phylogenetic markers from whole-genome sequencing (WGS) data."
