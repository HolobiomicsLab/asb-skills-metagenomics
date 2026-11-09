---
name: nucleotide-sequence-clustering
description: Use when a nucleotide FASTA of predicted coding sequences (e.g. ~5.5M
  CDSs from assembled vaginal metagenomes) must be de-replicated into a non-redundant
  gene catalog at ≥95% identity over ≥90% of the shorter gene length.
when_to_use_negative:
- Input sequences are proteins/amino acids — CD-HIT-EST is nucleotide-only; for protein
  families use the paper's VOG Jaccard-clustering workflow (Module 7, --linkscore=0.6,
  80% identity, 60% coverage)
- The VIRGO catalog already exists and the goal is per-sample abundance/annotation
  — map reads with VIRGO's runMapping.step1.sh against the existing non-redundant
  database instead of re-clustering
- You need orthologous-group or taxonomy-aware structure rather than nucleotide redundancy
  removal — the 95%/90% rule collapses any sufficiently similar nucleotide pair regardless
  of function or annotation
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0080
tools:
- name: CD-HIT-EST
  role: Greedy incremental nucleotide clustering engine run at -c 0.95 / -aS 0.90
    to cluster the ~5.5M predicted CDSs and emit the longest sequence of each cluster
    as the representative
- name: VIRGO
  role: Source of the predicted-CDS FASTA input and the V-prefixed gene-ID scheme
    that the non-redundant catalog must preserve for its 1_VIRGO annotation files
    and runMapping modules
  repo: github.com/ravel-lab/VIRGO
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-14677-3
    title: A comprehensive non-redundant gene catalog reveals extensive within-community
      intraspecies diversity in the human vagina
  source_content_hash: sha256:7662548ee6a90c90d53ba21662b28f7b095fc31a30fcd2451b3407ad4bad3574
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CD-HIT-EST
  - VIRGO
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# nucleotide-sequence-clustering

## Summary

Greedy nucleotide-level de-redundancy of predicted CDSs using CD-HIT-EST v4.6 (-c 0.95 identity, -aS 0.90 coverage of the shorter gene), which collapsed ~5.5 million predicted vaginal metagenomic CDSs into the 948,158-gene VIRGO non-redundant gene catalog, retaining the longest sequence per cluster as representative.

## When to use

Use when a nucleotide FASTA of predicted coding sequences (e.g. ~5.5M CDSs from assembled vaginal metagenomes) must be de-replicated into a non-redundant gene catalog at ≥95% identity over ≥90% of the shorter gene length.

## When NOT to use

- Input sequences are proteins/amino acids — CD-HIT-EST is nucleotide-only; for protein families use the paper's VOG Jaccard-clustering workflow (Module 7, --linkscore=0.6, 80% identity, 60% coverage)
- The VIRGO catalog already exists and the goal is per-sample abundance/annotation — map reads with VIRGO's runMapping.step1.sh against the existing non-redundant database instead of re-clustering
- You need orthologous-group or taxonomy-aware structure rather than nucleotide redundancy removal — the 95%/90% rule collapses any sufficiently similar nucleotide pair regardless of function or annotation

## Inputs

- Nucleotide FASTA of ~5.5 million predicted CDSs (V-prefixed IDs) predicted from assembled vaginal metagenomes
- CD-HIT-EST v4.6 parameter set: -c 0.95 identity, -aS 0.90 coverage of the shorter sequence; 99 bp minimum CDS length filter applied pre-clustering

## Outputs

- nr_gene_catalog.fasta: non-redundant gene catalog FASTA with 948,158 representative (longest-per-cluster) CDSs, each >99 bp
- V-prefixed gene identifiers retained for downstream VIRGO annotation and read-mapping modules

## How to apply

Load the predicted CDS nucleotide FASTA (~5.5 million sequences assembled from vaginal metagenomes) and first discard CDSs shorter than 99 bp so very short fragments cannot seed spurious clusters. Run CD-HIT-EST v4.6 in greedy incremental mode with sequence identity threshold -c 0.95 and alignment coverage of the shorter sequence -aS 0.90, so two CDSs join a cluster only at ≥95% identity over ≥90% of the shorter gene length. The greedy incremental algorithm sorts sequences longest-first and assigns each to the first cluster whose representative meets both thresholds, keeping the all-vs-all comparison tractable at millions of sequences. Write the cluster representatives (the longest sequence per cluster) to nr_gene_catalog.fasta as the non-redundant gene catalog, preserving the V-prefixed gene IDs consumed by downstream VIRGO annotation (1_VIRGO) and mapping (runMapping) modules. Validate by counting output sequences: 948,158 non-redundant CDSs, ~17.2% of the original 5.5 million, consistent with the reported 0.95-million-gene VIRGO catalog.

## Related tools

- **CD-HIT-EST** (Greedy incremental nucleotide clustering engine run at -c 0.95 / -aS 0.90 to cluster the ~5.5M predicted CDSs and emit the longest sequence of each cluster as the representative)
- **VIRGO** (Source of the predicted-CDS FASTA input and the V-prefixed gene-ID scheme that the non-redundant catalog must preserve for its 1_VIRGO annotation files and runMapping modules) — github.com/ravel-lab/VIRGO

## Examples

```
cd-hit-est -i predicted_cds.fasta -o nr_gene_catalog.fasta -c 0.95 -aS 0.90
```

## Evaluation signals

- Output FASTA contains exactly 948,158 sequences (~17.2% of the ~5.5M input CDSs), matching the published VIRGO catalog size
- No retained CDS is shorter than 99 bp, confirming the pre-cluster length filter was applied
- No catalog sequence matches a longer catalog sequence at ≥95% identity over ≥90% of the shorter gene length (representatives are the longest cluster members)
- All FASTA headers keep V-prefixed gene IDs (e.g. V1593031) resolvable against VIRGO's 1_VIRGO/1.taxon.tbl.txt and runMapping outputs
- Recorded parameters are CD-HIT-EST v4.6 with -c 0.95 and -aS 0.90; a re-run on the same input order reproduces the identical representative set

## Limitations

- CDSs shorter than 99 bp are discarded before clustering, so very short genes never enter the non-redundant catalog
- Any two CDSs ≥95% identical over ≥90% of the shorter sequence are merged, so near-identical recent paralogs or highly conserved genes may collapse into a single representative
- Greedy incremental clustering is length- and order-dependent, not a global optimum; representative choice follows sequence length rather than biological priority
- The resulting VIRGO database is primarily bacterial, with limited inclusion of viral and fungal gene sequences (10,908 fungal and 15,965 bacteriophage genes are annotated separately in 2_fungal_phage)

## Evidence

- [methods] Run CD-HIT-EST v4.6 in greedy incremental mode with sequence identity threshold -c 0.95 and alignment coverage of the shorter sequence -aS 0.90: [evidence span withheld — anchor status: not_found]
- [results] yielded a catalog of 948,158 non-redundant CDSs longer than 99 bp, representing 17.2% of the original 5.5 million predicted CDSs: [evidence span withheld — anchor status: not_found]
- [methods] Discard CDS sequences shorter than 99 bp to avoid spurious clusters of very short fragments.: [evidence span withheld — anchor status: not_found]
- [abstract] we assemble a comprehensive human vaginal non-redundant gene catalog (VIRGO) that includes 0.95 million non-redundant genes: "we assemble a comprehensive human vaginal non-redundant gene catalog (VIRGO) that includes 0.95 million non-redundant genes"
- [readme] is a non-redundant gene catalog for the microbial communities that inhabit the human vagina. It was built using a combination of metagenomic and urogential isolate genome sequences.: "is a non-redundant gene catalog for the microbial communities that inhabit the human vagina. It was built using a combination of metagenomic and urogential isolate genome sequences."
- [discussion] This version is focused on the gene-level de-redundancy and characterization of vaginal microbiome.: "This version is focused on the gene-level de-redundancy and characterization of vaginal microbiome."
