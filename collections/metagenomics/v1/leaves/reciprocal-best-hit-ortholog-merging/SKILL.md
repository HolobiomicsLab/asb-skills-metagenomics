---
name: reciprocal-best-hit-ortholog-merging
description: Use when a non-redundant gene/protein catalog needs to be grouped into
  orthologous families using Jaccard similarity clustering followed by reciprocal--hit
  merging of clusters.
when_to_use_negative:
- Input is raw nucleotide reads or unfiltered FASTQ rather than a non-redundant polypeptide
  set — BLASTP clustering requires translated proteins
- The ortholog families already exist (e.g. eggNOG/COG/KOfam annotation is sufficient)
  and no vaginal-specific protein family catalog is needed
- Sequences are primarily viral or fungal — the VIRGO/VOG database has limited inclusion
  of viral and fungal gene sequences
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Wu-blastp
  role: Performs the all-versus-all BLASTP whose BSML outputs and filtered hit pairs
    (80% identity, 70% coverage, E-value < 1e-10) feed the Jaccard clustering
  repo: github.com/ravel-lab/VIRGO
- name: clusterBsmlPairwiseAlignments-mod.pl
  role: Modified script that computes modified Jaccard similarity clusters (linkscore=0.6)
    from the filtered polypeptide pair list
  repo: github.com/ravel-lab/VIRGO
- name: CogBsmlLoader-mod.pl
  role: Converts Wu-blastp BSML results into the All-best-hits.btab best-hit summary
    with Jaccard cluster info appended
  repo: github.com/ravel-lab/VIRGO
- name: best_hit.pl
  role: Runs bidirectional best hit analysis and merges Jaccard clusters with reciprocal
    best hits into final-clusters.cog.gz
  repo: github.com/ravel-lab/VIRGO
- name: BLAST
  role: Underlying sequence search engine; VIRGO is also blastable in nucleotide and
    amino acid (blast 2.2.3+ compatible)
  repo: github.com/ravel-lab/VIRGO
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-14677-3
    title: A comprehensive non-redundant gene catalog reveals extensive within-community
      intraspecies diversity in the human vagina
  source_content_hash: sha256:f9d92aca80afed2a82f889c18b4393ca6b2775c9e97a6bf10c2308361964d519
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLAST
  - CogBsmlLoader-mod.pl
  - VIRGO
  - Wu-blastp
  - best_hit.pl
  - clusterBsmlPairwiseAlignments-mod.pl
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# reciprocal-best-hit-ortholog-merging

## Summary

Constructs Vaginal Orthologous Groups (VOGs) by merging modified-Jaccard protein clusters (JACs) into JOCs via bidirectional best hit analysis of Wu-blastp all-vs-all hits. Used in VIRGO (0.95M non-redundant vaginal genes) to yield 617,127 JACs and 552,679 JOCs, of which 177,684 contain ≥2 genes and 374,995 are singletons.

## When to use

Use when a non-redundant gene/protein catalog needs to be grouped into orthologous families using Jaccard similarity clustering followed by reciprocal-best-hit merging of clusters.

## When NOT to use

- Input is raw nucleotide reads or unfiltered FASTQ rather than a non-redundant polypeptide set — BLASTP clustering requires translated proteins
- The ortholog families already exist (e.g. eggNOG/COG/KOfam annotation is sufficient) and no vaginal-specific protein family catalog is needed
- Sequences are primarily viral or fungal — the VIRGO/VOG database has limited inclusion of viral and fungal gene sequences

## Inputs

- VIRGO non-redundant polypeptide (amino acid) FASTA set from the 0.95 million non-redundant gene catalog
- Wu-blastp.bsml.list — list of all-vs-all BLASTP BSML result files
- Filtered polypeptide pair list (20808.pairs.gz) at 80% identity, 70% coverage, E-value < 1e-10

## Outputs

- Jaccard-clusters.out — modified Jaccard clusters (JACs, linkscore=0.6)
- All-best-hits.btab — best-hit summary btab with Jaccard cluster info in the last column
- final-clusters.cog.gz — merged VOG clusters (JOCs) with 177,684 multi-gene clusters and 374,995 singletons

## How to apply

First run all-versus-all BLASTP on the VIRGO non-redundant polypeptide set with Wu-blastp, filtering hits at 80% percent identity, 70% percent coverage overlap, and E-value < 1e-10 to produce a filtered polypeptide pair list (20808.pairs.gz). Next compute modified Jaccard similarity clusters with clusterBsmlPairwiseAlignments-mod.pl using --linkscore=0.6 (with percent_identity=80, percent_coverage=60, p_value=1e-5), writing clusters to Jaccard-clusters.out. Convert the Wu-blastp BSML results to a summary btab of best hits with CogBsmlLoader-mod.pl, producing All-best-hits.btab with Jaccard cluster info in the last column. Finally run bidirectional best hit analysis (Step 3) and merge all clusters with reciprocal best hits using best_hit.pl to produce final-clusters.cog.gz; validate the cluster counts against 617,127 JACs and 552,679 JOCs.

## Related tools

- **Wu-blastp** (Performs the all-versus-all BLASTP whose BSML outputs and filtered hit pairs (80% identity, 70% coverage, E-value < 1e-10) feed the Jaccard clustering) — github.com/ravel-lab/VIRGO
- **clusterBsmlPairwiseAlignments-mod.pl** (Modified script that computes modified Jaccard similarity clusters (linkscore=0.6) from the filtered polypeptide pair list) — github.com/ravel-lab/VIRGO
- **CogBsmlLoader-mod.pl** (Converts Wu-blastp BSML results into the All-best-hits.btab best-hit summary with Jaccard cluster info appended) — github.com/ravel-lab/VIRGO
- **best_hit.pl** (Runs bidirectional best hit analysis and merges Jaccard clusters with reciprocal best hits into final-clusters.cog.gz) — github.com/ravel-lab/VIRGO
- **BLAST** (Underlying sequence search engine; VIRGO is also blastable in nucleotide and amino acid (blast 2.2.3+ compatible)) — github.com/ravel-lab/VIRGO

## Examples

```
./run-jaccard.sh
```

## Evaluation signals

- Final cluster tables contain exactly 617,127 JACs and 552,679 JOCs
- JOC breakdown yields 177,684 multi-gene clusters and 374,995 singletons (~38.5% of VOG proteins unique)
- All-best-hits.btab has Jaccard cluster information in the last column and best hits are bidirectional (reciprocal)
- Jaccard clustering was run with --linkscore=0.6 and BLASTP hits filtered at 80% identity, 70% coverage, E-value < 1e-10
- 5_VOG linker file 0.VOG.VIRGO.tbl.txt maps every VOG id back to VIRGO gene ids consistently

## Limitations

- The database is primarily focused on bacteria with limited inclusion of viral and fungal gene sequences
- This version is focused on gene-level de-redundancy; limitations of the reference-based approach of VIRGO are acknowledged
- Results depend on BLASTP filtering thresholds (80% identity, 70% coverage, 1e-10) and the linkscore=0.6 cutoff; adjusting these changes cluster granularity

## Evidence

- [readme] VOG is a comprehensive collection of orthlogous genes for the vaginal microbiota as defined by a novel jaccard clustering method.: "VOG is a comprehensive collection of orthlogous genes for the vaginal microbiota as defined by a novel jaccard clustering method."
- [readme] 4_Jaccard_index_clustering contains the script used to perform the jaccard clustering to identify VOGs: "4_Jaccard_index_clustering contains the script used to perform the jaccard clustering to identify VOGs"
- [methods] Run Jaccard clustering using modified clusterBsmlPairwiseAlignments script: "Run Jaccard clustering using modified clusterBsmlPairwiseAlignments script"
- [methods] 3b. Merge all clusters with reciprocal best hits.: "3b. Merge all clusters with reciprocal best hits."
- [intro] The genes identified in these data were further clustered into Vaginal Orthologous Groups (VOGs): "The genes identified in these data were further clustered into Vaginal Orthologous Groups (VOGs)"
