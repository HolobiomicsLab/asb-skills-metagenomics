---
name: cigar-string-parsing
description: Use when scoring suffix-prefix overlaps between NGS reads aligned with
  Minimap2 all-vs-all mode, where overlap quality must be derived from the CIGAR string
  rather than re-assessed across the full overlap length.
when_to_use_negative:
- Overlaps are already scored by a different scheme (e.g. FM-index based overlap enumeration
  as in SAVAGE) and CIGAR re-scoring is not needed
- Input reads are long noisy third-generation reads; the article notes the high error
  rate of third-generation sequencing reads introduces unresolved issues
- You only need approximate mapping coordinates without base-level alignment (no CIGAR
  requested from Minimap2)
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: Minimap2
  role: Aligns every read against every other read in overlap mode, retaining CIGAR
    strings from which identity and overlap length are parsed
  repo: https://github.com/lh3/minimap2
- name: StrainXpress
  role: Defines the weighted overlap-score formula and the 30 bp / 0.9 identity filters
    applied to parsed CIGAR-derived quantities
  repo: https://github.com/HaploKit/StrainXpress
- name: Pandas
  role: Used in the conda environment for handling the tabular overlap-score output
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/nar/gkac543
    title: 'StrainXpress: strain aware metagenome assembly from short reads'
  source_content_hash: sha256:48661184683d3e1b01672fb2cf659763ab4f2939281b5cce07b1920992eb2d45
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lh3/minimap2
  tools:
  - Minimap2
  - Pandas
  - Python
  - StrainXpress
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# cigar-string-parsing

## Summary

Parses CIGAR strings emitted by Minimap2 in all-vs-all overlap mode to derive alignment identity and overlap length, then computes StrainXpress's weighted overlap score D = 0.9*i + 0.1*o/((r1+r2)/2) for suffix-prefix read overlaps.

## When to use

Use when scoring suffix-prefix overlaps between NGS reads aligned with Minimap2 all-vs-all mode, where overlap quality must be derived from the CIGAR string rather than re-assessed across the full overlap length.

## When NOT to use

- Overlaps are already scored by a different scheme (e.g. FM-index based overlap enumeration as in SAVAGE) and CIGAR re-scoring is not needed
- Input reads are long noisy third-generation reads; the article notes the high error rate of third-generation sequencing reads introduces unresolved issues
- You only need approximate mapping coordinates without base-level alignment (no CIGAR requested from Minimap2)

## Inputs

- Interleaved FASTQ file of quality-controlled NGS reads (e.g. all_reads.fq)
- Minimap2 all-vs-all overlap alignments in PAF format with CIGAR strings (cg tag)

## Outputs

- TSV file (e.g. overlap_scores.tsv) with columns for the two read identifiers, overlap length, identity, and overlap score D

## How to apply

Run Minimap2 in all-vs-all overlap mode on interleaved FASTQ reads, retaining the CIGAR string (cg tag) for each alignment. Parse each CIGAR to extract alignment identity (i) and the overlap length (o) between the two reads with lengths r1 and r2. Score each overlap with D = 0.9*i + 0.1*o/((r1+r2)/2), which weights identity heavily while normalizing overlap length by mean read length. Discard overlaps shorter than 30 bp or with identity below 0.9. The rationale is that the weighted score avoids needing to assess overlap quality across the overlap's full length, keeping the overlap-graph stage computationally tractable.

## Related tools

- **Minimap2** (Aligns every read against every other read in overlap mode, retaining CIGAR strings from which identity and overlap length are parsed) — https://github.com/lh3/minimap2
- **StrainXpress** (Defines the weighted overlap-score formula and the 30 bp / 0.9 identity filters applied to parsed CIGAR-derived quantities) — https://github.com/HaploKit/StrainXpress
- **Pandas** (Used in the conda environment for handling the tabular overlap-score output)

## Examples

```
minimap2 -c all_reads.fq all_reads.fq > overlaps.paf; python scripts/score_overlaps.py --paf overlaps.paf --out overlap_scores.tsv
```

## Evaluation signals

- overlap_scores.tsv exists and is well-formed TSV with every row containing a read pair, overlap length, identity, and score D
- Every surviving row has overlap length >= 30 bp and identity >= 0.9
- Every overlap score D lies in the range [0, 1]
- Parsed identity and overlap length reproduce values recomputable from the Minimap2 CIGAR string (M/=/X and I/D operation counts)
- No read pair appears in the output that was filtered out for being shorter than 30 bp or below 0.9 identity

## Limitations

- The score is computed from the CIGAR only, so overlap quality is not assessed across the full overlap length
- Short Illumina-type read lengths constrain downstream contig lengths in the StrainXpress assembly
- High error rates of third-generation sequencing reads remain an unresolved issue for this approach
- Input must be interleaved FASTQ in the format StrainXpress expects (paired reads as /1 and /2 records in one file)

## Evidence

- [readme] CIGAR-derived weighted overlap score: [evidence span withheld — anchor status: external_doc]
- [readme] Input format for StrainXpress: "The input file must be interleaved FASTQ and format like below:"
- [readme] Dependency on Minimap2: "StrainXpress relies on the following dependencies:
- [minimap2](https://github.com/lh3/minimap2)"
- [discussion] Overlap-graph rationale: "patterns of co-occurring mutations—as the decisive characteristic sequential phenomenon of strains—can be highlighted optimally with overlap graphs"
