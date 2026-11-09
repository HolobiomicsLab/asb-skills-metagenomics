---
name: unsupervised-protein-family-clustering
description: Use when you have protein FASTA families with pre-existing HMM-based
  labels and want to test whether AAR-kmer-based unsupervised clustering recovers
  that family structure, or to sweep recoding alphabets and kmer lengths for de novo
  family determination.
when_to_use_negative:
- You already have labeled families and need a trained classifier for annotating new
  sequences — use Snekmer's `model`/`apply` supervised pipeline instead of clustering.
- Input is a precomputed feature/kmer table rather than raw protein FASTA sequences;
  Snekmer cluster expects sequence files to recode itself.
- You need exact-match kmer sensitivity on unrecoded sequences without evaluating
  AAR recoding trade-offs; note the exact-match kmer approach 'may be limited in terms
  of sensitivity and coverage'.
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3474
tools:
- name: Snekmer
  role: 'Runs the `cluster` operation mode: recodes proteins into AAR kmer vectors
    and performs hierarchical correlation-distance clustering with optional heatmap
    plots'
  repo: https://github.com/PNNL-CompBio/Snekmer
- name: Snakemake
  role: Workflow engine underlying snekmer cluster; Snakemake arguments are passed
    through directly for parallelization and reproducibility
  repo: https://github.com/PNNL-CompBio/Snekmer
- name: Python
  role: Runtime environment (3.11+) used for installation, config authoring, and downstream
    ROC AUC computation against HMM labels
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1093/bioadv/vbad005
    title: 'Snekmer: a scalable pipeline for protein sequence fingerprinting based
      on amino acid recoding'
  source_content_hash: sha256:606fbec0ad09c4443884064d547a81fac9058b934163c181c3cde34df6aef79f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/PNNL-CompBio/Snekmer
  tools:
  - Python
  - Snakemake
  - Snekmer
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# unsupervised-protein-family-clustering

## Summary

Reproduce de novo protein family structure with Snekmer's unsupervised `cluster` mode by recoding nitrogen cycling family FASTA sequences into AAR kmer vectors, hierarchically clustering them with correlation distance, and validating cluster agreement against HMM-derived family labels via per-family ROC AUC.

## When to use

Use when you have protein FASTA families with pre-existing HMM-based labels and want to test whether AAR-kmer-based unsupervised clustering recovers that family structure, or to sweep recoding alphabets and kmer lengths for de novo family determination.

## When NOT to use

- You already have labeled families and need a trained classifier for annotating new sequences — use Snekmer's `model`/`apply` supervised pipeline instead of clustering.
- Input is a precomputed feature/kmer table rather than raw protein FASTA sequences; Snekmer cluster expects sequence files to recode itself.
- You need exact-match kmer sensitivity on unrecoded sequences without evaluating AAR recoding trade-offs; note the exact-match kmer approach 'may be limited in terms of sensitivity and coverage'.

## Inputs

- Nitrogen cycling protein family FASTA files
- Ground-truth family labels from original HMM family designations
- config.yaml specifying kmer length k, alphabet (0-5 or None), and cluster.params (linkage, distance_threshold)

## Outputs

- AAR-kmer vectors for each sequence per alphabet/k combination
- Cluster memberships from correlation-based hierarchical clustering
- Per-family ROC AUC scores for each alphabet x kmer-length combination
- Summary heatmap of average AUC via cluster_plots

## How to apply

Place the nitrogen cycling family FASTA files into the Snekmer input directory and attach ground-truth labels from the original HMM family designations. In config.yaml, sweep the recoding alphabets 0-5 (hydro, standard, solvacc, hydrocharge, hydrostruct, miqs) plus None (size-20, no recoding) and kmer lengths, then run `snekmer cluster`, a Snakemake-driven workflow, to convert sequences into AAR-kmer vectors. Cluster the kmer vectors with the 'correlation' method (hierarchical clustering using correlation distance), configuring linkage and distance_threshold under cluster.params. Compare resulting cluster memberships to the HMM ground-truth labels and compute ROC AUC per family for each alphabet/kmer-length combination; average per-family AUCs per parameter combination and render a summary heatmap with cluster_plots enabled. The published result — average ROC AUC exceeding 0.9 for more than half of the alphabet/kmer-length combinations — serves as the reproduction benchmark.

## Related tools

- **Snekmer** (Runs the `cluster` operation mode: recodes proteins into AAR kmer vectors and performs hierarchical correlation-distance clustering with optional heatmap plots) — https://github.com/PNNL-CompBio/Snekmer
- **Snakemake** (Workflow engine underlying snekmer cluster; Snakemake arguments are passed through directly for parallelization and reproducibility) — https://github.com/PNNL-CompBio/Snekmer
- **Python** (Runtime environment (3.11+) used for installation, config authoring, and downstream ROC AUC computation against HMM labels)

## Examples

```
snekmer cluster --input input/ --alphabet 5 --k 8
```

## Evaluation signals

- Average per-family ROC AUC exceeds 0.9 for more than half of the examined alphabet and kmer-length combinations, matching the reported Figure 4 clustering performance.
- The cluster run completes for all 7 alphabet settings (0-5 plus None/size-20) at each configured kmer length without missing entries in the AUC summary.
- Cluster memberships reproduce HMM-derived family assignments better than random (ROC AUC >> 0.5) for the nitrogen cycling families.
- The rendered cluster_plots heatmap renders with finite AUC values for every parameter combination and matches the shape of the reported Figure 4 layout.
- AAR-kmer vector dimensions are consistent with the chosen alphabet size (2 for hydro, 7 for standard, 3 for solvacc/hydrocharge/hydrostruct, 10 for miqs, 20 for None).

## Limitations

- Exact-match kmer representation 'may be limited in terms of sensitivity and coverage'; AAR recoding trades fine-grained resolution for linking distant sequence similarity, so results vary with alphabet/k choice.
- The 'correlation' hierarchical clustering outcome depends on linkage and distance_threshold settings in cluster.params, which must be tuned; unsupervised clusters may not map one-to-one onto HMM families.
- Clustering was validated only on the nitrogen cycling family dataset and demo TIGRFAM data in the article; transfer to other protein families is unverified.

## Evidence

- [results] Cluster with correlation hierarchical clustering; benchmark vs HMM labels: [evidence span withheld — anchor status: not_found]
- [abstract] Average AUC > 0.9 for over half of parameter combinations: [evidence span withheld — anchor status: not_found]
- [intro] Exact-match kmer limitation; AAR links distant similarity: "this approach employs an exact-match methodology and thus may be limited in terms of sensitivity and coverage"
- [readme] Six modes including cluster; pip install instructions: "There are six operation modes for Snekmer: `cluster`, `model`, `search`, `learn`, `apply`, and `motif`,"
