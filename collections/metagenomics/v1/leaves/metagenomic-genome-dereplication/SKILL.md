---
name: metagenomic-genome-dereplication
description: Use when a genome set assembled/binned from metagenomes will be used
  as a concatenated mapping reference or for cross-tool strain-sharing benchmarks,
  and redundant near-identical genomes (e.g. >98% ANI) must be collapsed first.
when_to_use_negative:
- The analysis goal is strain/sub-species resolution within a single species population
  (strain clustering at 99.999% ANI), not collapsing redundant genomes — dereplication
  at 98% ANI would merge diversity you need to keep separate
- The genome set is already non-redundant or contains genomes from distinct taxa well
  below 98% ANI to each other, so no dereplication is needed
- Input is raw reads or a single reference genome rather than a collection of multiple
  genome .fasta files
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: dRep
  role: Performs genome de-replication, identifying groups of highly similar genomes
    and choosing the best representative genome at a 98% ANI threshold
  repo: https://github.com/MrOlm/drep
- name: inStrain
  role: Consumes the dereplicated genome set as the mapping reference for 'inStrain
    profile' and 'inStrain compare' (popANI, coverage overlap); recommends the 98%-or-lower
    dereplication threshold
  repo: https://github.com/MrOlm/instrain
- name: CoverM
  role: Alternative fast tool for calculating coverage and breadth of genomes from
    mappings, useful for sanity-checking detection on the dereplicated reference
  repo: https://github.com/wwood/CoverM
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41587-020-00797-0
    title: inStrain profiles population microdiversity from metagenomic data and sensitively
      detects shared microbial strains
  source_content_hash: sha256:c8b25e02eb5585f2a96bd8c9d39d9685c13112983ff34c710e799622bbe349b5
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/MrOlm/drep
  tools:
  - Bowtie2
  - CoverM
  - Matplotlib
  - Pandas
  - dRep
  - inStrain
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# metagenomic-genome-dereplication

## Summary

De-replicate a set of metagenomic genome bins into a non-redundant representative set using dRep at an ANI threshold of 98% or lower before read mapping and population-comparison analyses (e.g. inStrain profile/compare), so that highly similar genomes do not split read mappings or inflate strain-sharing counts.

## When to use

Use when a genome set assembled/binned from metagenomes will be used as a concatenated mapping reference or for cross-tool strain-sharing benchmarks, and redundant near-identical genomes (e.g. >98% ANI) must be collapsed first.

## When NOT to use

- The analysis goal is strain/sub-species resolution within a single species population (strain clustering at 99.999% ANI), not collapsing redundant genomes — dereplication at 98% ANI would merge diversity you need to keep separate
- The genome set is already non-redundant or contains genomes from distinct taxa well below 98% ANI to each other, so no dereplication is needed
- Input is raw reads or a single reference genome rather than a collection of multiple genome .fasta files

## Inputs

- Genome bin nucleotide .fasta files (e.g. infant metagenome bins from multiple samples)
- Optional genome quality information (completeness/contamination) for dRep scoring
- Downstream per-sample read sets (FASTQ) and Bowtie2 mappings (.bam) that will use the dereplicated reference

## Outputs

- Non-redundant representative genome set (dereplicated .fasta files or a list of representative genome IDs)
- dRep clustering/data tables reporting cluster membership and pairwise ANI
- Concatenated representative reference .fasta used for 'inStrain profile' mappings
- Per-tool shared-genome counts (e.g. supplemental_table_s1.csv) contributing to the Figure 2d benchmark

## How to apply

Run 'dRep dereplicate' on the genome bin .fasta files to cluster genomes and choose the best representative for each cluster, using a secondary ANI threshold of 98% or lower — the inStrain paper explicitly recommends this because genome de-replication at 98% ANI reduces multi-mapped reads in downstream Bowtie2 mappings. Feed the resulting representative-genome set (concatenated into a single .fasta) to 'inStrain profile' on each sample's mapping, then 'inStrain compare' to compute popANI and coverage overlap between sample pairs. Keep dereplication separate from strain-level clustering: strains are defined afterwards by 'average' hierarchical clustering of subspecies at a 99.999% ANI threshold, so do not collapse at that resolution during dereplication. In the twin-set benchmark, dRep's 98%-ANI sharing counts are tabulated alongside inStrain, MIDAS, and StrainPhlAn counts for Figure 2d and Supplemental Table S1.

## Related tools

- **dRep** (Performs genome de-replication, identifying groups of highly similar genomes and choosing the best representative genome at a 98% ANI threshold) — https://github.com/MrOlm/drep
- **inStrain** (Consumes the dereplicated genome set as the mapping reference for 'inStrain profile' and 'inStrain compare' (popANI, coverage overlap); recommends the 98%-or-lower dereplication threshold) — https://github.com/MrOlm/instrain
- **CoverM** (Alternative fast tool for calculating coverage and breadth of genomes from mappings, useful for sanity-checking detection on the dereplicated reference) — https://github.com/wwood/CoverM

## Examples

```
dRep dereplicate output_directory -g path/to/genomes/*.fasta
```

## Evaluation signals

- dRep dereplicate completes and reports genome clusters with secondary comparison ANI at the 98% (or lower) threshold, with one representative chosen per cluster
- Downstream Bowtie2 mappings against the dereplicated concatenated .fasta show reduced multi-mapped reads (per Supplemental Figure S4)
- inStrain compare runs pairwise over all sample-pair profiles and reports popANI and coverage overlap for every genome pair without ambiguous duplicate references
- Shared-genome counts at the 98% ANI criterion are tabulated for dRep alongside inStrain, MIDAS, and StrainPhlAn in supplemental_table_s1.csv and agree with the paper's Supplemental Table S1 / Figure 2d
- Detection of a sub-species in a sample is defined as >= 0.5 unmaskedBreadth, so dereplicated representatives must yield breadth values compatible with this criterion

## Limitations

- Dereplication at 98% ANI cannot distinguish strains below that divergence; strain-sharing claims at higher resolution (99.999% ANI) must come from inStrain popANI comparisons, not dRep
- Variant sites near the ends of genome fragments can be missed with inStrain, which can affect breadth-based detection of dereplicated representatives
- dRep dereplicate normally depends on CheckM for genome quality unless --genomeInfo or --ignoreGenomeQuality is supplied; skipping quality assessment may pick poor representatives

## Evidence

- [readme] dRep can also "de-replicate" a genome set by identifying groups of highly similar genomes and choosing the best representative genome for each genome set.: [evidence span withheld — anchor status: external_doc]
