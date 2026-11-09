---
name: votu-dereplication-clustering
description: Use when you have multi-FASTA viral scaffolds merged from several phage
  miners (e.g. VirFinder, VirSorter, Phigaro, VIBRANT) and need to reduce redundancy
  to species-level vOTUs before counting, overlap analysis, or classification.
when_to_use_negative:
- Input is raw reads or unfiltered assembled contigs that have not yet passed phage
  mining — clustering should act on miner-predicted viral scaffolds only
- Short viral scaffolds below the minimum length cutoff are of interest — the --minlen
  filter (e.g. 3000 bp) will discard them
- A strain-level (>95% ANI) clustering granularity is required — the 0.95/0.85 parameters
  correspond to phage species-level vOTUs
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: CD-HIT
  role: cd-hit-est dereplicates the merged viral scaffolds into vOTUs at 0.95 identity,
    word size 9, 0.85 alignment coverage
  repo: github.com/weizhongli/cdhit
- name: MetaPhage
  role: Wraps the dereplication step (--skip_dereplication, --minlen) downstream of
    the phage miners and produces the vOTU multifasta and miner-comparison report
  repo: https://github.com/MattiaPandolfoVR/MetaPhage
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1128/msystems.00741-22
    title: 'MetaPhage: an Automated Pipeline for Analyzing, Annotating, and Classifying
      Bacteriophages in Metagenomics Sequencing Data'
  source_content_hash: sha256:72c86c31d9ff84d63c8ccc40077c70375a0d87b54763ea407182c31171e7d67f
schema_version: 0.2.0
metadata:
  license_tier: open
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8125
n_claims: 16
---

# votu-dereplication-clustering

## Summary

Dereplicates viral scaffolds predicted by multiple phage miners into viral operational taxonomic units (vOTUs) using CD-HIT-EST at the phage species level (0.95 sequence identity, word size 9, 0.85 alignment coverage), with a minimum consensus scaffold length filter (e.g. --minlen 3000).

## When to use

Use when you have multi-FASTA viral scaffolds merged from several phage miners (e.g. VirFinder, VirSorter, Phigaro, VIBRANT) and need to reduce redundancy to species-level vOTUs before counting, overlap analysis, or classification.

## When NOT to use

- Input is raw reads or unfiltered assembled contigs that have not yet passed phage mining — clustering should act on miner-predicted viral scaffolds only
- Short viral scaffolds below the minimum length cutoff are of interest — the --minlen filter (e.g. 3000 bp) will discard them
- A strain-level (>95% ANI) clustering granularity is required — the 0.95/0.85 parameters correspond to phage species-level vOTUs

## Inputs

- Merged viral consensus scaffolds (multi-FASTA) from multiple phage miners
- Minimum consensus scaffold length parameter (--minlen, e.g. 3000 bp)
- CD-HIT-EST settings: identity threshold 0.95, word size 9, alignment coverage 0.85

## Outputs

- Dereplicated vOTU consensus multi-FASTA (final artifact)
- vOTU count table (total and per-miner)
- Per-miner pairwise/all-miner overlap summary rendered as an upSet plot

## How to apply

Merge the viral scaffolds predicted by all miners (VirFinder, VirSorter, Phigaro, VIBRANT, and optionally DeepVirFinder/VirSorter2 in MetaPhage v2) that were mined from MEGAHIT-assembled contigs. Run CD-HIT-EST dereplication with a sequence identity threshold of 0.95, word size of 9, and alignment coverage of 0.85, matching the phage species-level definition of ~95% average nucleotide identity over 85% of the aligned fraction. Apply a minimum consensus scaffold length via --minlen (e.g. 3000 bp to mirror the Liang et al. DNA virome analysis; the MetaPhage default is 1000 bp). From the dereplicated vOTU multifasta, compute per-miner vOTU counts and pairwise/all-miner overlaps, rendering the comparison as an upSet plot. Judge correctness by whether the vOTU count and per-miner overlap distribution show high concordance with published results (e.g. 5,284 vOTUs on the Liang et al. DNA virome, 14,274 on the shotgun metagenome set).

## Related tools

- **CD-HIT** (cd-hit-est dereplicates the merged viral scaffolds into vOTUs at 0.95 identity, word size 9, 0.85 alignment coverage) — github.com/weizhongli/cdhit
- **MetaPhage** (Wraps the dereplication step (--skip_dereplication, --minlen) downstream of the phage miners and produces the vOTU multifasta and miner-comparison report) — https://github.com/MattiaPandolfoVR/MetaPhage

## Examples

```
cd-hit-est -i viral_scaffolds.fasta -o vOTUs.fasta -c 0.95 -n 9 -aS 0.85 -d 0
```

## Evaluation signals

- Total vOTU count matches the expected published figure (e.g. 5,284 vOTUs on the Liang et al. DNA virome with minlen 3000; 14,274 on the shotgun metagenome set)
- No vOTU in the output multifasta is shorter than the --minlen cutoff (e.g. 3,000 bp)
- Per-miner overlap distribution in the upSet plot is concordant with the published 'high concordance in the number of vOTUs predicted' result
- Clustered sequences share ≥0.95 identity over ≥0.85 of the aligned fraction, consistent with the phage species-level definition
- The dereplicated vOTU multifasta is emitted as the pipeline's final artifact and is internally consistent with the vOTU count table

## Limitations

- The default minimum consensus scaffold length is 1,000 bp; species-level vOTU studies must override it (e.g. --minlen 3000) or results will not be comparable to the reference study
- The number and overlap of vOTUs depend strongly on which miners are used; CheckV showed that large differences in vOTU counts arise from predictions of more partial and low-quality genomes
- Per-miner overlap results are only meaningful if all miners were run on the same assembled contig input and merged before dereplication

## Evidence

- [intro] Species-level vOTU definition: "clustering in viral operational taxonomic units (vOTUs) that are aligned with the phage species level has emerged as the gold standard to reduce data set complexity"
- [discussion] CD-HIT-EST parameters: "with a sequence identity threshold of 0.95, word size of 9, and alignment coverage of 0.85"
- [methods] minlen default: "Specify the minimal length in bp for a viral _consensus_ scaffold. Default is `1000`."
- [discussion] Validation concordance: "we saw high concordance in the number of vOTUs predicted and a pattern of increased richness from month 0 to 4"
- [discussion] CheckV caveat: "Quality assessment with CheckV (29) showed that the large difference in number of vOTUs predicted was due to predictions of more partial and low-quality genomes"
