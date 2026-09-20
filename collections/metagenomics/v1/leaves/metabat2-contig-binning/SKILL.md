---
name: metabat2-contig-binning
description: Use when you have multi-sample assembled metagenomic contigs (e.g. metaSPAdes/MegaHIT
  output from CAMI II GI/Air/PA mock communities) that must be grouped into per-sample
  MAGs before taxonomic classification.
when_to_use_negative:
- Input is raw shotgun reads rather than assembled contigs — MetaBAT2 bins contigs,
  so assembly must precede binning
- Contigs are shorter than the 1,500 bp minimum used in this benchmark (--minS/-m
  setting), e.g. single-sample assemblies with only short contigs
- The goal is read-level taxonomic profiling rather than MAG-level classification,
  which is what kMetaShot/GTDBtk/CAMITAX consume downstream
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0176
- http://edamontology.org/topic_3697
tools:
- name: MetaBAT2 (v2:2.15)
  role: Contig binner producing per-sample MAGs with --maxP 99 --minS 98 -m 1500
- name: metaSPAdes v3.15.2
  role: PE read meta-assembler whose contig output is binned
- name: MegaHIT v1.2.9
  role: Alternative PE read meta-assembler (-k 35, 57, 79, 99) whose contig output
    is binned
- name: FastQC
  role: Quality check of reads before assembly and binning
- name: Sickle
  role: Read trimming (-q 32 -l 50) prior to assembly
- name: kMetaShot
  role: Downstream classifier consuming the MetaBAT2 bins directory via -b bins/
  repo: github.com/gdefazio/kMetaShot
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bib/bbae680
    title: 'kMetaShot: a fast and reliable taxonomy classifier for metagenome-assembled
      genomes'
  source_content_hash: sha256:112fd575be3244b9eb53cb132a03767d3359fcc4647f81c2ac20a3c40f44b435
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CAMITAX
  - FastQC
  - GTDBtk
  - MegaHIT v1.2.9
  - MetaBAT2
  - MetaBAT2 (v2:2.15)
  - Sickle
  - kMetaShot
  - metaSPAdes v3.15.2
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0176
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# metabat2-contig-binning

## Summary

Bins assembled metagenomic contigs into metagenome-assembled genomes (MAGs) with MetaBAT2 (v2:2.15), the binning step that produced the MAG inputs classified in the kMetaShot CAMI II benchmark against GTDBtk and CAMITAX.

## When to use

Use when you have multi-sample assembled metagenomic contigs (e.g. metaSPAdes/MegaHIT output from CAMI II GI/Air/PA mock communities) that must be grouped into per-sample MAGs before taxonomic classification.

## When NOT to use

- Input is raw shotgun reads rather than assembled contigs — MetaBAT2 bins contigs, so assembly must precede binning
- Contigs are shorter than the 1,500 bp minimum used in this benchmark (--minS/-m setting), e.g. single-sample assemblies with only short contigs
- The goal is read-level taxonomic profiling rather than MAG-level classification, which is what kMetaShot/GTDBtk/CAMITAX consume downstream

## Inputs

- Per-sample assembled contig FASTA files (from metaSPAdes v3.15.2 or MegaHIT v1.2.9, -k 35, 57, 79, 99)
- Quality-checked, trimmed PE reads (FastQC check; Sickle -q 32 -l 50) as the basis of the assemblies
- CAMI II multi-sample mock community contigs (GI-Illumina, GI-PacBio, Air-Illumina, Air-PacBio, PA-PacBio)

## Outputs

- Per-sample MAG FASTA files (bins/ directory, contigs >= 1,500 bp grouped by MetaBAT2)
- A bins directory directly consumable by kMetaShot_classifier_NV.py via -b bins/

## How to apply

Assemble quality-controlled PE reads (FastQC check, Sickle trimming at -q 32 -l 50) with metaSPAdes v3.15.2 and MegaHIT v1.2.9 (-k 35, 57, 79, 99), then bin the provided contigs separately per sample with MetaBAT2 v2:2.15 using --maxP 99 --minS 98 -m 1500, i.e. a minimum contig length of 1,500 bp, to obtain the MAGs. Each sample in the CAMI II multi-sample mock community (10 GI/Air samples per condition, 20 PA-PacBio samples) is binned independently, so outputs are per-sample MAG FASTA sets. Run the bins in a directory (e.g. bins/) that can be passed directly to downstream classifiers such as kMetaShot_classifier_NV.py (-b bins/). The 1,500 bp minimum-size threshold is the paper's concrete setting and should be preserved for reproducibility of the benchmark.

## Related tools

- **MetaBAT2 (v2:2.15)** (Contig binner producing per-sample MAGs with --maxP 99 --minS 98 -m 1500)
- **metaSPAdes v3.15.2** (PE read meta-assembler whose contig output is binned)
- **MegaHIT v1.2.9** (Alternative PE read meta-assembler (-k 35, 57, 79, 99) whose contig output is binned)
- **FastQC** (Quality check of reads before assembly and binning)
- **Sickle** (Read trimming (-q 32 -l 50) prior to assembly)
- **kMetaShot** (Downstream classifier consuming the MetaBAT2 bins directory via -b bins/) — github.com/gdefazio/kMetaShot

## Examples

```
metabat2 -i assembly_megahit.fasta -o bins/sample_mag --maxP 99 --minS 98 -m 1500
```

## Evaluation signals

- Binning is run per sample with the exact parameter set --maxP 99 --minS 98 -m 1500; verify no sample was pooled across conditions (10 GI/Air samples per condition, 20 PA-PacBio samples binned separately)
- Output bins must exclude contigs shorter than 1,500 bp (the -m minimum-size setting)
- The bins/ directory should be directly loadable by kMetaShot_classifier_NV.py -b bins/ without reformatting
- Downstream benchmark metrics (sensitivity/precision/BA/F1 vs CAMI expected taxonomy CE) are only valid if binning precedes kMetaShot classification with ass2ref 0.2
- kMetaShot sensitivity at species level should fall in the reported ranges (e.g. GI-Illumina species sensitivity median 69.75, IQR 3.86) when bins are reproducibly regenerated

## Limitations

- Binning is applied separately per sample on the provided contigs; pooling contigs across samples would deviate from the benchmark design
- The minimum contig length of 1,500 bp means shorter contigs are discarded and cannot contribute to MAGs
- MetaBAT2 MAG quality was not independently re-evaluated here — the benchmark measures classification accuracy of whatever bins MetaBAT2 produced, not binning completeness/contamination itself

## Evidence

- [methods] MetaBAT2 binning parameters: "the provided contigs were binned separately with MetaBAT2 (v2:2.15) [35] (--maxP 99 --minS 98 -m 1,500) to obtain MAGs"
- [methods] Assembly and QC chain: "After a quality check with FastQC [58] and trimming with Sickle [59] (-q 32 -l 50)"
- [methods] Assemblers used upstream of binning: "PE reads were meta-assembled with metaSpades (v3.15.2) [44] and MegaHIT (v1.2.9) [45] (for both -k 35, 57, 79, 99)"
- [results] Bins consumed by kMetaShot: "For kMetaShot, an ass2ref of 0.2 was set for all datasets."
- [readme] CLI accepting a bins directory: "Path to a directory containing bins fasta files or 
                        path to a multi-fasta file where each header corresponds"
