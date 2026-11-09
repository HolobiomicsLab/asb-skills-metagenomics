---
name: virus-host-proximity-graph-construction
description: Use when binning viral contigs from metagenomic Hi-C data and you need
  to build a virus-host proximity graph from shared host contacts to complement the
  sparse Hi-C interaction graph.
when_to_use_negative:
- Input Hi-C BAM is not name-sorted — the pipeline requires 'samtools sort' by read
  name
- No viral contig list is available or contigs are shorter than the --min-len default
  of 1000 bp
- The analysis goal is taxonomic classification of viral contigs from sequence homology
  rather than Hi-C-based binning
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: ViralCC
  role: Pipeline that builds the Hi-C interaction graph and host proximity graph,
    automatically selecting k, and integrates them for Leiden clustering
  repo: https://github.com/dyxstat/ViralCC
- name: pysam
  role: Python library used to load and filter the name-sorted Hi-C BAM alignments
    by MAPQ, match count, and contig length
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-35945-y
    title: ViralCC retrieves complete viral genomes and virus-host pairs from metagenomic
      Hi-C data
  source_content_hash: sha256:4c693ed261bdcd16f9b944843ba3f46f0c4090635aecdfb31f0e39d3d3a49ac9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dyxstat/ViralCC
  tools:
  - ViralCC
  - pysam
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# virus-host-proximity-graph-construction

## Summary

Constructs ViralCC's host proximity graph, in which two viral contigs are linked when they share at least k host contigs contacted via Hi-C interactions, with k chosen automatically so the proximity signal complements rather than overwhelms the Hi-C interaction signal.

## When to use

Use when binning viral contigs from metagenomic Hi-C data and you need to build a virus-host proximity graph from shared host contacts to complement the sparse Hi-C interaction graph.

## When NOT to use

- Input Hi-C BAM is not name-sorted — the pipeline requires 'samtools sort' by read name
- No viral contig list is available or contigs are shorter than the --min-len default of 1000 bp
- The analysis goal is taxonomic classification of viral contigs from sequence homology rather than Hi-C-based binning

## Inputs

- Name-sorted Hi-C alignment BAM file (e.g., MAP_SORTED.bam)
- Assembled contigs FASTA file (contigs >= 1000 bp)
- List of identified viral contigs (text file, e.g., viral_contigs.txt)

## Outputs

- Host proximity graph G_host linking pairs of viral contigs sharing >= k common host contigs (2698 edges at k = 30 on the mock human gut dataset)
- Selected threshold k for shared-host-contig linkage
- Integrated graph used downstream for Leiden clustering

## How to apply

Load the name-sorted Hi-C BAM with pysam, retaining alignments with MAPQ >= 30 (--min-mapq default), at least 30 matches (--min-match default), for contigs of length >= 1000 bp (--min-len default). Build the Hi-C interaction graph from the filtered read pairs and record its edge count |E_hic|. For candidate thresholds k >= 4 (lower bound set by --min-k), construct candidate host proximity graphs in which two viral contigs are connected when they share at least k host contigs contacted in Hi-C. Select the k that maximizes the number of host-proximity edges subject to the constraint |E_host| <= |E_hic|, so the proximity graph complements rather than overwhelms the Hi-C interaction signal; on the mock human gut dataset this yielded k = 30 and 2698 edges in G_host. The resulting host proximity graph is subsequently integrated with the Hi-C interaction graph before Leiden clustering to reconstruct draft viral genomes.

## Related tools

- **ViralCC** (Pipeline that builds the Hi-C interaction graph and host proximity graph, automatically selecting k, and integrates them for Leiden clustering) — https://github.com/dyxstat/ViralCC
- **pysam** (Python library used to load and filter the name-sorted Hi-C BAM alignments by MAPQ, match count, and contig length)

## Examples

```
python ./viralcc.py pipeline -v Test/final.contigs.fa Test/MAP_SORTED.bam Test/viral_contigs.txt Test/out_test
```

## Evaluation signals

- On the mock human gut dataset, the reconstructed edge rule reproduces 2698 host-proximity edges in G_host at the selected k = 30
- The selected k satisfies |E_host| <= |E_hic|, i.e., the host proximity graph never has more edges than the Hi-C interaction graph
- All edges in G_host connect viral contig pairs sharing at least k (= 30 on mock data) common host contigs in the Hi-C interaction graph
- The end-to-end pipeline run on the packaged simulated Test dataset completes: python ./viralcc.py pipeline -v Test/final.contigs.fa Test/MAP_SORTED.bam Test/viral_contigs.txt Test/out_test

## Limitations

- The fraction of spurious contacts in the host proximity graph cannot be obtained from real datasets because true labels of viral contigs are unknown
- Some proximity-ligation virus-bacteria associations might arise from proximity of viral and bacterial DNA from a mechanism other than infection and should be validated before drawing conclusions
- Whether the proximity structure can resolve closely related viruses residing in the same bacterial host remains unexplored

## Evidence

- [methods] host proximity edge rule: "ViralCC leverages the virus-host proximity structure as a complementary information source for the Hi-C interactions."
- [intro] graph construction and clustering: "ViralCC constructs the Hi-C interaction graph and the host proximity graph"
- [methods] filtering defaults: "--min-mapq: Minimum acceptable alignment quality (default 30)"
- [methods] BAM requirement: "BAM file needs to be sorted **by name** using 'samtools sort'"
- [readme] test invocation: "We provide a small simulated dataset, located under the Test directory, to demo and test the software"
