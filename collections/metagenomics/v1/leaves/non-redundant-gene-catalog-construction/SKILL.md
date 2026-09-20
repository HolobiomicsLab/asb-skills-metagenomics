---
name: non-redundant-gene-catalog-construction
description: Use when you have millions of predicted CDSs pooled from several sources
  (e.g., metagenome assemblies plus isolate genomes) and need to collapse redundant
  sequences into a single non-redundant gene catalog with stable gene IDs.
when_to_use_negative:
- Input is a small set of isolate genomes where pairwise ANI/dereplication at strain
  level is more appropriate than a community gene catalog
- You need taxonomic/functional profiling of reads against an existing catalog — use
  VIRGO's runMapping.step1.sh mapping modules instead of rebuilding the catalog
- Sequences are already de-replicated or you need nucleotide-variant (SNV) resolution,
  which the reference-based VIRGO approach does not yet provide
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: CD-HIT-EST
  role: Greedy incremental nucleotide clustering (-c 0.95, -aS 0.90) that de-replicates
    pooled CDSs into the non-redundant catalog
- name: Wu-blastp
  role: All-vs-all BLASTP comparison feeding sequence-similarity clustering and Jaccard
    protein clustering (BSML output)
- name: VIRGO
  role: Pipeline framework that pools CDSs, hosts the 0_db/1_VIRGO database structure,
    and assigns V-prefixed gene IDs used downstream
  repo: github.com/ravel-lab/VIRGO
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-14677-3
    title: A comprehensive non-redundant gene catalog reveals extensive within-community
      intraspecies diversity in the human vagina
  source_content_hash: sha256:4c76fdf3e1050036c15378841347977a68bb864ad2d96ea33322d8c0d53bf665
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CD-HIT-EST
  - VIRGO
  - Wu-blastp
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# non-redundant-gene-catalog-construction

## Summary

Builds a non-redundant gene catalog by pooling predicted CDSs from multiple sources (metagenomic assemblies and isolate genomes) and de-replicating them with nucleotide-level greedy clustering, as done for the 0.95-million-gene VIRGO human vaginal gene catalog.

## When to use

Use when you have millions of predicted CDSs pooled from several sources (e.g., metagenome assemblies plus isolate genomes) and need to collapse redundant sequences into a single non-redundant gene catalog with stable gene IDs.

## When NOT to use

- Input is a small set of isolate genomes where pairwise ANI/dereplication at strain level is more appropriate than a community gene catalog
- You need taxonomic/functional profiling of reads against an existing catalog — use VIRGO's runMapping.step1.sh mapping modules instead of rebuilding the catalog
- Sequences are already de-replicated or you need nucleotide-variant (SNV) resolution, which the reference-based VIRGO approach does not yet provide

## Inputs

- Nucleotide FASTA of predicted CDSs (~5.5 million sequences) pooled from assembled vaginal metagenomes and urogenital isolate genomes
- Quality-filtered metagenomic read sets (fastq) used for per-sample assembly and CDS prediction
- Protein sequences for all-vs-all Wu-blastp comparison (VOG clustering branch)

## Outputs

- nr_gene_catalog.fasta: 948,158 non-redundant CDS representatives (>99 bp) with V-prefixed gene IDs (~17.2% of original 5.5M CDSs)
- Companion annotation files in the 1_VIRGO directory structure (gene lengths, taxonomy table, protein-family, eggNOG, EC, product, KEGG annotations)
- VOG protein clusters (0.VOG.VIRGO.tbl.txt linker, 1.VOG.size.txt) for the Jaccard-clustering branch

## How to apply

Pool all predicted CDSs — in the VIRGO case, from per-sample de novo assemblies of 264 quality-filtered vaginal metagenomes, HMP metagenomes, and 308 urogenital isolate genomes — into one nucleotide FASTA. Discard CDSs shorter than 99 bp to avoid spurious clusters of very short fragments. Run CD-HIT-EST v4.6 in greedy incremental mode with sequence identity threshold -c 0.95 and alignment coverage of the shorter sequence -aS 0.90, so a cluster requires >=95% identity over >=90% of the shorter gene length; write the longest sequence per cluster as the representative. The protein-level complement uses Wu-blastp all-vs-all comparison followed by Jaccard clustering (clusterBsmlPairwiseAlignments-mod.pl with --percent_identity=80 --percent_coverage=60) and bidirectional best-hit merging to build vaginal orthologous groups (VOGs). Preserve V-prefixed gene IDs so downstream VIRGO annotation (0.geneLength, 1.taxon.table, eggNOG, EC, product, KEGG files) and read-mapping modules remain consistent.

## Related tools

- **CD-HIT-EST** (Greedy incremental nucleotide clustering (-c 0.95, -aS 0.90) that de-replicates pooled CDSs into the non-redundant catalog)
- **Wu-blastp** (All-vs-all BLASTP comparison feeding sequence-similarity clustering and Jaccard protein clustering (BSML output))
- **VIRGO** (Pipeline framework that pools CDSs, hosts the 0_db/1_VIRGO database structure, and assigns V-prefixed gene IDs used downstream) — github.com/ravel-lab/VIRGO

## Examples

```
cd-hit-est -i predicted_cds.fasta -o nr_gene_catalog.fasta -c 0.95 -aS 0.90 -g 1
```

## Evaluation signals

- Output FASTA contains exactly 948,158 sequences, matching the published ~0.95-million-gene VIRGO catalog
- Retained genes represent ~17.2% of the original 5.5 million predicted CDSs
- Partition counts per source are 634,288 (in-house metagenomes) / 54,500 (HMP metagenomes) / 371,099 (isolate genomes)
- No catalog sequence is shorter than 99 bp and all IDs carry the V prefix
- Every representative is the longest member of its CD-HIT-EST cluster and gene lengths in 0.geneLength agree with the FASTA

## Limitations

- Reference-based approach with gene-level de-redundancy; capability to identify nucleotide variants within a gene is not yet included
- Database is primarily focused on bacteria with limited inclusion of viral and fungal gene sequences (10,908 fungal and 15,965 bacteriophage genes are separate)
- Gaps may reflect limited isolate genomes or systematic bias in growth and recovery of vaginal species
- The paper's de-redundification combines nucleotide-level (CD-HIT-EST) and protein-level (Wu-blastp + Jaccard clustering) approaches; mixing branches changes the resulting catalog

## Evidence

- [methods] task_004 finding (methods): [evidence span withheld — anchor status: not_found]
- [abstract] abstract catalog size: "we assemble a comprehensive human vaginal non-redundant gene catalog (VIRGO) that includes 0.95 million non-redundant genes"
- [readme] README catalog definition: "**VIRGO** is a non-redundant gene catalog for the microbial communities that inhabit the human vagina. It was built using a combination of metagenomic and urogential isolate genome sequences."
- [methods] Jaccard clustering parameters: "--linkscore=0.6 \
 --percent_identity=80 \
 --percent_coverage=60 \
 --p_value=1e-5"
- [methods] VOG clustering method: "Run Jaccard clustering using modified clusterBsmlPairwiseAlignments script"
