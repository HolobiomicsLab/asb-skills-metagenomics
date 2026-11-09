---
name: gene-synteny-cluster-inference-from-contig-adjacency
description: Use when you have VAMB clusters.tsv, assembled contigs and reads from
  a metagenomics run and want to group genes into synteny clusters by contig adjacency,
  then attach eggNOG-mapper KEGG/GO/Pfam annotations to assess intra-cluster functional
  agreement.
when_to_use_negative:
- Accuracy is limited by MAGinator's lack of operon awareness, so avoid over-interpreting
  synteny clusters as true operons when precise operon boundaries matter
- If you only need taxonomic profiles or abundances without gene-content/pathway association,
  use MAGinator's signature-gene abundance output instead of the synteny-clustering
  step
- When input is raw reads without prior assembly, binning (VAMB clusters.tsv) and
  ORF calling, the pipeline prerequisites are missing
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: MAGinator
  role: Runs the ORF clustering and synteny-clustering step grouping gene clusters
    by contig adjacency (synteny_clusters.tab)
  repo: https://github.com/Russel88/MAGinator
- name: Snakemake
  role: Workflow engine that executes MAGinator with automatic dependency installation
  repo: https://github.com/Russel88/MAGinator
- name: EggNOG-mapper
  role: Diamond-mode functional annotation of the non-redundant gene catalogue; KEGG
    ortholog column 15 is parsed for intra-cluster module agreement
  repo: https://github.com/eggnogdb/eggnog-mapper
- name: VAMB
  role: Provides clusters.tsv mapping of contigs to bins that MAGinator consumes as
    input
  repo: https://github.com/RasmussenLab/vamb
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-49958-8
    title: MAGinator enables accurate profiling of de novo MAGs with strain-level
      phylogenies
  source_content_hash: sha256:2535047790c32853169ab7ae89204a19b8c83e5de272635298594ab04b82b80a
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
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# gene-synteny-cluster-inference-from-contig-adjacency

## Summary

MAGinator's synteny clustering groups gene clusters (ORFs) by how often they occur adjacent on contigs, producing synteny_clusters.tab associations that link shared pathways to subspecies-level MAG clusters. When combined with eggNOG-mapper KEGG ortholog annotations, it enables quantitative functional-coherence evaluation of each synteny cluster.

## When to use

Use when you have VAMB clusters.tsv, assembled contigs and reads from a metagenomics run and want to group genes into synteny clusters by contig adjacency, then attach eggNOG-mapper KEGG/GO/Pfam annotations to assess intra-cluster functional agreement.

## When NOT to use

- Accuracy is limited by MAGinator's lack of operon awareness, so avoid over-interpreting synteny clusters as true operons when precise operon boundaries matter
- If you only need taxonomic profiles or abundances without gene-content/pathway association, use MAGinator's signature-gene abundance output instead of the synteny-clustering step
- When input is raw reads without prior assembly, binning (VAMB clusters.tsv) and ORF calling, the pipeline prerequisites are missing

## Inputs

- clusters.tsv from VAMB (MAG/gene cluster membership mapping)
- Fasta file of all contigs with unique names
- CSV mapping SampleName, forward reads, reverse reads
- GTDB-tk database release 220
- Non-redundant gene catalogue FASTA (all_genes_nonredundant.fasta / all_genes_rep_seq.fasta)
- eggNOG-mapper annotations.tsv output (diamond mode)

## Outputs

- synteny_clusters.tab association table linking gene clusters to synteny clusters
- MAGfunctions.tsv merging eggNOG KEGG orthologs with clusters.tsv (sample/MAG_cluster/MAG/function)
- Per-cluster KEGG module agreement statistics (e.g. 746,251 synteny clusters; 28,341 with all genes sharing one KEGG module; 80.5% of modules with >80% intra-cluster agreement)

## How to apply

Run the MAGinator pipeline (snakemake + mamba installed via bioconda; maginator -v vamb_clusters.tsv -r reads.csv -c contigs.fasta -g GTDB-tk release220) so that its synteny-clustering step groups gene clusters by how frequently they are found adjacent on contigs, yielding the synteny_clusters.tab association table. Annotate the non-redundant gene catalogue (all_genes_nonredundant.fasta / all_genes_rep_seq.fasta, nucleotide representatives of gene clusters) with EggNOG-mapper in diamond mode (emapper.py -m diamond). Parse the eggNOG output by cutting the KEGG ortholog column 15, removing the header (first 5 lines), trailing summary lines (last 3), and unannotated '-' entries, then split comma-separated KO terms and strip the 'ko:' prefix. Merge the parsed annotations with VAMB clusters.tsv and the synteny cluster associations (e.g. via the README's join/awk one-liner producing MAGfunctions.tsv) to attach functions to each synteny cluster. For each qualifying synteny cluster (>=5 genes and >=2 annotated genes), determine the predominant KEGG module and the degree of intra-cluster functional agreement; in COPSAC2010 this yielded 35,798 qualifying clusters.

## Related tools

- **MAGinator** (Runs the ORF clustering and synteny-clustering step grouping gene clusters by contig adjacency (synteny_clusters.tab)) — https://github.com/Russel88/MAGinator
- **Snakemake** (Workflow engine that executes MAGinator with automatic dependency installation) — https://github.com/Russel88/MAGinator
- **EggNOG-mapper** (Diamond-mode functional annotation of the non-redundant gene catalogue; KEGG ortholog column 15 is parsed for intra-cluster module agreement) — https://github.com/eggnogdb/eggnog-mapper
- **VAMB** (Provides clusters.tsv mapping of contigs to bins that MAGinator consumes as input) — https://github.com/RasmussenLab/vamb

## Examples

```
emapper.py -i test/genes/all_genes_rep_seq.fasta --output test_out/functional_annotation -m diamond --cpu 38
```

## Evaluation signals

- Number of synteny clusters matches the reported 746,251 on COPSAC2010
- 28,341 synteny clusters have all genes assigned the same KEGG module
- 80.5% of KEGG modules show >80% intra-cluster agreement
- Filtering >=5 genes and >=2 annotated genes yields exactly 35,798 qualifying synteny clusters
- eggNOG parsing sanity check: header (first 5 lines), trailing 3 summary lines, '-' unannotated rows, comma-separated KO terms, and 'ko:' prefixes are all correctly handled before merging

## Limitations

- Currently, accuracy is limited by MAGinator's lack of operon awareness
- Users can also eliminate noise at the expense of sensitivity by altering a number of user-modifiable parameters
- Synteny cluster functional coherence depends on annotation coverage; only clusters with >=2 annotated genes are evaluated

## Evidence

- [readme] README: "MAGinator clusters gene clusters into synteny clusters - Syntenic genes are usually part of the same pathway or have similar functions"
- [intro] article intro: "By combining the information from both contigs and gene content we identify synteny clusters of genes within subspecies, yielding information on shared pathways for the genes"
- [readme] README functional annotation: "The eggNOG output can be merged with clusters.tsv and further processed to obtain functional annotations of the MAG, cluster or sample levels"
- [readme] README eggnog command: "emapper.py -i test/genes/all_genes_rep_seq.fasta --output test_out/functional_annotation -m diamond --cpu 38"
- [discussion] article discussion: "Currently, accuracy is limited by MAGinator’s lack of operon awareness."
