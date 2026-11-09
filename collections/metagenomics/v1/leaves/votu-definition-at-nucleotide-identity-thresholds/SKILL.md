---
name: votu-definition-at-nucleotide-identity-thresholds
description: Use when you have a set of quality-filtered viral contigs/genomes (e.g.
  CheckV-screened FASTA) and need to dereplicate them into species-level vOTUs and
  quantify overlap with reference virome catalogues.
when_to_use_negative:
- Input is raw metagenomic contigs that have not been screened for viral origin and
  completeness (run virus identification and CheckV-quality filtering first)
- The goal is prokaryotic species dereplication — the paper clusters those separately
  at 95% nucleotide similarity into SGBs, not vOTUs
- Only amino-acid-level gene clustering is needed (VMGC-50/90/95 gene catalogues)
  rather than whole-genome nucleotide clustering
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_0091
tools:
- name: BLAST
  role: All-vs-all nucleotide comparison of viral genomes for clustering, and BLASTn
    of vOTU representatives against the five external virome databases to assess novelty
- name: CheckV
  role: Estimates completeness of viral sequences to select genomes (>=50% completeness)
    entering vOTU clustering; 10.3% of viral sequences were estimated as complete
- name: Prodigal
  role: Extracts protein-coding genes from vOTU representatives in metagenomic mode
    for downstream annotation of the clustered viral set
- name: virusClstr.sh
  role: Clustering script (from the VMGC Pipelines) that groups viral genomes into
    vOTUs at 95% identity over 85% of the genome
  repo: https://github.com/RChGO/VMGC/blob/main/Pipelines/virusClstr.sh
- name: GNU parallel 20201122
  role: Parallelizes the pairwise comparisons and per-vOTU processing across the 14,224
    viral genomes
- name: NCBI RefSeq
  role: One of the five reference virome databases (viral genomes) used to compute
    the non-overlap fraction of VMGC vOTUs
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41564-024-01751-5
    title: A multi-kingdom collection of 33,804 reference genomes for the human vaginal
      microbiome
  source_content_hash: sha256:d6daa3e71a4ced2341a1f342c37000bad11ce784bf1b17f71dc7a64ecc688769
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLAST
  - CheckV
  - GNU parallel 20201122
  - NCBI RefSeq
  - Prodigal
  - virusClstr.sh
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# vOTU definition at nucleotide identity thresholds

## Summary

Defines species-level viral operational taxonomic units (vOTUs) by all-vs-all nucleotide comparison of quality-filtered viral genomes, clustering at 95% identity over 85% of the genome — in the VMGC study this reduced 14,224 viral genomes (>=50% completeness by CheckV) to 4,263 vOTUs whose novelty against external virome databases was then quantified.

## When to use

Use when you have a set of quality-filtered viral contigs/genomes (e.g. CheckV-screened FASTA) and need to dereplicate them into species-level vOTUs and quantify overlap with reference virome catalogues.

## When NOT to use

- Input is raw metagenomic contigs that have not been screened for viral origin and completeness (run virus identification and CheckV-quality filtering first)
- The goal is prokaryotic species dereplication — the paper clusters those separately at 95% nucleotide similarity into SGBs, not vOTUs
- Only amino-acid-level gene clustering is needed (VMGC-50/90/95 gene catalogues) rather than whole-genome nucleotide clustering

## Inputs

- Quality-filtered viral genome FASTA files (e.g. >=50% completeness by CheckV; VMGC_virus.tar.gz, n=14,224)
- Reference virome database sequences (Gut Virome Database, Gut Phage Database, Metagenomic Gut Virus catalogue, Oral Virus Database, NCBI RefSeq viral genomes)

## Outputs

- vOTU clusters with representative sequences (VMGC: 4,263 vOTUs)
- Protein-coding gene calls from vOTU representatives (Prodigal metagenomic mode)
- Per-vOTU database comparison results and non-overlap fraction (votu_overlap_results.csv; expected 85.8%)

## How to apply

Retrieve quality-filtered viral genomes (VMGC used 14,224 sequences with >=50% completeness estimated by CheckV, from VMGC_virus.tar.gz on Zenodo record 10457006). Perform an all-vs-all nucleotide comparison with BLASTn and cluster using the virusClstr.sh script from the RChGO/VMGC Pipelines, applying a 95% nucleotide identity threshold over 85% of the genome length to define vOTUs — this yielded 4,263 vOTUs with representative sequences. Downstream, extract protein-coding genes from the vOTU representatives with Prodigal in metagenomic mode. To quantify novelty, BLASTn the vOTU representatives against Gut Virome Database, Gut Phage Database, Metagenomic Gut Virus catalogue, Oral Virus Database and NCBI RefSeq viral genomes, count vOTUs with no significant match, and compute the non-overlap fraction (VMGC reported 85.8%).

## Related tools

- **BLAST** (All-vs-all nucleotide comparison of viral genomes for clustering, and BLASTn of vOTU representatives against the five external virome databases to assess novelty)
- **CheckV** (Estimates completeness of viral sequences to select genomes (>=50% completeness) entering vOTU clustering; 10.3% of viral sequences were estimated as complete)
- **Prodigal** (Extracts protein-coding genes from vOTU representatives in metagenomic mode for downstream annotation of the clustered viral set)
- **virusClstr.sh** (Clustering script (from the VMGC Pipelines) that groups viral genomes into vOTUs at 95% identity over 85% of the genome) — https://github.com/RChGO/VMGC/blob/main/Pipelines/virusClstr.sh
- **GNU parallel 20201122** (Parallelizes the pairwise comparisons and per-vOTU processing across the 14,224 viral genomes)
- **NCBI RefSeq** (One of the five reference virome databases (viral genomes) used to compute the non-overlap fraction of VMGC vOTUs)

## Examples

```
bash Pipelines/virusClstr.sh VMGC_virus.tar.gz  # cluster 14,224 viral genomes into vOTUs at 95% identity over 85% of the genome
```

## Evaluation signals

- Clustering at 95% identity / 85% genome coverage of the 14,224 quality-filtered viral genomes yields 4,263 vOTUs
- BLASTn of vOTU representatives against the five virome databases gives a non-overlap fraction of 85.8% within a small tolerance
- All representative vOTU sequences are drawn from the input set and no genome is assigned to more than one vOTU
- Per-vOTU comparison results in votu_overlap_results.csv are complete (4,263 rows) and each row records a match/no-match call against all five databases
- Prodigal in metagenomic mode produces gene calls for every vOTU representative with no empty output files

## Limitations

- Many phages in the VMGC are unknown and their functional exploration based on existing databases remains limited
- The 85.8% novelty figure depends on the coverage and content of the five reference databases; absence of a match is not proof of true novelty
- Microbial sequences captured by the VMGC are incomplete, especially for vaginal samples from non-western populations, which may bias vOTU diversity

## Evidence

- [other] Perform all-vs-all nucleotide comparison of the viral genomes with BLASTn and cluster them with virusClstr.sh to define vOTUs at 95% identity over 85% of the genome, yielding 4,263 vOTUs: [evidence span withheld — anchor status: not_found]
- [abstract] 85.8% of vOTUs absent from other databases: "The result revealed that 85.8% of vOTUs in the VMGC were not found in any of the other virome databases"
- [abstract] 14,224 viral sequences at >=50% completeness: "resulting in 14,224 viral sequences achieving at least 50% completeness"
- [readme] VMGC virus data deposit on Zenodo: "Full viral genomes (n=14,224) and vOTUs (n=4,263)| 197 MB | VMGC_virus.tar.gz"
- [methods] virusClstr.sh script available in the VMGC Pipelines repository: "[virusClstr.sh](https://github.com/RChGO/VMGC/blob/main/Pipelines/virusClstr.sh)"
