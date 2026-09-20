---
name: checkm-completeness-contamination-estimation
description: Use when benchmarking metagenome assemblies of a target species and extracted
  scaffold sets (e.g. B. longum 51A scaffolds from MetaPlatanus vs metaFlye/OPERA-MS/MEGAHIT)
  need completeness and contamination estimates to confirm which assembler produced
  a single near-complete bin.
when_to_use_negative:
- Input is raw short- or long-read data rather than assembled scaffold/contig FASTA
  sets — CheckM assesses assemblies, not reads
- The task is performing de novo assembly itself (contig-assembly, scaffolding, gap-closing
  with MetaPlatanus/MEGAHIT/metaFlye) rather than post-hoc quality assessment of existing
  assemblies
- The question is assembler contiguity statistics (N50, number of contigs) alone without
  a per-species completeness/contamination readout
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: CheckM
  role: Marker-gene-based estimation of genome completeness and contamination on the
    extracted per-assembler scaffold sets
- name: Minimap2
  role: asm5 assembly-to-assembly/reference alignment used to map extracted scaffolds
    to B. longum 51A and quantify edge error (minor 1k-mer rates) alongside CheckM
  repo: https://github.com/lh3/minimap2
- name: MetaPlatanus
  role: Source of the scaffold2094 assembly under quality assessment (V06-T-0501-S07
    human gut sample)
  repo: https://github.com/rkajitani/MetaPlatanus
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nar/gkab831
    title: 'MetaPlatanus: a metagenome assembler that combines long-range sequence
      links and species-specific features'
  source_content_hash: sha256:35eb8300f797b9337ff3c12f976616be69342fbbeb4a4010d1155c3d2ee3b2c5
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
  - CheckM
  - MetaPlatanus
  - Minimap2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# checkm-completeness-contamination-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Estimate genome completeness and contamination of extracted metagenome scaffold/contig sets with CheckM to validate near-complete, structurally correct assemblies — e.g. confirming that MetaPlatanus scaffold2094 (sample V06-T-0501-S07) captures the B. longum 51A genome while competitor assemblies are fragmented.

## When to use

Use when benchmarking metagenome assemblies of a target species and extracted scaffold sets (e.g. B. longum 51A scaffolds from MetaPlatanus vs metaFlye/OPERA-MS/MEGAHIT) need completeness and contamination estimates to confirm which assembler produced a single near-complete bin.

## When NOT to use

- Input is raw short- or long-read data rather than assembled scaffold/contig FASTA sets — CheckM assesses assemblies, not reads
- The task is performing de novo assembly itself (contig-assembly, scaffolding, gap-closing with MetaPlatanus/MEGAHIT/metaFlye) rather than post-hoc quality assessment of existing assemblies
- The question is assembler contiguity statistics (N50, number of contigs) alone without a per-species completeness/contamination readout

## Inputs

- Per-assembler scaffold/contig FASTA sets from the FigShare deposit doi:10.6084/m9.figshare.15090603.v1, including MetaPlatanus V06-T-0501-S07 scaffold2094 and corresponding metaFlye/OPERA-MS/MEGAHIT scaffolds
- Complete Bifidobacterium longum strain 51A reference genome (FASTA)
- Minimap2 asm5 PAF alignments of extracted scaffolds against the B. longum 51A reference

## Outputs

- CheckM genome completeness and contamination estimates for each extracted scaffold set
- Comparison table blongum_51A_scaffold_comparison.tsv (TSV with rows per assembler: scaffold counts, CheckM completeness/contamination, minor 1k-mer edge rates)

## How to apply

Retrieve the study assemblies (MetaPlatanus and competitors) from the FigShare deposit doi:10.6084/m9.figshare.15090603.v1, extract scaffold2094 from the MetaPlatanus V06-T-0501-S07 assembly and the corresponding scaffolds/contigs covering B. longum 51A from each competitor assembly (metaFlye, OPERA-MS, MEGAHIT). Align each extracted sequence set to the complete B. longum 51A reference using minimap2's asm5 assembly-to-assembly/asm-to-reference preset to confirm species correspondence and compute the minor 1k-mer rate within 1 kb of each scaffold edge as an independent structural-error readout. Run CheckM on each extracted scaffold set to obtain genome completeness and contamination estimates. Compile scaffold counts, CheckM completeness/contamination, and minor 1k-mer edge rates into blongum_51A_scaffold_comparison.tsv with one row per assembler. The decision point reproducing the paper's Figure 3 finding: only MetaPlatanus should show a single near-complete scaffold with low contamination, whereas competitor assemblies are fragmented into more than seven sequences with signs of interspecies misassemblies at their edges.

## Related tools

- **CheckM** (Marker-gene-based estimation of genome completeness and contamination on the extracted per-assembler scaffold sets)
- **Minimap2** (asm5 assembly-to-assembly/reference alignment used to map extracted scaffolds to B. longum 51A and quantify edge error (minor 1k-mer rates) alongside CheckM) — https://github.com/lh3/minimap2
- **MetaPlatanus** (Source of the scaffold2094 assembly under quality assessment (V06-T-0501-S07 human gut sample)) — https://github.com/rkajitani/MetaPlatanus

## Examples

```
minimap2 -cx asm5 B_longum_51A.fa extracted_scaffolds.fa > blongum_aln.paf
```

## Evaluation signals

- blongum_51A_scaffold_comparison.tsv exists in TSV format with one row for every assembler (MetaPlatanus, metaFlye, OPERA-MS, MEGAHIT)
- CheckM reports near-complete completeness with low contamination for the single MetaPlatanus scaffold2094
- Competitor assemblies covering B. longum 51A are fragmented into more than seven sequences, matching the paper's fragmentation criterion
- Minor 1k-mer rates within 1 kb of scaffold edges are consistent with interspecies misassemblies at competitor scaffold edges but not in the MetaPlatanus scaffold
- The compiled table reproduces the Figure 3 finding that only MetaPlatanus yields a single near-complete scaffold of B. longum 51A

## Limitations

- CheckM completeness/contamination do not by themselves detect structural errors such as interspecies misassemblies at scaffold edges; alignment-based edge metrics (minor 1k-mer rate) are needed alongside
- No tool versions were reported for the compared assemblers, and the study focuses on de novo methods — reference information 'has not been well examined' (Supplementary Tables S17, S24, S25 and Figure 6 were deferred to supplementary materials)
- Findings are grounded in the specific human gut benchmark (V06-T-0501-S07) and B. longum 51A reference; generalization to other samples or species was not established in this section

## Evidence

- [readme] ./minimap2 -cx asm5 asm1.fa asm2.fa > aln.paf             # intra-species asm-to-asm alignment: [evidence span withheld — anchor status: external_doc]
- [other] Run CheckM on the extracted scaffold sets to obtain genome completeness and contamination estimates.: [evidence span withheld — anchor status: not_found]
