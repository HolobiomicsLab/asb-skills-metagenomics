---
name: cross-tool-core-genome-comparison
description: Use when comparing pangenome/core-genome predictions from multiple tools
  (mOTUpan, Roary, PPanGGOLiN) on the same genome sets, especially MAGs spanning a
  wide completeness range.
when_to_use_negative:
- When only one pangenome tool's core-genome estimate is needed — no cross-tool benchmarking
  is required
- When genomes span such broad phylogenetic distances that a 95% ANI species cutoff
  cannot form coherent mOTUs (the method is for closely related genome sets)
- When GFF annotations are unavailable, since Roary and PPanGGOLiN require GFF-annotated
  genomes
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: mOTUpan
  role: Bayesian core-genome estimator run with --faas and, for benchmarking, re-run
    with --cog_file on converted gene clusters
  repo: github.com/moritzbuck/mOTUlizer
- name: mOTUlize.py
  role: Clusters genomes into species-level mOTUs using fastANI similarities at the
    95% ANI cutoff
  repo: github.com/moritzbuck/mOTUlizer
- name: fastANI
  role: Computes pairwise ANI similarities for mOTU-network generation (default similarity-cutoff
    95)
  repo: https://github.com/ParBLiSS/FastANI
- name: mOTUpan.py
  role: Computes core/accessory partitioning with mmseqs2 easy_cluster gene clusters
    or external clusters via --cog_file, iterating the Bayesian likelihood until convergence
  repo: github.com/moritzbuck/mOTUlizer
- name: mmseqs2
  role: Builds the gene clusters used by mOTUpan.py via easy_cluster
  repo: https://github.com/soedinglab/MMseqs2
- name: Roary
  role: State-of-the-art pangenome tool benchmarked against mOTUpan on GFF-annotated
    genomes (v3.13.0)
  repo: https://sanger-pathogens.github.io/Roary/
- name: PPanGGOLiN
  role: State-of-the-art pangenome tool benchmarked against mOTUpan on GFF-annotated
    genomes (v1.1.96)
  repo: https://github.com/labgem/PPanGGOLiN
- name: mOTUconvert.py
  role: Converts Roary and PPanGGOLiN outputs into mOTUpan --cog_file-compatible genome-to-trait
    files
  repo: github.com/moritzbuck/mOTUlizer
- name: CheckM
  role: Supplies the completeness/contamination prior consumed via --checkm
  repo: https://ecogenomics.github.io/CheckM/
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/nargab/lqac060
    title: 'mOTUpan: a robust Bayesian approach to leverage metagenome-assembled genomes
      for core-genome estimation'
  source_content_hash: sha256:605e30b7f54713fa6d7cdf4ec7e9c3e41ce78631aa5b8d6294353bef40a1a174
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CheckM
  - PPanGGOLiN
  - Roary
  - fastANI
  - mOTUconvert.py
  - mOTUlize.py
  - mOTUlizer
  - mOTUpan
  - mOTUpan.py
  - mmseqs2
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 16
---

# cross-tool-core-genome-comparison

## Summary

Benchmark core-genome estimates from mOTUpan against Roary and PPanGGOLiN by converting each tool's gene-cluster definitions into mOTUpan's genome-to-trait format and recomputing partitionings on the same genome sets, quantifying agreement via normalized residues.

## When to use

Use when comparing pangenome/core-genome predictions from multiple tools (mOTUpan, Roary, PPanGGOLiN) on the same genome sets, especially MAGs spanning a wide completeness range.

## When NOT to use

- When only one pangenome tool's core-genome estimate is needed — no cross-tool benchmarking is required
- When genomes span such broad phylogenetic distances that a 95% ANI species cutoff cannot form coherent mOTUs (the method is for closely related genome sets)
- When GFF annotations are unavailable, since Roary and PPanGGOLiN require GFF-annotated genomes

## Inputs

- Genome nucleotide FASTA files (.fna) for 11570 GTDB species genomes and 8955 StratFreshDB mOTU genomes
- GFF-annotated genome files for Roary and PPanGGOLiN runs
- Proteome files (.faa) for mOTUpan.py
- CheckM completeness/contamination table for the --checkm completeness prior
- Roary gene-presence output and PPanGGOLiN gene-cluster output

## Outputs

- Species-level mOTU assignments (mOTUlize.py output table)
- mOTUpan core/accessory gene-cluster partitionings per genome set
- Converted genome-to-trait files from mOTUconvert.py for Roary and PPanGGOLiN clusters
- Comparison table of normalized residues across mOTUpan, Roary and PPanGGOLiN (core_genome_comparison.tsv)

## How to apply

Run mOTUlize.py on nucleotide FASTAs (--fnas) of the GTDB (11570 genomes) and StratFreshDB (8955 genomes) sets with fastANI at the default 95% ANI cutoff to form species-level mOTUs. Run mOTUpan.py per genome set with --faas *.faa and a --checkm completeness file, letting mmseqs2 easy_cluster build gene clusters and iterating the Bayesian core/accessory partitioning until convergence. In parallel, run Roary v3.13.0 and PPanGGOLiN v1.1.96 on the GFF-annotated genomes. Convert Roary and PPanGGOLiN outputs with mOTUconvert.py (--in_type roary and --in_type PPanGGOLiN) into genome-to-trait files usable with mOTUpan's --cog_file option, then recompute core-genome partitionings with mOTUpan.py --cog_file using the converted definitions. Compute normalized residues comparing the three tools' core-genome predictions per genome set; the rationale is that recomputing with --cog_file isolates the effect of the gene-cluster definitions from the Bayesian core estimator itself.

## Related tools

- **mOTUpan** (Bayesian core-genome estimator run with --faas and, for benchmarking, re-run with --cog_file on converted gene clusters) — github.com/moritzbuck/mOTUlizer
- **mOTUlize.py** (Clusters genomes into species-level mOTUs using fastANI similarities at the 95% ANI cutoff) — github.com/moritzbuck/mOTUlizer
- **fastANI** (Computes pairwise ANI similarities for mOTU-network generation (default similarity-cutoff 95)) — https://github.com/ParBLiSS/FastANI
- **mOTUpan.py** (Computes core/accessory partitioning with mmseqs2 easy_cluster gene clusters or external clusters via --cog_file, iterating the Bayesian likelihood until convergence) — github.com/moritzbuck/mOTUlizer
- **mmseqs2** (Builds the gene clusters used by mOTUpan.py via easy_cluster) — https://github.com/soedinglab/MMseqs2
- **Roary** (State-of-the-art pangenome tool benchmarked against mOTUpan on GFF-annotated genomes (v3.13.0)) — https://sanger-pathogens.github.io/Roary/
- **PPanGGOLiN** (State-of-the-art pangenome tool benchmarked against mOTUpan on GFF-annotated genomes (v1.1.96)) — https://github.com/labgem/PPanGGOLiN
- **mOTUconvert.py** (Converts Roary and PPanGGOLiN outputs into mOTUpan --cog_file-compatible genome-to-trait files) — github.com/moritzbuck/mOTUlizer
- **CheckM** (Supplies the completeness/contamination prior consumed via --checkm) — https://ecogenomics.github.io/CheckM/

## Examples

```
mOTUpan.py --cog_file ppanggolin_converted.tsv --checkm checkm_output.tsv -o core_genome_comparison.tsv
```

## Evaluation signals

- Normalized residues of mOTUpan's core-genome predictions are comparable to PPanGGOLiN's on the more complete GTDB genome sets
- All three tools' outputs are present as tabular files (converted genome-to-trait tables plus core_genome_comparison.tsv)
- mOTUpan's core estimates remain stable as genome completeness decreases, whereas Roary's core-genome estimates drop away from mOTUpan's
- The mOTUlize.py output yields species-level mOTUs consistent with the default 95% ANI cutoff
- The Bayesian core/accessory partitioning converges within --max_iter iterations on each genome set

## Limitations

- Roary's core-genome estimates degrade away from mOTUpan's as genome completeness decreases, so Roary comparisons are only meaningful for high-quality genomes
- fastANI is memory greedy and slow on large genome sets; mOTUlize.py runs it in blocks, and --keep-simi-file can reuse the similarity file across cutoffs
- The similarity parser is fragile: it ignores any lines containing the word 'query' and assumes asymmetric similarities where both members must pass the threshold
- Confusingly, mOTUlizer (mOTUpan) is a different tool from the read-based mOTUs tool at motu-tool.org

## Evidence

- [abstract] comparable estimates: "It has shown comparable estimates to state-of-the-art tools Roary and PPanGGOLiN for high-quality genomes and is capable of using genomes at lower completeness thresholds."
- [methods] network cutoff: "`--similarity-cutoff`: similarity cutoff used to generate the mOTU network. By default 95, as in 95% ANI cutoff which has been recorded as a weirdly universal cutoff that separates species."
- [methods] conversion: "`mOTUconvert.py` converts the output of diverse programs into input files for `mOTUpan.py`"
- [methods] cog_file option: "`--cog_file` : can be used as an alternative to `--faas`, you can use it to provide you own gene-clusters (or other genetically encoded traits)."
- [methods] iterative convergence: "This new partitioning can be used to update our completeness prior, and recomputed iteratively until convergence."
- [readme] mOTUpan.py invocation: "``nmOTUpan.py --faas *.faa -o output.tsv
"
- [other] confusable tool names: "**DISCLAIMER, there is an other tool out there called mOTUs that creates OTU-tables directly from reads, if you are looking for that tool, this is the wrong page"
