---
name: nucleotide-protein-alignment-integration
description: Use when metagenomic contigs (e.g. VirMAP improved dual assembly) must
  be taxonomically classified using both nucleotide and protein alignment evidence,
  independent of genome coverage or read overlap.
when_to_use_negative:
- Input is raw metagenomic reads rather than assembled/improved contigs — VirMAP's
  classification stage operates on the improved dual assembly
- Reference database coverage of the target viral clade is insufficient; the paper
  notes insufficient database coverage of the viral tree of life limits existing approaches
- Goal is per-read (not per-contig) classification; current methods classify individual
  reads and/or contigs through alignments or kmer analyses, but this skill targets
  contig-level LCA assignment
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: VirMAP
  role: Implements the two-pass Virmap.pl bits-per-base taxonomic classification,
    viral superkingdom filtering, and rank-wise LCA algorithm
  repo: https://github.com/cmmr/VirMAP
- name: BBMap
  role: Performs nucleotide alignments of improved contigs against the viral nucleotide
    reference database
- name: DIAMOND
  role: Performs translated/protein alignments of contigs against the viral protein
    database (virDmnd.dmnd, gbBlastx.dmnd)
  repo: https://github.com/bbuchfink/diamond
- name: BLAST+
  role: Supports nucleotide/protein BLAST databases (gbBlastn, gbBlastx.dmnd) used
    by the classification stage
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-05658-8
    title: Maximal viral information recovery from sequence data using VirMAP
  source_content_hash: sha256:7a2008cc64b336c257caff5bea78f5f7f228bf7cf6cef08d35fde57069f649e8
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
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# nucleotide-protein-alignment-integration

## Summary

Taxonomically classify reconstructed viral contigs by merging nucleotide alignments (BBMap) and translated/protein alignments (DIAMOND) into a two-pass bits-per-base scoring system, followed by superkingdom-Virus filtering and a rank-wise LCA assignment.

## When to use

Use when metagenomic contigs (e.g. VirMAP improved dual assembly) must be taxonomically classified using both nucleotide and protein alignment evidence, independent of genome coverage or read overlap.

## When NOT to use

- Input is raw metagenomic reads rather than assembled/improved contigs — VirMAP's classification stage operates on the improved dual assembly
- Reference database coverage of the target viral clade is insufficient; the paper notes insufficient database coverage of the viral tree of life limits existing approaches
- Goal is per-read (not per-contig) classification; current methods classify individual reads and/or contigs through alignments or kmer analyses, but this skill targets contig-level LCA assignment

## Inputs

- Improved contigs from the VirMAP assembly/improvement stage (FASTA)
- VirMAP reference databases: Taxonomy.virmap, virDmnd.dmnd, virBbmap, gbBlastn, gbBlastx.dmnd
- Genbank viral (gbvrl) and phage (gbphg) division reference database spanning 16 divisions

## Outputs

- taxonomic_assignments.tsv with per-contig taxonomic assignments, bits-per-base scores, and LCA ranks
- Set of filtered contigs retained only if viral superkingdom criteria are met

## How to apply

Align improved contigs to the VirMAP reference databases (Taxonomy.virmap, virDmnd.dmnd, virBbmap, gbBlastn, gbBlastx.dmnd), using BBMap for nucleotide alignments and DIAMOND for translated/protein alignments. Run pass one of Virmap.pl, accumulating the maximum bit-score-per-base for every taxon aligned to each contig; run pass two computing relative weights per contig, weighting nucleotide alignment scores exponentially and protein alignment scores tetrationally. Filter each contig if its top-scoring pass-one taxon is not in the superkingdom Virus or if >50% of its aligning bases do not resolve to the superkingdom Virus. For retained contigs, consider all pass-one taxa scoring within a dynamically calculated radius and run the rank-wise LCA algorithm with majority thresholds computed dynamically from the database volume, writing per-contig assignments, scores, and LCA ranks to taxonomic_assignments.tsv.

## Related tools

- **VirMAP** (Implements the two-pass Virmap.pl bits-per-base taxonomic classification, viral superkingdom filtering, and rank-wise LCA algorithm) — https://github.com/cmmr/VirMAP
- **BBMap** (Performs nucleotide alignments of improved contigs against the viral nucleotide reference database)
- **DIAMOND** (Performs translated/protein alignments of contigs against the viral protein database (virDmnd.dmnd, gbBlastx.dmnd)) — https://github.com/bbuchfink/diamond
- **BLAST+** (Supports nucleotide/protein BLAST databases (gbBlastn, gbBlastx.dmnd) used by the classification stage)

## Examples

```
Virmap.pl --threads $(nproc) --useMegahit --useBbnorm --sampleName SRR9875293 --readF /home/$USER/VirmapTest/SRR9875293_1.fastq
```

## Evaluation signals

- Every retained contig's top-scoring pass-one taxon is in the superkingdom Virus
- More than 50% of each retained contig's aligning bases resolve to the superkingdom Virus
- Per-contig pass-one scores reflect maximum bit-score-per-base accumulation across all aligned taxa
- Pass-two relative weights computed for all alignment events, with exponential weighting for nucleotide and tetrational weighting for protein scores
- taxonomic_assignments.tsv contains assignment, score, and LCA rank for each filtered contig, with LCA taxa drawn from the dynamically calculated radius

## Limitations

- Dynamically calculated LCA radius and majority threshold formulas are not fully specified in the main text
- Classification depends on the custom Genbank database spanning 16 divisions; no Genbank snapshot/version or download date is specified
- High background noise and signal crosstalk confound viral taxonomic characterization from metagenomic data

## Evidence

- [intro] bits-per-base scoring: "a taxonomic classification algorithm that is centered around a bits-per-base scoring system"
- [discussion] filtering criterion: "A contig is filtered if its top scoring pass one taxon is not in the superkingdom virus, or if > 50% of its aligning bases do not resolve to the superkingdom virus."
- [discussion] aligners: "using BBMap19 and DIAMOND20, respectively"
- [discussion] LCA algorithm: "All pass one taxa scoring within a dynamically calculated radius per contig are considered in a rank-wise lowest common ancestor (LCA) algorithm."
- [abstract] rationale: "merge nucleotide and protein information to taxonomically classify viral reconstructions independent of genome coverage or read overlap"
- [methods] invocation: "`Virmap.pl --threads $(nproc) --readF /home/$USER/VirmapTest/SRR9875293_1.fastq"
