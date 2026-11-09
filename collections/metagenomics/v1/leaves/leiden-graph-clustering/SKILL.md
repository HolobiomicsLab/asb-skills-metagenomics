---
name: leiden-graph-clustering
description: Use when viral contigs from a metagenomic Hi-C experiment need to be
  binned into draft viral genomes by clustering a graph whose edges encode Hi-C interactions
  and/or virus-host proximity links.
when_to_use_negative:
- Hi-C alignment reads are not available or cannot be aligned to assembled contigs,
  since the Hi-C interaction graph is the core input
- The goal is prokaryotic MAG binning with marker genes (use CheckM-based tools instead);
  ViralCC targets viral genomes that lack universal single-copy marker genes
- Taxonomic annotation of contigs is required for binning (as in HiCBin/TAXAassign
  workflows), since TAXAassign can hardly annotate viral contigs
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0194
- http://edamontology.org/topic_3174
tools:
- name: ViralCC
  role: Constructs the Hi-C interaction graph from BAM/FASTA inputs, integrates it
    with the host proximity graph, and runs Leiden clustering to produce viral bins
  repo: https://github.com/dyxstat/ViralCC
- name: leidenalg
  role: Python library performing the Leiden graph partitioning on the integrated
    graph (random seed 42)
- name: CheckV
  role: Estimates completeness of the recovered vMAGs for evaluating clustering quality
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-35945-y
    title: ViralCC retrieves complete viral genomes and virus-host pairs from metagenomic
      Hi-C data
  source_content_hash: sha256:296068c4f4cae2085e366228fffca1690322f14712853edbaf22ddcf48320acf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dyxstat/ViralCC
  tools:
  - CheckV
  - ViralCC
  - leidenalg
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0194
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# leiden-graph-clustering

## Summary

Applies Leiden graph clustering (leidenalg, random seed 42) to an integrated graph combining the Hi-C interaction graph and the host proximity graph of viral contigs, to partition viral contigs into draft viral genomes (viral bins) in the ViralCC pipeline.

## When to use

Use when viral contigs from a metagenomic Hi-C experiment need to be binned into draft viral genomes by clustering a graph whose edges encode Hi-C interactions and/or virus-host proximity links.

## When NOT to use

- Hi-C alignment reads are not available or cannot be aligned to assembled contigs, since the Hi-C interaction graph is the core input
- The goal is prokaryotic MAG binning with marker genes (use CheckM-based tools instead); ViralCC targets viral genomes that lack universal single-copy marker genes
- Taxonomic annotation of contigs is required for binning (as in HiCBin/TAXAassign workflows), since TAXAassign can hardly annotate viral contigs

## Inputs

- Assembled contigs FASTA file (mock human gut dataset)
- Name-sorted Hi-C alignment BAM filtered with 'samtools view -F 0x904' (unmapped, supplementary, secondary alignments removed)
- Viral contig list from viral contig identification (e.g., VirSorter)
- Hi-C interaction graph edges (e.g., 2699 edges)
- Host proximity graph edges built with k=30 (e.g., 2698 edges)

## Outputs

- VIRAL_BIN FASTA files (draft viral genome bins)
- cluster_viral_contig.txt (two columns: contig name, group number)
- viral_contig_info.csv
- prokaryotic_contig_info.csv
- viralcc.log

## How to apply

First construct the Hi-C interaction graph from a name-sorted BAM (filtered with 'samtools view -F 0x904') via 'python ./viralcc.py pipeline' using ViralCC defaults --min-len 1000, --min-mapq 30, and --min-match 30; separately build the host proximity graph between viral and prokaryotic contigs using k=30. Integrate the two graphs (mock human gut example: 2699 Hi-C edges, 2698 host proximity edges, 4397 integrated edges) and run Leiden clustering with leidenalg (random seed 42) to partition viral contigs into draft viral bins. Evaluate by assessing vMAG completeness with CheckV; on the mock human gut dataset the integrated graph recovered 26 near-complete, 2 substantially complete, and 4 moderately complete vMAGs, versus 8/3/5 for the Hi-C graph alone and 12/2/0 for the host proximity graph alone, demonstrating that integration improves binning over either graph alone.

## Related tools

- **ViralCC** (Constructs the Hi-C interaction graph from BAM/FASTA inputs, integrates it with the host proximity graph, and runs Leiden clustering to produce viral bins) — https://github.com/dyxstat/ViralCC
- **leidenalg** (Python library performing the Leiden graph partitioning on the integrated graph (random seed 42))
- **CheckV** (Estimates completeness of the recovered vMAGs for evaluating clustering quality)

## Examples

```
python ./viralcc.py pipeline --min-len 1000 --min-mapq 30 --min-match 30 ASSEMBLY.final.contigs.fa hic.bam viral_contigs.txt OUTPUT_DIR
```

## Evaluation signals

- cluster_viral_contig.txt exists with exactly two columns (contig name, group number)
- Reported graph edge counts are reproduced: 2699 Hi-C interaction edges, 2698 host proximity edges, 4397 integrated edges
- Completeness-class counts of recovered vMAGs match expectation (integrated graph: 26 near-complete, 2 substantially complete, 4 moderately complete)
- Integrated-graph clustering outperforms single-graph clustering (Hi-C alone 8/3/5; host proximity alone 12/2/0)
- viralcc.log and viral_contig_info.csv / prokaryotic_contig_info.csv are written to the output directory

## Limitations

- Random binning control experiments are necessary when using CheckV completeness as the evaluation metric, since random binning does not reduce CheckV completeness
- The fraction of spurious contacts in the host proximity graph cannot be estimated from real datasets because true labels of viral contigs are unknown
- Effectiveness of the benchmarking is less convincing when there are few putative viral genomes

## Evidence

- [readme] The two graphs are then integrated together, followed by the Leiden graph clustering using the integrative graph to generate draft viral genomes.: "The two graphs are then integrated together, followed by the Leiden graph clustering using the integrative graph to generate draft viral genomes."
- [intro] After integrating two graphs, ViralCC employs Leiden clustering to reconstruct draft viral genomes: "After integrating two graphs, ViralCC employs Leiden clustering to reconstruct draft viral genomes"
- [methods] samtools with parameters ‘view -F 0x904’ is applied to remove unmapped reads, supplementary alignments, and secondary alignments: "samtools with parameters ‘view -F 0x904’ is applied to remove unmapped reads, supplementary alignments, and secondary alignments"
- [discussion] it is necessary to construct a random binning model as control experiments when the CheckV completeness is used as the evaluation metric: "it is necessary to construct a random binning model as control experiments when the CheckV completeness is used as the evaluation metric"
