---
name: protein-sequence-gene-clustering
description: Use when you have protein FASTA files (*.faa) for thousands of genomes
  (complete or MAGs/SAGs of mixed quality) and need gene clusters to feed a core/accessory
  genome analysis or other trait presence/absence computation.
when_to_use_negative:
- You want OTU tables computed directly from sequencing reads — that is the unrelated
  mOTUs tool (motu-tool.org), not mOTUlizer/mOTUpan
- You already have your own gene clusters or other genome-encoded traits — provide
  them via --cog_file instead of re-running clustering
- Genomes are so divergent that they do not form a closely related group suitable
  for a shared core-genome partitioning
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: mOTUpan.py
  role: Entry point that runs gene clustering with MMseqs2 easy_cluster on --faas
    inputs, or only the clustering via --genome2cog_only
  repo: github.com/moritzbuck/mOTUlizer
- name: MMseqs2
  role: Performs the actual protein sequence clustering via easy_cluster
  repo: https://github.com/soedinglab/MMseqs2
- name: mOTUlizer
  role: Software package implementing mOTUpan and its companion utilities (mOTUlize.py,
    mOTUconvert.py)
  repo: https://github.com/moritzbuck/mOTUlizer
- name: mOTUconvert.py
  role: Converts output of diverse programs (e.g. Roary, PPanGGOLiN) into input files
    for mOTUpan.py as an alternative to raw .faa clustering
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1093/nargab/lqac060
    title: 'mOTUpan: a robust Bayesian approach to leverage metagenome-assembled genomes
      for core-genome estimation'
  source_content_hash: sha256:00ed554f6f1fde7effb5b7f64797c9121ecf2cb6b185df53e8dd6f4af60cd90f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - MMseqs2
  - mOTUconvert.py
  - mOTUlizer
  - mOTUpan
  - mOTUpan.py
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4117647058823529
n_claims: 17
---

# protein-sequence-gene-clustering

## Summary

Clusters protein FASTA files (.faa) from many genomes into gene clusters using MMseqs2 easy_cluster, as the first stage of mOTUpan's Bayesian core-genome partitioning. This clustering stage is computationally efficient — reported at ~4 min on 24 threads for 9443 Staphylococcus aureus GTDB genomes.

## When to use

Use when you have protein FASTA files (*.faa) for thousands of genomes (complete or MAGs/SAGs of mixed quality) and need gene clusters to feed a core/accessory genome analysis or other trait presence/absence computation.

## When NOT to use

- You want OTU tables computed directly from sequencing reads — that is the unrelated mOTUs tool (motu-tool.org), not mOTUlizer/mOTUpan
- You already have your own gene clusters or other genome-encoded traits — provide them via --cog_file instead of re-running clustering
- Genomes are so divergent that they do not form a closely related group suitable for a shared core-genome partitioning

## Inputs

- Protein FASTA files (*.faa), one per genome (e.g. 9443 S. aureus GTDB genomes)
- Optional: precomputed gene clusters via a --cog_file JSON datastructure instead of --faas

## Outputs

- Gene clusters (genome-encoded traits) usable as --cog_file input for mOTUpan
- With --genome2cog_only: a JSON datastructure of gene clusters compatible with --cog_file
- Downstream: TAB-separated per-gene-cluster core/accessory partitioning written to output.tsv via -o

## How to apply

Gather the *.faa protein files for the genome set and run mOTUpan.py with --faas, which internally computes gene clusters via MMseqs2 easy_cluster; for 9443 S. aureus GTDB genomes this stage took ~4 min on 24 threads on a Ryzen 9 3900X. Alternatively, provide precomputed clusters via --cog_file, or run only the clustering step with --genome2cog_only, which returns a JSON datastructure compatible with --cog_file. The resulting gene clusters serve as the discrete genome-encoded traits whose presence/absence patterns mOTUpan evaluates with its iterative Bayesian likelihood-ratio core/accessory partitioning. Validation: confirm the clustering completes within the expected resource envelope and that the downstream output.tsv contains a core/accessory assignment for every gene cluster.

## Related tools

- **mOTUpan.py** (Entry point that runs gene clustering with MMseqs2 easy_cluster on --faas inputs, or only the clustering via --genome2cog_only) — github.com/moritzbuck/mOTUlizer
- **MMseqs2** (Performs the actual protein sequence clustering via easy_cluster) — https://github.com/soedinglab/MMseqs2
- **mOTUlizer** (Software package implementing mOTUpan and its companion utilities (mOTUlize.py, mOTUconvert.py)) — https://github.com/moritzbuck/mOTUlizer
- **mOTUconvert.py** (Converts output of diverse programs (e.g. Roary, PPanGGOLiN) into input files for mOTUpan.py as an alternative to raw .faa clustering)

## Examples

```
mOTUpan.py --faas *.faa -o output.tsv
```

## Evaluation signals

- The clustering stage completes within the reported resource envelope: ~4 min on 24 threads for 9443 S. aureus GTDB genomes
- The full mOTUpan run completes in ~2 h 15 min single-threaded using ~3 GB RAM for the same dataset
- output.tsv (written via -o) contains a core/accessory assignment for every gene cluster
- If --genome2cog_only was used, the returned JSON is accepted by mOTUpan.py via --cog_file
- Core-genome estimates are comparable to Roary and PPanGGOLiN when run on high-quality genomes

## Limitations

- mOTUpan requires MMseqs2 installed (and fastANI in PATH for mOTUlize.py if no --similarities file is provided)
- Existing core-genome analysis methods depend on high-quality complete genomes and are very sensitive to missing genes and fragmented genomes — mOTUpan is designed to be robust to incompleteness, but very low-quality input bins may still behave like SUBs rather than MAGs
- There is an unrelated tool called mOTUs that creates OTU-tables directly from reads; confusion between the two is a documented pitfall

## Evidence

- [methods] Simplest command to run (needs mmseqs2 installed): "Simplest command to run (needs mmseqs2 installed), but many options:

``nmOTUpan.py --faas *.faa -o output.tsv
"
- [methods] it computes gene clusters using mmseqs2: "it computes gene clusters using [`mmseqs2`](https://github.com/soedinglab/MMseqs2)"
- [methods] --genome2cog_only runs only the gene-clustering: "`--genome2cog_only` : only runs the gene-clustering (`mmseqs east_cluster`), returns a `JSON`-datastructure compatible with `--cog_file`"
- [results] 9443 S. aureus benchmark resources: [evidence span withheld — anchor status: not_found]
- [abstract] mOTUpan implemented in mOTUlizer: "mOTUpan is implemented in the mOTUlizer software package, and available at github.com/moritzbuck/mOTUlizer, under GPL 3.0 license."
