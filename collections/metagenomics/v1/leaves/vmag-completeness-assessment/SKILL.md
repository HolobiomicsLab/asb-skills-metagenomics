---
name: vmag-completeness-assessment
description: Use when you need to evaluate the quality of draft viral bins (e.g.,
  from ViralCC, VAMB, CoCoNet, vRhyme, bin3C, MetaTOR) against ground-truth or reference
  genomes, specifically counting near-complete and high-quality vMAGs.
when_to_use_negative:
- When the goal is to estimate contamination of vMAGs — CheckV cannot do this because
  no universal marker gene set exists for viruses
- When assessing prokaryotic MAGs — use CheckM's single-copy marker gene approach
  instead
- When the community has few putative viral genomes — the article notes benchmarking
  effectiveness is less convincing in that case
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_3174
tools:
- name: CheckV
  role: Estimates completeness of vMAGs by comparison to a curated reference database;
    used to count near-complete and high-quality viral bins per tool
- name: CheckM
  role: Contrasted with CheckV; assesses completeness/contamination of prokaryotic
    MAGs via universal single-copy marker genes and is not suitable for viral genomes
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-35945-y
    title: ViralCC retrieves complete viral genomes and virus-host pairs from metagenomic
      Hi-C data
  source_content_hash: sha256:11e69bca4a85783e90a2e4415f1b26278725626eec26c9d73edddcbc40a61bf2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CheckM
  - CheckV
  - CoCoNet
  - MetaTOR
  - VAMB
  - ViralCC
  - bin3C
  - leidenalg
  - vRhyme
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0781
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# vmag-completeness-assessment

## Summary

Assess the completeness of recovered viral MAGs (vMAGs) produced by Hi-C-based binning using CheckV, categorizing bins as near-complete, substantially complete, or moderately complete, and pairing the counts with a random binning control since CheckV cannot estimate vMAG contamination.

## When to use

Use when you need to evaluate the quality of draft viral bins (e.g., from ViralCC, VAMB, CoCoNet, vRhyme, bin3C, MetaTOR) against ground-truth or reference genomes, specifically counting near-complete and high-quality vMAGs.

## When NOT to use

- When the goal is to estimate contamination of vMAGs — CheckV cannot do this because no universal marker gene set exists for viruses
- When assessing prokaryotic MAGs — use CheckM's single-copy marker gene approach instead
- When the community has few putative viral genomes — the article notes benchmarking effectiveness is less convincing in that case

## Inputs

- Draft viral bin FASTA files (e.g., VIRAL_BIN/*.fa from ViralCC)
- Ground-truth genome labels of mock viral contigs (e.g., the 1010 mock human gut viral contigs)
- CheckV database curated from NCBI GenBank and environmental samples
- Random binning control assignments

## Outputs

- Per-vMAG completeness estimates and CheckV-style categories (near-complete / substantially complete / moderately complete)
- Per-tool counts of near-complete and high-quality vMAGs for the benchmarking table
- Random binning control completeness counts for comparison

## How to apply

Run CheckV (v0.7.0 in the paper) on each tool's draft viral bins — e.g., the VIRAL_BIN/*.fa produced by `python ./viralcc.py pipeline` — to estimate completeness by comparing bins to a curated reference database derived from NCBI GenBank and environmental samples. Classify recovered vMAGs into CheckV-style completeness categories (near-complete, substantially complete, moderately complete) and count them per binning tool. Because CheckV completeness is used as the evaluation metric, construct a random binning model as a control experiment, since random binning does not reduce CheckV completeness. Aggregate per-tool counts of near-complete and high-quality vMAGs (e.g., 26 near-complete for ViralCC vs 22 for MetaTOR and 5 for bin3C on the 1010-contig mock human gut dataset) into the benchmarking table alongside clustering metrics (F-score, ARI, NMI, homogeneity).

## Related tools

- **CheckV** (Estimates completeness of vMAGs by comparison to a curated reference database; used to count near-complete and high-quality viral bins per tool)
- **CheckM** (Contrasted with CheckV; assesses completeness/contamination of prokaryotic MAGs via universal single-copy marker genes and is not suitable for viral genomes)

## Examples

```
checkv end_to_end VIRAL_BIN/ checkv_output/ --restart 1
```

## Evaluation signals

- ViralCC on the mock human gut dataset should recover 26 near-complete vMAGs (vs 22 MetaTOR, 5 VAMB/bin3C, 1 CoCoNet, 0 vRhyme baselines), matching published Fig. 2 values within numerical tolerance
- Integrated-graph clustering should yield 26 near-complete, 2 substantially complete, and 4 moderately complete vMAGs versus 8/3/5 (Hi-C graph alone) and 12/2/0 (host proximity graph alone)
- A random binning control must be included when CheckV completeness is the evaluation metric, since random binning does not reduce CheckV completeness
- Completeness categories must align with CheckV-style classification and preserve the reported performance ordering (ViralCC outperforming all comparators)

## Limitations

- CheckV is unable to estimate contamination of vMAGs since no universal marker gene set is available for viruses
- Since CheckV assessment is not comprehensive, a systematic benchmarking strategy plus random binning control is required
- The benchmarking strategy underestimates true viral community diversity because only single-contig-recoverable viral genomes were chosen, and putative viral genomes in the benchmarking tend to be small

## Evidence

- [intro] CheckV is unable to estimate the contamination of vMAGs since there is no such marker gene set available for viruses: "CheckV is unable to estimate the contamination of vMAGs since there is no such marker gene set available for viruses"
- [discussion] CheckV completeness rationale: "it is necessary to construct a random binning model as control experiments when the CheckV completeness is used as the evaluation metric"
- [intro] CheckV methodology: "CheckV has been widely used to estimate the completeness of vMAGs by comparing them to a large database curated from NCBI GenBank and environmental samples"
- [discussion] Benchmarking scope caveat: "Since we only choose viral genomes that can be recovered by a single contig from the whole community, our benchmarking method inevitably underestimates the true diversity of the virus community."
- [readme] CheckV version used: [evidence span withheld — anchor status: external_doc]
