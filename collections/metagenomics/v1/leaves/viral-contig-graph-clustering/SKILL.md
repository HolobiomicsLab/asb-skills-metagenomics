---
name: viral-contig-graph-clustering
description: Use when you have assembled contigs (FASTA), a name-sorted Hi-C alignment
  BAM, and a viral contig list, and need to bin viral contigs into draft genomes using
  Hi-C linkages plus virus-host proximity.
when_to_use_negative:
- No Hi-C (proximity-ligation) data are available — the Hi-C interaction graph is
  a core input, so co-abundance-only tools like CoCoNet or vRhyme may be more appropriate
- Viral contigs have not yet been identified — the pipeline requires a pre-computed
  viral contig list (VIRAL_file), not raw assemblies
- The BAM is not sorted by name — ViralCC requires name-sorted alignments before graph
  construction
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0194
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: ViralCC
  role: Pipeline that builds the host proximity graph (G_host) and Hi-C interaction
    graph (G_hic), integrates them into G_int, and runs Leiden clustering to produce
    draft viral bins
  repo: https://github.com/dyxstat/ViralCC
- name: leidenalg
  role: Python library implementing Leiden clustering on the integrative graph G_int
    with resolution parameter sweep
- name: scikit-learn
  role: Computes silhouette scores used to select the Leiden resolution r maximizing
    clustering quality
- name: samtools
  role: Sorts the Hi-C alignment BAM by name prior to graph construction
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-35945-y
    title: ViralCC retrieves complete viral genomes and virus-host pairs from metagenomic
      Hi-C data
  source_content_hash: sha256:818626bee3c0648312741f02849ce3a9b0572a140f94c998d4c0915e7204d9d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dyxstat/ViralCC
  tools:
  - ViralCC
  - leidenalg
  - samtools
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0194
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# viral-contig-graph-clustering

## Summary

Integrates ViralCC's Hi-C interaction graph and host proximity graph into a unified graph G_int and applies Leiden clustering to reconstruct draft viral genomes (vMAGs) from metagenomic Hi-C data.

## When to use

Use when you have assembled contigs (FASTA), a name-sorted Hi-C alignment BAM, and a viral contig list, and need to bin viral contigs into draft genomes using Hi-C linkages plus virus-host proximity.

## When NOT to use

- No Hi-C (proximity-ligation) data are available — the Hi-C interaction graph is a core input, so co-abundance-only tools like CoCoNet or vRhyme may be more appropriate
- Viral contigs have not yet been identified — the pipeline requires a pre-computed viral contig list (VIRAL_file), not raw assemblies
- The BAM is not sorted by name — ViralCC requires name-sorted alignments before graph construction

## Inputs

- Assembled contigs FASTA (e.g., final.contigs.fa)
- Name-sorted Hi-C alignment BAM (MAP_SORTED.bam, e.g., from samtools sort by name)
- Viral contig list (viral_contigs.txt) from viral contig identification (e.g., VirSorter)

## Outputs

- cluster_viral_contig.txt (2 columns: viral contig name and group number, no header)
- Draft viral bin FASTA files (VIRAL_BIN0000.fa, VIRAL_BIN0001.fa, ...) in the VIRAL_BIN folder
- viral_contig_info.csv and prokaryotic_contig_info.csv (contig name, length, GC-content)
- viralcc.log

## How to apply

Load the assembled contigs (final.contigs.fa), the name-sorted Hi-C alignment (MAP_SORTED.bam), and the viral contig list (viral_contigs.txt). Build the host proximity graph G_host with ViralCC's --min-k parameter (lower bound of k, default 4) and the Hi-C interaction graph G_hic from alignments filtered with --min-mapq (default 30) and --min-match (default 30). Integrate the two graphs into G_int, then run Leiden clustering (leidenalg) sweeping the resolution parameter r and selecting the value maximizing the silhouette score (scikit-learn) under Reichardt-Bornholdt modularity, fixing --random-seed at 42 for reproducibility. Export cluster_viral_contig.txt (viral contig name and group number, 2 columns, no header) and write each draft viral bin as a fasta (VIRAL_BIN0000.fa, ...) in the VIRAL_BIN folder.

## Related tools

- **ViralCC** (Pipeline that builds the host proximity graph (G_host) and Hi-C interaction graph (G_hic), integrates them into G_int, and runs Leiden clustering to produce draft viral bins) — https://github.com/dyxstat/ViralCC
- **leidenalg** (Python library implementing Leiden clustering on the integrative graph G_int with resolution parameter sweep)
- **scikit-learn** (Computes silhouette scores used to select the Leiden resolution r maximizing clustering quality)
- **samtools** (Sorts the Hi-C alignment BAM by name prior to graph construction)

## Examples

```
python ./viralcc.py pipeline --min-k 4 --min-mapq 30 --min-match 30 --random-seed 42 final.contigs.fa MAP_SORTED.bam viral_contigs.txt OUTPUT
```

## Evaluation signals

- Reproduced clustering on the Test dataset matches reference output in Test/out_test: cluster_viral_contig.txt exists with 2 columns of contig names and group numbers
- VIRAL_BIN directory contains the expected draft viral bin fasta files (VIRAL_BIN0000.fa, VIRAL_BIN0001.fa, ...)
- On the mock human gut dataset, Leiden clustering on G_int reconstructs 26 near-complete, 2 substantially complete, and 4 moderately complete vMAGs
- Auxiliary tables viral_contig_info.csv and prokaryotic_contig_info.csv contain contig name, length, and GC-content columns
- Selected Leiden resolution r maximizes silhouette score under Reichardt-Bornholdt modularity with --random-seed 42, so re-running reproduces identical bins

## Limitations

- The fraction of spurious contacts in the host proximity graph cannot be quantified on real datasets because true labels of viral contigs are unknown
- Whether the method resolves closely related viruses residing in the same bacterial host based on virus-host proximities remains unexplored
- Discovering co-infection of multiple phages within the same cell via Hi-C proximity ligation is left as future work

## Evidence

- [readme] The two graphs are then integrated together, followed by the Leiden graph clustering using the integrative graph to generate draft viral genomes.: "The two graphs are then integrated together, followed by the Leiden graph clustering using the integrative graph to generate draft viral genomes."
- [intro] ViralCC constructs the Hi-C interaction graph and the host proximity graph: "ViralCC constructs the Hi-C interaction graph and the host proximity graph"
- [intro] After integrating two graphs, ViralCC employs Leiden clustering to reconstruct draft viral genomes: "After integrating two graphs, ViralCC employs Leiden clustering to reconstruct draft viral genomes"
- [methods] --min-mapq: Minimum acceptable alignment quality (default 30): "--min-mapq: Minimum acceptable alignment quality (default 30)"
- [methods] python ./viralcc.py pipeline [Parameters] FASTA_file BAM_file VIRAL_file OUTPUT_directory: "python ./viralcc.py pipeline [Parameters] FASTA_file BAM_file VIRAL_file OUTPUT_directory"
