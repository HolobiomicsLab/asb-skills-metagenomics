---
name: transposable-element-annotation
description: Use when you have a FASTA catalogue of candidate insertion sequences
  (e.g. from Palidis/ISC) and need to determine which have known homologs in ISfinder
  and at what stringency.
when_to_use_negative:
- Input sequences are not insertion sequences or lack inverted terminal repeats —
  the ISC is restricted to DDE-type ISs with ITRs of 25 or more nucleotides, so results
  will not generalize to ITR-less elements
- You need de novo IS discovery from reads rather than annotation of an existing catalogue
  — use Palidis first to generate the IS sequences
- The catalogue derives from sources outside healthy human oral cavity and stool metagenomes
  — ISC v1.0.0 is recommended as a reference only for isolates from those sources
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_0622
tools:
- name: ISfinder
  role: Reference database of known transposable elements queried via BLAST to assign
    homology status to each catalogue IS
- name: Palidis
  role: Upstream pipeline that discovered the insertion sequences compiled into the
    ISC catalogue being annotated
  repo: https://github.com/blue-moon22/palidis
- name: Insertion Sequence Catalogue (ISC)
  role: The FASTA catalogue of 879 ISs that serves as the query input for the ISfinder
    homology breakdown
  repo: https://github.com/blue-moon22/ISC
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1099/mgen.0.000917
    title: 'Palidis: fast discovery of novel insertion sequences'
  source_content_hash: sha256:e451f040f3bdaa55242803960b4d0815b156a57db6c3d82a1d7ee99116b002d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/blue-moon22/palidis
  tools:
  - ISfinder
  - Insertion Sequence Catalogue (ISC)
  - Palidis
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# transposable-element-annotation

## Summary

Annotates a catalogue of discovered insertion sequences (ISs) by BLASTing each sequence against ISfinder and classifying hits into strict homology (e-value <1e-50), loose homology (0.01 > e-value >= 1e-50), or novel (no hit at the 0.01 threshold), enabling quantification of how much of a novel IS catalogue is already characterized.

## When to use

Use when you have a FASTA catalogue of candidate insertion sequences (e.g. from Palidis/ISC) and need to determine which have known homologs in ISfinder and at what stringency.

## When NOT to use

- Input sequences are not insertion sequences or lack inverted terminal repeats — the ISC is restricted to DDE-type ISs with ITRs of 25 or more nucleotides, so results will not generalize to ITR-less elements
- You need de novo IS discovery from reads rather than annotation of an existing catalogue — use Palidis first to generate the IS sequences
- The catalogue derives from sources outside healthy human oral cavity and stool metagenomes — ISC v1.0.0 is recommended as a reference only for isolates from those sources

## Inputs

- FASTA file of insertion sequences (e.g. ISC v1.0.0, 879 ISs from github.com/blue-moon22/ISC)
- ISfinder database accessed via its online BLAST service

## Outputs

- Per-sequence homology classification (strict hit, loose hit, or novel) with e-values
- Summary table (e.g. isfinder_homology_benchmark.csv) of hit counts and percentages

## How to apply

Download the catalogue FASTA (e.g. the 879-sequence ISC v1.0.0 from github.com/blue-moon22/ISC). Query each sequence against ISfinder's online BLAST service with an e-value cutoff of 0.01 and default parameters (as of 7 October 2022 for the published benchmark). Classify each catalogue entry by e-value: strict hit if e-value <1e-50, loose hit if 0.01 > e-value >= 1e-50, and novel if no hit passes the 0.01 threshold. Tabulate the counts and proportions, expecting for ISC v1.0.0 360 (41.0%) ISfinder hits split as 60 strict + 300 loose, with 519 novel sequences, and write the summary to a CSV such as isfinder_homology_benchmark.csv. The rationale is that very low e-values indicate near-certain assignment to a known IS family, whereas looser hits suggest distant homology warranting manual inspection.

## Related tools

- **ISfinder** (Reference database of known transposable elements queried via BLAST to assign homology status to each catalogue IS)
- **Palidis** (Upstream pipeline that discovered the insertion sequences compiled into the ISC catalogue being annotated) — https://github.com/blue-moon22/palidis
- **Insertion Sequence Catalogue (ISC)** (The FASTA catalogue of 879 ISs that serves as the query input for the ISfinder homology breakdown) — https://github.com/blue-moon22/ISC

## Examples

```
blast the ISC FASTA against ISfinder's online BLAST service (e-value cutoff 0.01, default parameters), then classify: strict if e-value <1e-50, loose if 0.01 > e-value >= 1e-50, else novel; write isfinder_homology_benchmark.csv
```

## Evaluation signals

- Total classified sequences must sum to the catalogue size (360 hits + 519 novel = 879 for ISC v1.0.0)
- Strict (e-value <1e-50) and loose (0.01 > e-value >= 1e-50) counts must sum to total hits: 60 + 300 = 360 (41.0%)
- Every catalogue sequence receives exactly one of three labels: strict, loose, or novel (no hit at 0.01 threshold)
- Novel count matches the independently reported figure of 519 novel ISs
- Summary CSV (isfinder_homology_benchmark.csv) contains counts consistent with the published proportions

## Limitations

- BLAST parameters and ISfinder contents are time-sensitive; the published breakdown used ISfinder as of 7 October 2022 with default parameters, so rerunning later may shift counts
- ISC v1.0.0 only contains common DDE-type ISs with ITRs of 25 or more nucleotides; ISs lacking ITRs or with shorter ITRs (as short as ten nucleotides) are not represented and thus cannot be annotated by this workflow
- ISC v1.0.0 was built only from healthy human oral cavity and stool metagenomes, limiting its use as an annotation reference for isolates from other sources
- ISs flanked by ITRs outside the 500–3000 bp proximity range (e.g. lengthened by nested insertions) are absent from the catalogue and hence from the homology breakdown

## Evidence

- [results] 360 (41.0%) of the 879 ISC ISs have hits in ISfinder while 519 are novel; of the hits, 60 show strict homology (e-value <1e-50) and 300 show loose homology (0.01>e-value ≥1e-50).: [evidence span withheld — anchor status: not_found]
- [intro] Microbial genomes can be annotated for ISs by querying reference databases of known transposable elements, such as ISfinder: "Microbial genomes can be annotated for ISs by querying reference databases of known transposable elements, such as ISfinder"
- [discussion] we have released the first version of ISC, a catalogue containing 879 ISs: "we have released the first version of ISC, a catalogue containing 879 ISs"
- [readme] This repository contains a catalogue of insertion sequences in FASTA format generated using palidis4: [evidence span withheld — anchor status: external_doc]
- [discussion] the catalogue currently contains ISs with ITRs that are 25 or greater nucleotides in length as generated by Palidis, although ITRs can be as short as ten nucleotides in length: "the catalogue currently contains ISs with ITRs that are 25 or greater nucleotides in length as generated by Palidis, although ITRs can be as short as ten nucleotides"
