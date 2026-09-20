---
name: rank-wise-lowest-common-ancestor-assignment
description: Use when improved viral contigs must be taxonomically classified with
  per-contig scores and LCA ranks, especially for low-coverage or divergent viruses
  where single -hit assignment is unreliable.
when_to_use_negative:
- Input is raw reads or pre-assembly contigs that have not passed through VirMAP's
  dereplication, normalization, assembly, and improvement stages
- The goal is host or non-viral sequence classification — the algorithm filters contigs
  to the superkingdom Virus and discards everything else
- Only a single best-hit taxonomic label is needed without ambiguity handling — a
  simple top-hit BLAST/DIAMOND report suffices and avoids the two-pass scoring overhead
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: VirMAP
  role: Implements the two-pass bits-per-base scoring and rank-wise LCA classification
    in Virmap.pl (Perl)
  repo: https://github.com/cmmr/VirMAP
- name: BBMap
  role: Produces the nucleotide alignments of improved contigs to the reference nucleotide
    database used for pass-one scoring
- name: DIAMOND
  role: Produces the translated/protein alignments whose scores are weighed tetrationally
    in pass two
  repo: https://github.com/bbuchfink/diamond
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-018-05658-8
    title: Maximal viral information recovery from sequence data using VirMAP
  source_content_hash: sha256:2baf0b09d82e044798b8b05c1f6aa0308603b4030e00b66f3c0acd3433f306ce
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
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# rank-wise-lowest-common-ancestor-assignment

## Summary

Assigns taxonomy to viral contigs by running Virmap.pl's two-pass bits-per-base scoring followed by a rank-wise lowest common ancestor (LCA) over all pass-one taxa falling within a dynamically calculated radius, with majority thresholds computed from the reference database volume.

## When to use

Use when improved viral contigs must be taxonomically classified with per-contig scores and LCA ranks, especially for low-coverage or divergent viruses where single best-hit assignment is unreliable.

## When NOT to use

- Input is raw reads or pre-assembly contigs that have not passed through VirMAP's dereplication, normalization, assembly, and improvement stages
- The goal is host or non-viral sequence classification — the algorithm filters contigs to the superkingdom Virus and discards everything else
- Only a single best-hit taxonomic label is needed without ambiguity handling — a simple top-hit BLAST/DIAMOND report suffices and avoids the two-pass scoring overhead

## Inputs

- Improved contigs from the VirMAP improvement stage (FASTA)
- VirMAP reference databases: Taxonomy.virmap, virDmnd.dmnd, virBbmap, gbBlastn, gbBlastx.dmnd (built from Genbank viral gbvrl and phage gbphg divisions)
- BBMap nucleotide alignments of contigs to the nucleotide database
- DIAMOND translated/protein alignments of contigs to the protein database

## Outputs

- taxonomic_assignments.tsv with per-contig taxonomic assignments, bits-per-base scores, and LCA ranks
- Set of retained viral contigs passing the viral superkingdom filter

## How to apply

Align improved contigs against the VirMAP reference databases (Taxonomy.virmap, virDmnd.dmnd, virBbmap, gbBlastn, gbBlastx.dmnd) using BBMap for nucleotide and DIAMOND for translated/protein alignments. In pass one of Virmap.pl, accumulate the maximum bits-per-base score for every taxon aligned to each contig. In pass two, compute relative weights for all alignment events per contig, weighing nucleotide alignment scores exponentially and protein alignment scores tetrationally, merging nucleotide and protein evidence. Before LCA assignment, filter contigs: remove any whose top pass-one taxon is outside the superkingdom Virus or where >50% of aligning bases do not resolve to Virus. For retained contigs, consider all pass-one taxa scoring within a dynamically calculated radius per contig and run the rank-wise LCA algorithm with majority thresholds computed dynamically from the database volume, then write per-contig assignments, scores, and LCA ranks to taxonomic_assignments.tsv.

## Related tools

- **VirMAP** (Implements the two-pass bits-per-base scoring and rank-wise LCA classification in Virmap.pl (Perl)) — https://github.com/cmmr/VirMAP
- **BBMap** (Produces the nucleotide alignments of improved contigs to the reference nucleotide database used for pass-one scoring)
- **DIAMOND** (Produces the translated/protein alignments whose scores are weighed tetrationally in pass two) — https://github.com/bbuchfink/diamond

## Examples

```
Virmap.pl --threads $(nproc) --readF /home/$USER/VirmapTest/SRR9875293_1.fastq --useMegahit --useBbnorm --sampleName SRR9875293
```

## Evaluation signals

- Every retained contig's top-scoring pass-one taxon is in the superkingdom Virus
- For every retained contig, >50% of its aligning bases resolve to the superkingdom Virus
- taxonomic_assignments.tsv contains one row per retained contig with taxon, bits-per-base score, and LCA rank populated
- Contigs whose top pass-one taxon falls outside Virus are absent from the retained set (correctly filtered)
- On validated mock communities (e.g. Viral Mock Community PRJNA431646, NetoVir PRJNA319556), assigned taxa match expected viral compositions including RNA viruses, DNA viruses, and bacteriophages

## Limitations

- The dynamically calculated LCA radius per contig and the formulas for dynamically computed majority thresholds are not fully specified in the article text
- Classification is bounded by Genbank viral/phage database coverage (gbvrl/gbphg, 16 divisions); the Genbank snapshot version is not stated in the article
- Classification accuracy depends on sufficient assembly coverage; VirMAP was specifically built to address low-coverage and highly divergent viruses, but very short contigs may still be filtered by the >50% aligning-base criterion

## Evidence

- [intro] Bits-per-base scoring and LCA definition: "a taxonomic classification algorithm that is centered around a bits-per-base scoring system"
- [discussion] Pass-one filter criterion: "A contig is filtered if its top scoring pass one taxon is not in the superkingdom virus, or if > 50% of its aligning bases do not resolve to the superkingdom virus."
- [discussion] LCA radius and rank-wise algorithm: "All pass one taxa scoring within a dynamically calculated radius per contig are considered in a rank-wise lowest common ancestor (LCA) algorithm."
- [abstract] Merged nucleotide and protein evidence: "merge nucleotide and protein information to taxonomically classify viral reconstructions independent of genome coverage or read overlap"
- [discussion] Aligner choices: "using BBMap19 and DIAMOND20, respectively"
