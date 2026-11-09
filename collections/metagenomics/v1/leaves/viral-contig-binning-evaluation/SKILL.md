---
name: viral-contig-binning-evaluation
description: Use when a metagenomic Hi-C dataset has assembled contigs, a name-sorted
  filtered BAM, and an identified viral contig list, and you need to bin viral contigs
  and quantify whether graph integration improves vMAG recovery.
when_to_use_negative:
- The input is shotgun-only data with no Hi-C proximity-ligation BAM, since the Hi-C
  interaction graph is the core input to ViralCC
- You need contamination estimates for vMAGs — CheckV cannot estimate contamination
  of vMAGs because no viral marker gene set exists
- Comparing methods using CheckV completeness alone without a random binning control,
  which the paper says is necessary because random binning does not reduce CheckV
  completeness
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0102
tools:
- name: ViralCC
  role: Constructs the Hi-C interaction graph and host proximity graph, integrates
    them, and writes draft viral bins; run via 'viralcc.py pipeline' with --min-len
    1000, --min-mapq 30, --min-match 30
  repo: https://github.com/dyxstat/ViralCC
- name: leidenalg
  role: Leiden graph clustering on the integrated graph (random seed 42) to partition
    viral contigs into draft viral bins
  repo: https://github.com/dyxstat/Reproduce_ViralCC
- name: CheckV
  role: Estimates completeness of recovered vMAGs to classify near-complete, substantially
    complete, and moderately complete bins
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-35945-y
    title: ViralCC retrieves complete viral genomes and virus-host pairs from metagenomic
      Hi-C data
  source_content_hash: sha256:df10cfd800237c0304c2b8210c254906945e666f04024136ba4b61d5c4bad85a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dyxstat/ViralCC
  tools:
  - CheckV
  - ViralCC
  - leidenalg
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# viral-contig-binning-evaluation

## Summary

Reproduce and evaluate ViralCC-style viral binning by integrating the Hi-C interaction graph with a host proximity graph (k=30) and running Leiden clustering (leidenalg, seed 42) on a mock human gut dataset, then assessing vMAG completeness with CheckV against graph-only ablation baselines.

## When to use

Use when a metagenomic Hi-C dataset has assembled contigs, a name-sorted filtered BAM, and an identified viral contig list, and you need to bin viral contigs and quantify whether graph integration improves vMAG recovery.

## When NOT to use

- The input is shotgun-only data with no Hi-C proximity-ligation BAM, since the Hi-C interaction graph is the core input to ViralCC
- You need contamination estimates for vMAGs — CheckV cannot estimate contamination of vMAGs because no viral marker gene set exists
- Comparing methods using CheckV completeness alone without a random binning control, which the paper says is necessary because random binning does not reduce CheckV completeness

## Inputs

- Assembled metagenomic contigs FASTA (mock human gut dataset)
- Name-sorted Hi-C alignment BAM filtered with 'samtools view -F 0x904'
- Viral contig list from viral contig identification (e.g., VirSorter)

## Outputs

- VIRAL_BIN fasta files (draft viral genome bins)
- cluster_viral_contig.txt (two columns: contig name, group number)
- viral_contig_info.csv
- prokaryotic_contig_info.csv
- viralcc.log

## How to apply

Load the assembled contigs FASTA, the name-sorted Hi-C BAM (filtered with 'samtools view -F 0x904' to remove unmapped, supplementary, and secondary alignments), and the viral contig list. Run 'python ./viralcc.py pipeline' with ViralCC defaults --min-len 1000, --min-mapq 30, and --min-match 30 to construct the Hi-C interaction graph (2699 edges on the mock human gut data). Build the host proximity graph using k=30 to link viral and prokaryotic contigs (2698 edges), integrate the two graphs (4397 edges), and run Leiden clustering with leidenalg and random seed 42. For evaluation, cluster on each graph alone as ablations and estimate vMAG completeness with CheckV, comparing near-complete / substantially complete / moderately complete counts: integrated graph yields 26/2/4 versus Hi-C-only 8/3/5 and host-only 12/2/0. Confirm outputs include VIRAL_BIN fasta files, cluster_viral_contig.txt (two columns: contig name, group number), viral_contig_info.csv, prokaryotic_contig_info.csv, and viralcc.log.

## Related tools

- **ViralCC** (Constructs the Hi-C interaction graph and host proximity graph, integrates them, and writes draft viral bins; run via 'viralcc.py pipeline' with --min-len 1000, --min-mapq 30, --min-match 30) — https://github.com/dyxstat/ViralCC
- **leidenalg** (Leiden graph clustering on the integrated graph (random seed 42) to partition viral contigs into draft viral bins) — https://github.com/dyxstat/Reproduce_ViralCC
- **CheckV** (Estimates completeness of recovered vMAGs to classify near-complete, substantially complete, and moderately complete bins)

## Examples

```
python ./viralcc.py pipeline --min-len 1000 --min-mapq 30 --min-match 30 contigs.fasta aligned.bam viral_contigs.txt OUTDIR
```

## Evaluation signals

- Graph edge counts are reproduced: 2699 Hi-C interaction edges, 2698 host proximity edges, 4397 integrated edges
- Completeness-class counts match the ablation expectations: integrated graph 26 near-complete / 2 substantially complete / 4 moderately complete, versus Hi-C-only 8/3/5 and host-only 12/2/0
- cluster_viral_contig.txt exists and has exactly two columns (contig name, group number)
- Output directory contains VIRAL_BIN fasta files, viral_contig_info.csv, prokaryotic_contig_info.csv, and viralcc.log
- Integrated-graph clustering outperforms both single-graph ablations in near-complete vMAG count

## Limitations

- CheckV cannot estimate contamination of vMAGs since no universal marker gene set exists for viruses, and its assessment is not comprehensive
- When CheckV completeness is used as the evaluation metric, a random binning control is necessary because random binning does not reduce CheckV completeness
- The benchmarking strategy underestimates true viral community diversity because only single-contig-recoverable viral genomes were chosen

## Evidence

- [methods] Pipeline invocation: "python ./viralcc.py pipeline [Parameters] FASTA_file BAM_file VIRAL_file OUTPUT_directory"
- [intro] Leiden clustering on integrated graph: "After integrating two graphs, ViralCC employs Leiden clustering to reconstruct draft viral genomes"
- [abstract] Host proximity rationale: "ViralCC leverages the virus-host proximity structure as a complementary information source for the Hi-C interactions."
- [intro] CheckV completeness evaluation: "CheckV has been widely used to estimate the completeness of vMAGs by comparing them to a large database curated from NCBI GenBank and environmental samples"
- [readme] README graph integration description: "The two graphs are then integrated together, followed by the Leiden graph clustering using the integrative graph to generate draft viral genomes."
