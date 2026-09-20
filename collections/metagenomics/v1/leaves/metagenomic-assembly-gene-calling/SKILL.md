---
name: metagenomic-assembly-gene-calling
description: Use when building a body-site non-redundant gene catalog from vaginal
  metagenomic FASTQ read sets and isolate genomes, requiring per-sample assembly,
  CDS prediction, and all-vs-all BLASTP de-redundification.
when_to_use_negative:
- The analysis goal is taxonomic/functional profiling of new samples against an existing
  catalog — use VIRGO's read-mapping module (runMapping.step1.sh) instead of rebuilding
  the catalog
- The input community is primarily viral or fungal — VIRGO is 'primarily focused on
  bacteria with limited inclusion of viral and fungal gene sequences' and separate
  annotation files (2_fungal_phage) cover only 10,908 fungal and 15,965 bacteriophage
  genes
- The goal is identifying nucleotide variants within genes — this capability is not
  yet included in VIRGO
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3174
tools:
- name: VIRGO
  role: Pipeline and reference gene catalog framework for assembling, predicting,
    and de-redundifying vaginal CDSs into the non-redundant gene catalog
  repo: github.com/ravel-lab/VIRGO
- name: Wu-blastp
  role: All-vs-all BLASTP comparison of pooled CDSs feeding sequence-similarity clustering
    for de-redundification
- name: BLAST
  role: Sequence similarity search; blast 2.2.3+ is the compatible version for the
    VIRGO database
- name: clusterBsmlPairwiseAlignments-mod.pl
  role: Modified script performing Jaccard clustering of BLASTP BSML pairwise alignments
    to generate protein clusters (VOGs)
  repo: github.com/ravel-lab/VIRGO
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-14677-3
    title: A comprehensive non-redundant gene catalog reveals extensive within-community
      intraspecies diversity in the human vagina
  source_content_hash: sha256:c29b2cfd9aedd1465b863605b2efa5295d7488342799c739939444cfa03d498f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLAST
  - VIRGO
  - Wu-blastp
  - clusterBsmlPairwiseAlignments-mod.pl
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# metagenomic-assembly-gene-calling

## Summary

De novo assembles quality-filtered vaginal metagenomic reads per sample and predicts coding sequences, which are pooled with CDSs from urogenital isolate genomes and de-redundified into the VIRGO non-redundant gene catalog. This is the core gene-content construction step that yielded 948,158 non-redundant CDSs from ~5.5 million original predictions.

## When to use

Use when building a body-site non-redundant gene catalog from vaginal metagenomic FASTQ read sets and isolate genomes, requiring per-sample assembly, CDS prediction, and all-vs-all BLASTP de-redundification.

## When NOT to use

- The analysis goal is taxonomic/functional profiling of new samples against an existing catalog — use VIRGO's read-mapping module (runMapping.step1.sh) instead of rebuilding the catalog
- The input community is primarily viral or fungal — VIRGO is 'primarily focused on bacteria with limited inclusion of viral and fungal gene sequences' and separate annotation files (2_fungal_phage) cover only 10,908 fungal and 15,965 bacteriophage genes
- The goal is identifying nucleotide variants within genes — this capability is not yet included in VIRGO

## Inputs

- 264 quality-filtered vaginal metagenomic FASTQ read sets (human and rRNA reads removed)
- 308 urogenital isolate genome sequences
- HMP vaginal metagenomic assemblies

## Outputs

- Non-redundant gene catalog of 948,158 CDSs >99 bp (VIRGO)
- Per-partition CDS counts: 634,288 (in-house metagenomes) / 54,500 (HMP metagenomes) / 371,099 (isolate genomes)
- Companion annotation files: 0.geneLength, 1.taxon.table, 2.A/2.B.proteinFamily.txt, 3.eggnog.txt, 4.GC.txt, 5.EC.txt, 6.product.txt, 7.rxn.txt, 8.A-C KEGG files

## How to apply

Preprocess the 264 vaginal metagenomic read sets by removing human-derived reads and rRNA sequences, with per-sample QC on the remaining FASTQ reads. De novo assemble the quality-filtered reads per sample and predict coding sequences; simultaneously predict CDSs from the 308 urogenital isolate genomes. Pool all predicted CDSs from the three source partitions (in-house metagenomes, HMP metagenomes, isolate genomes) and run an all-vs-all BLASTP comparison with Wu-blastp, followed by sequence-similarity clustering to de-redundify genes into a single non-redundant catalog. Retain only non-redundant CDSs longer than 99 bp. Generate the companion annotation files mirroring the 1_VIRGO directory structure (gene lengths, taxonomy table, protein-family, eggNOG, EC, product, and KEGG annotation files). Rationale: pooling assembled-metagenome CDSs with isolate-genome CDSs maximizes the gene space captured for the vaginal microbiome before clustering collapses redundant sequences.

## Related tools

- **VIRGO** (Pipeline and reference gene catalog framework for assembling, predicting, and de-redundifying vaginal CDSs into the non-redundant gene catalog) — github.com/ravel-lab/VIRGO
- **Wu-blastp** (All-vs-all BLASTP comparison of pooled CDSs feeding sequence-similarity clustering for de-redundification)
- **BLAST** (Sequence similarity search; blast 2.2.3+ is the compatible version for the VIRGO database)
- **clusterBsmlPairwiseAlignments-mod.pl** (Modified script performing Jaccard clustering of BLASTP BSML pairwise alignments to generate protein clusters (VOGs)) — github.com/ravel-lab/VIRGO

## Examples

```
./run-jaccard.sh  # after running all-vs-all Wu-blastp, cluster BLASTP BSML alignments with clusterBsmlPairwiseAlignments-mod.pl (--linkscore=0.6 --percent_identity=80 --percent_coverage=60 --p_value=1e-5)
```

## Evaluation signals

- Final catalog totals 948,158 non-redundant CDSs (~0.95 million as reported in the abstract)
- Partition counts match 634,288 / 54,500 / 371,099 for in-house metagenomes / HMP metagenomes / isolate genomes
- Retained CDSs represent ~17.2% of the original ~5.5 million predicted CDSs
- All retained non-redundant CDSs are longer than 99 bp
- Companion annotation files in the 1_VIRGO directory structure (geneLength, taxon.table, proteinFamily, eggnog, EC, product, KEGG) are all generated and populated

## Limitations

- VIRGO is focused on bacteria with limited inclusion of viral and fungal gene sequences
- The reference-based approach of VIRGO has acknowledged limitations; this version is focused on gene-level de-redundancy and characterization
- Capability to identify nucleotide variants within a gene is not yet included and is planned for the future
- Gaps may reflect limited isolate genomes available for a species or systematic bias in the growth and recovery of vaginal species

## Evidence

- [abstract] we assemble a comprehensive human vaginal non-redundant gene catalog (VIRGO) that includes 0.95 million non-redundant genes: "we assemble a comprehensive human vaginal non-redundant gene catalog (VIRGO) that includes 0.95 million non-redundant genes"
- [readme] VIRGO is a non-redundant gene catalog for the microbial communities that inhabit the human vagina: "**VIRGO** is a non-redundant gene catalog for the microbial communities that inhabit the human vagina. It was built using a combination of metagenomic and urogential isolate genome sequences."
- [readme] Run Jaccard clustering using modified clusterBsmlPairwiseAlignments script: "Run Jaccard clustering using modified clusterBsmlPairwiseAlignments script"
- [intro] The genes identified in these data were further clustered into Vaginal Orthologous Groups (VOGs): "The genes identified in these data were further clustered into Vaginal Orthologous Groups (VOGs), providing a catalog of functional protein families common to vaginal microbiomes."
- [discussion] The database is primarily focused on bacteria with limited inclusion of viral and fungal gene sequences.: "The database is primarily focused on bacteria with limited inclusion of viral and fungal gene sequences."
