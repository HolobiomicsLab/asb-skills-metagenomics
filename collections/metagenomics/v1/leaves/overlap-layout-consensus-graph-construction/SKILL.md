---
name: overlap-layout-consensus-graph-construction
description: Use when assembling individual strain genomes (haplotigs) from short-read
  metagenomic data that has already been split into per-cluster interleaved FASTQ
  read sets, and strain-resolved contigs are needed.
when_to_use_negative:
- Input reads have not been pre-clustered by strain — overlap-graph assembly on the
  full metagenome is computationally intractable; run the clustering stage first
- The goal is maximum contig length (N50) rather than strain resolution — MEGAHIT-type
  assemblers outperform StrainXpress on contig length
- Data are long third-generation reads with high error rates — the paper notes error
  rates of third-generation reads introduce issues that still need to be resolved
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: StrainXpress
  role: Implements the ploidy-aware OLC local assembly of cluster read sets into strain
    haplotigs
  repo: https://github.com/HaploKit/StrainXpress
- name: minimap2
  role: Computes read-to-read approximate overlaps within each cluster, replacing
    FM-index seeding
  repo: https://github.com/lh3/minimap2
- name: HaploConduct
  role: Provides the adopted ploidy-aware overlap-graph assembly strategy (SAVAGE/POLYTE
    lineage)
  repo: https://github.com/HaploConduct/HaploConduct
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkac543
    title: 'StrainXpress: strain aware metagenome assembly from short reads'
  source_content_hash: sha256:0fee6fe0f786231cf04f7fb510f745894bbf8402bf789013b3043c127cca0e2b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HaploKit/StrainXpress
  tools:
  - HaploConduct
  - StrainXpress
  - minimap2
  edam_operation: http://edamontology.org/operation_0525
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# overlap-layout-consensus-graph-construction

## Summary

Constructs overlap-layout-consensus (OLC) overlap graphs per read cluster to reconstruct strain-aware haplotigs, using minimap2 read-to-read overlaps within a HaploConduct-derived ploidy-aware assembly strategy as implemented in StrainXpress.

## When to use

Use when assembling individual strain genomes (haplotigs) from short-read metagenomic data that has already been split into per-cluster interleaved FASTQ read sets, and strain-resolved contigs are needed.

## When NOT to use

- Input reads have not been pre-clustered by strain — overlap-graph assembly on the full metagenome is computationally intractable; run the clustering stage first
- The goal is maximum contig length (N50) rather than strain resolution — MEGAHIT-type assemblers outperform StrainXpress on contig length
- Data are long third-generation reads with high error rates — the paper notes error rates of third-generation reads introduce issues that still need to be resolved

## Inputs

- Per-cluster interleaved FASTQ read sets from StrainXpress clustering (paired reads labeled /1 and /2)
- StrainXpress installation with minimap2 and HaploConduct dependencies

## Outputs

- final_contigs.fasta containing concatenated strain-aware haplotigs in the stageb folder
- Per-cluster overlap graphs and resolved strain layouts (intermediate)

## How to apply

Load the per-cluster interleaved FASTQ read sets produced by StrainXpress's clustering stage and process each cluster independently. For each cluster, compute read-to-read approximate overlaps with minimap2, replacing the FM-index-based seeding of the HaploConduct (SAVAGE/POLYTE) ploidy-aware strategy — the paper reports this Local Assembly stage is 8 times faster than the original template. Build the OLC overlap graph where nodes are reads/contigs and edges connect reads representing identical haplotypic sequence, resolving layouts that capture co-occurring mutation patterns of individual strains. Generate consensus haplotig sequences per resolved strain layout, then concatenate all cluster haplotigs into final_contigs.fasta in the stageb folder. Keeping cluster sizes small (a product of the upstream clustering) is what makes strain-aware overlap-graph assembly computationally feasible.

## Related tools

- **StrainXpress** (Implements the ploidy-aware OLC local assembly of cluster read sets into strain haplotigs) — https://github.com/HaploKit/StrainXpress
- **minimap2** (Computes read-to-read approximate overlaps within each cluster, replacing FM-index seeding) — https://github.com/lh3/minimap2
- **HaploConduct** (Provides the adopted ploidy-aware overlap-graph assembly strategy (SAVAGE/POLYTE lineage)) — https://github.com/HaploConduct/HaploConduct

## Examples

```
python ../scripts/strainxpress.py -fq all_reads.fq
```

## Evaluation signals

- final_contigs.fasta exists in the stageb folder and parses as valid FASTA
- Haplotigs reproduce co-occurring mutation patterns per strain, i.e. strain-resolved contigs are recovered per read cluster
- Local Assembly runtime is consistent with the reported ~8x speedup over the original FM-index-based template
- Assembly quality assessed with Quast/metaQuast shows improved strain-specific genome fraction relative to generic assemblers (e.g. 93.45% vs 74.74% for SPAdes on low-complexity 2x250 bp data)

## Limitations

- Short Illumina read lengths constrain contig length, preventing strain-specific contigs that span whole genomes
- High error rates of third-generation sequencing reads remain an unresolved issue for this overlap-graph approach
- No specific version numbers for StrainXpress or dependencies are stated, complicating exact reproduction

## Evidence

- [readme] OLC paradigm and ploidy-aware assembly with minimap2 overlaps: "StrainXpress is a de novo assembly method which base on overlap-layout-consensus (OLC) paradigm"
- [readme] minimap2 as dependency for overlap computation: "StrainXpress relies on the following dependencies: - [minimap2](https://github.com/lh3/minimap2) - [HaploConduct](https://github.com/HaploConduct/HaploConduct)"
- [results] Local Assembly stage is 8x faster than original template: [evidence span withheld — anchor status: not_found]
- [discussion] Overlap graphs optimally highlight co-occurring mutation patterns: "patterns of co-occurring mutations—as the decisive characteristic sequential phenomenon of strains—can be highlighted optimally with overlap graphs"
- [readme] Interleaved FASTQ input requirement: "The input file must be interleaved FASTQ and format like below:"
