---
name: mock-community-reference-based-evaluation
description: Use when a known-composition mock metagenome (e.g. GIS20 bacteria with
  Illumina paired-end plus ONT or PacBio long reads) is being assembled and contiguity/accuracy
  must be compared across assemblers against reference genomes.
when_to_use_negative:
- Real environmental metagenomes or clinical samples without known reference genomes
  — the article notes 'this study focuses on de novo methods, and reference information
  has not been well examined', and the reference-based NGA50/interspecies-misassembly
  evaluation requires known mock composition
- Evaluating a single assembly rather than comparing multiple assemblers on identical
  read sets — the benchmark design depends on the same Illumina PE plus ONT/PacBio
  inputs being run through each assembler
- Assembly questions that depend on local graph structures or polishing quality alone,
  where metaQUAST NGA50/misassembly statistics are not the deciding metric
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: MetaPlatanus
  role: Hybrid metagenome assembler under test; one invocation performs MEGAHIT contig
    assembly, scaffolding, MetaBAT2 binning, TGS-GapCloser gap closing and NextPolish
    polishing
  repo: https://github.com/rkajitani/MetaPlatanus
- name: metaQUAST
  role: Reference-based evaluation of all assemblies against the known mock bacteria
    reference genomes to yield NGA50, misassemblies, identity, and interspecies misassemblies
- name: metaSPAdes
  role: Compared assembler, run in hybrid mode on the identical read sets
- name: OPERA-MS
  role: Compared assembler, run in de novo mode
- name: metaFlye
  role: Compared assembler, with its polishing module executed as the default action
- name: MEGAHIT
  role: Initial contig assembler invoked internally by MetaPlatanus (default k-mer
    coverage cutoff 4)
  repo: https://github.com/voutcn/megahit
- name: TGS-GapCloser
  role: Gap-closing step integrated into the MetaPlatanus pipeline being benchmarked
  repo: https://github.com/BGI-Qingdao/TGS-GapCloser
- name: NextPolish
  role: Polishing step integrated into the MetaPlatanus pipeline being benchmarked
  repo: https://github.com/Nextomics/NextPolish
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nar/gkab831
    title: 'MetaPlatanus: a metagenome assembler that combines long-range sequence
      links and species-specific features'
  source_content_hash: sha256:2901da150857e7101ddd20f1f8cbb1ecfef175295894d39cb113b83720594e10
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
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# mock-community-reference-based-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark metagenome assemblers (MetaPlatanus v1.3.0 vs metaSPAdes hybrid, OPERA-MS de novo, metaFlye) on a mock community with known reference genomes, using QUAST/metaQUAST to extract NGA50, misassemblies, sequence identity, and interspecies misassemblies.

## When to use

Use when a known-composition mock metagenome (e.g. GIS20 bacteria with Illumina paired-end plus ONT or PacBio long reads) is being assembled and contiguity/accuracy must be compared across assemblers against reference genomes.

## When NOT to use

- Real environmental metagenomes or clinical samples without known reference genomes — the article notes 'this study focuses on de novo methods, and reference information has not been well examined', and the reference-based NGA50/interspecies-misassembly evaluation requires known mock composition
- Evaluating a single assembly rather than comparing multiple assemblers on identical read sets — the benchmark design depends on the same Illumina PE plus ONT/PacBio inputs being run through each assembler
- Assembly questions that depend on local graph structures or polishing quality alone, where metaQUAST NGA50/misassembly statistics are not the deciding metric

## Inputs

- GIS20 mock community Illumina paired-end reads (FASTQ, from the SRA accession cited in ref. 22; MP mock raw reads at DDBJ BioProject PRJDB4377)
- Mock community ONT long-read FASTQ
- Mock community PacBio long-read FASTQ
- Reference genomes of the known mock bacteria (FASTA, for metaQUAST)

## Outputs

- Final assemblies per assembler (e.g. MetaPlatanus out_result/out_final.fa; all study assemblies deposited at FigShare doi:10.6084/m9.figshare.15090603.v1)
- metaQUAST report per assembly with NGA50, misassembly count, mean sequence identity, and interspecies misassembly count
- Combined benchmark table gis20_benchmark_quast.tsv comparable to Table 1 and Supplementary Tables S8-S10

## How to apply

Assemble the identical read sets with each assembler: run MetaPlatanus once (it automatically performs MEGAHIT contig assembly with default k-mer coverage cutoff 4, scaffolding, MetaBAT2 binning at 97% minimum mapping identity, TGS-GapCloser gap closing and NextPolish polishing, writing out_result/out_final.fa), then metaSPAdes in hybrid mode, OPERA-MS in de novo mode, and metaFlye with its polishing module as the default action. Run metaQUAST on all four assemblies against the reference genomes of the known mock bacteria. Extract NGA50, misassembly count, mean sequence identity, and interspecies misassembly count per assembler into a comparison table (gis20_benchmark_quast.tsv). Judge correctness by comparing the extracted values against those reported in the paper's Table 1 and Supplementary Tables S8-S10; MetaPlatanus is expected to show the highest NGA50s for both ONT and PacBio long-read datasets.

## Related tools

- **MetaPlatanus** (Hybrid metagenome assembler under test; one invocation performs MEGAHIT contig assembly, scaffolding, MetaBAT2 binning, TGS-GapCloser gap closing and NextPolish polishing) — https://github.com/rkajitani/MetaPlatanus
- **metaQUAST** (Reference-based evaluation of all assemblies against the known mock bacteria reference genomes to yield NGA50, misassemblies, identity, and interspecies misassemblies)
- **metaSPAdes** (Compared assembler, run in hybrid mode on the identical read sets)
- **OPERA-MS** (Compared assembler, run in de novo mode)
- **metaFlye** (Compared assembler, with its polishing module executed as the default action)
- **MEGAHIT** (Initial contig assembler invoked internally by MetaPlatanus (default k-mer coverage cutoff 4)) — https://github.com/voutcn/megahit
- **TGS-GapCloser** (Gap-closing step integrated into the MetaPlatanus pipeline being benchmarked) — https://github.com/BGI-Qingdao/TGS-GapCloser
- **NextPolish** (Polishing step integrated into the MetaPlatanus pipeline being benchmarked) — https://github.com/Nextomics/NextPolish

## Examples

```
metaplatanus -t 8 -m 64 -IP1 PE_1.fq PE_2.fq -ont ONT.fq -p pacbio.fq && metaquast.py out_result/out_final.fa metaflye_scaffolds.fasta metaphlan_hybrid.fasta opera_ms_assembly.fasta -r mock_refs/ -o gis20_quast
```

## Evaluation signals

- MetaPlatanus achieves the highest NGA50s on both the ONT and PacBio GIS20 long-read datasets compared to metaSPAdes, OPERA-MS, and metaFlye, as reported in Table 1 and Supplementary Tables S8-S10
- Interspecies misassembly counts remain low for MetaPlatanus relative to the compared assemblers, matching the paper's claim of 'high contiguity ... with a few interspecies misassemblies'
- All four assemblers are run on identical read sets and evaluated by the same metaQUAST invocation against the same set of mock reference genomes
- Extracted gis20_benchmark_quast.tsv contains NGA50, misassembly count, mean sequence identity, and interspecies misassembly count for every assembler
- Reproduced assemblies can be checked against the deposited FigShare assemblies (doi:10.6084/m9.figshare.15090603.v1) and raw reads at DDBJ PRJDB4377

## Limitations

- Versions of the compared assemblers (metaFlye, OPERA-MS, MEGAHIT, metaSPAdes) are not reported in the article, so exact reproduction may vary
- metaFlye also requires additional short-read sequencing and polishing for rigorous downstream analysis, which affects fair comparison of its raw output
- Key quantitative evidence is deferred to supplementary tables (S8-S10, S17, S24, S25) that are not in the main text

## Evidence
