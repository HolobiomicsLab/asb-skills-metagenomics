---
name: short-read-reference-mapping-with-bowtie2
description: Use when you need to quantify how completely a raw or subsetted set of
  short paired-end reads (FASTQ) covers a specific reference contig (FASTA), e.g.
  tracking cas operon coverage across read-subset expansion cycles.
when_to_use_negative:
- The reads are already assembled contigs or long reads (PacBio/ONT) — bowtie2 end-to-end
  short-read paired-end mapping is designed for short Illumina reads, not de novo
  contig or long-read alignment
- You need annotation or assembly of the CRISPR/Cas locus rather than a coverage quantification
  — use CasCollect's own assembly (SPAdes/metaSPAdes) and CRISPRCasFinder annotation
  steps instead
- The reference contig is not the target of interest (e.g. no cas operon reference
  FASTA has been extracted yet) — mapping cannot proceed without a defined reference
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: bowtie2
  role: Maps seed read subsets and the complete raw dataset onto the cas operon reference
    in end-to-end paired-end mode
- name: SAMtools
  role: Extracts per-base pileup coverage ('samtools depth -a') from the alignment
    so the covered fraction of every operon base can be computed
- name: CasCollect
  role: Produces the seed read subsets whose coverage is tracked, via vsearch-driven
    expansion cycles at the default 95% identity match with cycle counts overridden
    by '-cycle N'
  repo: https://github.com/sandialabs/CasCollect
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/nargab/lqaa063
    title: 'CasCollect: targeted assembly of CRISPR-associated operons from high-throughput
      sequencing data'
  source_content_hash: sha256:ef94a5109fb8e501213a0be8643f19fc67f028459f6110fc65a40d40db33ea56
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sandialabs/CasCollect
  tools:
  - CasCollect
  - HMMER
  - SAMtools
  - bowtie2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# Short-read reference mapping with bowtie2

## Summary

Maps short paired-end sequencing reads (e.g. CasCollect seed read subsets) onto a reference FASTA such as the largest ground water cas operon contig using bowtie2 in end-to-end paired-end mode, then quantifies per-base coverage with 'samtools depth -a' to compute the fraction of reference bases covered.

## When to use

Use when you need to quantify how completely a raw or subsetted set of short paired-end reads (FASTQ) covers a specific reference contig (FASTA), e.g. tracking cas operon coverage across read-subset expansion cycles.

## When NOT to use

- The reads are already assembled contigs or long reads (PacBio/ONT) — bowtie2 end-to-end short-read paired-end mapping is designed for short Illumina reads, not de novo contig or long-read alignment
- You need annotation or assembly of the CRISPR/Cas locus rather than a coverage quantification — use CasCollect's own assembly (SPAdes/metaSPAdes) and CRISPRCasFinder annotation steps instead
- The reference contig is not the target of interest (e.g. no cas operon reference FASTA has been extracted yet) — mapping cannot proceed without a defined reference

## Inputs

- Paired-end FASTQ reads (e.g. SRA accession SRR10598175 ground water metagenome, full dataset or CasCollect seed subsets)
- Reference FASTA of the largest ground water cas operon contig
- bowtie2 index built from the reference FASTA

## Outputs

- SAM/BAM alignment of each read set against the cas operon reference
- Per-base depth table from 'samtools depth -a' covering every reference position
- Percent operon coverage per read set (CSV, e.g. cas_operon_coverage_progression.csv with coverage vs. cycle count and full-dataset baseline)

## How to apply

Index the reference FASTA (the extracted cas operon contig) with bowtie2-build, then map each read set in end-to-end paired-end mode with bowtie2 (-x reference -1 R1.fq -2 R2.fq). Here the read sets are the seed read subsets produced by CasCollect after each vsearch-driven expansion cycle (default 95% identity match, cycle count overridden with '-cycle N' from 0 to 5) plus the complete raw SRR10598175 dataset as a baseline. Convert the bowtie2 SAM output and run 'samtools depth -a' to obtain per-base pileup depth at every reference position, including zero-coverage positions. Compute the fraction of operon bases with nonzero depth per read set and tabulate percent coverage versus cycle count (e.g. cas_operon_coverage_progression.csv) to test whether expansion infills coverage gaps.

## Related tools

- **bowtie2** (Maps seed read subsets and the complete raw dataset onto the cas operon reference in end-to-end paired-end mode)
- **SAMtools** (Extracts per-base pileup coverage ('samtools depth -a') from the alignment so the covered fraction of every operon base can be computed)
- **CasCollect** (Produces the seed read subsets whose coverage is tracked, via vsearch-driven expansion cycles at the default 95% identity match with cycle counts overridden by '-cycle N') — https://github.com/sandialabs/CasCollect

## Examples

```
bowtie2 -x cas_operon_ref -1 SRR10598175_1.fq -2 SRR10598175_2.fq -S aln.sam; samtools depth -a aln.sam > depth.txt
```

## Evaluation signals

- Reproduced coverage progression matches reported values within a few percent: ~33.2% for the initial seed set, 96.1% after one expansion cycle, and 99.5% after two cycles
- Coverage rises monotonically with expansion cycle count and reaches saturation (no remaining gaps) by cycle two
- 'samtools depth -a' output contains one row for every reference position, including zero-depth bases, so the covered fraction is computed over the full operon length
- The full raw dataset mapping serves as a baseline and should meet or exceed the highest expanded-subset coverage
- Paired-end read pairing is preserved (end-to-end paired-end mode) so both mates of each read contribute to the pileup

## Limitations

- The initial seed read subset covers only a fraction of the operon (~33.2% for the ground water cas operon); coverage conclusions depend on running enough expansion cycles
- Expansion sensitivity is governed by the vsearch percent-match threshold (default 95%, adjustable with -match) and default five cycles (adjustable with -cycle); changing these alters which reads are mapped
- No software version numbers for the pipeline tools are given in the article, so exact numeric reproduction may vary across bowtie2/SAMtools versions

## Evidence

- [other] Map the seed read subset from each cycle count, as well as the complete raw dataset, onto the cas operon reference with bowtie2 (end-to-end paired-end mode).: [evidence span withheld — anchor status: not_found]
- [other] samtools depth per-base coverage extraction: [evidence span withheld — anchor status: not_found]
- [results] default 95% identity match: "The default identity match has been set to a moderately high value of 95%, however, this can be adjusted with the -match command."
- [results] default five expansion cycles: "The default number of these read subset expansion cycles is set at five and can be adjusted with the -cycle command."
