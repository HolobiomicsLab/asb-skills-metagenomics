---
name: paf-mapping-output-generation
description: Use when converting matching seed hash values from raw nanopore signals
  into a mapped genomic region (e.g., D1 SARS-CoV-2 r9.4 FAST5 reads vs ref.ind/ref.fasta),
  or when validating mapping.paf against minimap2 ground truth.
when_to_use_negative:
- Input is already basecalled FASTQ/sequence reads — minimap2-style sequence mapping,
  not raw-signal chaining, is appropriate.
- No reference genome index exists — indexing (task_006-style ref.ind generation)
  must precede this skill.
- The goal is reference-free all-vs-all overlap finding between raw signals — use
  Rawsamble, which produces PAF overlaps without a reference.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: RawHash
  role: Performs chaining over seed matches of raw nanopore signals against the hash
    index and reports mapping positions
  repo: https://github.com/CMU-SAFARI/RawHash
- name: Sigmap
  role: Source of the chaining algorithm (with distance coefficient disabled) and
    of the PAF reporting template
  repo: https://github.com/haowenz/sigmap
- name: UNCALLED
  role: Provides the PAF template followed to report mapping information
  repo: https://github.com/skovaka/UNCALLED
- name: minimap2
  role: Provides ground-truth mapping positions used to validate chain-derived PAF
    mappings
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad272
    title: 'RawHash: enabling fast and accurate real-time analysis of raw nanopore
      signals for large genomes'
  source_content_hash: sha256:fd4612fc7143595bd25280f8935e9f0ccc8a54a877a33044327c460953eddf69
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - RawHash
  - Sigmap
  - UNCALLED
  - minimap2
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# paf-mapping-output-generation

## Summary

Generates the mapping output of RawHash by chaining seed matches with the Sigmap chaining algorithm (distance coefficient disabled) and reporting chain-derived positions in the modified PAF template used by UNCALLED and Sigmap, enabling real-time raw nanopore signal mapping.

## When to use

Use when converting matching seed hash values from raw nanopore signals into a mapped genomic region (e.g., D1 SARS-CoV-2 r9.4 FAST5 reads vs ref.ind/ref.fasta), or when validating mapping.paf against minimap2 ground truth.

## When NOT to use

- Input is already basecalled FASTQ/sequence reads — minimap2-style sequence mapping, not raw-signal chaining, is appropriate.
- No reference genome index exists — indexing (task_006-style ref.ind generation) must precede this skill.
- The goal is reference-free all-vs-all overlap finding between raw signals — use Rawsamble, which produces PAF overlaps without a reference.

## Inputs

- Nanopore raw signal reads in FAST5 (also POD5, SLOW5, or BLOW5 supported by RawHash)
- RawHash hash-table index over the reference genome (e.g., ref.ind built from ref.fasta)
- Reference genome in sequence format (ref.fasta)

## Outputs

- mapping.paf — a PAF-formatted table of chain-derived mapping positions, using the modified PAF template from UNCALLED and Sigmap

## How to apply

Load the nanopore raw signals (e.g., from the d1_sars-cov-2_r94 FAST5 directory) and the hash index (ref.ind) built over the reference genome (ref.fasta). For each read, generate seed hashes from the raw signal and look up matching genomic positions in the hash-table index built during the indexing step. Apply the Sigmap-style chaining algorithm over the retrieved seed matches with the distance coefficient disabled, since RawHash does not calculate distances between seed matches, to form colinear seed chains. Select the best-scoring colinear chain per read as the mapped region, following the real-time behavior of stopping once a read prefix maps, and report the chain-derived positions in the modified PAF template used by UNCALLED and Sigmap.

## Related tools

- **RawHash** (Performs chaining over seed matches of raw nanopore signals against the hash index and reports mapping positions) — https://github.com/CMU-SAFARI/RawHash
- **Sigmap** (Source of the chaining algorithm (with distance coefficient disabled) and of the PAF reporting template) — https://github.com/haowenz/sigmap
- **UNCALLED** (Provides the PAF template followed to report mapping information) — https://github.com/skovaka/UNCALLED
- **minimap2** (Provides ground-truth mapping positions used to validate chain-derived PAF mappings) — https://github.com/lh3/minimap2

## Examples

```
minimap2 -x map-ont ref.fasta reads.fq > ground_truth.paf  # ground truth for validating mapping.paf
```

## Evaluation signals

- mapping.paf exists and parses as a PAF-formatted table (tab-delimited records with query/target name, coordinates, strand, mapping quality fields).
- Chain-derived mapping positions on the evaluated read subset agree with minimap2-derived ground-truth positions.
- Chaining is confirmed to run with the distance coefficient disabled, matching RawHash's design of not calculating distances between seed matches.
- Per read, exactly one best-scoring colinear chain is selected, consistent with real-time prefix-mapping stop behavior.
- Output records follow the modified PAF template of UNCALLED/Sigmap rather than a generic SAM/PAF variant.

## Limitations

- RawHash does not calculate distances between seed matches, so the Sigmap chaining distance coefficient is disabled, which constrains chain scoring compared to distance-aware chainers.
- Real-time behavior stops once a read prefix maps, so mappings reflect prefix-based regions rather than full-read end-to-end alignments.
- UNCALLED-based PAF reporting applies to legacy r9.4.1 data; other chemistries are outside this template's validated scope.

## Evidence

- [methods] Chaining over seed matches: "To map the raw signal to a reference genome, RawHash performs chaining over the seed matches."
- [intro] Mapping step queries hash table: "In the mapping step, RawHash generates the hash values from raw signals and queries the hash table"
- [methods] PAF template from UNCALLED and Sigmap: "We follow the similar PAF template used in [UNCALLED](https://github.com/skovaka/UNCALLED)"
- [readme] PAF output for downstream use: "The overlap information is generated in a PAF output and can be used by assemblers such as `miniasm` to construct *de novo* assemblies."
- [intro] Real-time mapping scope: "RawHash (and RawHash2) is a hash-based mechanism to map raw nanopore signals to a reference genome in real-time."
