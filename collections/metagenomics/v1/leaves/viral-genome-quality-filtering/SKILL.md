---
name: viral-genome-quality-filtering
description: Use when metagenome-assembled viral contigs/genomes need completeness
  and quality assessment (CheckV) so that only genomes passing a completeness threshold
  feed downstream vOTU clustering or database comparison.
when_to_use_negative:
- Input is a set of already-clustered vOTU representatives — quality filtering should
  precede clustering, not be re-run on representatives
- Assessing prokaryotic MAG quality — use CheckM2/CheckM/BUSCO (MIMAG-revised criteria),
  not the viral CheckV completeness workflow
- Estimating population abundance or taxonomic composition — this skill produces a
  genome set, not per-sample abundance profiles
edam_topics:
- http://edamontology.org/topic_0781
tools:
- name: CheckV
  role: Estimates completeness of viral sequences and assigns complete/high/medium
    quality tiers used for the ≥50% completeness filter
- name: virusFind_flow.sh
  role: Upstream combined feature- and homology-based identification of viral genomes
    on metagenome-assembled contigs before quality filtering
  repo: https://github.com/RChGO/VMGC/blob/main/Pipelines/virusFind_flow.sh
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41564-024-01751-5
    title: A multi-kingdom collection of 33,804 reference genomes for the human vaginal
      microbiome
  source_content_hash: sha256:ae45952df8daf999321ef8fa865340170da2216325f7a14b3f1dc938874a5a86
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLAST
  - CheckV
  - NCBI RefSeq
  - Prodigal
  - virusFind_flow.sh
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0781
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# viral-genome-quality-filtering

## Summary

Quality-assessment and completeness filtering of metagenome-assembled viral genomes using CheckV, as applied to the VMGC to retain 14,224 viral sequences with at least 50% completeness and classify them into complete/high/medium-quality tiers before vOTU clustering.

## When to use

Use when metagenome-assembled viral contigs/genomes need completeness and quality assessment (CheckV) so that only genomes passing a completeness threshold feed downstream vOTU clustering or database comparison.

## When NOT to use

- Input is a set of already-clustered vOTU representatives — quality filtering should precede clustering, not be re-run on representatives
- Assessing prokaryotic MAG quality — use CheckM2/CheckM/BUSCO (MIMAG-revised criteria), not the viral CheckV completeness workflow
- Estimating population abundance or taxonomic composition — this skill produces a genome set, not per-sample abundance profiles

## Inputs

- Metagenome-assembled viral contigs/genomes in FASTA format (candidates from combined feature- and homology-based virus identification)
- Viral genome candidates from vaginal metagenomic samples (4,472 samples across 32 studies in the VMGC pipeline)

## Outputs

- Quality-filtered viral genome set (14,224 genomes at ≥50% completeness in VMGC)
- Per-genome CheckV quality annotations (complete / high quality ≥90% / medium quality tiers, completeness estimates)
- Filtered genomes (VMGC_virus.tar.gz, n=14,224) plus VMGC_virus.info annotations, ready for vOTU clustering at 95% identity over 85% genome coverage

## How to apply

Viral candidates are first identified on metagenome-assembled contigs with a combined feature- and homology-based approach (virusFind_flow.sh from the VMGC pipelines). Each candidate viral genome is then assessed with CheckV, which estimates completeness and assigns quality tiers: complete, high quality (completeness ≥90%) and medium quality. Apply the study's filter step — keep only viral genomes with ≥50% completeness — which, for the VMGC, reduced candidates to 14,224 viral sequences (10.3% complete, 29.3% high quality, 60.4% medium quality; median genome size 33.5 kbp). Rationale: low-completeness viral fragments inflate OTU counts and produce spurious 'novel' vOTUs, so filtering before the 95% identity / 85% genome-coverage clustering (virusClstr.sh) keeps the 4,263 vOTU set interpretable.

## Related tools

- **CheckV** (Estimates completeness of viral sequences and assigns complete/high/medium quality tiers used for the ≥50% completeness filter)
- **virusFind_flow.sh** (Upstream combined feature- and homology-based identification of viral genomes on metagenome-assembled contigs before quality filtering) — https://github.com/RChGO/VMGC/blob/main/Pipelines/virusFind_flow.sh

## Evaluation signals

- Number of retained viral genomes matches the expected filter yield (14,224 for VMGC inputs)
- Quality-tier proportions are reproducible: ~10.3% complete, ~29.3% high quality (completeness ≥90%), ~60.4% medium quality
- Median genome size of retained viral genomes is around 33.5 kbp
- Downstream all-vs-all clustering (95% identity over 85% of genome) of the filtered set yields approximately 4,263 vOTUs
- Per-genome annotation file (VMGC_virus.info-style) carries taxonomy/quality/clustering/host fields for every retained genome

## Limitations

- CheckV-based completeness can be underestimated for some organisms (the paper notes CheckM2 completeness underestimation for Atopobium species, illustrating general tool-dependence of quality estimates)
- Quality filtering does not address contamination from experimental operations or kits present in the public vaginal metagenome datasets
- Genomes passing the ≥50% completeness threshold may still be incomplete; most retained sequences (60.4%) are only medium quality

## Evidence

- [abstract] Based on the CheckV algorithm18, 10.3% of viral sequences were estimated as complete viral genomes: "Based on the CheckV algorithm18, 10.3% of viral sequences were estimated as complete viral genomes"
- [abstract] Viral genomes filtered to at least 50% completeness: "resulting in 14,224 viral sequences achieving at least 50% completeness"
- [abstract] Combined feature- and homology-based virus identification: "we performed virus identification on the metagenome-assembled contigs using a combined feature- and homology-based approach"
- [readme] VMGC_virus.tar.gz deposit contents: "Full viral genomes (n=14,224) and vOTUs (n=4,263)| 197 MB | VMGC_virus.tar.gz"
