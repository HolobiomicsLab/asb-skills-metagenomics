---
name: paf-output-parsing
description: Use when you have PAF files in the UNCALLED-style template emitted by
  RawHash/UNCALLED/Sigmap and need to derive precision/recall, relative abundance,
  or read-prefix/chunk statistics from them.
when_to_use_negative:
- Input is not raw-signal mapper output but basecalled-read alignments (e.g. minimap2
  SAM/BAM) — use sequence-level alignment scoring instead
- The mapper emits SAM rather than the UNCALLED-style PAF template; field offsets
  differ and pafstats cannot consume it directly
- You need base-level CIGAR alignment information — the PAF template used here reports
  approximate mapping coordinates only
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: RawHash
  role: Produces the PAF mapping outputs being parsed; its PAF records include the
    sequenced read prefix length since RawHash stops mapping once a read prefix maps
  repo: https://github.com/CMU-SAFARI/RawHash
- name: UNCALLED
  role: Defines the PAF template followed for mapping output; its pafstats utility
    scores PAF outputs against minimap2 ground truth
  repo: https://github.com/skovaka/UNCALLED
- name: Sigmap
  role: Baseline mapper whose PAF records are parsed for comparison of accuracy and
    chunk counts
  repo: https://github.com/haowenz/sigmap
- name: Minimap2
  role: Generates basecalled-read ground-truth alignments that PAF outputs are scored
    against
  repo: https://github.com/lh3/minimap2
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_001
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad272
    title: 'RawHash: enabling fast and accurate real-time analysis of raw nanopore
      signals for large genomes'
  source_content_hash: sha256:a6445b4215528c9a5a11bb5647fbe77e845337fb573927d2c886cc9737a73b92
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CMU-SAFARI/RawHash
  tools:
  - Minimap2
  - RawHash
  - RawHash2
  - Sigmap
  - UNCALLED
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# paf-output-parsing

## Summary

Parse the modified-PAF mapping outputs produced by RawHash, UNCALLED, and Sigmap on raw nanopore signals to extract mapped-read records, sequenced read-prefix lengths, and per-genome mapping counts used in accuracy, abundance, and sequencing-cost analyses.

## When to use

Use when you have PAF files in the UNCALLED-style template emitted by RawHash/UNCALLED/Sigmap and need to derive precision/recall, relative abundance, or read-prefix/chunk statistics from them.

## When NOT to use

- Input is not raw-signal mapper output but basecalled-read alignments (e.g. minimap2 SAM/BAM) — use sequence-level alignment scoring instead
- The mapper emits SAM rather than the UNCALLED-style PAF template; field offsets differ and pafstats cannot consume it directly
- You need base-level CIGAR alignment information — the PAF template used here reports approximate mapping coordinates only

## Inputs

- RawHash mapping.paf (modified PAF template, raw nanopore signal alignments)
- UNCALLED PAF output on FAST5/BLOW5 read sets (datasets d1-d5, e.g. d5_human_na12878_r94)
- Sigmap PAF output on the same read sets
- minimap2 basecalled-read alignments used as ground truth for scoring

## Outputs

- Per-dataset, per-tool precision/recall/error-rate metrics table
- Per-genome relative abundance estimates and Euclidean distances
- table4_prefix_chunk_stats.csv with average sequenced bases per read and chunks per read
- Estimation distances and read-usage fractions for Sequence Until analyses

## How to apply

Read each mapped-read record from the tool's PAF output (the UNCALLED-style template shared by UNCALLED and Sigmap), splitting tab-delimited fields for query name, coordinates, and target coordinates. For accuracy scoring, feed the PAF plus minimap2 basecalled-read ground-truth alignments through UNCALLED's pafstats utility to obtain error rate, precision, and recall. For abundance estimation, tally mapped reads per reference genome and convert counts to relative abundances; for time/cost analyses, extract the sequenced read prefix length in bases (RawHash stops as soon as a read prefix maps to the reference genome) and the number of sequenced signal chunks consumed per mapped read. Average these per dataset and tool before assembling comparison tables.

## Related tools

- **RawHash** (Produces the PAF mapping outputs being parsed; its PAF records include the sequenced read prefix length since RawHash stops mapping once a read prefix maps) — https://github.com/CMU-SAFARI/RawHash
- **UNCALLED** (Defines the PAF template followed for mapping output; its pafstats utility scores PAF outputs against minimap2 ground truth) — https://github.com/skovaka/UNCALLED
- **Sigmap** (Baseline mapper whose PAF records are parsed for comparison of accuracy and chunk counts) — https://github.com/haowenz/sigmap
- **Minimap2** (Generates basecalled-read ground-truth alignments that PAF outputs are scored against) — https://github.com/lh3/minimap2

## Examples

```
import pandas as pd; cols=['qname','qlen','qstart','qend','strand','tname','tlen','tstart','tend','nmatch','alen','mapq']; df=pd.read_csv('mapping.paf',sep='\t',header=None,names=cols)
```

## Evaluation signals

- PAF parses with the expected UNCALLED-template field count (at least the 12 standard PAF columns) for all three tools' outputs
- Scoring with pafstats yields finite precision/recall values in [0,1] per dataset and tool, with RawHash better on large genomes (human) per the paper's ordering
- Computed average sequenced prefix lengths are shorter for RawHash than UNCALLED on large genomes (Green Algae, Human), while chunk counts are higher than Sigmap (≈1.58x on average)
- Per-genome mapped-read counts sum to the number of mapped reads and abundance values fall in [0,1]
- Sequence Until stop fractions approximate the reported ~7% of reads consumed before safe termination

## Limitations

- UNCALLED can only be applied to legacy r9.4.1 data, so the PAF-parsing workflow is validated on r9.4 FAST5 datasets
- PAF records here carry approximate coordinates without base-level alignment, so parse-derived statistics cannot replace CIGAR-based error analysis
- Comparisons of bases-per-read and chunks-per-read are tool-dependent (RawHash uses size-matched presets, e.g. -x sensitive for small genomes and -x fast for large ones), so parsed values are only comparable under matched presets

## Evidence

- [methods] We follow the similar PAF template used in [UNCALLED] to report the mapping information.: "We follow the similar PAF template used in [UNCALLED]"
- [full_text] RawHash stops as soon as a read prefix maps to the reference genome.: [evidence span withheld — anchor status: not_found]
- [full_text] Score each tool's PAF output against the minimap2 ground truth using UNCALLED's pafstats utility to obtain error rate, precision, and recall per dataset and tool.: [evidence span withheld — anchor status: not_found]
- [full_text] which are reported in the modified PAF template shared by UNCALLED and Sigmap.: [evidence span withheld — anchor status: not_found]
- [readme] A read mapper which rapidly aligns raw nanopore signal to DNA references: [evidence span withheld — anchor status: external_doc]
