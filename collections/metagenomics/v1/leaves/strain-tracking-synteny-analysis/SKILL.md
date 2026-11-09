---
name: strain-tracking-synteny-analysis
description: Use when comparing conspecific microbial strains from metagenomic assemblies
  or assembled genomes where SNP-based tools are insufficient (e.g., structural variation,
  phages, plasmids) and a per-region synteny-based relatedness score is needed.
when_to_use_negative:
- When the research question is only about point mutations / SNP differences and no
  structural variation is of interest — the paper notes most SNP-quantifying methods
  are insensitive to structural changes, but SynTracker's strength is synteny, not
  SNP calling
- When strains are too divergent for BLAST hits to pass the 97% identity / 70% coverage
  thresholds on 5-kbp regions (default -length 5000), since homologous regions will
  not be recovered
- When fewer than n homologous regions per sample pair exist (e.g., small phages/plasmids
  relative to the subsampling depth) — such pairs are excluded from downstream analysis
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: SynTracker
  role: 'Main pipeline: fragments reference genomes, runs BLASTn against targets,
    computes per-region synteny scores and subsampled APSS values'
  repo: https://github.com/leylabmpi/SynTracker
- name: BLAST
  role: BLASTn search of 1-kbp central reference regions against a database of target
    metagenomic assemblies/genomes with identity >= 97% and coverage >= 70%; 'blastcmddb'
    retrieves 2-kbp flanking regions
- name: DECIPHER R package
  role: FindSynteny function performs all-versus-all pairwise alignments within each
    region bin (maxGap=15, maxSep=15) to identify syntenic blocks underlying each
    synteny score
- name: phangorn R package
  role: Builds UPGMA phylogenetic trees from synteny distances derived from APSS values
- name: TreeDist R package
  role: Computes Information-based generalized Robinson-Foulds distances between synteny-based
    and reference trees with label shuffling (100,000 iterations) for empirical P-values
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_005
  source_papers:
  - doi: 10.1038/s41587-024-02276-2
    title: Strain tracking in complex microbiomes using synteny analysis reveals per-species
      modes of evolution
  source_content_hash: sha256:0303634cb3b5b6d5a64266f7fdc0a62e6041cbfbab0413c370bd0c90bc8bbf44
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leylabmpi/SynTracker
  tools:
  - BLAST
  - DECIPHER R package
  - SynTracker
  - TreeDist R package
  - phangorn R package
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# strain-tracking-synteny-analysis

## Summary

Track closely related microbial strains by comparing genome synteny (the order of sequence blocks in homologous genomic regions) between pairs of metagenomic assemblies or genomes using SynTracker, which converts region-level synteny scores into an average pairwise synteny score (APSS) for phylogenetic and strain-delineation analyses.

## When to use

Use when comparing conspecific microbial strains from metagenomic assemblies or assembled genomes where SNP-based tools are insufficient (e.g., structural variation, phages, plasmids) and a per-region synteny-based relatedness score is needed.

## When NOT to use

- When the research question is only about point mutations / SNP differences and no structural variation is of interest — the paper notes most SNP-quantifying methods are insensitive to structural changes, but SynTracker's strength is synteny, not SNP calling
- When strains are too divergent for BLAST hits to pass the 97% identity / 70% coverage thresholds on 5-kbp regions (default -length 5000), since homologous regions will not be recovered
- When fewer than n homologous regions per sample pair exist (e.g., small phages/plasmids relative to the subsampling depth) — such pairs are excluded from downstream analysis

## Inputs

- Reference genome FASTA files (complete genomes or contigs of the same genome in a single .fasta; one directory of references per run)
- Per-sample metagenomic assembly FASTA files, or one FASTA per target genome, all in a single 'target directory'
- Optional: pre-built BLAST database directory produced by syntracker_makeDB.py

## Outputs

- Raw per-region pairwise synteny score table: synteny_scores_per_region.csv (per genome and combined summary_output/)
- APSS matrices at each subsampling length: avg_synteny_scores_[40|60|80|100|200]_regions.csv
- APSS over all regions without subsampling: avg_synteny_scores_all_regions.csv
- Per-bin, per-pair synteny score/block-count/overlap TSV tables within [genome_name]/final_output/

## How to apply

Fragment each per-species reference genome into 1-kbp central regions and run BLASTn (minimal identity 97%, minimal query coverage 70%) of each central region against a BLAST database built from the target metagenomic assemblies or genomes. For each hit, retrieve 2 kbp on each side with 'blastcmddb', excluding hits lacking sufficient flanking sequence. Within each region bin, run all-versus-all pairwise alignment using the FindSynteny function in the DECIPHER R package (maxGap and maxSep both set to 15), retain only pairwise comparisons with a minimal overlap of 4,800 bp, and compute the synteny score synScore = 1 + log10((Ov/len)/B), which is inversely proportional to the number of syntenic blocks B and directly proportional to the sequence overlap. Then randomly subsample n genomic regions per pair of samples (default n = 40, 60, 80, 100, 200; seed=1 for reproducibility unless --no_seed) and average the individual pairwise synteny scores into the APSS, excluding pairs with fewer than n regions.

## Related tools

- **SynTracker** (Main pipeline: fragments reference genomes, runs BLASTn against targets, computes per-region synteny scores and subsampled APSS values) — https://github.com/leylabmpi/SynTracker
- **BLAST** (BLASTn search of 1-kbp central reference regions against a database of target metagenomic assemblies/genomes with identity >= 97% and coverage >= 70%; 'blastcmddb' retrieves 2-kbp flanking regions)
- **DECIPHER R package** (FindSynteny function performs all-versus-all pairwise alignments within each region bin (maxGap=15, maxSep=15) to identify syntenic blocks underlying each synteny score)
- **phangorn R package** (Builds UPGMA phylogenetic trees from synteny distances derived from APSS values)
- **TreeDist R package** (Computes Information-based generalized Robinson-Foulds distances between synteny-based and reference trees with label shuffling (100,000 iterations) for empirical P-values)

## Examples

```
python syntracker.py -target Sample_Data/Input_example/Target_genomes/ -ref Sample_Data/Input_example/Reference_genomes/ -out SynTracker_output/
```

## Evaluation signals

- All synteny scores are <= 1 (score of 1 corresponds to a single syntenic block with 100% overlap); APSS values lie in [0, 1]
- Output TSV/CSV files contain the expected score/block-count/overlap columns for every region bin and sample pair, and APSS tables exist for each default subsampling length (40, 60, 80, 100, 200) plus all_regions
- APSS-based UPGMA trees recapitulate an independently derived phylogeny: e.g., Information-based generalized Robinson-Foulds distances against a pruned Mash tree yield P < 1e-5 under 100,000 label shuffles, even with < 2% of the genome sampled
- Pairs with fewer than n available regions are excluded from the APSS matrix at the corresponding subsampling length
- Runs are reproducible: with default seed=1, repeated runs yield identical APSS values (unless --no_seed is set, in which case subsampling varies)

## Limitations

- Strain delineation thresholds (e.g., the APSS 0.95 cutoff for designating the same strain in the human gut) are subjective and species-dependent; the 0.95 cutoff was based on averaging per-species cutoffs and may not be optimal for all settings
- Users must set their own cutoffs considering evolvability, selection, and their interpretation of 'strain'; tailored cutoffs require a training dataset relevant to the research question
- A small number of regions per pairwise comparison mostly results in lower accuracy; a trade-off exists between accuracy and the number of regions sampled
- SynTracker is designed for conspecific (same-species) strain comparison and requires user-provided reference genomes; no database requirement, but it cannot classify strains without a suitable reference

## Evidence

- [abstract] SynTracker, a tool that compares microbial strains using genome synteny-the order of sequence blocks in homologous genomic regions-in pairs of metagenomic assemblies or genomes.: "We introduce SynTracker, a tool that compares microbial strains using genome synteny-the order of sequence blocks in homologous genomic regions-in pairs of metagenomic assemblies or genomes."
- [discussion] BLAST search parameters and flanking retrieval: "performs a BLAST search, for each of the central regions, against the newly created BLAST database with a minimal identity of 97% and a minimal query coverage of 70%"
- [discussion] FindSynteny parameters and minimal overlap threshold: "using the ‘FindSynteny’ function in the DECIPHER R package21, with parameters ‘maxGap’ and ‘maxSep’ both set to 15"
- [discussion] APSS calculation by subsampling: "n genomic regions are randomly selected per pair of metagenomic samples or genomes. The APSS is calculated by averaging the individual pairwise synteny scores"
- [readme] Default subsampling lengths and seed: "By default, N equals to 40, 60, 80, 100, 200 regions per pair of samples."
- [discussion] APSS cutoff for strain designation: "we set an APSS score cutoff of 0.95 to designate the same strain in the human gut"
