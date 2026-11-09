---
name: bits-per-base-alignment-scoring
description: Use when assigning taxonomy to reconstructed viral contigs from metagenomic
  assemblies using combined nucleotide (BBMap) and translated/protein (DIAMOND) alignments,
  especially for low-coverage or divergent viruses that defeat read- or kmer-based
  classifiers.
when_to_use_negative:
- Input is raw reads rather than assembled contigs — the bits-per-base scoring operates
  on improved contigs, not individual reads
- The analysis goal is host or non-viral organism classification — this engine explicitly
  filters to the superkingdom Virus and discards contigs mapping outside it
- A kmer/read-level classifier (e.g. Kraken-style) suffices — the article targets
  cases where reads, contigs, or kmers map equally well to different genomes and coverage
  is too low for confident assembly
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0659
tools:
- name: VirMAP (Virmap.pl)
  role: Implements the two-pass bits-per-base scoring, virus-superkingdom filtering,
    and rank-wise LCA classification
  repo: https://github.com/cmmr/VirMAP
- name: BBMap
  role: Produces the nucleotide alignments of improved contigs whose bit scores feed
    pass one
- name: DIAMOND
  role: Produces the translated/protein alignments whose scores are weighed tetrationally
    in pass two
  repo: https://github.com/bbuchfink/diamond
- name: BLAST+
  role: Provides reference alignment scoring for the Genbank database comparisons
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-05658-8
    title: Maximal viral information recovery from sequence data using VirMAP
  source_content_hash: sha256:9ca8810932d9df1b66de9cba1018c3506e116e9694f5624ad7ece249af417c45
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cmmr/VirMAP
  tools:
  - BBMap
  - BLAST+
  - DIAMOND
  - Perl
  - VirMAP
  - VirMAP (Virmap.pl)
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# bits-per-base-alignment-scoring

## Summary

VirMAP's two-pass taxonomic scoring engine: accumulate maximum bits-per-base scores per taxon in pass one, then compute pass-two relative weights (exponential for nucleotide, tetrational for protein alignments) followed by virus-superkingdom filtering and rank-wise LCA. This enables classification of low-coverage, divergent viral contigs independent of genome coverage or read overlap.

## When to use

Use when assigning taxonomy to reconstructed viral contigs from metagenomic assemblies using combined nucleotide (BBMap) and translated/protein (DIAMOND) alignments, especially for low-coverage or divergent viruses that defeat read- or kmer-based classifiers.

## When NOT to use

- Input is raw reads rather than assembled contigs — the bits-per-base scoring operates on improved contigs, not individual reads
- The analysis goal is host or non-viral organism classification — this engine explicitly filters to the superkingdom Virus and discards contigs mapping outside it
- A kmer/read-level classifier (e.g. Kraken-style) suffices — the article targets cases where reads, contigs, or kmers map equally well to different genomes and coverage is too low for confident assembly

## Inputs

- Improved contigs from VirMAP's improvement stage (dual nucleotide/protein assembly)
- VirMAP reference databases: Taxonomy.virmap, virDmnd.dmnd, virBbmap, gbBlastn, gbBlastx.dmnd built from Genbank viral (gbvrl) and phage (gbphg) divisions
- Nucleotide alignments from BBMap
- Translated/protein alignments from DIAMOND

## Outputs

- taxonomic_assignments.tsv (per-contig taxonomic assignments, scores, and LCA ranks)
- Set of filtered contigs: only those with a top pass-one taxon in the superkingdom Virus and >50% of aligning bases resolving to Virus

## How to apply

Align the improved dual-assembly contigs to VirMAP reference databases (Taxonomy.virmap, virDmnd.dmnd, virBbmap, gbBlastn, gbBlastx.dmnd) using BBMap for nucleotide and DIAMOND for translated/protein alignments. In pass one of the Virmap.pl algorithm, accumulate the maximum bit-score-per-base for every taxon aligned to each contig. In pass two, compute relative weights for all alignment events per contig, weighing nucleotide alignment scores exponentially and protein alignment scores tetrationally — this merging of nucleotide and protein information is what makes classification independent of coverage. Filter contigs whose top pass-one taxon is not in the superkingdom Virus, or where >50% of aligning bases do not resolve to the superkingdom Virus. For retained contigs, consider all pass-one taxa within a dynamically calculated radius per contig and run the rank-wise LCA algorithm, with majority thresholds computed dynamically from the database volume; write assignments to taxonomic_assignments.tsv.

## Related tools

- **VirMAP (Virmap.pl)** (Implements the two-pass bits-per-base scoring, virus-superkingdom filtering, and rank-wise LCA classification) — https://github.com/cmmr/VirMAP
- **BBMap** (Produces the nucleotide alignments of improved contigs whose bit scores feed pass one)
- **DIAMOND** (Produces the translated/protein alignments whose scores are weighed tetrationally in pass two) — https://github.com/bbuchfink/diamond
- **BLAST+** (Provides reference alignment scoring for the Genbank database comparisons)

## Examples

```
Virmap.pl --threads $(nproc) --readF /home/$USER/VirmapTest/SRR9875293_1.fastq --useMegahit --useBbnorm --sampleName SRR9875293
```

## Evaluation signals

- Every retained contig has a top pass-one taxon in the superkingdom Virus
- Every retained contig has >50% of its aligning bases resolving to the superkingdom Virus
- taxonomic_assignments.tsv exists with per-contig taxonomic assignment, score, and LCA rank columns for all filtered contigs
- Pass-two weighting distinguishes nucleotide (exponential) from protein (tetrational) score events for each contig
- LCA majority thresholds are computed dynamically from the database volume, and the radius per contig covers all pass-one taxa it should include

## Limitations

- The dynamically calculated LCA radius and majority threshold formulas are not fully specified in the paper's main text
- Supplementary Data 14 (sensitivity/specificity tuning parameters) is referenced but not provided in the document
- Supplementary Note 4 describing the 16-division custom Genbank database is not provided; Genbank database snapshot/version or download date is not specified
- Classification depends on the Genbank viral and phage database coverage — insufficient coverage of the viral tree of life is an acknowledged problem of alignment-based approaches

## Evidence

- [intro] VirMAP uses a taxonomic classification algorithm that is centered around a bits-per-base scoring system.: [evidence span withheld — anchor status: not_found]
- [discussion] Virus-superkingdom filter criterion: "A contig is filtered if its top scoring pass one taxon is not in the superkingdom virus, or if > 50% of its aligning bases do not resolve to the superkingdom virus."
- [discussion] Dynamically calculated LCA radius: "All pass one taxa scoring within a dynamically calculated radius per contig are considered in a rank-wise lowest common ancestor (LCA) algorithm."
- [discussion] Nucleotide and protein alignment tools: "using BBMap19 and DIAMOND20, respectively"
- [abstract] Merging rationale for coverage-independent classification: "merge nucleotide and protein information to taxonomically classify viral reconstructions independent of genome coverage or read overlap"
