---
name: host-proximity-graph-linkage
description: Use when binning metagenomic viral contigs from a Hi-C experiment where
  the Hi-C interaction graph alone is too sparse, and assembled prokaryotic contigs
  are available to infer virus-host proximity links.
when_to_use_negative:
- The assembly contains no prokaryotic contigs or the dataset provides no basis for
  virus-host proximity inference (host proximity edges cannot be formed)
- The task is assessing vMAG completeness/contamination with CheckV rather than binning
  — this skill ends at draft bins and does not include quality assessment
- Only shotgun co-abundance data are available without Hi-C alignments — the proximity-ligation
  BAM is required input
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0114
tools:
- name: ViralCC
  role: Constructs the host proximity graph (--min-k), integrates it with the Hi-C
    interaction graph into G_int, and runs the pipeline end-to-end
  repo: https://github.com/dyxstat/ViralCC
- name: leidenalg
  role: Leiden graph clustering of the integrative graph G_int with resolution sweep
    and random seed 42
- name: scikit-learn
  role: Computes the silhouette score used to select the Leiden resolution parameter
    under Reichardt-Bornholdt modularity
- name: samtools
  role: Filters and name-sorts the Hi-C alignment BAM ('samtools view -F 0x904', 'samtools
    sort') prior to graph construction
provenance:
  source_task_ids:
  - task_002
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-35945-y
    title: ViralCC retrieves complete viral genomes and virus-host pairs from metagenomic
      Hi-C data
  source_content_hash: sha256:4c46a61d0f48b71f380e6bde49ed69e6857bf93df5078741fc6c0baf0d481997
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dyxstat/ViralCC
  tools:
  - ViralCC
  - leidenalg
  - samtools
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0114
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.46153846153846156
n_claims: 13
---

# host-proximity-graph-linkage

## Summary

A ViralCC skill that builds a host proximity graph linking viral contigs to prokaryotic contigs, providing complementary information to the sparse Hi-C interaction graph before Leiden clustering generates draft viral genomes (vMAGs).

## When to use

Use when binning metagenomic viral contigs from a Hi-C experiment where the Hi-C interaction graph alone is too sparse, and assembled prokaryotic contigs are available to infer virus-host proximity links.

## When NOT to use

- The assembly contains no prokaryotic contigs or the dataset provides no basis for virus-host proximity inference (host proximity edges cannot be formed)
- The task is assessing vMAG completeness/contamination with CheckV rather than binning — this skill ends at draft bins and does not include quality assessment
- Only shotgun co-abundance data are available without Hi-C alignments — the proximity-ligation BAM is required input

## Inputs

- Assembled contigs FASTA (e.g., final.contigs.fa, megahit --min-contig-len 1000)
- Name-sorted Hi-C alignment BAM filtered with samtools view -F 0x904
- Viral contig list (e.g., viral_contigs.txt from VirSorter-based identification)
- Prokaryotic contig set from the same assembly

## Outputs

- Host proximity graph edges integrated into the unified graph G_int
- cluster_viral_contig.txt (2 columns: viral contig name, group number; no header)
- VIRAL_BIN folder with draft viral bin fasta files (VIRAL_BIN0000.fa, VIRAL_BIN0001.fa, ...)
- viral_contig_info.csv and prokaryotic_contig_info.csv (contig name, length, GC-content)
- viralcc.log

## How to apply

From the name-sorted Hi-C alignment BAM (filtered with 'samtools view -F 0x904' to remove unmapped, supplementary, and secondary alignments) and the assembled contigs FASTA, construct the host proximity graph using ViralCC's --min-k parameter (lower bound of k for host proximity links; default 4, with k=30 used in the mock human gut ablation) to connect viral and prokaryotic contigs. Alignments used for graph construction pass --min-mapq 30 and --min-match 30 thresholds. Integrate G_host with the Hi-C interaction graph G_hic into the unified graph G_int, then run Leiden clustering (leidenalg, random seed 42), sweeping the resolution parameter r to maximize the silhouette score (scikit-learn) under Reichardt-Bornholdt modularity. The rationale is that virus-host proximity structure compensates for the remarkably sparse Hi-C interaction map among viral contigs; on the mock human gut dataset the integrated graph (4397 edges vs 2698 host-proximity and 2699 Hi-C edges) recovered 26 near-complete vMAGs versus 12 (host-only) and 8 (Hi-C-only).

## Related tools

- **ViralCC** (Constructs the host proximity graph (--min-k), integrates it with the Hi-C interaction graph into G_int, and runs the pipeline end-to-end) — https://github.com/dyxstat/ViralCC
- **leidenalg** (Leiden graph clustering of the integrative graph G_int with resolution sweep and random seed 42)
- **scikit-learn** (Computes the silhouette score used to select the Leiden resolution parameter under Reichardt-Bornholdt modularity)
- **samtools** (Filters and name-sorts the Hi-C alignment BAM ('samtools view -F 0x904', 'samtools sort') prior to graph construction)

## Examples

```
python ./viralcc.py pipeline final.contigs.fa MAP_SORTED.bam viral_contigs.txt OUTPUT_directory
```

## Evaluation signals

- Reproduce the reported mock human gut graph edge counts: 2699 Hi-C interaction edges, 2698 host proximity edges, 4397 integrated edges
- Confirm the clustering outcome on G_int recovers 26 near-complete, 2 substantially complete, and 4 moderately complete vMAGs, outperforming G_hic-only (8/3/5) and G_host-only (12/2/0)
- Check that cluster_viral_contig.txt exists with exactly 2 columns (contig name, group number) and no header
- Verify the VIRAL_BIN directory contains the expected draft viral bin fasta files matching the group numbers
- Re-run with --random-seed 42 fixed and confirm the clustering partition is reproducible

## Limitations

- The fraction of spurious contacts in the host proximity graph cannot be estimated on real datasets because true labels of viral contigs are unknown
- Some proximity-ligation virus-bacteria associations may arise from mechanisms other than infection and should be validated before drawing conclusions
- Whether existing binning methods can resolve closely related viruses residing in the same bacterial host remains unexplored

## Evidence

- [abstract] ViralCC leverages the virus-host proximity structure as a complementary information source for the Hi-C interactions.: "ViralCC leverages the virus-host proximity structure as a complementary information source for the Hi-C interactions."
- [intro] ViralCC constructs the Hi-C interaction graph and the host proximity graph: "ViralCC constructs the Hi-C interaction graph and the host proximity graph"
- [readme] README overview on host proximity graph: "ViralCC not only considers the Hi-C interaction graph, but also puts forward a novel host proximity graph of viral contigs"
- [methods] samtools filtering for graph construction inputs: "samtools with parameters ‘view -F 0x904’ is applied to remove unmapped reads, supplementary alignments, and secondary alignments"
