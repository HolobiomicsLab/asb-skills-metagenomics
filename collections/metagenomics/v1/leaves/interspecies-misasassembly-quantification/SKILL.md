---
name: interspecies-misasassembly-quantification
description: Use when comparing metagenome assemblers (e.g. MetaPlatanus, metaSPAdes,
  OPERA-MS, metaFlye) on a mock community whose bacterial reference genomes are known,
  and misassembly/NGA50 counts must be extracted with metaQUAST.
when_to_use_negative:
- Input is real (unknown-composition) metagenome data with no reference genomes for
  the constituent species — reference-based QUAST evaluation of interspecies misassemblies
  is impossible; use binning-based or read-mapping QC instead
- The goal is reference-informed or contig binning evaluation — the article notes
  'this study focuses on de novo methods, and reference information has not been well
  examined'
edam_operation: http://edamontology.org/operation_3180
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: metaQUAST
  role: Reference-based evaluation of assemblies against known mock bacteria reference
    genomes to produce NGA50, misassembly and interspecies misassembly counts
- name: MetaPlatanus
  role: One of the assemblers under evaluation; its output (out_final.fa) is scored
    for contiguity and interspecies misassemblies
  repo: https://github.com/rkajitani/MetaPlatanus
- name: metaFlye
  role: Comparator assembler; its polishing module is executed as the default action
    before evaluation
- name: OPERA-MS
  role: Comparator assembler run in de novo mode
- name: metaSPAdes
  role: Comparator assembler run in hybrid mode
- name: MEGAHIT
  role: Initial contig assembler invoked inside MetaPlatanus (default k-mer coverage
    cutoff 4)
  repo: https://github.com/voutcn/megahit
- name: TGS-GapCloser
  role: Gap-closing step inside MetaPlatanus prior to evaluation
  repo: https://github.com/BGI-Qingdao/TGS-GapCloser
- name: NextPolish
  role: Polishing step inside MetaPlatanus prior to evaluation
  repo: https://github.com/Nextomics/NextPolish
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkab831
    title: 'MetaPlatanus: a metagenome assembler that combines long-range sequence
      links and species-specific features'
  source_content_hash: sha256:d95e8be2611e951af227dc7d8cf7c34ab5f40d303d4d50bce921ea93a8cf7e77
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/rkajitani/MetaPlatanus
  repo_url: https://github.com/rkajitani/MetaPlatanus
  tools:
  - MEGAHIT
  - MetaPlatanus
  - NextPolish
  - OPERA-MS
  - TGS-GapCloser
  - metaFlye
  - metaQUAST
  - metaSPAdes
  edam_operation: http://edamontology.org/operation_3180
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# interspecies-misasassembly-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies misassemblies and, specifically, interspecies misassemblies of metagenome assemblies on mock communities with known reference genomes using QUAST/metaQUAST, as done in the MetaPlatanus GIS20 mock benchmark alongside NGA50 and mean sequence identity metrics.

## When to use

Use when comparing metagenome assemblers (e.g. MetaPlatanus, metaSPAdes, OPERA-MS, metaFlye) on a mock community whose bacterial reference genomes are known, and misassembly/NGA50 counts must be extracted with metaQUAST.

## When NOT to use

- Input is real (unknown-composition) metagenome data with no reference genomes for the constituent species — reference-based QUAST evaluation of interspecies misassemblies is impossible; use binning-based or read-mapping QC instead
- The goal is reference-informed or contig binning evaluation — the article notes 'this study focuses on de novo methods, and reference information has not been well examined'

## Inputs

- Assembled contig/scaffold FASTA files from each assembler (e.g. MetaPlatanus out_final.fa, metaSPAdes, OPERA-MS de novo, metaFlye outputs)
- Reference genomes (FASTA) of the known GIS20 mock bacteria
- Illumina paired-end reads plus ONT and PacBio long-read sets from the public GIS20 mock dataset

## Outputs

- metaQUAST report per assembly (NGA50, misassembly count, mean sequence identity, interspecies misassembly count)
- Consolidated comparison table, e.g. gis20_benchmark_quast.tsv, mapping each assembler's metrics to Table 1 / Supplementary Tables S8–S10 values

## How to apply

Assemble the identical read sets (Illumina paired-end plus ONT or PacBio long reads from the GIS20 mock dataset) with each assembler under test. Run metaQUAST on all assemblies against the reference genomes of the known mock bacteria so that each contig/scaffold is assigned to its source species. Extract per-assembler NGA50, total misassembly count, mean sequence identity, and interspecies misassembly count into a table (e.g. gis20_benchmark_quast.tsv) and compare against the values reported in the paper's Table 1 and Supplementary Tables S8–S10. MetaPlatanus was benchmarked with its defaults: MEGAHIT-based contig assembly with k-mer coverage cutoff 4, MetaBAT2 binning with minimum mapping identity 97%, TGS-GapCloser gap closing and NextPolish polishing.

## Related tools

- **metaQUAST** (Reference-based evaluation of assemblies against known mock bacteria reference genomes to produce NGA50, misassembly and interspecies misassembly counts)
- **MetaPlatanus** (One of the assemblers under evaluation; its output (out_final.fa) is scored for contiguity and interspecies misassemblies) — https://github.com/rkajitani/MetaPlatanus
- **metaFlye** (Comparator assembler; its polishing module is executed as the default action before evaluation)
- **OPERA-MS** (Comparator assembler run in de novo mode)
- **metaSPAdes** (Comparator assembler run in hybrid mode)
- **MEGAHIT** (Initial contig assembler invoked inside MetaPlatanus (default k-mer coverage cutoff 4)) — https://github.com/voutcn/megahit
- **TGS-GapCloser** (Gap-closing step inside MetaPlatanus prior to evaluation) — https://github.com/BGI-Qingdao/TGS-GapCloser
- **NextPolish** (Polishing step inside MetaPlatanus prior to evaluation) — https://github.com/Nextomics/NextPolish

## Examples

```
metaplatanus -t 8 -m 64 -IP1 PE_1.fq PE_2.fq -ont ONT.fq -p pacbio.fq && metaquast.py --min-contig 500 metaplatanus/out_final.fa metaspades.fasta operams.fasta metaflye.fasta -o quast_out -r mock_refs/
```

## Evaluation signals

- metaQUAST completes on all four assemblies and assigns each scaffold to a known mock reference genome
- Extracted NGA50 values are finite and reproducible; MetaPlatanus should show the highest NGA50s on both ONT and PacBio GIS20 datasets relative to metaSPAdes, OPERA-MS and metaFlye
- Interspecies misassembly count for MetaPlatanus is low, consistent with 'high contiguity ... with a few interspecies misassemblies'
- Per-assembler metrics in gis20_benchmark_quast.tsv match Table 1 and Supplementary Tables S8–S10 within reporting tolerance
- Assemblies compared were generated from the identical read sets, so differences are attributable to the assemblers

## Limitations

- Interspecies misassembly quantification requires mock communities with known reference genomes; it cannot be computed for real environmental samples
- No tool versions were reported for the compared assemblers (metaFlye, OPERA-MS, MEGAHIT, etc.), so exact reproduction may require version pinning from the paper's SI
- Reference-based methods are not well examined in the study; the metric is specific to de novo assembler comparison
- Key benchmark evidence is in supplementary materials (Supplementary Tables S8–S10, S17, S24, S25) which must be retrieved separately

## Evidence

- [results] On the GIS20 mock benchmark, MetaPlatanus achieved the highest NGA50s for both the ONT and PacBio long-read datasets compared to the other tested assemblers.: [evidence span withheld — anchor status: not_found]
