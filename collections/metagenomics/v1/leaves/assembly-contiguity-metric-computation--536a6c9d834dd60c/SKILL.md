---
name: assembly-contiguity-metric-computation
description: Use when FASTA assemblies (contigs/scaffolds) from metagenome assemblers
  must be scored against a known reference to compare contiguity (NGA50), misassemblies,
  and base-level accuracy across tools or ablation modes.
when_to_use_negative:
- Reference genomes are unavailable or the study is fully de novo with unknown community
  composition — NGA50 and misassembly counts are reference-based and cannot be computed.
- Input is raw reads rather than an assembled FASTA; use a metagenome assembler (e.g.,
  MetaPlatanus) first.
- Only base-level error profiling of a single assembly is needed without contiguity
  or misjoin assessment across tools.
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: minimap2
  role: Aligns assembled scaffolds to the reference genome set to derive misassembly,
    NGA50, and identity statistics
  repo: https://github.com/lh3/minimap2
- name: MetaPlatanus
  role: Produces the out_final.fa assemblies (default, -no_tgsgapcloser, -no_nextpolish
    modes) that are scored for NGA50 and misassemblies
  repo: https://github.com/rkajitani/MetaPlatanus
- name: MEGAHIT
  role: Competitor short-read assembler whose contigs are evaluated for NGA50 comparison
    in the MP mock benchmark
  repo: https://github.com/voutcn/megahit
provenance:
  source_task_ids:
  - task_007
  - task_002
  source_papers:
  - doi: 10.1093/nar/gkab831
    title: 'MetaPlatanus: a metagenome assembler that combines long-range sequence
      links and species-specific features'
  source_content_hash: sha256:2747d09139a8a7cd179d9fa4d05a0b9ae812f2a1282cf87c56fc3a7bd2132562
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/rkajitani/MetaPlatanus
  repo_url: https://github.com/lh3/minimap2
  tools:
  - IDBA-UD
  - MEGAHIT
  - MetaPlatanus
  - NextPolish
  - SPAdes
  - TGS-GapCloser
  - metaSPAdes
  - minimap2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# assembly-contiguity-metric-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute reference-based assembly-quality metrics — misassembly counts, NGA50, and mean sequence identity — to quantify contiguity and accuracy of metagenome assemblies (e.g., MetaPlatanus out_final.fa) and to validate ablation and assembler-comparison benchmarks.

## When to use

Use when FASTA assemblies (contigs/scaffolds) from metagenome assemblers must be scored against a known reference to compare contiguity (NGA50), misassemblies, and base-level accuracy across tools or ablation modes.

## When NOT to use

- Reference genomes are unavailable or the study is fully de novo with unknown community composition — NGA50 and misassembly counts are reference-based and cannot be computed.
- Input is raw reads rather than an assembled FASTA; use a metagenome assembler (e.g., MetaPlatanus) first.
- Only base-level error profiling of a single assembly is needed without contiguity or misjoin assessment across tools.

## Inputs

- Assembled scaffold/contig FASTA files (e.g., MetaPlatanus out_final.fa from default, 'w/o closing', and 'w/o correction' modes)
- Reference genome FASTA set (e.g., the 20 known bacteria of the GIS20 or MP mock communities)
- Assembler run metadata (flags used, e.g., -IP1, -ont, -p, -no_tgsgapcloser, -no_nextpolish, -OP mate-pair libraries)

## Outputs

- Per-assembly metrics: misassembly count, NGA50, mean sequence identity
- Comparison table (TSV) mirroring Supplementary Table S9 / S14 (e.g., nga50_comparison.tsv)
- Bar chart of NGA50 per assembler per dataset analogous to Supplementary Figure S5

## How to apply

Take each scaffold set (e.g., MetaPlatanus out_final.fa produced with -IP1 paired-end plus -ont or -p long reads, or the -no_tgsgapcloser and -no_nextpolish ablation variants) and align it to the reference genome set using an assembler-to-reference aligner such as minimap2. From the alignments compute, per assembly, the number of misassemblies (chimeric/interspecies joins), NGA50 (contiguity of aligned blocks relative to reference length), and mean sequence identity of aligned regions. Tabulate the per-assembly metrics into a single comparison table mirroring Table 1A-B, Supplementary Table S9, or Supplementary Table S14, and optionally plot NGA50 per assembler per dataset as a bar chart analogous to Supplementary Figure S5. Validation is differential: for the ablation comparison, the 'w/o correction' run must show increased misassemblies and decreased NGA50 relative to the full run, while the 'w/o closing' run must show the highest mean sequence identity and fewest misassemblies; for the MP mock comparison, MetaPlatanus must attain the largest NGA50 across mock cases 1-3.

## Related tools

- **minimap2** (Aligns assembled scaffolds to the reference genome set to derive misassembly, NGA50, and identity statistics) — https://github.com/lh3/minimap2
- **MetaPlatanus** (Produces the out_final.fa assemblies (default, -no_tgsgapcloser, -no_nextpolish modes) that are scored for NGA50 and misassemblies) — https://github.com/rkajitani/MetaPlatanus
- **MEGAHIT** (Competitor short-read assembler whose contigs are evaluated for NGA50 comparison in the MP mock benchmark) — https://github.com/voutcn/megahit

## Examples

```
minimap2 -cx asm5 gis20_reference.fa out_final.fa > asm_to_ref.paf  # then compute misassemblies, NGA50 and mean identity from the alignment
```

## Evaluation signals

- The 'w/o correction' (-no_nextpolish) run shows increased misassemblies and decreased NGA50 relative to the full MetaPlatanus run on the GIS20 mock.
- The 'w/o closing' (-no_tgsgapcloser) run shows the highest mean sequence identity and fewest misassemblies among the compared runs.
- In the MP mock benchmark (HiSeq2500/MiSeq paired-end, 4 kbp and 8 kbp mate-pair libraries from PRJDB4377), MetaPlatanus attains the largest NGA50 for mock cases 1-3.
- Every assembly in the comparison has all three metrics (misassemblies, NGA50, mean sequence identity) populated, and values match the structure of Supplementary Table S9 / S14.

## Limitations

- Assessment is reference-based; it cannot be applied to real environmental metagenomes without known genomes (the paper notes reference information was not well examined in its de novo focus).
- NGA50 and misassembly counts measure contiguity and interspecies misjoins but not full biological element recovery; key evidence (Supplementary Tables S17, S24, S25) is deferred to supplementary materials.
- MetaPlatanus requires at least one short-read paired-end library, so metric comparisons of purely long-read assemblies fall outside this workflow's inputs.

## Evidence

- [abstract] Benchmarking using mock datasets consisting of known bacteria with long reads or mate pairs revealed the high contiguity MetaPlatanus with a few interspecies misassemblies.: "Benchmarking using mock datasets consisting of known bacteria with long reads or mate pairs revealed the high contiguity MetaPlatanus with a few interspecies misassemblies."
- [methods] Contig-assembly, scaffolding, gap-closing and binning are automatically executed at once.: "(3) Contig-assembly, scaffolding, gap-closing and binning are automatically executed at once."
- [supplementary] Raw reads of the MP mock samples were deposited in the DDBJ Sequence Read Archive under BioProject ID PRJDB4377.: "Raw reads of the MP mock samples were deposited in the DDBJ Sequence Read Archive under BioProject ID PRJDB4377."
- [readme] MetaPlatanus requires at least one short-read paired-end library.: "(4) MetaPlatanus requires at least one short-read paired-end library."
