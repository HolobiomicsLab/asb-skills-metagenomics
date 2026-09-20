---
name: hic-contact-graph-construction
description: 'Use when assembling draft viral genomes (vMAGs) from metagenomic Hi-C
  data: you have assembled contigs (FASTA), a name-sorted Hi-C alignment BAM, and
  a viral contig list, and need contact/proximity graphs for binning.'
when_to_use_negative:
- When no Hi-C (proximity-ligation) data are available — shotgun co-abundance-based
  binning tools like CoCoNet or vRhyme are the relevant alternatives, though they
  need enough samples for reliable co-abundance profiles
- When the input BAM is not name-sorted or still contains unmapped/supplementary/secondary
  alignments (unfiltered with -F 0x904), as ViralCC requires a name-sorted, filtered
  BAM
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: ViralCC
  role: Pipeline that builds the Hi-C interaction graph and host proximity graph,
    integrates them into G_int, and runs Leiden clustering to output draft viral bins
  repo: https://github.com/dyxstat/ViralCC
- name: leidenalg
  role: Python library used to partition viral contigs on the integrative graph, with
    resolution sweeping and random seed 42 for reproducibility
- name: CheckV
  role: Used downstream to estimate completeness classes of recovered vMAGs for validation
provenance:
  source_task_ids:
  - task_002
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-35945-y
    title: ViralCC retrieves complete viral genomes and virus-host pairs from metagenomic
      Hi-C data
  source_content_hash: sha256:7f61b05de5497f9f039aeb6dddc79bbef474308d08e1cf20cc4d2e0f5a89bb02
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dyxstat/ViralCC
  tools:
  - CheckV
  - ViralCC
  - leidenalg
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# hic-contact-graph-construction

## Summary

Constructs the ViralCC Hi-C interaction graph (G_hic) from a name-sorted Hi-C alignment BAM and the host proximity graph (G_host) from k-mer-based virus-host links, which are then integrated into G_int for Leiden clustering of viral contigs into draft viral genomes.

## When to use

Use when assembling draft viral genomes (vMAGs) from metagenomic Hi-C data: you have assembled contigs (FASTA), a name-sorted Hi-C alignment BAM, and a viral contig list, and need contact/proximity graphs for binning.

## When NOT to use

- When no Hi-C (proximity-ligation) data are available — shotgun co-abundance-based binning tools like CoCoNet or vRhyme are the relevant alternatives, though they need enough samples for reliable co-abundance profiles
- When the input BAM is not name-sorted or still contains unmapped/supplementary/secondary alignments (unfiltered with -F 0x904), as ViralCC requires a name-sorted, filtered BAM

## Inputs

- Assembled contigs FASTA (e.g., final.contigs.fa from MEGAHIT, --min-contig-len 1000)
- Name-sorted Hi-C alignment BAM filtered with 'samtools view -F 0x904'
- Viral contig list (e.g., viral_contigs.txt from viral contig identification)

## Outputs

- cluster_viral_contig.txt (2 columns: viral contig name and group number, no header)
- Draft viral bin FASTA files (VIRAL_BIN0000.fa, VIRAL_BIN0001.fa, ...) in the VIRAL_BIN folder
- viral_contig_info.csv and prokaryotic_contig_info.csv (contig name, length, GC-content)
- viralcc.log

## How to apply

Align Hi-C paired-end reads to assembled contigs, filter alignments with 'samtools view -F 0x904' to remove unmapped, supplementary, and secondary alignments, and sort the BAM by name. Run ViralCC's 'viralcc.py pipeline' on the contig FASTA, BAM, and viral contig list, keeping defaults --min-len 1000, --min-mapq 30, and --min-match 30 to build the Hi-C interaction graph, and use k=30 for host proximity links between viral and prokaryotic contigs (lower bound k controlled by --min-k, default 4). Integrate G_hic and G_host into G_int, then run Leiden clustering (leidenalg, random seed 42) sweeping the resolution parameter to maximize silhouette score under Reichardt-Bornholdt modularity. On the mock human gut dataset this integration reconstructs 26 near-complete, 2 substantially complete, and 4 moderately complete vMAGs, versus 8/3/5 for G_hic alone and 12/2/0 for G_host alone, so validation should compare against these counts and the reported edge counts (2699 Hi-C, 2698 host proximity, 4397 integrated edges).

## Related tools

- **ViralCC** (Pipeline that builds the Hi-C interaction graph and host proximity graph, integrates them into G_int, and runs Leiden clustering to output draft viral bins) — https://github.com/dyxstat/ViralCC
- **leidenalg** (Python library used to partition viral contigs on the integrative graph, with resolution sweeping and random seed 42 for reproducibility)
- **CheckV** (Used downstream to estimate completeness classes of recovered vMAGs for validation)

## Examples

```
python ./viralcc.py pipeline --min-len 1000 --min-mapq 30 --min-match 30 final.contigs.fa MAP_SORTED.bam viral_contigs.txt OUTPUT_directory
```

## Evaluation signals

- cluster_viral_contig.txt exists with exactly two columns (contig name, group number) and no header
- VIRAL_BIN directory contains draft viral bin fasta files named VIRAL_BIN0000.fa, VIRAL_BIN0001.fa, ...
- Reported graph edge counts are reproduced: 2699 Hi-C interaction edges, 2698 host proximity edges, 4397 integrated edges
- CheckV completeness-class counts on the mock human gut dataset match 26 near-complete / 2 substantially complete / 4 moderately complete vMAGs for G_int
- viral_contig_info.csv and prokaryotic_contig_info.csv contain contig name, length, and GC-content columns; viralcc.log is written

## Limitations

- The host proximity graph's spurious-contact fraction cannot be estimated on real datasets because true viral contig labels are unknown
- The Hi-C interaction graph is remarkably sparse, which is why the host proximity graph is added as complementary information
- Random binning controls are necessary when using CheckV completeness as an evaluation metric because random binning does not reduce CheckV completeness

## Evidence

- [abstract] ViralCC leverages the virus-host proximity structure as a complementary information source for the Hi-C interactions.: "ViralCC leverages the virus-host proximity structure as a complementary information source for the Hi-C interactions."
- [methods] samtools -F 0x904 filter: "samtools with parameters 'view -F 0x904' is applied to remove unmapped reads, supplementary alignments, and secondary alignments"
- [methods] BAM sorted by name: "BAM file needs to be sorted **by name** using 'samtools sort'"
- [intro] graph construction and Leiden clustering steps: "ViralCC constructs the Hi-C interaction graph and the host proximity graph"
- [readme] ViralCC CLI from README: "python ./viralcc.py pipeline [Parameters] FASTA_file BAM_file VIRAL_file OUTPUT_directory"
