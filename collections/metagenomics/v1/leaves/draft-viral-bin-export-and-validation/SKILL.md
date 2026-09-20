---
name: draft-viral-bin-export-and-validation
description: Use when ViralCC's inputs (final.contigs.fa, name-sorted MAP_SORTED.bam,
  viral_contigs.txt) are ready and you must cluster G_int with Leiden to produce draft
  vMAG bins and validate the exported clustering files against the Test dataset reference.
when_to_use_negative:
- The BAM file is not sorted by name — ViralCC requires the Hi-C alignment BAM name-sorted
  before its pipeline stage runs.
- The goal is taxonomic classification of viral contigs (e.g. to order/family level)
  rather than reconstruction of draft viral genome bins from Hi-C data.
- Viral contigs have not yet been identified from the assembled contigs, since the
  clustering stage operates only on the supplied viral contig list.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: ViralCC
  role: Implements the graph integration (G_host + G_hic -> G_int), Leiden clustering,
    and export of draft viral bins (cluster_viral_contig.txt, VIRAL_BIN folder, info
    CSVs, viralcc.log)
  repo: https://github.com/dyxstat/ViralCC
- name: leidenalg
  role: Leiden graph clustering of G_int, sweeping resolution r under Reichardt-Bornholdt
    modularity
- name: scikit-learn
  role: Computes the silhouette score used to select the optimal Leiden resolution
    parameter r
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-35945-y
    title: ViralCC retrieves complete viral genomes and virus-host pairs from metagenomic
      Hi-C data
  source_content_hash: sha256:32fc06265d6d4e0d5a827f58ceb3f85235aafbcf6950832a33d29bb72a50b4a5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dyxstat/ViralCC
  tools:
  - ViralCC
  - leidenalg
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# draft-viral-bin-export-and-validation

## Summary

Runs the final stage of the ViralCC pipeline, in which the integrative graph G_int (Hi-C interaction graph plus host proximity graph) is clustered with the Leiden algorithm and the resulting viral contig groups are exported as draft viral genome bins (cluster_viral_contig.txt and VIRAL_BIN*.fa) with auxiliary tables, then validated against the reference Test/out_test output.

## When to use

Use when ViralCC's inputs (final.contigs.fa, name-sorted MAP_SORTED.bam, viral_contigs.txt) are ready and you must cluster G_int with Leiden to produce draft vMAG bins and validate the exported clustering files against the Test dataset reference.

## When NOT to use

- The BAM file is not sorted by name — ViralCC requires the Hi-C alignment BAM name-sorted before its pipeline stage runs.
- The goal is taxonomic classification of viral contigs (e.g. to order/family level) rather than reconstruction of draft viral genome bins from Hi-C data.
- Viral contigs have not yet been identified from the assembled contigs, since the clustering stage operates only on the supplied viral contig list.

## Inputs

- assembled contigs FASTA (final.contigs.fa)
- name-sorted Hi-C alignment BAM (MAP_SORTED.bam, sorted by 'samtools sort' by name)
- viral contig list (viral_contigs.txt) from viral contig identification
- host proximity graph G_host (built with --min-k, default 4)
- Hi-C interaction graph G_hic (alignments filtered at --min-mapq 30, --min-match 30)

## Outputs

- cluster_viral_contig.txt (2 columns: viral contig name and group number, no header)
- VIRAL_BIN folder with draft viral bin FASTA files (VIRAL_BIN0000.fa, VIRAL_BIN0001.fa, ...)
- viral_contig_info.csv (contig name, length, GC-content)
- prokaryotic_contig_info.csv (contig name, length, GC-content)
- viralcc.log

## How to apply

Load the mock-dataset inputs — final.contigs.fa, the name-sorted Hi-C alignment MAP_SORTED.bam, and viral_contigs.txt — and build the host proximity graph with --min-k (default 4, lower bound of k). Construct the Hi-C interaction graph from alignments filtered at --min-mapq 30 and --min-match 30, then integrate G_host and G_hic into G_int. Run Leiden clustering (leidenalg) on G_int, sweeping resolution r and selecting the value maximizing the silhouette score (scikit-learn) under Reichardt-Bornholdt modularity, with --random-seed 42 fixed for reproducibility. Export clustering results as cluster_viral_contig.txt (2 columns: viral contig name and group number, no header), write each draft viral bin as VIRAL_BIN0000.fa, VIRAL_BIN0001.fa, ... in the VIRAL_BIN folder, and write viral_contig_info.csv and prokaryotic_contig_info.csv (contig name, length, GC-content) plus viralcc.log. On the mock human gut dataset this yields 26 near-complete, 2 substantially complete, and 4 moderately complete vMAGs.

## Related tools

- **ViralCC** (Implements the graph integration (G_host + G_hic -> G_int), Leiden clustering, and export of draft viral bins (cluster_viral_contig.txt, VIRAL_BIN folder, info CSVs, viralcc.log)) — https://github.com/dyxstat/ViralCC
- **leidenalg** (Leiden graph clustering of G_int, sweeping resolution r under Reichardt-Bornholdt modularity)
- **scikit-learn** (Computes the silhouette score used to select the optimal Leiden resolution parameter r)

## Examples

```
python ./viralcc.py pipeline [Parameters] FASTA_file BAM_file VIRAL_file OUTPUT_directory
```

## Evaluation signals

- cluster_viral_contig.txt exists with exactly 2 columns (viral contig name and group number) and no header row.
- The VIRAL_BIN directory contains the expected draft viral bin FASTA files following the VIRAL_BIN0000.fa, VIRAL_BIN0001.fa naming pattern.
- The reproduced clustering output on the Test dataset matches the reference output in Test/out_test.
- viral_contig_info.csv and prokaryotic_contig_info.csv are written with contig name, length, and GC-content columns.
- The run is reproducible with --random-seed fixed at 42 and the selected resolution maximizing silhouette score under Reichardt-Bornholdt modularity.

## Limitations

- CheckV is unable to estimate the contamination of vMAGs since there is no such marker gene set available for viruses, so completeness-based evaluation alone is not comprehensive; a random binning control model is necessary when using CheckV completeness as the evaluation metric.
- The benchmarking effectiveness is less convincing if there are few putative viral genomes, and only viral genomes recoverable by a single contig were chosen, underestimating true viral community diversity.
- Some proximity-ligation virus-bacteria associations might result from proximity of viral and bacterial DNA from mechanisms other than infection, so downstream virus-host conclusions from the recovered bins require validation (e.g. CRISPR spacer analyses).

## Evidence

- [intro] Leiden clustering reconstructs draft viral genomes: "After integrating two graphs, ViralCC employs Leiden clustering to reconstruct draft viral genomes"
- [methods] Run the ViralCC pipeline command: "python ./viralcc.py pipeline [Parameters] FASTA_file BAM_file VIRAL_file OUTPUT_directory"
- [readme] Integrative graph clustered by Leiden to generate draft viral genomes: "The two graphs are then integrated together, followed by the Leiden graph clustering using the integrative graph to generate draft viral genomes."
- [other] Exported clustering output and reference validation: [evidence span withheld — anchor status: not_found]
