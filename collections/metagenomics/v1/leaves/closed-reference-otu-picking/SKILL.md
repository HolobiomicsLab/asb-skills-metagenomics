---
name: closed-reference-otu-picking
description: Use when a 16S rRNA OTU table must be built against a fixed reference
  (Greengenes+IMG) so downstream PICRUSt metagenome prediction can map OTUs to reference
  genomes.
when_to_use_negative:
- The analysis goal is de novo discovery of OTUs without reference-genome mapping
  (closed-reference discards reads not matching Greengenes+IMG)
- Downstream analysis is not PICRUSt metagenome prediction — a standard open-reference
  or de novo OTU table suffices
- The community is dominated by viral or eukaryotic sequences, which the 16S bacterial/archaeal
  reference framework does not cover
edam_operation: http://edamontology.org/operation_3234
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3174
tools:
- name: QIIME 1.5.0-dev
  role: Executes the closed-reference OTU picking protocol via pick_reference_otus_through_otu_table.py
    to build the PICRUSt-compatible OTU table
- name: uclust
  role: Clustering/search engine used to match 16S reads against the Greengenes+IMG
    reference in closed-reference mode
- name: PICRUSt
  role: Downstream consumer of the closed-reference OTU table; requires tips to map
    to reference genomes for gene-content prediction
  repo: https://github.com/picrust/picrust
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/nbt.2676
    title: Predictive functional profiling of microbial communities using 16S rRNA
      marker gene sequences
  source_content_hash: sha256:47f8e70fa98b12ce5304bf06e9b45be2cbbd94aa188d6af2656ba2d8f2bfaca2
schema_version: 0.2.0
metadata:
  license_tier: unknown
  repo_url: https://github.com/picrust/picrust
  tools:
  - PICRUSt
  - QIIME 1.5.0-dev
  - STAMP
  - uclust
  edam_operation: http://edamontology.org/operation_3234
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# Closed-reference OTU picking

## Summary

Picks OTUs by closed-reference clustering of 16S reads against a fixed reference database (Greengenes+IMG), producing a PICRUSt-compatible OTU table as a prerequisite for metagenome prediction.

## When to use

Use when a 16S rRNA OTU table must be built against a fixed reference (Greengenes+IMG) so downstream PICRUSt metagenome prediction can map OTUs to reference genomes.

## When NOT to use

- The analysis goal is de novo discovery of OTUs without reference-genome mapping (closed-reference discards reads not matching Greengenes+IMG)
- Downstream analysis is not PICRUSt metagenome prediction — a standard open-reference or de novo OTU table suffices
- The community is dominated by viral or eukaryotic sequences, which the 16S bacterial/archaeal reference framework does not cover

## Inputs

- 16S rRNA sequence data (e.g. 335 coral mucus samples, QIIME Study ID 1854)
- Greengenes+IMG reference sequence collection with associated gene content tables

## Outputs

- PICRUSt-compatible closed-reference OTU table (BIOM format)
- OTU-to-reference mapping used by PICRUSt normalization and metagenome prediction

## How to apply

Construct the OTU table using the closed-reference OTU picking protocol in QIIME 1.5.0-dev via the pick_reference_otus_through_otu_table.py script, with uclust as the clustering method and the Greengenes+IMG reference collection (e.g. the tax2tree version of Greengenes with KO/COG annotations from IMG v3.5) as the reference database. For the coral mucus case this was applied to the 335-sample 16S dataset (QIIME Study ID 1854, microbio.me) to yield a PICRUSt-compatible OTU table; closed-reference (rather than de novo) picking is essential because every OTU must map to a reference tip that carries predicted gene contents. Reads that fail to hit the reference are discarded, which is acceptable here because downstream prediction depends on the reference mapping rather than exhaustive read recovery.

## Related tools

- **QIIME 1.5.0-dev** (Executes the closed-reference OTU picking protocol via pick_reference_otus_through_otu_table.py to build the PICRUSt-compatible OTU table)
- **uclust** (Clustering/search engine used to match 16S reads against the Greengenes+IMG reference in closed-reference mode)
- **PICRUSt** (Downstream consumer of the closed-reference OTU table; requires tips to map to reference genomes for gene-content prediction) — https://github.com/picrust/picrust

## Examples

```
pick_reference_otus_through_otu_table.py -i seqs.fna -r greengenes_img_ref_seqs.fna -t greengenes_img_tax.txt -p closed_ref_params.txt -o coral_mucus_otus/
```

## Evaluation signals

- Every retained OTU in the output table maps to a Greengenes+IMG reference sequence (closed-reference invariant)
- The resulting OTU table is accepted by PICRUSt's copy-number normalization and metagenome prediction steps without format errors
- Sample count is preserved as expected (e.g. 335 coral mucus samples from Study ID 1854)
- Downstream per-sample weighted NSTI (0.12 +/- 0.02 s.d. for the coral data) confirms reads were picked against a reference containing relevant genomes

## Limitations

- Reads without a reference hit are discarded, so novel lineages are excluded from the predicted metagenome
- Reference database has phylogenetic gaps for some environments; missing reference genomes degrade downstream prediction (reflected in higher NSTI)

## Evidence

- [methods] uclust used as clustering engine: [evidence span withheld — anchor status: not_found]
