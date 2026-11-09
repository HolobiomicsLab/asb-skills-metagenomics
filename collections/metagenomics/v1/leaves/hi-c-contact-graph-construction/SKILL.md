---
name: hi-c-contact-graph-construction
description: Use when you have Hi-C paired-end read alignments (BAM sorted by name)
  to assembled contigs plus a list of identified viral contigs, and you need to bin
  viral contigs using virus-host proximity structure.
when_to_use_negative:
- Input is shotgun-only data with no proximity-ligation (Hi-C) read pairs — the host
  proximity structure cannot be derived.
- The Hi-C BAM is sorted by coordinate rather than by name, so read-pair mates cannot
  be reliably paired.
- Contigs have not passed the minimum length threshold (1000 bp), which the pipeline
  filters out via --min-len.
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: ViralCC
  role: Implements the Hi-C interaction and host proximity graph construction and
    the automatic k selection via `viralcc.py pipeline`.
  repo: https://github.com/dyxstat/ViralCC
- name: pysam
  role: Used to load the name-sorted Hi-C BAM and filter alignments (MAPQ >= 30, >=
    30 matches, contig length >= 1000 bp).
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-35945-y
    title: ViralCC retrieves complete viral genomes and virus-host pairs from metagenomic
      Hi-C data
  source_content_hash: sha256:84b7a08f26e869f430fe3f843f165bfbd7128a4d96035e8c4facbe14c907ee8b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dyxstat/ViralCC
  tools:
  - ViralCC
  - pysam
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# hi-c-contact-graph-construction

## Summary

Constructs the Hi-C interaction graph and the complementary host proximity graph from a name-sorted Hi-C BAM: viral contigs are linked when they share at least k common host contigs contacted in Hi-C, with k auto-selected so |E_host| <= |E_hic|. In ViralCC this dual graph feeds Leiden clustering to reconstruct draft viral genomes.

## When to use

Use when you have Hi-C paired-end read alignments (BAM sorted by name) to assembled contigs plus a list of identified viral contigs, and you need to bin viral contigs using virus-host proximity structure.

## When NOT to use

- Input is shotgun-only data with no proximity-ligation (Hi-C) read pairs — the host proximity structure cannot be derived.
- The Hi-C BAM is sorted by coordinate rather than by name, so read-pair mates cannot be reliably paired.
- Contigs have not passed the minimum length threshold (1000 bp), which the pipeline filters out via --min-len.

## Inputs

- Hi-C paired-end alignment BAM sorted by name via samtools sort
- Assembled contigs FASTA (e.g. from megahit with --min-contig-len 1000)
- File listing identified viral contigs (e.g. VirSorter output)

## Outputs

- Hi-C interaction graph over viral contigs with recorded edge count |E_hic|
- Host proximity graph G_host linking viral contigs sharing >= k host contigs (k auto-selected; 2698 edges at k=30 on the mock human gut dataset)
- Integrated graph for downstream Leiden clustering into draft viral genomes

## How to apply

Load the name-sorted Hi-C BAM with pysam, retaining alignments with MAPQ >= 30 (--min-mapq), at least 30 matches (--min-match), for contigs of length >= 1000 bp (--min-len). Build the Hi-C interaction graph from the filtered read pairs and record its edge count |E_hic|. For candidate thresholds k >= 4 (--min-k), construct candidate host proximity graphs in which two viral contigs are connected when they share at least k host contigs contacted in Hi-C; select the k that maximizes |E_host| subject to |E_host| <= |E_hic| so the proximity signal complements rather than overwhelms the Hi-C signal. On the ViralCC mock human gut dataset this selects k = 30, yielding 2698 edges in G_host. The two graphs are then integrated and clustered (Leiden) to recover draft viral genomes.

## Related tools

- **ViralCC** (Implements the Hi-C interaction and host proximity graph construction and the automatic k selection via `viralcc.py pipeline`.) — https://github.com/dyxstat/ViralCC
- **pysam** (Used to load the name-sorted Hi-C BAM and filter alignments (MAPQ >= 30, >= 30 matches, contig length >= 1000 bp).)

## Examples

```
python ./viralcc.py pipeline -v Test/final.contigs.fa Test/MAP_SORTED.bam Test/viral_contigs.txt Test/out_test
```

## Evaluation signals

- The reconstructed edge rule reproduces the reported mock-dataset result: 2698 host-proximity edges at the selected k = 30.
- The selected k satisfies the constraint |E_host| <= |E_hic|, i.e. host-proximity edges never exceed Hi-C interaction edges.
- The selected k respects the lower bound --min-k (default 4).
- Alignment filtering is verifiable: retained alignments have MAPQ >= 30, at least 30 matches, and belong to contigs of length >= 1000 bp.
- The packaged simulated dataset (Test directory) runs end-to-end via `python ./viralcc.py pipeline -v Test/final.contigs.fa Test/MAP_SORTED.bam Test/viral_contigs.txt Test/out_test`.

## Limitations

- The fraction of spurious contacts in the host proximity graph cannot be estimated on real datasets because true labels of viral contigs are unknown.
- k was tuned on the mock human gut dataset (k = 30); the automatic k selection maximizes |E_host| under the |E_host| <= |E_hic| constraint and may pick different values on other datasets.
- Some proximity-ligation virus–bacteria associations may arise from mechanisms other than infection, so host proximity edges should be validated (e.g. CRISPR spacer analyses) before concluding infection.

## Evidence

- [results] two viral contigs are linked when they share at least k (tuned to 30 on the mock human gut dataset) common host contigs by Hi-C interaction, resulting in 2698 edges in G_host.: [evidence span withheld — anchor status: not_found]
- [intro] ViralCC constructs the Hi-C interaction graph and the host proximity graph: "ViralCC constructs the Hi-C interaction graph and the host proximity graph"
- [intro] After integrating two graphs, ViralCC employs Leiden clustering to reconstruct draft viral genomes: "After integrating two graphs, ViralCC employs Leiden clustering to reconstruct draft viral genomes"
- [methods] --min-mapq: Minimum acceptable alignment quality (default 30): "--min-mapq: Minimum acceptable alignment quality (default 30)"
- [readme] `ViralCC` not only considers the Hi-C interaction graph, but also puts forward a novel host proximity graph of viral contigs: "`ViralCC` not only considers the Hi-C interaction graph, but also puts forward a novel host proximity graph of viral contigs"
