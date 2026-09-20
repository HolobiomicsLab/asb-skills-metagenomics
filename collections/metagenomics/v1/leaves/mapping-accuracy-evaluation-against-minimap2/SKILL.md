---
name: mapping-accuracy-evaluation-against-minimap2
description: Use when a raw nanopore signal mapper (e.g., RawHash) has produced mapping.paf
  and its mapping positions must be validated against sequence-level ground truth
  before downstream analysis.
when_to_use_negative:
- Output is not raw-signal mapping (e.g., Rawsamble all-vs-all overlaps for de novo
  assembly via miniasm) — there is no sequence-mapper ground truth to compare against.
- The mapper being validated does not report positions in the UNCALLED/Sigmap PAF
  template, making positional comparison ill-defined.
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: RawHash
  role: Produces the chain-derived mapping positions under evaluation via Sigmap-style
    chaining over seed matches
  repo: https://github.com/CMU-SAFARI/RawHash
- name: minimap2
  role: Generates ground-truth mapping positions from sequence-level alignment for
    comparison
  repo: https://github.com/lh3/minimap2
- name: Sigmap
  role: Source of the chaining algorithm RawHash uses; its PAF output template is
    the validation format
  repo: https://github.com/haowenz/sigmap
- name: UNCALLED
  role: Co-source of the PAF template used to report and validate the mapping output
  repo: https://github.com/skovaka/UNCALLED
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btad272
    title: 'RawHash: enabling fast and accurate real-time analysis of raw nanopore
      signals for large genomes'
  source_content_hash: sha256:e6ef894ff198abe41bc5a0fc4536bec7a3d941e609ba67f9a70539cf84de8cce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - RawHash
  - Sigmap
  - UNCALLED
  - minimap2
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# mapping-accuracy-evaluation-against-minimap2

## Summary

Validates raw-signal mapping output by checking that chain-derived mapping positions from RawHash's Sigmap-style chaining agree with minimap2-derived ground-truth positions, and that the output parses as the modified PAF template used by UNCALLED and Sigmap.

## When to use

Use when a raw nanopore signal mapper (e.g., RawHash) has produced mapping.paf and its mapping positions must be validated against sequence-level ground truth before downstream analysis.

## When NOT to use

- Output is not raw-signal mapping (e.g., Rawsamble all-vs-all overlaps for de novo assembly via miniasm) — there is no sequence-mapper ground truth to compare against.
- The mapper being validated does not report positions in the UNCALLED/Sigmap PAF template, making positional comparison ill-defined.

## Inputs

- mapping.paf produced by RawHash (modified UNCALLED/Sigmap PAF template)
- minimap2-derived ground-truth mapping positions for the same read set
- SARS-CoV-2 reference genome (ref.fasta) and hash index (ref.ind) used for mapping
- d1_sars-cov-2_r94 FAST5 raw signal directory (D1 nanopore raw signals)

## Outputs

- Validation verdict that mapping.paf exists and parses as a PAF-formatted table
- Per-read comparison of chain-derived RawHash positions versus minimap2 ground-truth positions on the evaluated read subset

## How to apply

After RawHash generates seed hashes from the raw FAST5 signals and queries the hash-table index (ref.ind), it applies the Sigmap chaining algorithm with the distance coefficient disabled — because it does not calculate distances between seed matches — and selects the best colinear chain of seed matches as the mapped region. Collect these chain-derived positions from mapping.paf in the PAF template used by UNCALLED and Sigmap. In parallel, generate ground-truth positions by aligning the basecalled or reference-derived sequences with minimap2 (e.g., the map-ont preset, with an optional pre-built .mmi index). Then compare the two position sets on the evaluated read subset, checking that RawHash's chain-derived coordinates agree with minimap2's alignments. Correct application is judged by mapping.paf existing, parsing as a PAF-formatted table, and positional concordance with the minimap2 ground truth.

## Related tools

- **RawHash** (Produces the chain-derived mapping positions under evaluation via Sigmap-style chaining over seed matches) — https://github.com/CMU-SAFARI/RawHash
- **minimap2** (Generates ground-truth mapping positions from sequence-level alignment for comparison) — https://github.com/lh3/minimap2
- **Sigmap** (Source of the chaining algorithm RawHash uses; its PAF output template is the validation format) — https://github.com/haowenz/sigmap
- **UNCALLED** (Co-source of the PAF template used to report and validate the mapping output) — https://github.com/skovaka/UNCALLED

## Examples

```
minimap2 -x map-ont -d ref-ont.mmi ref.fasta && minimap2 -a ref-ont.mmi reads.fq > ground_truth.sam
```

## Evaluation signals

- mapping.paf exists in the working directory
- mapping.paf parses as a PAF-formatted table (correct column count and fields per line)
- Chain-derived mapping positions on the evaluated read subset agree with minimap2-derived ground-truth positions
- Each reported region corresponds to the best-scoring colinear chain of seed matches for that read

## Limitations

- RawHash disables the distance coefficient in chaining since it does not calculate distances between seed matches, so agreement with minimap2 is approximate rather than exact coordinate identity.
- Validation covers only the evaluated read subset, not the full read set.
- RawHash stops once a read prefix maps (real-time behavior), so reported regions may reflect prefix-based mapping rather than full-read alignments.

## Evidence

- [methods] chaining selects mapped region: "To map the raw signal to a reference genome, RawHash performs chaining over the seed matches."
- [methods] paf template: "We follow the similar PAF template used in [UNCALLED](https://github.com/skovaka/UNCALLED)"
- [methods] sigmap paf template: "and [Sigmap](https://github.com/haowenz/sigmap) to report the mapping information."
- [readme] minimap2 ground truth usage: "# long sequences against a reference genome
./minimap2 -a test/MT-human.fa test/MT-orang.fa > test.sam"
