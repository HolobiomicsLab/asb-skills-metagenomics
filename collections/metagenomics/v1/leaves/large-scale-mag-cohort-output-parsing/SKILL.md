---
name: large-scale-mag-cohort-output-parsing
description: 'Use when you have MAGinator outputs from a cohort (e.g. COPSAC2010:
  746,251 synteny clusters) plus eggNOG-mapper annotations and need to attach KEGG
  functions to synteny clusters and measure their coherence.'
when_to_use_negative:
- You only need sample- or MAG-level functional profiles rather than synteny-cluster
  coherence — use the README's MAGfunctions.tsv one-liner instead
- Your genes have no eggNOG-mapper KEGG ortholog annotations (unannotated entries
  are dropped, so agreement metrics cannot be computed)
- Your gene-to-contig adjacency comes from a tool other than MAGinator, since the
  synteny_clusters.tab schema and qualifying-cluster cutoffs are specific to its output
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: MAGinator
  role: Produces the synteny_clusters.tab association table by grouping gene clusters
    based on genomic adjacency on contigs
  repo: https://github.com/Russel88/MAGinator
- name: EggNOG-mapper
  role: Annotates the non-redundant gene catalogue with KEGG orthologs (emapper.py
    -m diamond); KEGG ortholog column 15 is extracted for merging
  repo: https://github.com/eggnogdb/eggnog-mapper
- name: VAMB
  role: Supplies the clusters.tsv gene-to-MAG-cluster mapping that is joined with
    the eggNOG annotations
  repo: https://github.com/RasmussenLab/vamb
- name: Snakemake
  role: Workflow engine in which MAGinator is implemented to distribute the large-scale
    cohort processing
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-49958-8
    title: MAGinator enables accurate profiling of de novo MAGs with strain-level
      phylogenies
  source_content_hash: sha256:2e9d059637bbf5b0ac50264c05ae08e21da26895301fc8758e1b514302b25fc9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Russel88/MAGinator
  tools:
  - EggNOG-mapper
  - MAGinator
  - Snakemake
  - VAMB
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# large-scale-mag-cohort-output-parsing

## Summary

Parse and merge large-scale MAGinator cohort outputs (VAMB clusters.tsv, synteny_clusters.tab, eggNOG-mapper annotations) with shell text-processing (join/awk/sed/cut) to attach KEGG functional annotations to synteny clusters and quantify intra-cluster functional agreement.

## When to use

Use when you have MAGinator outputs from a cohort (e.g. COPSAC2010: 746,251 synteny clusters) plus eggNOG-mapper annotations and need to attach KEGG functions to synteny clusters and measure their coherence.

## When NOT to use

- You only need sample- or MAG-level functional profiles rather than synteny-cluster coherence — use the README's MAGfunctions.tsv one-liner instead
- Your genes have no eggNOG-mapper KEGG ortholog annotations (unannotated entries are dropped, so agreement metrics cannot be computed)
- Your gene-to-contig adjacency comes from a tool other than MAGinator, since the synteny_clusters.tab schema and qualifying-cluster cutoffs are specific to its output

## Inputs

- MAGinator synteny cluster association table (synteny_clusters.tab)
- VAMB clusters.tsv gene-to-cluster mapping file
- eggNOG-mapper output (annotations.tsv) run in diamond mode on all_genes_nonredundant.fasta
- Non-redundant gene catalogue FASTA (all_genes_nonredundant.fasta / all_genes_rep_seq.fasta)

## Outputs

- Synteny-cluster-level functional agreement summary (e.g. 35,798 qualifying clusters; 28,341 with all genes in one KEGG module; 80.5% of modules with >80% intra-cluster agreement)
- Merged per-gene function table linking KEGG orthologs, VAMB clusters, MAGs, MAG clusters and synteny clusters

## How to apply

Run the MAGinator synteny-clustering step to produce synteny_clusters.tab associations, then annotate the non-redundant gene catalogue (all_genes_nonredundant.fasta, nucleotide sequences of gene cluster representatives) with eggNOG-mapper in diamond mode. Parse the emapper output by extracting the KEGG ortholog column (column 15), stripping header and trailing summary lines and removing unannotated entries, mirroring the README's merge one-liner (tail -n +6 | head -n -3 | cut -f1,15 | grep -v '\-$'). Merge parsed annotations with the VAMB clusters.tsv mapping and synteny cluster associations so each synteny cluster carries functional labels. For each qualifying synteny cluster (>=5 genes and >=2 annotated genes), determine the predominant KEGG module and the fraction of intra-cluster agreement, then summarize: total clusters, clusters where all genes share one KEGG module, and the percentage of modules exceeding 80% agreement.

## Related tools

- **MAGinator** (Produces the synteny_clusters.tab association table by grouping gene clusters based on genomic adjacency on contigs) — https://github.com/Russel88/MAGinator
- **EggNOG-mapper** (Annotates the non-redundant gene catalogue with KEGG orthologs (emapper.py -m diamond); KEGG ortholog column 15 is extracted for merging) — https://github.com/eggnogdb/eggnog-mapper
- **VAMB** (Supplies the clusters.tsv gene-to-MAG-cluster mapping that is joined with the eggNOG annotations) — https://github.com/RasmussenLab/vamb
- **Snakemake** (Workflow engine in which MAGinator is implemented to distribute the large-scale cohort processing)

## Examples

```
emapper.py -i test/genes/all_genes_rep_seq.fasta --output test_out/functional_annotation -m diamond --cpu 38
```

## Evaluation signals

- Summary reproduces the reported figures: 746,251 total synteny clusters, 35,798 qualifying clusters (>=5 genes, >=2 annotated), 28,341 clusters with all genes sharing a single KEGG module
- 80.5% of KEGG modules show more than 80% intra-cluster agreement in the recapitulated run
- eggNOG-mapper parsing correctly strips 5 header lines and 3 trailing summary lines (tail -n +6 | head -n -3) and removes entries with '-' in the KEGG column
- Join of clusters.tsv (awk '{print $2 "\t" $1}' | sort) with the parsed annotation column produces a non-empty keyed merge with no orphan gene IDs
- Every qualifying synteny cluster has >=5 total genes and >=2 annotated genes by construction

## Limitations

- Accuracy is limited by MAGinator's lack of operon awareness, so synteny clusters may group genes that are adjacent but not functionally coupled
- Users eliminating synteny noise via parameter changes do so at the expense of sensitivity
- Only annotated genes count toward the qualifying-cluster threshold; heavily unannotated clusters are dropped, biasing coherence statistics toward well-characterized functions

## Evidence

- [readme] Improve your functional annotation by grouping your genes in synteny clusters based on genomic adjacency: "MAGinator clusters gene clusters into synteny clusters - Syntenic genes are usually part of the same pathway or have similar functions"
- [readme] eggNOG merge one-liner with KEGG column 15: "In this case the KEGG ortholog column 15 was picked from the eggNOG-mapper output. But by cutting e.g. column number 13, one would obtain GO terms instead."
- [readme] eggNOG-mapper invocation on gene representatives: "emapper.py -i test/genes/all_genes_rep_seq.fasta --output test_out/functional_annotation -m diamond --cpu 38"
- [intro] Synteny cluster identification as a MAGinator workflow stage: "By combining the information from both contigs and gene content we identify synteny clusters of genes within subspecies, yielding information on shared pathways for the genes"
- [discussion] Operon-awareness limitation: "Currently, accuracy is limited by MAGinator’s lack of operon awareness."
