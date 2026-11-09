---
name: paired-end-read-alignment
description: Use when metagenomic FASTQ (paired-end WGSIM-simulated or real shotgun)
  reads must be mapped to a bacterial reference set to produce unique/split read BAMs
  for HGT breakpoint detection.
when_to_use_negative:
- When the goal is de novo assembly or strain typing without a reference — BWA mapping
  against a fixed reference set will miss reads from genomes absent from the 16,093-species
  reference
- When analyzing transcript-level fusions or isoforms (STAR-Fusion's role in the paper),
  not genomic HGT breakpoints from shotgun DNA reads
- When only ancient/lineage HGT inference from single genomes is needed rather than
  breakpoint-level detection from population shotgun data
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: BWA
  role: Maps paired-end shotgun reads against the reference genomes to identify donors,
    receptors, and mapped-read coverage
  repo: https://github.com/lichen2018/LEMON
- name: samtools
  role: Filters the aligned BAM to unique reads (view -q 20) required by LEMON's get_raw_bkp.py
  repo: https://github.com/samtools/htslib
- name: LUMPY
  role: Provides extractSplitReads_BwaMem for split-read extraction; also run with
    default parameters as the comparison breakpoint caller
  repo: https://github.com/lichen2018/LEMON
- name: WGSIM
  role: Generates the paired-end reads (2X/5X/70X) from simulated local strains before
    alignment
- name: HTSlib
  role: Core BAM library dependency (v1.9) that LEMON's getAccBkp is compiled against
  repo: https://github.com/samtools/htslib
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1186/s12859-019-3301-8
    title: 'LEMON: a method to construct the local strains at horizontal gene transfer
      sites in gut metagenomics'
  source_content_hash: sha256:408ca98616379c015ce154fe2def8d6f07a196aa809bfd7c773ac1c9ce4f4178
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen2018/LEMON
  tools:
  - BWA
  - HTSlib
  - LEMON
  - LUMPY
  - WGSIM
  - samtools
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# paired-end-read-alignment

## Summary

Aligns paired-end shotgun metagenomic reads against a reference set of microbiome genomes using BWA, then partitions the mapped BAM into unique-read and split-read files that feed LEMON's HGT breakpoint detection and LUMPY benchmarking.

## When to use

Use when metagenomic FASTQ (paired-end WGSIM-simulated or real shotgun) reads must be mapped to a bacterial reference set to produce unique/split read BAMs for HGT breakpoint detection.

## When NOT to use

- When the goal is de novo assembly or strain typing without a reference — BWA mapping against a fixed reference set will miss reads from genomes absent from the 16,093-species reference
- When analyzing transcript-level fusions or isoforms (STAR-Fusion's role in the paper), not genomic HGT breakpoints from shotgun DNA reads
- When only ancient/lineage HGT inference from single genomes is needed rather than breakpoint-level detection from population shotgun data

## Inputs

- Paired-end shotgun metagenomic reads (FASTQ, e.g. WGSIM-simulated at 2X, 5X, 20X or 70X coverage)
- Metagenomic reference genome set (multi-FASTA of bacterial genomes/scaffolds)
- BWA index built on the reference FASTA

## Outputs

- Coordinate-sorted aligned BAM (BWA mem output with read groups and -M flag)
- Unique-reads BAM filtered at MAPQ >= 20 (samtools view -q 20) for LEMON get_raw_bkp.py and get_acc_bkp
- Split-reads BAM (lumpy-sv extractSplitReads_BwaMem) for LEMON get_acc_bkp
- Coverage/count files from mapped reads used for HGT strain reconstruction

## How to apply

First map paired-end shotgun reads against the metagenomic reference (e.g. the 16,093-species, 1,246,881-scaffold NCBI bacterial set) with BWA, adding a read group via '@RG\tID:id\tSM:sample\tLB:lib' and the -M flag so split/supplementary alignments are marked. From the mapped BAM, extract unique reads with 'samtools view -q 20' (mapping quality ≥ 20) since get_raw_bkp.py requires a unique-reads BAM, and extract split reads with lumpy-sv's extractSplitReads_BwaMem since get_acc_bkp requires a split-read BAM. The alignment quality filters are a deliberate decision point: unique reads drive raw breakpoint-pair discovery while split reads refine exact breakpoint positions, so a permissive alignment step would pollute both candidate sets. Benchmark the same BAMs against LUMPY with default parameters, then score breakpoint calls within a 20 bp tolerance against simulated truth (10k bp inserted segments) to compute sensitivity and FDR at 2X/5X/70X coverage.

## Related tools

- **BWA** (Maps paired-end shotgun reads against the reference genomes to identify donors, receptors, and mapped-read coverage) — https://github.com/lichen2018/LEMON
- **samtools** (Filters the aligned BAM to unique reads (view -q 20) required by LEMON's get_raw_bkp.py) — https://github.com/samtools/htslib
- **LUMPY** (Provides extractSplitReads_BwaMem for split-read extraction; also run with default parameters as the comparison breakpoint caller) — https://github.com/lichen2018/LEMON
- **WGSIM** (Generates the paired-end reads (2X/5X/70X) from simulated local strains before alignment)
- **HTSlib** (Core BAM library dependency (v1.9) that LEMON's getAccBkp is compiled against) — https://github.com/samtools/htslib

## Examples

```
bwa mem -M -t 8 -R "@RG\tID:id\tSM:sample\tLB:lib" ref.fa reads_1.fq reads_2.fq > aln.sam; samtools view -b -q 20 aln.bam > unique.bam
```

## Evaluation signals

- The unique-reads BAM contains only alignments with MAPQ >= 20 and the split-reads BAM contains supplementary/aligned read pairs flagged by extractSplitReads_BwaMem
- Downstream get_acc_bkp runs without 'missing input' errors, confirming the raw breakpoints file (-b), unique BAM (-u), and split BAM (-s) are all consistent with the same BWA run
- Benchmark sensitivity/FDR at each coverage level falls close to the reported values (LEMON 22.39%/51.19% sensitivity at 2X/5X; FDR 0.002 at 2X, 0.0034 at 70X; LUMPY 6.86%/31.12% sensitivity) and LEMON outperforms LUMPY
- True breakpoints are recovered within the 20 bp tolerance of simulated 10k bp transferred-segment junctions
- BAM is coordinate-sorted and indexed (htslib/samtools) so LEMON and LUMPY can random-access it

## Limitations

- LEMON's breakpoint detection depends on reads that map uniquely or as split reads; repetitive or multi-mapped regions of the 1,246,881-scaffold reference yield no usable signal
- At very low coverage (2X) sensitivity is limited (LEMON 22.39%, LUMPY 6.86%), so the alignment step alone cannot guarantee breakpoint recovery
- The BWA -M flag and read-group string are required by the LEMON workflow; omitting them can break split-read extraction downstream

## Evidence

- [methods] Firstly, we map paired-end shotgun reads against reference genomes using Burrows Wheeler Aligner (BWA).: "Firstly, we map paired-end shotgun reads against reference genomes using Burrows Wheeler Aligner (BWA)."
- [methods] we utilize BWA[20] to align reads against the references to identify the set of donors and receptors: "we utilize BWA[20] to align reads against the references to identify the set of donors and receptors"
- [methods] The coverage of each segment is calculated according to the number of mapped reads on it.: "The coverage of each segment is calculated according to the number of mapped reads on it."
- [readme] bwa mem -M -t 8 -R "@RG\tID:id\tSM:sample\tLB:lib": "bwa mem -M -t 8 -R "@RG\tID:id\tSM:sample\tLB:lib""
- [readme] -u     FILE  unique reads bam file: "-u     FILE  unique reads bam file"
