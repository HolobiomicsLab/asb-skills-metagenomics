---
name: isolate-genome-gene-prediction
description: Use when building a non-redundant gene catalog from pooled sources and
  you need gene predictions from complete/draft isolate genomes (e.g., 308 urogenital
  isolates) to complement CDSs predicted from per-sample metagenomic assemblies.
when_to_use_negative:
- Input is raw vaginal metagenomic reads to be profiled against the finished catalog
  — map reads to VIRGO with runMapping.step1.sh instead of re-predicting genes
- Input is already a non-redundant gene catalog or gene abundance table — de-redundification/prediction
  would duplicate existing catalog structure
- Target is primarily viral or fungal sequence — VIRGO's database is 'primarily focused
  on bacteria with limited inclusion of viral and fungal gene sequences' (fungal/phage
  genes are handled separately in 2_fungal_phage)
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: VIRGO
  role: Pipeline whose scripts perform CDS prediction from isolate genomes and subsequent
    pooling/de-redundification into the non-redundant catalog
  repo: github.com/ravel-lab/VIRGO
- name: Wu-blastp
  role: All-vs-all BLASTP comparison of pooled predicted CDSs (including isolate-derived
    genes) used for sequence-similarity clustering to de-redundify genes
- name: BLAST
  role: Downstream makes the finished catalog blastable in nucleotide and amino acid
    (blastn/blastp, blast 2.2.3+ compatible)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-14677-3
    title: A comprehensive non-redundant gene catalog reveals extensive within-community
      intraspecies diversity in the human vagina
  source_content_hash: sha256:45f6f34ca3cc6437528f2900d76a060bddcba55d1c24660aaa63ebe465b6f378
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLAST
  - VIRGO
  - Wu-blastp
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# isolate-genome-gene-prediction

## Summary

Predicts coding sequences (CDSs) from cultured isolate genome sequences — in VIRGO's case 308 urogenital isolate genomes — so their genes can be pooled with metagenomic-assembly CDSs and de-redundified into a non-redundant body-site gene catalog.

## When to use

Use when building a non-redundant gene catalog from pooled sources and you need gene predictions from complete/draft isolate genomes (e.g., 308 urogenital isolates) to complement CDSs predicted from per-sample metagenomic assemblies.

## When NOT to use

- Input is raw vaginal metagenomic reads to be profiled against the finished catalog — map reads to VIRGO with runMapping.step1.sh instead of re-predicting genes
- Input is already a non-redundant gene catalog or gene abundance table — de-redundification/prediction would duplicate existing catalog structure
- Target is primarily viral or fungal sequence — VIRGO's database is 'primarily focused on bacteria with limited inclusion of viral and fungal gene sequences' (fungal/phage genes are handled separately in 2_fungal_phage)

## Inputs

- Draft or complete genome sequences of urogenital isolate strains (308 isolate genomes in VIRGO; FASTA assemblies)
- Quality-filtered vaginal metagenomic read sets (264 samples) already de novo assembled per sample, whose CDS predictions will be pooled with isolate predictions
- VIRGO directory structure (0_db, 1_VIRGO) from github.com/ravel-lab/VIRGO for downstream annotation files

## Outputs

- Predicted CDS nucleotide/protein sequences per isolate genome (FASTA), pooled with metagenomic CDS predictions
- Partition-level CDS counts (e.g., 371,099 non-redundant CDSs from isolate genomes vs 634,288 in-house metagenomes and 54,500 HMP metagenomes)
- Genes carried forward into the 948,158-gene non-redundant VIRGO catalog and companion annotation files (gene lengths, taxonomy table, protein-family, eggNOG, EC, product, KEGG files under 1_VIRGO)

## How to apply

Run CDS prediction independently on each isolate genome assembly, in parallel with the per-sample de novo assembly of quality-filtered vaginal metagenomic reads and its own CDS prediction. Pool the predicted CDSs from the isolate genomes with those from the in-house and HMP metagenome assemblies (~5.5 million CDSs total from three sources) before de-redundification. Retain only CDSs longer than 99 bp, then feed the combined protein set into all-vs-all BLASTP (Wu-blastp) and sequence-similarity clustering to collapse redundancies into the final catalog. Rationale: isolate genomes contribute high-quality complete genes that metagenomic assemblies fragment or miss, raising catalog completeness (VIRGO reaches ~0.95 million non-redundant genes, 371,099 attributable to the isolate-genome partition).

## Related tools

- **VIRGO** (Pipeline whose scripts perform CDS prediction from isolate genomes and subsequent pooling/de-redundification into the non-redundant catalog) — github.com/ravel-lab/VIRGO
- **Wu-blastp** (All-vs-all BLASTP comparison of pooled predicted CDSs (including isolate-derived genes) used for sequence-similarity clustering to de-redundify genes)
- **BLAST** (Downstream makes the finished catalog blastable in nucleotide and amino acid (blastn/blastp, blast 2.2.3+ compatible))

## Evaluation signals

- Pooled CDS pool before de-redundification totals ~5.5 million CDSs across the three sources (in-house metagenomes, HMP metagenomes, isolate genomes)
- After de-redundification the catalog totals 948,158 non-redundant CDSs longer than 99 bp, i.e. ~0.95 million genes as reported (17.2% of the original predictions)
- Partition counts of the non-redundant catalog match 634,288 (in-house metagenomes) / 54,500 (HMP metagenomes) / 371,099 (isolate genomes)
- Every retained gene has a companion entry in the 1_VIRGO annotation files (0.geneLength, 1.taxon.table, proteinFamily, eggNOG, EC, product, KEGG files)
- Test run of the pipeline reproduces expected _test_run outputs (summary.Abundance.txt, summary.Count.txt, summary.geneRichness.txt, etc.)

## Limitations

- Isolate-genome representation is uneven: gaps may reflect limited isolate genomes per species or systematic bias in growth and recovery of vaginal species, requiring targeted isolation
- Metagenomic subspecies contain non-redundant genes not observed in isolate genome sequences for that species, so isolate predictions alone under-represent within-species accessory-gene diversity
- VIRGO's catalog is primarily bacterial; fungal and bacteriophage genes (10,908 and 15,965) are annotated separately and not fully integrated

## Evidence

- [readme] VIRGO built from metagenomic and isolate sequences: "VIRGO is a non-redundant gene catalog for the microbial communities that inhabit the human vagina. It was built using a combination of metagenomic and urogential isolate genome sequences."
- [abstract] Catalog size and scope: "we assemble a comprehensive human vaginal non-redundant gene catalog (VIRGO) that includes 0.95 million non-redundant genes"
- [intro] Catalog construction workflow: "The genes identified in these data were further clustered into Vaginal Orthologous Groups (VOGs), providing a catalog of functional protein families common to vaginal microbiomes."
- [intro] Annotation curation of predicted genes: "We meticulously curated the gene catalog with taxonomic assignments as well as functional features using 17 diverse protein databases."
- [discussion] Isolate-derived genes absent from isolate genomes: "the observed metagenomic subspecies contain non-redundant genes, which have not been observed in isolate genome sequences for that species"
