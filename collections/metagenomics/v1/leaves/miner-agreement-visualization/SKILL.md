---
name: miner-agreement-visualization
description: Use when metagenomic/virome contigs must be mined with multiple phage
  predictors and the goal is a vOTU set plus a per-miner overlap visualization to
  assess cross-miner concordance.
when_to_use_negative:
- Input is raw reads rather than assembled contigs — the mining step expects MEGAHIT-assembled
  consensus scaffolds, not FASTQ
- A single-miner viral prediction is sufficient and no cross-miner concordance assessment
  is needed — the multi-miner merge and upSet visualization add runtime (e.g., all-versus-all
  comparisons are time-consuming) without benefit
- Short contigs below the --minlen cutoff are the target — the dereplication step
  discards consensus scaffolds under the threshold (default 1000 bp, 3000 bp for Liang
  et al. reproduction)
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MetaPhage
  role: Nextflow pipeline orchestrating the four-miner phage detection, dereplication,
    and reporting
  repo: https://github.com/MattiaPandolfoVR/MetaPhage
- name: Nextflow
  role: Workflow manager enabling parallel execution of the miner tasks
  repo: https://github.com/MattiaPandolfoVR/MetaPhage
- name: Megahit
  role: Assembles quality-controlled virome reads into the contigs fed to the miners
  repo: https://github.com/voutcn/megahit
- name: VirFinder
  role: k-mer-based machine-learning miner for viral sequence prediction
  repo: https://github.com/jessieren/VirFinder
- name: VirSorter
  role: Homology-based miner detecting viral hallmark genes
  repo: https://github.com/simroux/VirSorter
- name: Phigaro
  role: Prophage-detection miner using pVOG HMMs and a smoothing-window algorithm
  repo: https://github.com/bobeobibo/phigaro
- name: VIBRANT
  role: Hybrid miner for virus identification by iterative annotation
- name: CD-HIT
  role: Dereplicates merged viral scaffolds into vOTUs at 0.95 identity, word size
    9, 0.85 coverage
  repo: https://github.com/weizhongli/cdhit
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1128/msystems.00741-22
    title: 'MetaPhage: an Automated Pipeline for Analyzing, Annotating, and Classifying
      Bacteriophages in Metagenomics Sequencing Data'
  source_content_hash: sha256:09063e969a70c02ea3cd57f554cf077aaa8c33327842a679976a93809d4b8d41
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
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# miner-agreement-visualization

## Summary

Reproduce the vOTU identification stage of the MetaPhage pipeline and quantify agreement across phage miners by merging VirFinder, VirSorter, Phigaro, and VIBRANT predictions, dereplicating with CD-HIT-EST, and rendering per-miner overlaps as an upSet plot.

## When to use

Use when metagenomic/virome contigs must be mined with multiple phage predictors and the goal is a vOTU set plus a per-miner overlap visualization to assess cross-miner concordance.

## When NOT to use

- Input is raw reads rather than assembled contigs — the mining step expects MEGAHIT-assembled consensus scaffolds, not FASTQ
- A single-miner viral prediction is sufficient and no cross-miner concordance assessment is needed — the multi-miner merge and upSet visualization add runtime (e.g., all-versus-all comparisons are time-consuming) without benefit
- Short contigs below the --minlen cutoff are the target — the dereplication step discards consensus scaffolds under the threshold (default 1000 bp, 3000 bp for Liang et al. reproduction)

## Inputs

- MEGAHIT-assembled FASTA contigs from quality-controlled virome shotgun reads (e.g., Liang et al. 2020 infant virome samples)
- MetaPhage miner outputs (VirFinder, VirSorter, Phigaro, VIBRANT viral scaffold predictions)
- Reference miner databases required by MetaPhage (Inphared, Kraken2/minikraken, Phigaro, PhiX, Vibrant, Virsorter)

## Outputs

- Dereplicated vOTU consensus multifasta
- Per-miner vOTU counts and pairwise/all-miner overlap matrix
- UpSet plot visualizing miner agreement
- Per-vOTU statistics (e.g., counts of vOTUs with more than 10 ORFs)

## How to apply

Take MEGAHIT-assembled contigs from quality-controlled reads (e.g., the Liang et al. 2020 infant virome) and run four miners in parallel — VirFinder (k-mer machine learning), VirSorter (viral hallmark gene homology), Phigaro (prophage detection via pVOG HMMs with a smoothing-window algorithm), and VIBRANT (hybrid approach). Merge the viral scaffolds from all miners and dereplicate into vOTUs with CD-HIT-EST using a 0.95 sequence identity threshold, word size 9, and 0.85 alignment coverage (the phage-species-level clustering standard), discarding consensus scaffolds shorter than the --minlen cutoff (use --minlen 3000 to match the original Liang et al. study rather than the MetaPhage default of 1000). Compute per-miner vOTU counts and pairwise/all-miner overlaps, render the comparison as an upSet plot, and export the dereplicated vOTU multifasta as the final artifact. Optionally count vOTUs exceeding an ORF-content threshold (e.g., more than 10 open reading frames) for richness characterization. On the Liang et al. DNA virome dataset with --minlen 3000, MetaPhage recovered 5,284 vOTUs, of which 2,507 contained more than 10 ORFs.

## Related tools

- **MetaPhage** (Nextflow pipeline orchestrating the four-miner phage detection, dereplication, and reporting) — https://github.com/MattiaPandolfoVR/MetaPhage
- **Nextflow** (Workflow manager enabling parallel execution of the miner tasks) — https://github.com/MattiaPandolfoVR/MetaPhage
- **Megahit** (Assembles quality-controlled virome reads into the contigs fed to the miners) — https://github.com/voutcn/megahit
- **VirFinder** (k-mer-based machine-learning miner for viral sequence prediction) — https://github.com/jessieren/VirFinder
- **VirSorter** (Homology-based miner detecting viral hallmark genes) — https://github.com/simroux/VirSorter
- **Phigaro** (Prophage-detection miner using pVOG HMMs and a smoothing-window algorithm) — https://github.com/bobeobibo/phigaro
- **VIBRANT** (Hybrid miner for virus identification by iterative annotation)
- **CD-HIT** (Dereplicates merged viral scaffolds into vOTUs at 0.95 identity, word size 9, 0.85 coverage) — https://github.com/weizhongli/cdhit

## Examples

```
git clone https://github.com/MattiaPandolfoVR/MetaPhage && nextflow run MetaPhage --input samplesheet.csv --minlen 3000
```

## Evaluation signals

- vOTU count and per-miner overlap distribution show high concordance with published Liang et al. results (5,284 vOTUs at --minlen 3000, of which 2,507 contain more than 10 ORFs)
- CD-HIT-EST parameters match the phage-species-level standard: identity threshold 0.95, word size 9, alignment coverage 0.85
- The upSet plot accounts for all miners (VirFinder, VirSorter, Phigaro, VIBRANT) and per-miner overlap sets sum consistently with the merged scaffold set
- Dereplicated vOTU multifasta contains no consensus scaffold shorter than the --minlen cutoff (3,000 bp for the Liang et al. reproduction)
- Richness pattern across samples (e.g., increased richness from month 0 to 4 in the infant time series) is reproducible

## Limitations

- vConTACT2-based classification does not directly classify vOTUs, requiring manual inspection absent graphanalyzer, and the network is rebuilt whenever new data are added, extending runtime
- All-versus-all protein comparison against large reference databases can require a long time
- Users should be mindful of the biases inherent to virome analyses, including recall/precision trade-offs in virus prediction
- MetaPhage requires a Linux environment; the default --minlen of 1000 bp differs from the 3,000 bp used in the original Liang et al. study and must be overridden explicitly

## Evidence

- [discussion] CD-HIT-EST parameters: "with a sequence identity threshold of 0.95, word size of 9, and alignment coverage of 0.85"
- [discussion] high concordance with Liang et al.: "we saw high concordance in the number of vOTUs predicted and a pattern of increased richness from month 0 to 4"
- [intro] vOTU clustering rationale: "clustering in viral operational taxonomic units (vOTUs) that are aligned with the phage species level has emerged as the gold standard to reduce data set complexity"
- [methods] minlen parameter: "Specify the minimal length in bp for a viral _consensus_ scaffold. Default is `1000`."
- [readme] MetaPhage pipeline purpose: "This is MetaPhage, a nextflow pipeline for automatic phage discovery."
