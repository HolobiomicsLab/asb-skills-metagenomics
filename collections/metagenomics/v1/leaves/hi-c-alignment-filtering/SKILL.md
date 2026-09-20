---
name: hi-c-alignment-filtering
description: Use when raw Hi-C paired-end reads have been aligned to assembled contigs
  (BAM available) and downstream binning or graph construction requires a curated
  set of high-confidence inter-contig contact pairs.
when_to_use_negative:
- Input is raw uncleaned FASTQ — first run adaptor removal, quality trimming (bbduk
  trimq=10 qtrim=r ftm=5 minlen=50), Hi-C first-10-nt trimming (ftl=10), and PCR duplicate
  removal (clumpify.sh) before alignment
- The BAM is coordinate-sorted rather than name-sorted — ViralCC requires name sorting
  to pair Hi-C mates
- Goal is taxonomic annotation of contigs rather than Hi-C contact curation — use
  TAXAassign/Demovir instead
edam_operation: http://edamontology.org/operation_3680
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3168
tools:
- name: ViralCC
  role: Applies the --min-mapq (30), --min-match (30), and --min-len (1000) thresholds
    while loading the name-sorted Hi-C BAM with pysam to build the filtered interaction
    graph
  repo: https://github.com/dyxstat/ViralCC
- name: samtools
  role: Removes unmapped, supplementary, and secondary alignments (view -F 0x904)
    and name-sorts the BAM; v1.15.1 used in the paper's reproduction
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-023-35945-y
    title: ViralCC retrieves complete viral genomes and virus-host pairs from metagenomic
      Hi-C data
  source_content_hash: sha256:6d8afbc01e1e90cadb90ad265e637749f8663ca80a7dca3b44c5cec03faa9bda
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/dyxstat/ViralCC
  tools:
  - ViralCC
  - pysam
  - samtools
  edam_operation: http://edamontology.org/operation_3680
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# Hi-C alignment filtering

## Summary

Filters metagenomic Hi-C alignments before graph construction by removing unmapped/supplementary/secondary records with samtools view -F 0x904, enforcing a name-sorted BAM, and applying ViralCC's per-alignment thresholds (MAPQ ≥ 30, ≥ 30 matches, contigs ≥ 1000 bp) so only reliable inter-contig contacts feed the Hi-C interaction graph.

## When to use

Use when raw Hi-C paired-end reads have been aligned to assembled contigs (BAM available) and downstream binning or graph construction requires a curated set of high-confidence inter-contig contact pairs.

## When NOT to use

- Input is raw uncleaned FASTQ — first run adaptor removal, quality trimming (bbduk trimq=10 qtrim=r ftm=5 minlen=50), Hi-C first-10-nt trimming (ftl=10), and PCR duplicate removal (clumpify.sh) before alignment
- The BAM is coordinate-sorted rather than name-sorted — ViralCC requires name sorting to pair Hi-C mates
- Goal is taxonomic annotation of contigs rather than Hi-C contact curation — use TAXAassign/Demovir instead

## Inputs

- BAM file of Hi-C paired-end reads aligned to assembled contigs (must be sorted by name via samtools sort)
- Assembled contigs FASTA (contigs ≥ 1000 bp retained)
- List of putative viral contig names (viral_contigs.txt)

## Outputs

- Filtered Hi-C interaction graph of inter-contig contact pairs with recorded edge count |E_hic|
- Name-sorted, filtered BAM accepted by ViralCC's pipeline stage

## How to apply

Align Hi-C paired-end reads to assembled contigs (e.g. BWA MEM against MEGAHIT contigs assembled with --min-contig-len 1000), then remove unmapped reads, supplementary alignments, and secondary alignments using samtools view -F 0x904. Sort the resulting BAM by read NAME (not coordinate) with samtools sort, because ViralCC's pysam loader walks the name-sorted BAM to mate read pairs. ViralCC then retains only alignments with mapping quality ≥ 30 (--min-mapq default), at least 30 matched bases (--min-match default), on contigs of length ≥ 1000 bp (--min-len default). The rationale is that the Hi-C interaction map is remarkably sparse, so weak or spurious alignments would inject noise into the interaction graph; the filtered contact pairs also set the reference edge count |E_hic| used to cap the host proximity graph (|E_host| ≤ |E_hic|) during automatic k selection.

## Related tools

- **ViralCC** (Applies the --min-mapq (30), --min-match (30), and --min-len (1000) thresholds while loading the name-sorted Hi-C BAM with pysam to build the filtered interaction graph) — https://github.com/dyxstat/ViralCC
- **samtools** (Removes unmapped, supplementary, and secondary alignments (view -F 0x904) and name-sorts the BAM; v1.15.1 used in the paper's reproduction)

## Examples

```
samtools view -F 0x904 MAP_SORTED.bam | samtools sort -n -o MAP_SORTED_NAME.bam; python ./viralcc.py pipeline -v Test/final.contigs.fa Test/MAP_SORTED.bam Test/viral_contigs.txt Test/out_test
```

## Evaluation signals

- BAM is confirmed name-sorted (samtools sort on read names) before ViralCC's pysam loading step
- All retained records pass the 0x904 flag filter and have MAPQ ≥ 30 with ≥ 30 matched bases on contigs ≥ 1000 bp
- The resulting Hi-C interaction graph edge count |E_hic| is recorded and constrains the host proximity graph such that |E_host| ≤ |E_hic| (mock dataset: |E_host| = 2698 edges at k = 30)
- End-to-end run on the packaged Test dataset (python ./viralcc.py pipeline -v Test/final.contigs.fa Test/MAP_SORTED.bam Test/viral_contigs.txt Test/out_test) completes and reproduces expected downstream clustering

## Limitations

- Spurious proximity-ligation contacts from mechanisms other than infection can survive filtering; such virus-host associations should be validated (e.g. via CRISPR spacer analyses) before drawing conclusions
- CheckV-like downstream assessments do not capture filtering-induced biases, and benchmarking only uses single-contig-recoverable viral genomes, underestimating true community diversity
- The fraction of spurious contacts in the derived graphs cannot be estimated on real datasets because true viral contig labels are unknown

## Evidence

- [methods] samtools with parameters 'view -F 0x904' is applied to remove unmapped reads, supplementary alignments, and secondary alignments: "samtools with parameters 'view -F 0x904' is applied to remove unmapped reads, supplementary alignments, and secondary alignments"
- [methods] BAM sorted by name: "BAM file needs to be sorted **by name** using 'samtools sort'"
- [methods] ViralCC filtering parameters: "--min-mapq: Minimum acceptable alignment quality (default 30); --min-match: Accepted alignments must be at least N matches (default 30)"
- [results] k selection cap rationale: [evidence span withheld — anchor status: not_found]
- [readme] Readme overview of ViralCC inputs: "`ViralCC` not only considers the Hi-C interaction graph, but also puts forward a novel host proximity graph of viral contigs"
