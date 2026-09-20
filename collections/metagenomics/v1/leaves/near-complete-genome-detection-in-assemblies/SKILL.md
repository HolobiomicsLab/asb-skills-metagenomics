---
name: near-complete-genome-detection-in-assemblies
description: Use when comparing metagenome assembler outputs (FASTA scaffolds, e.g.
  MetaPlatanus out_final.fa) and you must count near-complete genomes, genes, gene
  clusters or viral sequences to rank assemblers on high-depth ONT/PromethION + Illumina
  data.
when_to_use_negative:
- Input is raw reads rather than assembled scaffold FASTA — this skill evaluates assemblies,
  it does not run the assembly itself beyond the benchmark comparison step
- Reference-based evaluation is required — the study explicitly notes reference information
  has not been well examined, focusing on de novo methods
- Data lack long-range links (only short paired-end reads from low-abundance communities)
  — near-complete recovery relies on long reads (ONT/PromethION) or mate-pair links
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: MetaPlatanus
  role: Hybrid metagenome assembler whose contig-assembly, scaffolding, gap-closing
    and binning output (out_final.fa) is evaluated for near-complete scaffolds; run
    with -IP1 for paired-end and -ont for ONT reads
  repo: https://github.com/rkajitani/MetaPlatanus
- name: MEGAHIT
  role: Short-read-only comparator assembler and the initial contig assembler inside
    MetaPlatanus (k-mer coverage cutoff 4)
  repo: https://github.com/voutcn/megahit
- name: metaFlye
  role: Long-read comparator assembler run with its default polishing module
- name: OPERA-MS
  role: Hybrid short+long read comparator assembler with a binning-like clustering
    step
- name: minimap2
  role: Read mapper supporting the binning read mapping step underlying scaffold detection/evaluation
  repo: https://github.com/lh3/minimap2
- name: TGS-GapCloser
  role: Optional gap-closing tool integrated in the MetaPlatanus pipeline
  repo: https://github.com/BGI-Qingdao/TGS-GapCloser
- name: NextPolish
  role: Optional polishing tool integrated in the MetaPlatanus pipeline
  repo: https://github.com/Nextomics/NextPolish
- name: Racon
  role: Optional consensus module usable within the long-read error-correction path
  repo: https://github.com/isovic/racon
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nar/gkab831
    title: 'MetaPlatanus: a metagenome assembler that combines long-range sequence
      links and species-specific features'
  source_content_hash: sha256:44452619b040a6ec69729f6117dad95b305478b10a2f69a4b529f41846625571
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
  - Racon
  - TGS-GapCloser
  - metaFlye
  - minimap2
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# near-complete-genome-detection-in-assemblies

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detects biologically important elements — coding genes, gene clusters, viral sequences, and near-complete genomes — in metagenome assemblies to quantify which assembler recovers high-abundance bacterial genomes as near-complete scaffolds, as demonstrated on the MetaPlatanus human saliva PromethION benchmark where 22 scaffolds were assembled near-complete exclusively by MetaPlatanus.

## When to use

Use when comparing metagenome assembler outputs (FASTA scaffolds, e.g. MetaPlatanus out_final.fa) and you must count near-complete genomes, genes, gene clusters or viral sequences to rank assemblers on high-depth ONT/PromethION + Illumina data.

## When NOT to use

- Input is raw reads rather than assembled scaffold FASTA — this skill evaluates assemblies, it does not run the assembly itself beyond the benchmark comparison step
- Reference-based evaluation is required — the study explicitly notes reference information has not been well examined, focusing on de novo methods
- Data lack long-range links (only short paired-end reads from low-abundance communities) — near-complete recovery relies on long reads (ONT/PromethION) or mate-pair links

## Inputs

- Final scaffold FASTA per assembler (e.g. MetaPlatanus out_final.fa)
- MetaPlatanus bin assignment files (out_finalClusters, out_finalClusters.tsv)
- Illumina paired-end FASTQ and ONT PromethION FASTQ/FASTA read sets per saliva sample
- Assembler outputs from MEGAHIT (short-read-only), metaFlye (with default polishing), OPERA-MS

## Outputs

- Per-assembler table of detected elements: coding genes, gene clusters, viral sequences, near-complete genomes (Supplementary Tables S22, S24, S25 style)
- Nx-length contiguity plot comparing all assemblers
- List of scaffolds assembled near-complete exclusively by one assembler (expected: 22 MetaPlatanus-only on saliva)

## How to apply

Run each assembler on the same samples (here four human saliva samples with 28-75 Gbp ONT PromethION reads plus Illumina paired ends) and collect the final scaffold FASTA files (e.g. MetaPlatanus out_final.fa, with bin files out_finalClusters and out_finalClusters.tsv). Detect biologically important elements (coding genes, gene clusters, viral sequences, near-complete genomes) in every assembly and tabulate per-assembler counts, checking which scaffolds achieve near-complete genome status. As a benchmark reference, verify the 22 scaffolds assembled near-complete only by MetaPlatanus and that the Nx-length contiguity plot shows MetaPlatanus dominating MEGAHIT, metaFlye and OPERA-MS (Figures 4-6, Supplementary Tables S22, S24, S25). For the binning-based scaffold extension that enables near-complete recovery, MetaPlatanus uses a 97% minimum identity in read mapping for binning (-min_map_idt_binning) and a k-mer coverage cutoff of 4 with MEGAHIT (-min_cov_contig).

## Related tools

- **MetaPlatanus** (Hybrid metagenome assembler whose contig-assembly, scaffolding, gap-closing and binning output (out_final.fa) is evaluated for near-complete scaffolds; run with -IP1 for paired-end and -ont for ONT reads) — https://github.com/rkajitani/MetaPlatanus
- **MEGAHIT** (Short-read-only comparator assembler and the initial contig assembler inside MetaPlatanus (k-mer coverage cutoff 4)) — https://github.com/voutcn/megahit
- **metaFlye** (Long-read comparator assembler run with its default polishing module)
- **OPERA-MS** (Hybrid short+long read comparator assembler with a binning-like clustering step)
- **minimap2** (Read mapper supporting the binning read mapping step underlying scaffold detection/evaluation) — https://github.com/lh3/minimap2
- **TGS-GapCloser** (Optional gap-closing tool integrated in the MetaPlatanus pipeline) — https://github.com/BGI-Qingdao/TGS-GapCloser
- **NextPolish** (Optional polishing tool integrated in the MetaPlatanus pipeline) — https://github.com/Nextomics/NextPolish
- **Racon** (Optional consensus module usable within the long-read error-correction path) — https://github.com/isovic/racon

## Examples

```
metaplatanus -IP1 PE_1.fq PE_2.fq -ont ONT.fq  # then detect near-complete scaffolds in out_final.fa against comparator assemblies
```

## Evaluation signals

- Element detection table contains 22 near-complete scaffolds assembled exclusively by MetaPlatanus and none by MEGAHIT, metaFlye or OPERA-MS on the saliva benchmark
- Nx-length contiguity plot reproduces MetaPlatanus dominance over compared assemblers (Figures 4-6)
- Bin assignment files (out_finalClusters.tsv) are consistent with the scaffold set in out_final.fa
- Per-assembler element counts match reported Supplementary Tables S22, S24, S25
- MetaPlatanus run used the documented defaults: k-mer coverage cutoff 4 with MEGAHIT and 97% minimum identity for binning read mapping

## Limitations

- Key benchmark evidence (Supplementary Tables S17, S24, S25; Figure 6) is deferred to supplementary materials not present in the main text
- No tool versions were reported for the compared assemblers (metaFlye, OPERA-MS, MEGAHIT), complicating exact reproduction
- metaFlye requires additional short-read sequencing and polishing for rigorous downstream analysis; near-complete recovery is limited to high-abundance genomes since low-abundance species yield few long-range links

## Evidence
