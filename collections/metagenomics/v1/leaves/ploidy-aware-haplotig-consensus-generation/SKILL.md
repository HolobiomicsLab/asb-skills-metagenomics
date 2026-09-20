---
name: ploidy-aware-haplotig-consensus-generation
description: Use when assembling strain-resolved haplotigs from pre-clustered metagenomic
  short reads (interleaved FASTQ per cluster), where co-occurring mutation patterns
  must be preserved per strain rather than collapsed into a consensus.
when_to_use_negative:
- Input reads have not been pre-partitioned into strain-aware clusters by the StrainXpress
  'Clustering Reads' stage — overlap-graph assembly on unclustered high-complexity
  metagenome input is computationally infeasible
- The goal is maximum contig length (N50) rather than strain resolution — the paper
  notes MEGAHIT outperforms all methods in contig length, and short Illumina reads
  constrain haplotig length
- Input is third-generation long-read data with high error rates — the paper states
  these 'introduce a range of issues that still need to be resolved'
edam_operation: http://edamontology.org/operation_0525
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3697
tools:
- name: StrainXpress
  role: Pipeline whose Local Assembly stage performs the OLC ploidy-aware haplotig
    construction and consensus generation per cluster
  repo: https://github.com/HaploKit/StrainXpress
- name: minimap2
  role: Computes approximate read-to-read overlaps within each cluster, replacing
    FM-index-based seeding
  repo: https://github.com/lh3/minimap2
- name: HaploConduct
  role: Source of the adopted overlap-graph-based ploidy-aware assembly strategy (SAVAGE/POLYTE);
    original template that StrainXpress's local assembly is 8x faster than
  repo: https://github.com/HaploConduct/HaploConduct
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/nar/gkac543
    title: 'StrainXpress: strain aware metagenome assembly from short reads'
  source_content_hash: sha256:4ee7a5b7feae9626b7b5e20902809881ccc8cd376998f25eea7c8dd3f3fb5e43
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
yes_rate: 0.7
n_claims: 10
---

# ploidy-aware-haplotig-consensus-generation

## Summary

Per-read-cluster local assembly stage of StrainXpress that reconstructs strain-aware contigs (haplotigs) using an overlap-graph-based (OLC) ploidy-aware strategy from HaploConduct, with read-to-read overlaps computed by minimap2 instead of FM-index seeding.

## When to use

Use when assembling strain-resolved haplotigs from pre-clustered metagenomic short reads (interleaved FASTQ per cluster), where co-occurring mutation patterns must be preserved per strain rather than collapsed into a consensus.

## When NOT to use

- Input reads have not been pre-partitioned into strain-aware clusters by the StrainXpress 'Clustering Reads' stage — overlap-graph assembly on unclustered high-complexity metagenome input is computationally infeasible
- The goal is maximum contig length (N50) rather than strain resolution — the paper notes MEGAHIT outperforms all methods in contig length, and short Illumina reads constrain haplotig length
- Input is third-generation long-read data with high error rates — the paper states these 'introduce a range of issues that still need to be resolved'

## Inputs

- Per-cluster interleaved FASTQ read files produced by the StrainXpress clustering stage (e.g. reads with /1 and /2 mate tags like @S0R0/1, @S0R0/2)

## Outputs

- final_contigs.fasta in the stageb folder, containing concatenated strain-aware haplotigs (consensus sequences per resolved strain layout per cluster)

## How to apply

Load the interleaved FASTQ read sets produced by the upstream StrainXpress 'Clustering Reads' stage and process each cluster independently. For each cluster, compute approximate read-to-read overlaps with minimap2, which replaces the FM-index-based overlap computation of the adopted HaploConduct (SAVAGE-style) ploidy-aware strategy and makes the stage substantially faster while cluster sizes remain small enough for overlap-graph assembly. Build the OLC overlap graph and resolve layouts representing co-occurring mutation patterns of individual strains, then generate consensus haplotig sequences for each resolved strain layout. Finally concatenate all cluster haplotigs into final_contigs.fasta in the stageb folder. The 8x speedup over the original template (HaploConduct) comes from minimap2-based overlap detection, per the paper's results section.

## Related tools

- **StrainXpress** (Pipeline whose Local Assembly stage performs the OLC ploidy-aware haplotig construction and consensus generation per cluster) — https://github.com/HaploKit/StrainXpress
- **minimap2** (Computes approximate read-to-read overlaps within each cluster, replacing FM-index-based seeding) — https://github.com/lh3/minimap2
- **HaploConduct** (Source of the adopted overlap-graph-based ploidy-aware assembly strategy (SAVAGE/POLYTE); original template that StrainXpress's local assembly is 8x faster than) — https://github.com/HaploConduct/HaploConduct

## Examples

```
python ../scripts/strainxpress.py -fq all_reads.fq
```

## Evaluation signals

- final_contigs.fasta exists in the stageb folder and parses as valid FASTA
- Qualitative acceptance: each cluster yields strain-resolved haplotigs via the overlap-graph approach, reflecting co-occurring mutation patterns rather than a single collapsed consensus
- Runtime check: the Local Assembly stage runs approximately 8x faster than the original HaploConduct template when overlaps are computed with minimap2
- Downstream evaluation with Quast/metaQuast should show strain-specific genome fraction improvements consistent with the paper (e.g. 93.45% vs 74.74% for SPAdes on low-complexity 2x250 bp data)

## Limitations

- Short Illumina-type read length constrains contig length, preventing full-length strain-specific contigs spanning entire genomes
- High error rate of third-generation sequencing reads remains unresolved by this approach
- Feasibility depends on small cluster sizes; clustering quality upstream directly affects local assembly quality
- No specific software version numbers for StrainXpress or dependencies are stated in the paper, complicating exact reproduction

## Evidence

- [results] Local Assembly is 8 times faster than the original template: [evidence span withheld — anchor status: not_found]
- [readme] OLC paradigm and strain resolution: "StrainXpress is a de novo assembly method which base on overlap-layout-consensus (OLC) paradigm and can fast and accurately assemble high complexity metagenome sequencing data at strain resolution."
- [readme] HaploConduct overlap graph strategy: [evidence span withheld — anchor status: external_doc]
- [discussion] Overlap graphs highlight co-occurring mutations: "patterns of co-occurring mutations—as the decisive characteristic sequential phenomenon of strains—can be highlighted optimally with overlap graphs"
- [readme] minimap2 dependency and stageb output: "- [minimap2](https://github.com/lh3/minimap2)
- [HaploConduct](https://github.com/HaploConduct/HaploConduct)*
- g++ >=5.5.0 and with boost libraries"
