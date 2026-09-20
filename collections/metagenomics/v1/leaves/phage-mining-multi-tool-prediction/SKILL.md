---
name: phage-mining-multi-tool-prediction
description: Use when FASTQ metagenomic or virome reads need viral contig identification
  and vOTU counting, and a single-miner prediction would miss phages detectable only
  by complementary k-mer, homology, or HMM-based approaches.
when_to_use_negative:
- Input is raw reads without prior assembly — this skill mines assembled contigs,
  not reads
- Non-virome shotgun microbiome data where nonviral scaffold removal with --virome
  would discard host-associated viral signal you want to keep
- Only a single miner's output is available, so no multi-miner consensus or overlap
  analysis can be computed
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0085
tools:
- name: MetaPhage
  role: Nextflow pipeline orchestrating miner execution, consensus scaffold dereplication,
    and report generation
  repo: https://github.com/MattiaPandolfoVR/MetaPhage
- name: VirFinder
  role: k-mer-based machine-learning miner for viral sequence prediction
  repo: https://github.com/jessieren/VirFinder
- name: VirSorter
  role: homology-based miner detecting viral hallmark genes
  repo: https://github.com/simroux/VirSorter
- name: Phigaro
  role: prophage detection miner using pVOG HMMs with a smoothing-window algorithm
  repo: https://github.com/bobeobibo/phigaro
- name: VIBRANT
  role: hybrid-approach miner for virus identification
  repo: https://github.com/AnantharamanLab/VIBRANT
- name: CD-HIT
  role: cd-hit-est dereplication of merged viral scaffolds into vOTUs at 0.95 identity
    / 0.85 coverage
  repo: https://github.com/weizhongli/cdhit
- name: Megahit
  role: per-sample assembly of quality-controlled reads supplying contigs to the miners
  repo: https://github.com/voutcn/megahit
- name: Nextflow
  role: workflow manager enabling parallelized, reproducible execution of the mining
    steps
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1128/msystems.00741-22
    title: 'MetaPhage: an Automated Pipeline for Analyzing, Annotating, and Classifying
      Bacteriophages in Metagenomics Sequencing Data'
  source_content_hash: sha256:22a02084e0c5e94f305ed824dd585a39e1a4b37269f2bd53dad2af7605f226d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MattiaPandolfoVR/MetaPhage
  tools:
  - CD-HIT
  - DeepVirFinder
  - Megahit
  - MetaPhage
  - Nextflow
  - Phigaro
  - VIBRANT
  - VirFinder
  - VirSorter
  - VirSorter2
  - bamtocov
  - bowtie2
  - fastp
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9411764705882353
n_claims: 17
---

# phage-mining-multi-tool-prediction

## Summary

Run multiple phage miners (VirFinder/DeepVirFinder, VirSorter/VirSorter2, Phigaro, VIBRANT) in parallel on assembled metagenomic contigs, then dereplicate the merged viral scaffolds into vOTUs with CD-HIT-EST at 0.95 identity / 0.85 coverage. This captures the complementary detection strategies (k-mer ML, hallmark genes, pVOG HMM prophage detection, hybrid) used by MetaPhage to maximize viral signal recovery.

## When to use

Use when FASTQ metagenomic or virome reads need viral contig identification and vOTU counting, and a single-miner prediction would miss phages detectable only by complementary k-mer, homology, or HMM-based approaches.

## When NOT to use

- Input is raw reads without prior assembly — this skill mines assembled contigs, not reads
- Non-virome shotgun microbiome data where nonviral scaffold removal with --virome would discard host-associated viral signal you want to keep
- Only a single miner's output is available, so no multi-miner consensus or overlap analysis can be computed

## Inputs

- MEGAHIT-assembled contigs from quality-controlled shotgun or VLP virome reads (FASTA)
- MetaPhage project configuration generated with newProject.py (-i reads directory, -m CSV metadata, -v metadata variable, -s project.conf)
- Phage miner databases (Inphared, Kraken2/minikraken, Phigaro, PhiX, VIBRANT, VirSorter)

## Outputs

- Dereplicated vOTU consensus multifasta (final artifact)
- Per-miner vOTU counts and pairwise/all-miner overlap table
- Miner-comparison upSet plot
- vOTU ORF content counts (e.g., vOTUs with >10 open reading frames)

## How to apply

First assemble quality-controlled reads per sample with MEGAHIT, using the contigs as miner input. Run the miners in parallel on the contigs: VirFinder or DeepVirFinder (k-mer-based machine learning), VirSorter or VirSorter2 (viral hallmark gene homology), Phigaro (prophage detection via pVOG HMMs with a smoothing-window algorithm), and VIBRANT (hybrid approach). Merge all predicted viral scaffolds and dereplicate them into vOTUs with CD-HIT-EST using a sequence identity threshold of 0.95, word size of 9, and alignment coverage of 0.85, discarding consensus scaffolds shorter than the --minlen threshold (3,000 bp in the Liang et al. reproductions, matching the phage species-level clustering gold standard of 95% ANI over 85% of the aligned fraction). For virome datasets, run with the --virome option to increase sensitivity by removing nonviral scaffolds. Compute per-miner vOTU counts and pairwise/all-miner overlaps and render the comparison as an upSet plot. Note that the MetaPhage v2 beta configuration swaps in DeepVirFinder and VirSorter2 for VirFinder and VirSorter, so state which miner set was used.

## Related tools

- **MetaPhage** (Nextflow pipeline orchestrating miner execution, consensus scaffold dereplication, and report generation) — https://github.com/MattiaPandolfoVR/MetaPhage
- **VirFinder** (k-mer-based machine-learning miner for viral sequence prediction) — https://github.com/jessieren/VirFinder
- **VirSorter** (homology-based miner detecting viral hallmark genes) — https://github.com/simroux/VirSorter
- **Phigaro** (prophage detection miner using pVOG HMMs with a smoothing-window algorithm) — https://github.com/bobeobibo/phigaro
- **VIBRANT** (hybrid-approach miner for virus identification) — https://github.com/AnantharamanLab/VIBRANT
- **CD-HIT** (cd-hit-est dereplication of merged viral scaffolds into vOTUs at 0.95 identity / 0.85 coverage) — https://github.com/weizhongli/cdhit
- **Megahit** (per-sample assembly of quality-controlled reads supplying contigs to the miners) — https://github.com/voutcn/megahit
- **Nextflow** (workflow manager enabling parallelized, reproducible execution of the mining steps)

## Examples

```
cd-hit-est -i merged_viral_scaffolds.fa -o vOTUs.fa -c 0.95 -n 9 -aS 0.85
```

## Evaluation signals

- Per-miner vOTU counts and overlap distribution show high concordance with the published Liang et al. results ('high concordance in the number of vOTUs predicted')
- On the Liang et al. DNA virome dataset with --minlen 3000, expect 5,284 vOTUs, of which 2,507 contain more than 10 ORFs
- On the Liang et al. 59-sample shotgun metagenome with the five-miner v2 configuration, expect 14,274 vOTUs and DeepVirFinder highest with 10,508 predictions
- Miner-overlap composition check: 24.8% of vOTUs predicted only by DeepVirFinder and VIBRANT, and 2.3% by all five miners
- Every dereplicated vOTU in the consensus multifasta is at least the --minlen length (e.g., 3,000 bp)

## Limitations

- Users are urged to be mindful of the biases inherent to virome analyses, including the recall/precision trade-off in virus prediction
- vConTACT2 does not directly classify a vOTU and would leave classification up to the user's extensive manual inspection absent graphanalyzer
- The default --minlen is 1,000 bp, so reproducing the Liang et al. vOTU counts requires explicitly setting --minlen 3000

## Evidence

- [intro] clustering in viral operational taxonomic units (vOTUs) that are aligned with the phage species level has emerged as the gold standard: "clustering in viral operational taxonomic units (vOTUs) that are aligned with the phage species level has emerged as the gold standard to reduce data set complexity"
- [discussion] with a sequence identity threshold of 0.95, word size of 9, and alignment coverage of 0.85: "with a sequence identity threshold of 0.95, word size of 9, and alignment coverage of 0.85"
- [discussion] The option –virome increases sensitivity for virome data sets by removing nonviral scaffolds: "The option –virome increases sensitivity for virome data sets by removing nonviral scaffolds"
- [methods] Specify the minimal length in bp for a viral consensus scaffold. Default is 1000.: "Specify the minimal length in bp for a viral _consensus_ scaffold. Default is `1000`."
- [readme] The prediction method is based on sequence signatures: "The method has good prediction accuracy for short (~1kb) and noval viral sequences.

The prediction method is based on"
