---
name: hgt-breakpoint-pair-position-matching
description: Use when candidate transferred segments from reconstructed HGT local
  strains must be scored against reference segments to compute segment similarity,
  breakpoint-match flags, and reconstruction accuracy (RAi in [0,1]).
when_to_use_negative:
- Reference and query are short sequencing reads rather than transferred-segment sequences
  — LEMON uses BWA for read-to-reference mapping, not Smith-Waterman segment scoring
- The goal is global full-length genome alignment rather than local alignment of putative
  transferred segments at HGT breakpoints
- No reference/true segment sequences exist (e.g. de novo real-data analysis without
  ground truth), so RAi cannot be computed
edam_operation: http://edamontology.org/operation_0297
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0091
tools:
- name: Smith-Waterman algorithm
  role: Local alignment algorithm used to measure similarity between segment sequences
    of Hi and hi
- name: ssw-py
  role: Python binding providing the Smith-Waterman local alignment implementation
    used by LEMON's scripts
  repo: https://github.com/lichen2018/LEMON
- name: LEMON
  role: Pipeline that detects accurate HGT breakpoints (sample.acc.txt), reconstructs
    local strains, and scores reconstructed segments against reference segments
  repo: https://github.com/lichen2018/LEMON
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1186/s12859-019-3301-8
    title: 'LEMON: a method to construct the local strains at horizontal gene transfer
      sites in gut metagenomics'
  source_content_hash: sha256:ff0145d37b7654c43de8a57646a6e0c5f852369bc2745f17b0e44c2ec153b81a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen2018/LEMON
  tools:
  - LEMON
  - Smith-Waterman algorithm
  - ssw-py
  edam_operation: http://edamontology.org/operation_0297
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# hgt-breakpoint-pair-position-matching

## Summary

Quantifies how accurately reconstructed local HGT strains match true transferred segments by aligning each reconstructed segment (Hi) to its reference segment (hi) with the Smith-Waterman local alignment algorithm (match_score=1, mismatch_score=-1) and flagging segment pairs whose breakpoint positions agree within 20 bp, yielding per-strain reconstruction accuracy RAi.

## When to use

Use when candidate transferred segments from reconstructed HGT local strains must be scored against reference segments to compute segment similarity, breakpoint-match flags, and reconstruction accuracy (RAi in [0,1]).

## When NOT to use

- Reference and query are short sequencing reads rather than transferred-segment sequences — LEMON uses BWA for read-to-reference mapping, not Smith-Waterman segment scoring
- The goal is global full-length genome alignment rather than local alignment of putative transferred segments at HGT breakpoints
- No reference/true segment sequences exist (e.g. de novo real-data analysis without ground truth), so RAi cannot be computed

## Inputs

- Accurate HGT breakpoints file (e.g. sample.acc.txt) from LEMON's getAccBkp stage
- Candidate transferred segment sequences extracted from reconstructed local strains (Hi)
- True/reference transferred segment sequences (hi) from the metagenomic reference FASTA (donor/receptor genomes)
- Number of segments mi per true strain

## Outputs

- segment_similarity_scores.csv containing per-segment Smith-Waterman alignment scores SW(hi,Hi), 20 bp breakpoint-match flags, and reconstruction accuracy RAi per strain

## How to apply

Load the candidate transferred segments derived from reconstructed local strains (from the accurate HGT breakpoints file, e.g. sample.acc.txt) together with the corresponding donor/receptor sequences from the metagenomic reference FASTA. Align each reconstructed segment sequence Hi against each true/reference segment sequence hi using the Smith-Waterman local alignment algorithm implemented via the ssw-py Python binding, with match_score set to 1 and mismatch_score set to -1. Record the alignment score SW(hi,Hi) and compute reconstruction accuracy RAi = SW(hi,Hi)/(mi*match_score), where mi is the number of segments in the true strain (RAi=0 if the reconstruction is empty). Additionally flag a segment pair as matched when both breakpoint-pair positions of the reconstructed segment lie within 20 bp of the true breakpoint-pair positions. Write all per-segment alignment scores, match flags, and RA values to segment_similarity_scores.csv.

## Related tools

- **Smith-Waterman algorithm** (Local alignment algorithm used to measure similarity between segment sequences of Hi and hi)
- **ssw-py** (Python binding providing the Smith-Waterman local alignment implementation used by LEMON's scripts) — https://github.com/lichen2018/LEMON
- **LEMON** (Pipeline that detects accurate HGT breakpoints (sample.acc.txt), reconstructs local strains, and scores reconstructed segments against reference segments) — https://github.com/lichen2018/LEMON

## Examples

```
python LEMON/Scripts/reconstruct_HGT_strain.py -r metagenomic_reference.fa -c coverage.txt -s sample -a sample.acc.txt -o results/
```

## Evaluation signals

- Every RAi value falls in the range [0, 1]
- RAi = 1 when all transferred segments are matched, i.e. SW(hi,Hi) equals mi*match_score
- RAi = 0 for any strain whose reconstruction is empty
- Every segment row in segment_similarity_scores.csv contains a finite Smith-Waterman score
- Matched segment pairs have both breakpoint positions within 20 bp of the true breakpoint-pair positions

## Limitations

- Scores depend on the fixed scoring scheme (match_score=1, mismatch_score=-1); changing it changes the RAi normalization mi*match_score
- Breakpoint matching requires an exact 20 bp tolerance on both breakpoint-pair positions; reconstructions with slightly offset breakpoints may be unmatched despite high sequence similarity
- Validated in simulation against true local strains generated with WGSIM (20X paired-end reads); real-data applications lack exact ground truth for RAi

## Evidence

- [methods] Smith-Waterman similarity measurement with scoring scheme: "We apply Smith-Waterman algorithm to measure the similarity between segment sequences of Hi and hi."
- [readme] ssw-py dependency in README requirements: "- Python packages(3.5+)
  - pysam
  - sklearn
  - numpy
  - scipy
  - lmfit
  - ssw-py"
- [intro] LEMON detects breakpoints, identifies transferred segments, and reconstructs the local strain: "LEMON(https://github.com/lichen2018/LEMON) takes use of existing shotgun NGS datasets to detect HGT breakpoints, identify the transferred segments, and reconstruct the local strain"
- [abstract] Simulation validation of transferred segment identification accuracy: "The simulated results indicate that the method can identify transferred segments accurately on reference sequences of the microbiome."
