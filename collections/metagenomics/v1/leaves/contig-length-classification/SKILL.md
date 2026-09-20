---
name: contig-length-classification
description: Use when a long-read-only assembly (e.g. Flye nanopore assembly of a
  bacterial isolate) must be split into chromosome and putative-plasmid contigs before
  chromosomal read removal or plasmid read extraction.
when_to_use_negative:
- When the genome has a chromosome-length contig that is actually a plasmid (e.g.
  large megaplasmids near the chromosome threshold) or chromosomal fragments split
  across contigs — length alone will misclassify them
- When input is already a curated plasmid set (e.g. completed GenBank genomes with
  replicon annotations) — explicit replicon labels should be used instead of length
  heuristics
- When using the long-only kmer_mode, which the article describes as experimental
  and unvalidated especially on small plasmids
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_0080
tools:
- name: Flye
  role: Produces the long-read-only assembly whose contigs are classified by length
    (Plassembler requires Flye >=2.9; Raven >=1.8 is an optional alternative assembler)
  repo: https://github.com/fenderglass/Flye
- name: Plassembler
  role: Implements the length-based chromosome/plasmid contig classification via the
    -c chromosome threshold and drives downstream chromosomal read removal
  repo: https://github.com/gbouras13/plassembler
- name: minimap2
  role: Maps long and short read sets to the Flye assembly so that classification
    into chromosome vs non-chromosome contigs can be used for read extraction (requires
    minimap2 >=2.11)
  repo: https://github.com/lh3/minimap2
- name: SAMtools
  role: Extracts reads mapping to non-chromosome contigs plus unmapped reads after
    the length classification has designated putative plasmid contigs
  repo: https://github.com/samtools/samtools
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad409
    title: 'Plassembler: an automated bacterial plasmid assembly tool'
  source_content_hash: sha256:282b82e4448d29ba994aa724e301c6550a3f544d37419adc1cf4c8e2c5edfbd2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/fenderglass/Flye
  tools:
  - Flye
  - Plassembler
  - SAMtools
  - minimap2
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# contig-length-classification

## Summary

Classifies contigs from a long-read-only assembly (e.g. Flye output from Plassembler) into chromosomal versus non-chromosome (putative plasmid) classes using a contig length threshold, so downstream read-extraction can retain plasmid reads while discarding chromosomal reads.

## When to use

Use when a long-read-only assembly (e.g. Flye nanopore assembly of a bacterial isolate) must be split into chromosome and putative-plasmid contigs before chromosomal read removal or plasmid read extraction.

## When NOT to use

- When the genome has a chromosome-length contig that is actually a plasmid (e.g. large megaplasmids near the chromosome threshold) or chromosomal fragments split across contigs — length alone will misclassify them
- When input is already a curated plasmid set (e.g. completed GenBank genomes with replicon annotations) — explicit replicon labels should be used instead of length heuristics
- When using the long-only kmer_mode, which the article describes as experimental and unvalidated especially on small plasmids

## Inputs

- Long-read FASTQ file (e.g. nanopore reads, run in --nano-hq mode)
- Flye long-read-only assembly contigs FASTA
- Chromosome length threshold (-c parameter)

## Outputs

- Set of chromosomal contigs (contigs meeting the length threshold)
- Set of non-chromosome (putative plasmid) contigs
- Downstream: filtered_plasmid_reads.fastq.gz (reads mapping to putative plasmid contigs plus unmapped reads)

## How to apply

First assemble long reads with Flye (Plassembler uses 'flye --nano-hq' for high-quality nanopore reads; Flye >=2.9 is required) to obtain an initial long-read-only assembly. Then classify the resulting contigs by length against the chromosome threshold supplied via the '-c' parameter: contigs meeting the threshold (largest contig(s)) are designated chromosomal, while non-chromosome contigs are treated as putative plasmids. This length-based classification is the decision point that controls which contigs downstream Minimap2/SAMtools extraction will target — reads mapping to non-chromosome contigs plus unmapped reads are kept, chromosomal reads are removed. The rationale (per Bouras et al., Bioinformatics 2023) is that plasmids are typically much shorter than the chromosome, so a length cutoff reliably separates replicons prior to Unicycler hybrid assembly of the filtered plasmid reads.

## Related tools

- **Flye** (Produces the long-read-only assembly whose contigs are classified by length (Plassembler requires Flye >=2.9; Raven >=1.8 is an optional alternative assembler)) — https://github.com/fenderglass/Flye
- **Plassembler** (Implements the length-based chromosome/plasmid contig classification via the -c chromosome threshold and drives downstream chromosomal read removal) — https://github.com/gbouras13/plassembler
- **minimap2** (Maps long and short read sets to the Flye assembly so that classification into chromosome vs non-chromosome contigs can be used for read extraction (requires minimap2 >=2.11)) — https://github.com/lh3/minimap2
- **SAMtools** (Extracts reads mapping to non-chromosome contigs plus unmapped reads after the length classification has designated putative plasmid contigs) — https://github.com/samtools/samtools

## Examples

```
flye --nano-hq input_longreads.fastq --out-dir flye_assembly --threads 8
```

## Evaluation signals

- Verify the classification partition is exhaustive: every Flye contig is assigned either to the chromosomal set (>= -c threshold) or the putative plasmid set, with no contig left unclassified
- Confirm the downstream filtered_plasmid_reads.fastq.gz contains reads aligned to the non-chromosome (putative plasmid) contigs plus unmapped reads, and no reads mapping solely to the designated chromosomal contig(s)
- Validate output FASTQs are well-formed with a validate_fastq-style check (sequence and quality lengths match per read)
- Check that plasmid contig lengths recovered after hybrid assembly are consistent with ground-truth plasmid sizes (e.g. 1934 bp in K. variicola INF345, 10697 bp in K. oxytoca MSB1 2C), indicating classification did not drop small plasmids
- Compare against Unicycler on the same inputs: correct classification should allow recovery of plasmids Unicycler misses (e.g. only 1 missed plasmid vs 7 for Unicycler in simulated benchmarking)

## Limitations

- Length thresholds are heuristics: plasmids close in size to the chromosome cutoff (or very large plasmids) may be misclassified as chromosomal and have their reads removed
- If Flye misses a plasmid entirely from the long-read assembly, classification alone cannot recover it — Plassembler also retains unmapped reads to recover possible plasmids Flye has missed
- Flye --deterministic is not deterministic between thread counts, so contig sets may vary across thread settings even with --deterministic
- The long-only kmer_mode alternative is described as experimental and unvalidated on more data, especially with small plasmids

## Evidence

- [other] Classify Flye contigs by length against the '-c' chromosome threshold to designate the largest/chromosomal contig(s) versus putative plasmid (non-chromosome) contigs.: [evidence span withheld — anchor status: not_found]
- [methods] Flye or Raven initial assembly: "`plassembler` uses Flye (Kolmogorov et al 2019) or optionally Raven (Vaser and Šikić 2021) to create an initial long-read only assembly."
- [methods] Extraction of plasmid and unmapped reads: "Reads that map to the non-chromosome contigs (i.e. the putative plasmids), and reads that are unmapped  (to recover possible plasmids Flye has missed) are extracted and combined."
- [abstract] Mapping approach for chromosomal read removal: "It achieves increased accuracy and computational efficiency compared to the existing gold standard tool Unicycler by removing chromosomal reads from the input read sets using a mapping approach"
