---
name: mock-dataset-graph-validation
description: Use when implementing or modifying the host proximity graph code in ViralCC
  and you need a reproducible check that the automatic k-selection rule (maximize
  |E_host| subject to |E_host| ≤ |E_hic|) behaves as published.
when_to_use_negative:
- When validating steps unrelated to graph construction (e.g., viral contig identification
  or virus-host pair detection), since the expected 2698-edge result only constrains
  the host proximity graph rule
- When the Hi-C BAM is sorted by coordinate rather than by name, as the ViralCC pipeline
  requires a name-sorted BAM
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0194
- http://edamontology.org/topic_0622
tools:
- name: ViralCC
  role: Pipeline under test; its viralcc.py pipeline command constructs the Hi-C interaction
    graph and host proximity graph on the simulated Test dataset
  repo: https://github.com/dyxstat/ViralCC
- name: pysam
  role: Used to load the name-sorted Hi-C BAM and filter alignments by MAPQ, match
    count, and contig length when rebuilding the graph
- name: samtools
  role: Sorts the Hi-C BAM by name prior to graph construction and applies 'view -F
    0x904' to remove unmapped, supplementary, and secondary alignments
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-35945-y
    title: ViralCC retrieves complete viral genomes and virus-host pairs from metagenomic
      Hi-C data
  source_content_hash: sha256:c3ac0a691d9c2d499d02d399dce4a9f9ccbc7dbc143dad60ce6964c78aed8998
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dyxstat/ViralCC
  tools:
  - ViralCC
  - pysam
  - samtools
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0194
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# mock-dataset-graph-validation

## Summary

End-to-end validation of ViralCC's host proximity graph construction by running the packaged simulated test dataset and confirming the k-selection rule reproduces the reported result of 2698 host-proximity edges at k=30.

## When to use

Use when implementing or modifying the host proximity graph code in ViralCC and you need a reproducible check that the automatic k-selection rule (maximize |E_host| subject to |E_host| ≤ |E_hic|) behaves as published.

## When NOT to use

- When validating steps unrelated to graph construction (e.g., viral contig identification or virus-host pair detection), since the expected 2698-edge result only constrains the host proximity graph rule
- When the Hi-C BAM is sorted by coordinate rather than by name, as the ViralCC pipeline requires a name-sorted BAM

## Inputs

- Packaged simulated test dataset under the Test directory (Test/final.contigs.fa, Test/MAP_SORTED.bam, Test/viral_contigs.txt)
- Name-sorted Hi-C BAM file (sorted by name with samtools)
- List of identified viral contigs (viral_contigs.txt)
- Assembly FASTA of contigs ≥ 1000 bp

## Outputs

- Reconstructed host proximity graph G_host with edge count |E_host| (expected 2698 edges at k = 30 on the mock dataset)
- Selected k threshold from automatic k selection
- Hi-C interaction graph edge count |E_hic| used as the upper bound constraint

## How to apply

Load the name-sorted Hi-C BAM with pysam, retaining alignments with MAPQ ≥ 30 (--min-mapq default), at least 30 matches (--min-match default), for contigs of length ≥ 1000 bp (--min-len default). Build the Hi-C interaction graph from the filtered read pairs and record its edge count |E_hic|. For candidate thresholds k ≥ 4 (--min-k lower bound), construct candidate host proximity graphs in which two viral contigs are connected when they share at least k host contigs contacted in Hi-C, and select the k that maximizes host-proximity edges subject to |E_host| ≤ |E_hic| so the proximity signal complements rather than overwhelms the Hi-C interaction signal. Then run the full pipeline on the Test directory dataset and verify the reconstructed edge rule reproduces the mock human gut dataset result of 2698 edges in G_host at the selected k = 30.

## Related tools

- **ViralCC** (Pipeline under test; its viralcc.py pipeline command constructs the Hi-C interaction graph and host proximity graph on the simulated Test dataset) — https://github.com/dyxstat/ViralCC
- **pysam** (Used to load the name-sorted Hi-C BAM and filter alignments by MAPQ, match count, and contig length when rebuilding the graph)
- **samtools** (Sorts the Hi-C BAM by name prior to graph construction and applies 'view -F 0x904' to remove unmapped, supplementary, and secondary alignments)

## Examples

```
python ./viralcc.py pipeline -v Test/final.contigs.fa Test/MAP_SORTED.bam Test/viral_contigs.txt Test/out_test
```

## Evaluation signals

- The host proximity graph built from the Test directory dataset yields exactly 2698 edges in G_host at the selected k = 30
- The selected k satisfies |E_host| ≤ |E_hic| for every candidate k ≥ 4 considered, with k chosen to maximize |E_host|
- Alignment filtering reproduces the defaults: MAPQ ≥ 30, ≥ 30 matches, contigs ≥ 1000 bp (--min-mapq, --min-match, --min-len)
- The end-to-end pipeline command exits successfully on Test/final.contigs.fa, Test/MAP_SORTED.bam, Test/viral_contigs.txt with output in Test/out_test

## Limitations

- The fraction of spurious contacts in the host proximity graph cannot be assessed on real datasets because true labels of viral contigs are unknown
- k = 30 was tuned specifically on the mock human gut dataset; other communities may warrant different thresholds
- The packaged Test directory is a small simulated dataset intended for demo and testing, not a benchmark of biological performance

## Evidence

- [other] k tuned to 30 on the mock human gut dataset with 2698 edges: [evidence span withheld — anchor status: not_found]
- [other] k-selection constraint: [evidence span withheld — anchor status: not_found]
- [methods] alignment filter defaults: "--min-len: Minimum acceptable contig length (default 1000)"
- [methods] test dataset packaging: "We provide a small simulated dataset, located under the Test directory, to demo and test the software"
- [readme] pipeline invocation: "`ViralCC` not only considers the Hi-C interaction graph, but also puts forward a novel host proximity graph of viral contigs"
