---
name: jaccard-distance-genomic-read-set-filtering
description: Use when reads have been assigned to viral genomes from nucleotide and
  translated alignments against Genbank gbvrl/gbphage and a genome's read set shows
  high non-unique read membership relative to larger sets, causing cross-genome ambiguity
  that would corrupt per-genome pileup assembly.
when_to_use_negative:
- The goal is classification of individual reads or contigs by taxonomic scoring rather
  than removing cross-genome read-set ambiguity before per-genome assembly.
- Input has not yet been assigned to viral genomes via both nucleotide and translated
  alignments — the Jaccard filter operates on per-genome read sets, not raw FASTQ
  files.
- Viruses are absent from or under-represented in Genbank gbvrl/gbphage, since the
  mapping assembly (and hence this filter) depends on clustered alignments to that
  reference database.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0114
- http://edamontology.org/topic_3174
tools:
- name: VirMAP
  role: Perl-implemented pipeline that performs the per-genome read assignment, computes
    Jaccard-distance filtering of genomic read sets, and carries non-filtered sets
    into pileup assembly and super-scaffold merging
  repo: https://github.com/cmmr/VirMAP
- name: BBMap
  role: Nucleotide aligner (90% identity / 5% aligning-bases threshold) producing
    the nucleotide-alignment read assignments whose per-genome sets are subject to
    Jaccard filtering
- name: DIAMOND
  role: Translated aligner (20% identity / 8% aligning-bases threshold) producing
    the amino-acid-alignment read assignments that feed the same per-genome read sets
  repo: https://github.com/bbuchfink/diamond
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-018-05658-8
    title: Maximal viral information recovery from sequence data using VirMAP
  source_content_hash: sha256:137e07a6d6b6580a0ba3d6c3df72a010267135becd0ad257cb5df5cca92269c4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cmmr/VirMAP
  tools:
  - BBMap
  - DIAMOND
  - Perl
  - VirMAP
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0114
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# Jaccard-distance genomic read set filtering

## Summary

VirMAP's intermediate filtering stage that drops genomically ambiguous read sets: after reads are assigned to viral genomes via nucleotide (BBMap) and translated (DIAMOND) alignments, read sets showing high numbers of non-unique read membership relative to larger sets are filtered by Jaccard distance before naive pileup assembly and super-scaffold construction.

## When to use

Use when reads have been assigned to viral genomes from nucleotide and translated alignments against Genbank gbvrl/gbphage and a genome's read set shows high non-unique read membership relative to larger sets, causing cross-genome ambiguity that would corrupt per-genome pileup assembly.

## When NOT to use

- The goal is classification of individual reads or contigs by taxonomic scoring rather than removing cross-genome read-set ambiguity before per-genome assembly.
- Input has not yet been assigned to viral genomes via both nucleotide and translated alignments — the Jaccard filter operates on per-genome read sets, not raw FASTQ files.
- Viruses are absent from or under-represented in Genbank gbvrl/gbphage, since the mapping assembly (and hence this filter) depends on clustered alignments to that reference database.

## Inputs

- Per-genome read sets assigned by nucleotide alignments to the Genbank viral (gbvrl) division
- Per-genome read sets assigned by translated alignments (DIAMOND) to the Genbank viral protein database
- Dereplicated and digitally normalized paired-end input reads (BBTools bbnorm, default parameters)

## Outputs

- Set of non-filtered genomic read sets eligible for naive pileup assembly per genome
- Set of excluded/ambiguous genomic read sets that will not receive a super-scaffold

## How to apply

First assign processed reads to viral genomes by both nucleotide alignments (BBMap against Genbank gbvrl/gbphage nucleotide sequences) and translated alignments (DIAMOND against the viral protein database), yielding one read set per genome. Compute Jaccard distance between genomic read sets and filter a genome's read set when high numbers of non-unique read membership are observed relative to larger sets — the rationale is that reads mapping equally well to different genomes (ambiguous mapping, a core limitation VirMAP addresses) signal crosstalk that would corrupt the subsequent naive pileup reconstruction of that genome. Only non-filtered genome read sets proceed to individual reconstruction and to the tiered merge in which reconstructed protein coding sequences are tiled on top of the originating genome's nucleotide scaffold, with nucleotide precedence resolving disagreements. The filter is a precision safeguard, not a primary classifier: it acts on set-level overlap, complementing VirMAP's bits-per-base taxonomic classification and the pass-one taxon (>50% aligning bases resolving to superkingdom virus) contig filters.

## Related tools

- **VirMAP** (Perl-implemented pipeline that performs the per-genome read assignment, computes Jaccard-distance filtering of genomic read sets, and carries non-filtered sets into pileup assembly and super-scaffold merging) — https://github.com/cmmr/VirMAP
- **BBMap** (Nucleotide aligner (90% identity / 5% aligning-bases threshold) producing the nucleotide-alignment read assignments whose per-genome sets are subject to Jaccard filtering)
- **DIAMOND** (Translated aligner (20% identity / 8% aligning-bases threshold) producing the amino-acid-alignment read assignments that feed the same per-genome read sets) — https://github.com/bbuchfink/diamond

## Examples

```
Virmap.pl --threads $(nproc) --readF /home/$USER/VirmapTest/SRR9875293_1.fastq --useMegahit --useBbnorm --sampleName SRR9875293
```

## Evaluation signals

- Each non-filtered genome yields exactly one merged nucleotide super-scaffold in virmap_superscaffolds.fasta; genomes whose read sets were Jaccard-filtered must be absent.
- Filtering must be applied only after read assignment from both nucleotide and translated alignments has produced a read set per genome — filter output on raw or pre-assignment reads is invalid.
- Filtered sets should be those with high numbers of non-unique read membership relative to larger sets; unambiguous genomes with distinct read membership must survive.
- Downstream, reconstructed protein coding sequences (CDS-derived protein calls) tile onto nucleotide scaffolds of non-filtered genomes only, with nucleotide precedence resolving disagreements.
- The output FASTA parses as valid FASTA and each super-scaffold corresponds to a single non-filtered genome.

## Limitations

- The exact Jaccard distance threshold or formula is not specified in the article text; sensitivity/specificity tuning parameters are only in Supplementary Data 14, which is referenced but not provided here.
- The filter depends on Genbank gbvrl/gbphage coverage; highly divergent viruses with insufficient database coverage may produce ambiguous sets or be missed entirely.
- No software versions are stated for VirMAP, BBMap, or DIAMOND, so reproducibility of the filtering behavior depends on the repository state at run time.

## Evidence

- [discussion] Reads are assigned to viral genomes by nucleotide and translated alignments resulting in a set of reads per genome. Genomic read sets are then filtered by Jaccard distance: "Reads are assigned to viral genomes by nucleotide and translated alignments resulting in a set of reads per genome. Genomic read sets are then filtered by Jaccard distance"
- [discussion] Jaccard filter trigger condition: "Genomic read sets are then filtered by Jaccard distance if high numbers of non-unique read membership are observed relative to larger sets."
- [discussion] Downstream use of non-filtered genomes: "Each nucleotide and amino acid entry associated with a non-filtered genome is individually reconstructed using a naive pileup assembly."
- [intro] Motivation: ambiguous mapping across genomes: "These approaches are all restricted by similar problems including: reads, contigs, or kmers mapping equally well to different genomes"
