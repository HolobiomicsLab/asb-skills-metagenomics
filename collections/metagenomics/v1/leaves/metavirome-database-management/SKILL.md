---
name: metavirome-database-management
description: Use when reproducing phage discovery on virome FASTQ/contigs with MetaPhage,
  e.g. running multiple miners (VirFinder, VirSorter, Phigaro, VIBRANT) and vOTU clustering
  on infant gut or similar shotgun/VLP metagenomes.
when_to_use_negative:
- Input scaffolds are already dereplicated vOTUs or a curated viral genome set — no
  mining/dereplication step is needed
- The goal is prophage detection in a single isolate genome rather than mining virome
  metagenomes with multiple miners
- Scaffolds shorter than 3,000 bp must be retained, since reproducing the Liang et
  al. vOTU result requires discarding them via --minlen 3000
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: MetaPhage
  role: Nextflow reads-to-report pipeline orchestrating QC, assembly, phage mining,
    classification, and quantification
  repo: https://github.com/MattiaPandolfoVR/MetaPhage
- name: Nextflow
  role: Workflow manager enabling parallelization and reproducible execution of the
    pipeline
- name: Megahit
  role: Assembles quality-controlled virome reads into contigs fed to the miners
  repo: https://github.com/voutcn/megahit
- name: VirFinder
  role: k-mer-based machine-learning miner for viral contig prediction
  repo: https://github.com/jessieren/VirFinder
- name: VirSorter
  role: Homology-based miner detecting viral hallmark genes
  repo: https://github.com/simroux/VirSorter
- name: Phigaro
  role: Prophage miner using pVOG HMMs with a smoothing-window algorithm
  repo: https://github.com/bobeobibo/phigaro
- name: VIBRANT
  role: Hybrid miner combining annotation- and sequence-based viral prediction
  repo: https://github.com/AnantharamanLab/VIBRANT
- name: CD-HIT
  role: cd-hit-est dereplication of merged viral scaffolds into vOTUs at 0.95 identity,
    word size 9, coverage 0.85
  repo: https://github.com/weizhongli/cdhit
- name: fastp
  role: Sliding-window quality trimming and filtering of raw FASTQ reads
  repo: https://github.com/OpenGene/fastp
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/msystems.00741-22
    title: 'MetaPhage: an Automated Pipeline for Analyzing, Annotating, and Classifying
      Bacteriophages in Metagenomics Sequencing Data'
  source_content_hash: sha256:400bf3e487e9af7ca02119528e3065d97f7ccf699509aede0da24df9516cbe5d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MattiaPandolfoVR/MetaPhage
  tools:
  - CD-HIT
  - Megahit
  - MetaPhage
  - Nextflow
  - Phigaro
  - VIBRANT
  - VirFinder
  - VirSorter
  - fastp
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# metavirome-database-management

## Summary

Set up and manage the reference databases and mining tools that the MetaPhage Nextflow pipeline requires for automated phage discovery in virome metagenomes, then dereplicate mined scaffolds into vOTUs with CD-HIT-EST (0.95 identity, 0.85 coverage).

## When to use

Use when reproducing phage discovery on virome FASTQ/contigs with MetaPhage, e.g. running multiple miners (VirFinder, VirSorter, Phigaro, VIBRANT) and vOTU clustering on infant gut or similar shotgun/VLP metagenomes.

## When NOT to use

- Input scaffolds are already dereplicated vOTUs or a curated viral genome set — no mining/dereplication step is needed
- The goal is prophage detection in a single isolate genome rather than mining virome metagenomes with multiple miners
- Scaffolds shorter than 3,000 bp must be retained, since reproducing the Liang et al. vOTU result requires discarding them via --minlen 3000

## Inputs

- Quality-controlled shotgun/VLP FASTQ reads (e.g. Liang et al. 2020 infant metagenomes, 20 samples)
- MEGAHIT-assembled contigs from the QC'd reads
- MetaPhage reference databases: Inphared, Kraken2/minikraken, Phigaro, PhiX, VIBRANT, VirSorter
- Container environment (Singularity v3.7.1, Docker, or Conda) with Nextflow v21.04.0

## Outputs

- Dereplicated vOTU consensus multifasta (CD-HIT-EST, identity 0.95 / coverage 0.85, minlen 3000)
- Per-miner vOTU counts and pairwise/all-miner overlap table rendered as an upSet plot
- Exhaustive HTML report with summary visualizations and interactive filterable tables
- Read profiling, assembly, and quantification outputs from the reads-to-report pipeline

## How to apply

Install MetaPhage (git clone from GitHub) and provision its required set of databases: Inphared, Kraken2/minikraken, Phigaro, PhiX, VIBRANT, and VirSorter; tools run in Singularity/Docker/Conda containers for reproducibility. Take MEGAHIT-assembled contigs from quality-controlled reads (fastp trimming at cut_mean_quality 15, phiX174 removal with SeqScreener) and, for virome data, enable the --virome option to increase sensitivity by removing nonviral scaffolds. Run the four phage miners in parallel, merge predicted viral scaffolds, and dereplicate into vOTUs with CD-HIT-EST at a sequence identity threshold of 0.95, word size of 9, and alignment coverage of 0.85 — the species-level clustering gold standard — using --minlen 3000 when reproducing the Liang et al. infant virome analysis (default is 1000). Collect the dereplicated vOTU multifasta, per-miner counts, and overlap upSet plot from the exhaustive report. Validate against the reference result: 5,284 vOTUs with 2,507 containing more than 10 ORFs on the DNA virome dataset, matching the paper's reported high concordance with Liang et al.

## Related tools

- **MetaPhage** (Nextflow reads-to-report pipeline orchestrating QC, assembly, phage mining, classification, and quantification) — https://github.com/MattiaPandolfoVR/MetaPhage
- **Nextflow** (Workflow manager enabling parallelization and reproducible execution of the pipeline)
- **Megahit** (Assembles quality-controlled virome reads into contigs fed to the miners) — https://github.com/voutcn/megahit
- **VirFinder** (k-mer-based machine-learning miner for viral contig prediction) — https://github.com/jessieren/VirFinder
- **VirSorter** (Homology-based miner detecting viral hallmark genes) — https://github.com/simroux/VirSorter
- **Phigaro** (Prophage miner using pVOG HMMs with a smoothing-window algorithm) — https://github.com/bobeobibo/phigaro
- **VIBRANT** (Hybrid miner combining annotation- and sequence-based viral prediction) — https://github.com/AnantharamanLab/VIBRANT
- **CD-HIT** (cd-hit-est dereplication of merged viral scaffolds into vOTUs at 0.95 identity, word size 9, coverage 0.85) — https://github.com/weizhongli/cdhit
- **fastp** (Sliding-window quality trimming and filtering of raw FASTQ reads) — https://github.com/OpenGene/fastp

## Examples

```
git clone https://github.com/MattiaPandolfoVR/MetaPhage; nextflow run MattiaPandolfoVR/MetaPhage --input '*.fastq.gz' --virome --minlen 3000
```

## Evaluation signals

- vOTU count shows high concordance with the published Liang et al. results; on the DNA virome dataset with --minlen 3000 expect 5,284 vOTUs
- 2,507 of the 5,284 vOTUs contain more than 10 open reading frames when ORFs are called
- Per-miner overlap distribution matches the published pattern (paper's 'high concordance in the number of vOTUs predicted' and increased richness from month 0 to 4)
- Dereplication used the exact CD-HIT-EST parameters: sequence identity threshold 0.95, word size 9, alignment coverage 0.85
- All required databases (Inphared, Kraken2/minikraken, Phigaro, PhiX, VIBRANT, VirSorter) are present before launch; the pipeline completes without missing-database errors

## Limitations

- Recall/precision trade-off in virus prediction means biases inherent to virome analyses should be kept in mind when interpreting counts
- vConTACT2 network is rebuilt every time new data are added, extending runtime; all-versus-all protein comparison against large reference databases is time-consuming
- vConTACT2 does not directly classify a vOTU, leaving classification to manual inspection without graphanalyzer
- --minlen defaults to 1000 bp; analyses must set --minlen 3000 explicitly to reproduce species-level vOTU results like Liang et al.

## Evidence

- [methods] MetaPhage requires a set of databases: "MetaPhage requires a set of databases:
* Inphared
* Kraken2/minikraken
* Phigaro
* PhiX
* Vibrant
* Virsorter"
- [discussion] vOTU clustering parameters: "with a sequence identity threshold of 0.95, word size of 9, and alignment coverage of 0.85"
- [abstract] Reads-to-report pipeline: "a comprehensive reads-to-report pipeline that streamlines the use of multiple phage miners and generates an exhaustive report"
- [discussion] Concordance with Liang et al.: "we saw high concordance in the number of vOTUs predicted and a pattern of increased richness from month 0 to 4"
- [intro] vOTU species-level clustering rationale: "clustering in viral operational taxonomic units (vOTUs) that are aligned with the phage species level has emerged as the gold standard to reduce data set complexity"
- [readme] Repo install command: "This is MetaPhage, a nextflow pipeline for automatic phage discovery."
