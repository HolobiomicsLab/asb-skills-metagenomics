---
name: multi-method-bin-count-aggregation
description: Use when benchmarking a viral binning method against baselines on assembled
  metagenomic Hi-C data and you need per-method counts of high/medium-completion bins
  (CheckV) to quantify improvement percentages.
when_to_use_negative:
- The task is to bin prokaryotic MAGs rather than viral genomes — CheckM marker-gene
  assessment, not CheckV, applies and HiCBin/TAXAassign are appropriate
- The dataset has too few putative viral genomes for the benchmarking to be convincing,
  or the goal is to estimate contamination of vMAGs (CheckV cannot estimate contamination
  since no marker gene set exists for viruses)
- Only a single binning method is being run without baselines — no improvement percentages
  can be computed
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: ViralCC
  role: Primary Hi-C-based binning method whose high/medium-completion bin counts
    are compared against baselines
  repo: https://github.com/dyxstat/ViralCC
- name: CoCoNet
  role: Shotgun-based viral binning baseline included in the count comparison
- name: vRhyme
  role: Shotgun-based viral binning baseline included in the count comparison
- name: MetaTOR
  role: Hi-C-based binning baseline included in the count comparison
- name: bin3C
  role: Hi-C-based binning baseline included in the count comparison
- name: HiCBin
  role: Hi-C-based baseline (requires TAXAassign; cannot bin viral contigs)
- name: CheckV
  role: Estimates completeness of every recovered viral bin against its curated GenBank/environmental
    database and assigns high/medium completion
- name: MEGAHIT
  role: Assembles shotgun reads into contigs (v1.2.9, --min-contig-len 1000) shared
    by all methods
- name: BBTools
  role: bbduk adaptor/quality/ftl=10 trimming and clumpify.sh duplicate removal to
    standardize inputs across methods
- name: samtools
  role: Filters alignments (view -F 0x904) and name-sorts the BAM for binning input
    (v1.15.1)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-35945-y
    title: ViralCC retrieves complete viral genomes and virus-host pairs from metagenomic
      Hi-C data
  source_content_hash: sha256:ef131af45a2be45999459fca017967f8e3a0e8d258126cfca64f47afc678cbb9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dyxstat/ViralCC
  tools:
  - BBTools
  - BWA
  - CheckV
  - CoCoNet
  - HiCBin
  - MEGAHIT
  - MetaTOR
  - TAXAassign
  - ViralCC
  - bin3C
  - samtools
  - vRhyme
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# multi-method-bin-count-aggregation

## Summary

Runs multiple metagenomic binning tools (ViralCC, MetaTOR, bin3C, HiCBin, CoCoNet, vRhyme, VAMB) on the same shotgun+Hi-C dataset and aggregates per-method counts of high/medium-completion viral bins as scored by CheckV, then computes ViralCC's percentage improvements over each baseline.

## When to use

Use when benchmarking a viral binning method against baselines on assembled metagenomic Hi-C data and you need per-method counts of high/medium-completion bins (CheckV) to quantify improvement percentages.

## When NOT to use

- The task is to bin prokaryotic MAGs rather than viral genomes — CheckM marker-gene assessment, not CheckV, applies and HiCBin/TAXAassign are appropriate
- The dataset has too few putative viral genomes for the benchmarking to be convincing, or the goal is to estimate contamination of vMAGs (CheckV cannot estimate contamination since no marker gene set exists for viruses)
- Only a single binning method is being run without baselines — no improvement percentages can be computed

## Inputs

- Cleaned shotgun and Hi-C FASTQ files from public SRA/ENA accessions (SRR6131122-24, ERX2333418/ERX2548555-56, SRR8239392-93)
- MEGAHIT assembly FASTA of contigs (min-contig-len 1000)
- Name-sorted BAM of Hi-C reads aligned with BWA MEM and filtered via samtools view -F 0x904
- VirSorter VIRAL_file listing viral contigs
- Bins produced by each method: ViralCC, MetaTOR, bin3C, HiCBin, CoCoNet, vRhyme, VAMB

## Outputs

- Per-method, per-dataset counts of high- and medium-completion draft viral genomes (CheckV categories)
- Percentage improvements of ViralCC over each baseline method (Fig. 3 values)
- Source-data tables comparable to the dyxstat/Reproduce_ViralCC Source Data folder

## How to apply

For each real Hi-C dataset (human gut SRR6131122-24, cow fecal ERX2333418/ERX2548555-56, wastewater SRR8239392-93), clean reads with BBTools (bbduk adaptor/quality trimming with ftl=10 for Hi-C, clumpify.sh duplicate removal), assemble with MEGAHIT (--min-contig-len 1000), align Hi-C reads with BWA MEM -5SP, filter with samtools view -F 0x904, name-sort the BAM, and identify viral contigs with VirSorter to build the VIRAL_file. Run ViralCC (python ./viralcc.py pipeline FASTA_file BAM_file VIRAL_file OUTPUT_directory) and each baseline (MetaTOR, bin3C, HiCBin, CoCoNet, vRhyme, VAMB) on identical inputs. Estimate completeness of every recovered bin with CheckV (v0.7.0) against its curated GenBank/environmental database, assign high/medium completion categories, aggregate per-method per-dataset counts, and compute ViralCC's percentage improvement over each baseline (e.g., on cow fecal: 161% over VAMB, 140% over CoCoNet, 66.7% over vRhyme, 93.5% over bin3C, 62.1% over MetaTOR). Success is judged by whether reproduced counts and improvements match the paper's Fig. 3, whose source data is provided in the dyxstat/Reproduce_ViralCC repository.

## Related tools

- **ViralCC** (Primary Hi-C-based binning method whose high/medium-completion bin counts are compared against baselines) — https://github.com/dyxstat/ViralCC
- **CoCoNet** (Shotgun-based viral binning baseline included in the count comparison)
- **vRhyme** (Shotgun-based viral binning baseline included in the count comparison)
- **MetaTOR** (Hi-C-based binning baseline included in the count comparison)
- **bin3C** (Hi-C-based binning baseline included in the count comparison)
- **HiCBin** (Hi-C-based baseline (requires TAXAassign; cannot bin viral contigs))
- **CheckV** (Estimates completeness of every recovered viral bin against its curated GenBank/environmental database and assigns high/medium completion)
- **MEGAHIT** (Assembles shotgun reads into contigs (v1.2.9, --min-contig-len 1000) shared by all methods)
- **BBTools** (bbduk adaptor/quality/ftl=10 trimming and clumpify.sh duplicate removal to standardize inputs across methods)
- **samtools** (Filters alignments (view -F 0x904) and name-sorts the BAM for binning input (v1.15.1))

## Examples

```
python ./viralcc.py pipeline FASTA_file BAM_file VIRAL_file OUTPUT_directory
```

## Evaluation signals

- Reproduced per-method high/medium-completion counts match the values reported in Fig. 3 of the paper (source data available in dyxstat/Reproduce_ViralCC/Source Data)
- Computed percentage improvements match the reported values (cow fecal: 161% over VAMB, 140% over CoCoNet, 66.7% over vRhyme, 93.5% over bin3C, 62.1% over MetaTOR)
- A random binning control model is included, since random binning does not reduce CheckV completeness and the comparison is meaningless without it
- Every bin evaluated by CheckV originates from a viral genome recoverable by a single contig, per the benchmarking strategy
- Software versions match the paper: BBTools v37.25, MEGAHIT v1.2.9, BWA v0.7.17, samtools v1.15.1, VirSorter v1.0.6, CheckV 0.7.0

## Limitations

- CheckV assessment is not comprehensive and cannot estimate contamination of vMAGs because no universal marker gene set exists for viruses
- The benchmarking underestimates true viral community diversity because only viral genomes recoverable by a single contig were chosen, and putative viral genomes in the benchmarking tend to be small (size bias, Supplementary Note 6)
- Random binning does not reduce CheckV completeness, so a random binning control is necessary whenever CheckV completeness is the evaluation metric

## Evidence

- [results] cow fecal improvement percentages over baselines: [evidence span withheld — anchor status: not_found]
- [intro] CheckV role in completeness estimation: "CheckV has been widely used to estimate the completeness of vMAGs by comparing them to a large database curated from NCBI GenBank and environmental samples"
- [methods] ViralCC pipeline invocation: "python ./viralcc.py pipeline [Parameters] FASTA_file BAM_file VIRAL_file OUTPUT_directory"
- [discussion] random binning control requirement: "it is necessary to construct a random binning model as control experiments when the CheckV completeness is used as the evaluation metric"
- [readme] CheckV version used in reproduction: [evidence span withheld — anchor status: external_doc]
- [discussion] benchmarking scope limitation: "Since we only choose viral genomes that can be recovered by a single contig from the whole community, our benchmarking method inevitably underestimates the true diversity of the virus community."
