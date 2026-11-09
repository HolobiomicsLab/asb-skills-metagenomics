---
name: viral-taxonomic-classification
description: Use when improved dual-assembly viral contigs need taxonomic labels and
  read-level or simple BLAST--hit classification is expected to fail due to low coverage,
  divergent viruses, or ambiguous mapping.
when_to_use_negative:
- Input is a set of unassembled reads requiring taxonomic binning only (VirMAP classifies
  reconstructed contigs, not individual reads)
- The analysis target is non-viral taxa or host/microbiome classification outside
  the viral superkingdom
- Genome coverage is high and non-divergent, making a standard MEGAHIT+BLASTn/BLASTx
  approach sufficient (VirMAP's advantage is largest for low coverage and divergent
  viruses)
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0781
- http://edamontology.org/topic_3174
tools:
- name: VirMAP
  role: Implements the two-pass bits-per-base classification algorithm, viral superkingdom
    filtering, and rank-wise LCA (Virmap.pl)
  repo: https://github.com/cmmr/VirMAP
- name: BBMap
  role: Performs nucleotide alignments of improved contigs to the viral nucleotide
    database (virBbmap, gbBlastn)
- name: DIAMOND
  role: Performs translated/protein alignments of contigs to the protein database
    (virDmnd.dmnd, gbBlastx.dmnd)
  repo: https://github.com/bbuchfink/diamond
- name: Perl
  role: Language in which VirMAP (Virmap.pl) is implemented
provenance:
  source_task_ids:
  - task_008
  - task_001
  - task_003
  - task_005
  source_papers:
  - doi: 10.1038/s41467-018-05658-8
    title: Maximal viral information recovery from sequence data using VirMAP
  source_content_hash: sha256:1807e52bbf24068cd01c42afd4902acc48141bc73379aca5527966083c932541
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cmmr/VirMAP
  tools:
  - BBMap
  - BLAST+
  - DIAMOND
  - MEGAHIT
  - Perl
  - Statistics::Basic
  - VirMAP
  - fasterq-dump
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0781
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# viral-taxonomic-classification

## Summary

Taxonomic classification of reconstructed viral contigs using VirMAP's two-pass bits-per-base scoring system, which merges nucleotide (BBMap) and translated/protein (DIAMOND) alignment information against the Genbank viral/phage database before rank-wise LCA assignment.

## When to use

Use when improved dual-assembly viral contigs need taxonomic labels and read-level or simple BLAST-best-hit classification is expected to fail due to low coverage, divergent viruses, or ambiguous mapping.

## When NOT to use

- Input is a set of unassembled reads requiring taxonomic binning only (VirMAP classifies reconstructed contigs, not individual reads)
- The analysis target is non-viral taxa or host/microbiome classification outside the viral superkingdom
- Genome coverage is high and non-divergent, making a standard MEGAHIT+BLASTn/BLASTx approach sufficient (VirMAP's advantage is largest for low coverage and divergent viruses)

## Inputs

- Improved viral contigs from the VirMAP improvement stage
- Pre-built VirMAP databases: Taxonomy.virmap, virDmnd.dmnd, virBbmap, gbBlastn, gbBlastx.dmnd
- Paired-end FASTQ reads (e.g. SRA accessions such as SRR9875293 or SRR3458562)

## Outputs

- taxonomic_assignments.tsv with per-contig taxonomic assignments, scores, and LCA ranks
- Filtered table of taxa IDs (e.g. minimum 1000 bits/taxa ID) and raw unfiltered output (e.g. 300 bits/contig minimum)

## How to apply

Align improved contigs to the VirMAP reference databases (Taxonomy.virmap, virDmnd.dmnd, virBbmap, gbBlastn, gbBlastx.dmnd) with BBMap for nucleotide and DIAMOND for translated alignments. Run pass one of Virmap.pl to accumulate the maximum bits-per-base score for every taxon aligned to each contig. In pass two, compute relative weights for all alignment events per contig, weighting nucleotide scores exponentially and protein scores tetrationally. Filter a contig if its top-scoring pass-one taxon is outside the superkingdom Virus or if >50% of its aligning bases do not resolve to Virus. For retained contigs, consider all pass-one taxa within a dynamically calculated radius and run a rank-wise LCA with majority thresholds computed from the database volume.

## Related tools

- **VirMAP** (Implements the two-pass bits-per-base classification algorithm, viral superkingdom filtering, and rank-wise LCA (Virmap.pl)) — https://github.com/cmmr/VirMAP
- **BBMap** (Performs nucleotide alignments of improved contigs to the viral nucleotide database (virBbmap, gbBlastn))
- **DIAMOND** (Performs translated/protein alignments of contigs to the protein database (virDmnd.dmnd, gbBlastx.dmnd)) — https://github.com/bbuchfink/diamond
- **Perl** (Language in which VirMAP (Virmap.pl) is implemented)

## Examples

```
Virmap.pl --threads $(nproc) --useMegahit --useBbnorm --taxaJson /scratch/VirmapDb/Taxonomy.virmap --virDmnd /scratch/VirmapDb/virDmnd.dmnd --virBbmap /scratch/VirmapDb/virBbmap --gbBlastn /scratch/VirmapDb/gbBlastn --gbBlastx /scratch/VirmapDb/gbBlastx.dmnd
```

## Evaluation signals

- Every retained contig has a top pass-one taxon in the superkingdom Virus
- More than 50% of each retained contig's aligning bases resolve to the superkingdom Virus
- On the Viral Mock Community (PRJNA431646), expect precision ≈0.88, recall = 1.0, and F-score ≈0.94 across the seven intended constituents
- On the NetoVir mock virome (SRR3458562), ~37.04% of reads classified viral across 10 lineages with R² = 0.9972 against Conceição-Neto per-lineage proportions
- taxonomic_assignments.tsv exists and is populated with per-contig scores and LCA ranks

## Limitations

- LCA radius and majority threshold formulas are dynamically calculated from database volume and are not fully specified in the paper text
- Supplementary Data 14 for tuning sensitivity/specificity tradeoff and Supplementary Note 4 describing the 16-division custom Genbank database are not provided in the main document
- No software versions stated for VirMAP, BBMap, DIAMOND, MEGAHIT, or Tadpole; Genbank database snapshot/version not specified
- Precision on the mock community was lowered by a cell-culture contaminant (bosavirus MS-2016a), illustrating sensitivity to contaminant detection

## Evidence

- [abstract] merge nucleotide and protein information to taxonomically classify viral reconstructions independent of genome coverage or read overlap: "merge nucleotide and protein information to taxonomically classify viral reconstructions independent of genome coverage or read overlap"
- [intro] bits-per-base scoring: "a taxonomic classification algorithm that is centered around a bits-per-base scoring system"
- [discussion] superkingdom virus filtering: "A contig is filtered if its top scoring pass one taxon is not in the superkingdom virus, or if > 50% of its aligning bases do not resolve to the superkingdom virus."
- [discussion] LCA radius: "All pass one taxa scoring within a dynamically calculated radius per contig are considered in a rank-wise lowest common ancestor (LCA) algorithm."
- [methods] Virmap.pl invocation: "`Virmap.pl --threads $(nproc) --readF /home/$USER/VirmapTest/SRR9875293_1.fastq"
