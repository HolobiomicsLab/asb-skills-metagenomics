---
name: popani-strain-delineation
description: Use when pairwise isolate genomes or metagenomic assemblies of the same
  species must be classified as same vs. different strain, especially when structural
  variation may be missed by SNP-only comparisons.
when_to_use_negative:
- When only a continuous measure of genomic difference is needed — the authors recommend
  avoiding same/different strain classifications in favor of continuous measures because
  thresholds are subjective and species-dependent
- When the target sequences are phages, plasmids, or other low-data contexts where
  SNP-based tools like inStrain underperform relative to SynTracker alone (SynTracker
  achieved a significantly higher AUC on phages and plasmids)
- When pairs have fewer than n available homologous regions — SynTracker excludes
  such pairs from downstream analysis
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: SynTracker
  role: Computes average pairwise synteny scores (APSS) between genome pairs via BLAST
    and synteny-block alignment to capture structural differences
  repo: https://github.com/leylabmpi/SynTracker
- name: inStrain
  role: Computes popANI values for every isolate/library pair to quantify point-mutation
    (SNP) based similarity
- name: BLAST
  role: BLAST+ database search of reference-derived 1-kbp central regions against
    target genomes (minimal identity 97%, coverage 70%) within SynTracker
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41587-024-02276-2
    title: Strain tracking in complex microbiomes using synteny analysis reveals per-species
      modes of evolution
  source_content_hash: sha256:691018f76b7f4487078517ed47d556eee258d1c8a7414b57fa01dbc1d287fafa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/leylabmpi/SynTracker
  tools:
  - BLAST
  - SynTracker
  - inStrain
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# popani-strain-delineation

## Summary

Delineate whether two conspecific genomes/metagenome assemblies represent the same strain by combining an SNP-based popANI metric (inStrain) with synteny-based APSS scores (SynTracker), so that strains enriched in point mutations versus structural changes are distinguishable.

## When to use

Use when pairwise isolate genomes or metagenomic assemblies of the same species must be classified as same vs. different strain, especially when structural variation may be missed by SNP-only comparisons.

## When NOT to use

- When only a continuous measure of genomic difference is needed — the authors recommend avoiding same/different strain classifications in favor of continuous measures because thresholds are subjective and species-dependent
- When the target sequences are phages, plasmids, or other low-data contexts where SNP-based tools like inStrain underperform relative to SynTracker alone (SynTracker achieved a significantly higher AUC on phages and plasmids)
- When pairs have fewer than n available homologous regions — SynTracker excludes such pairs from downstream analysis

## Inputs

- Per-isolate genome FASTA files or per-sample metagenomic assemblies, one file per sample, in a single target directory
- Per-species complete reference genomes (e.g., NC_000913.3 E. coli, CP032479.1 H. pylori, GCF_000331185.2 S. rimosus, GCF_900087635.2 N. gonorrhoeae)
- Sequencing reads from NCBI SRA for inStrain popANI computation (Supplementary Tables 2-5)

## Outputs

- Per-pair popANI values from inStrain
- Per-pair APSS values from SynTracker 'avg_synteny_scores_all_regions' tables (and subsampled tables at 40/60/80/100/200 regions)
- Four same/different-strain classification tables (e.g., 66 N. gonorrhoeae, 185 hypermutator E. coli, H. pylori within-host, S. rimosus library pairs) corresponding to Fig. 3a-d

## How to apply

Run inStrain on every genome/library pair from the SRA collections (Supplementary Tables 2-5) to compute popANI, and run SynTracker (BLAST identity ≥97%, coverage ≥70%, compared region length 5000, seed=1 for reproducible subsampling) against the appropriate per-species reference (e.g., GCF_900087635.2 for N. gonorrhoeae, NC_000913.3 for E. coli, CP032479.1 for H. pylori, GCF_000331185.2 for S. rimosus). Extract per-pair APSS values from the SynTracker 'avg_synteny_scores_all_regions' tables, using up to 200 regions per pair, which yields maximal sensitivity (98.6%) and specificity (97%). Apply the same-strain cutoffs APSS > 0.955 and popANI > 0.99999 jointly: concordant calls indicate clonal or divergent strains, while discordance (clonal popANI but wide APSS range, as seen across 185 S. rimosus pairwise comparisons) signals structural rather than point-mutation evolution. Note that strain delineation thresholds are subjective and species-dependent; the human-gut APSS 0.95 cutoff was an average of per-species cutoffs and may not be optimal for every setting.

## Related tools

- **SynTracker** (Computes average pairwise synteny scores (APSS) between genome pairs via BLAST and synteny-block alignment to capture structural differences) — https://github.com/leylabmpi/SynTracker
- **inStrain** (Computes popANI values for every isolate/library pair to quantify point-mutation (SNP) based similarity)
- **BLAST** (BLAST+ database search of reference-derived 1-kbp central regions against target genomes (minimal identity 97%, coverage 70%) within SynTracker)

## Examples

```
python syntracker.py -target isolate_genomes_dir/ -ref reference_dir/ -out SynTracker_output/
```

## Evaluation signals

- Each species classification table exists and contains all pairwise comparisons (66 rows for N. gonorrhoeae, 185 for hypermutator E. coli)
- Spearman rho = 0.985 between APSS and popANI over the 66 N. gonorrhoeae comparisons
- All 185 hypermutator E. coli pairs are called same strain by both tools (APSS > 0.955 and popANI > 0.99999)
- S. rimosus pairs show clonal popANI but a wide APSS range, confirming structural-difference-driven variation
- SynTracker run uses seed=1 (no --no_seed flag) so APSS values are reproducible between runs

## Limitations

- Strain delineation thresholds (inStrain popANI 99.999%, APSS 0.95) are subjective, depend on the species, and the 0.95 cutoff was an average of per-species cutoffs that may not be optimal for all settings
- Small numbers of regions per pairwise comparison mostly result in lower accuracy; a trade-off exists between accuracy and sample size
- SynTracker's seed-based subsampling makes APSS values change between runs if --no_seed is used
- APSS values reflect synteny and are insensitive to SNP-level differences, so the two tools must be interpreted jointly; discordant classifications reveal the mode of evolution rather than tool error

## Evidence

- [methods] apply the same-strain cutoffs APSS > 0.955 and popANI > 0.99999: [evidence span withheld — anchor status: not_found]
- [discussion] maximal sensitivity and specificity with 200 regions: "maximal average sensitivity and specificity values of 98.6% and 97% for comparisons calculated using 200 regions per pairwise comparison"
- [discussion] S. rimosus structural differences with low point mutations: "S. rimosus, in which we observed a high level of structural differences with a low level of point mutations"
- [discussion] APSS 0.95 cutoff is an averaged, species-dependent threshold: "this APSS cutoff was based on averaging different per-species cutoffs and may not be optimal for all settings."
- [readme] SynTracker run command with default parameters: "python syntracker.py -target Sample_Data/Input_example/Target_genomes/ -ref Sample_Data/Input_example/Reference_genomes/ -out SynTracker_output/"
