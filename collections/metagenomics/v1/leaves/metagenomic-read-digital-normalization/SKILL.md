---
name: metagenomic-read-digital-normalization
description: Use when raw paired-end virome FASTQ reads (e.g. SRA runs like SRR3458562
  or SRR9875293) feed into VirMAP's mapping assembly, pileup rebuilding, or DIAMOND/BBMap
  alignment and read depth is uneven or redundant.
when_to_use_negative:
- Input is not raw sequencing reads but already-assembled contigs, super-scaffolds,
  or a read-assignment table (normalization applies to the read stage before alignment).
- The VirMAP run is invoked without --useBbnorm, in which case reads are not normalized
  and this skill should not be assumed in the downstream analysis.
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3168
tools:
- name: VirMAP
  role: Pipeline driver whose pre-processing stage dereplicates and digitally normalizes
    input reads; normalization is toggled with --useBbnorm
  repo: https://github.com/cmmr/VirMAP
- name: BBMap (bbnorm)
  role: BBTools component performing read dereplication and digital normalization
    with default parameters
- name: khmer (normalize-by-median.py)
  role: Referenced implementation of digital normalization for metagenomic reads
  repo: https://github.com/dib-lab/khmer
provenance:
  source_task_ids:
  - task_003
  - task_006
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-05658-8
    title: Maximal viral information recovery from sequence data using VirMAP
  source_content_hash: sha256:30618afdee47fc6b6ee6615bb4d04c5559aff65b5d05e9b533ad511561c2939b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cmmr/VirMAP
  tools:
  - BBMap
  - BBMap (bbnorm)
  - DIAMOND
  - MEGAHIT
  - Perl
  - Statistics::Basic
  - Tadpole
  - VirMAP
  - fasterq-dump
  - khmer (normalize-by-median.py)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# metagenomic-read-digital-normalization

## Summary

Dereplicate and digitally normalize metagenomic/virome FASTQ reads before VirMAP alignment and assembly, using BBTools bbnorm (or khmer's normalize-by-median.py) with default parameters to flatten coverage and reduce redundant read depth.

## When to use

Use when raw paired-end virome FASTQ reads (e.g. SRA runs like SRR3458562 or SRR9875293) feed into VirMAP's mapping assembly, pileup rebuilding, or DIAMOND/BBMap alignment and read depth is uneven or redundant.

## When NOT to use

- Input is not raw sequencing reads but already-assembled contigs, super-scaffolds, or a read-assignment table (normalization applies to the read stage before alignment).
- The VirMAP run is invoked without --useBbnorm, in which case reads are not normalized and this skill should not be assumed in the downstream analysis.

## Inputs

- Paired-end metagenomic/virome FASTQ reads (e.g. retrieved from SRA with fasterq-dump, as for SRR3458562 or SRR9875293)

## Outputs

- Dereplicated, digitally normalized FASTQ read set used as input to VirMAP's mapping-assembly stage
- Normalized reads feeding per-genome naive pileup assembly and the VirmapRun output directory

## How to apply

As VirMAP's pre-processing stage, input reads are dereplicated and then digitally normalized with BBTools bbnorm using default parameters, matching the normalization the authors applied before any alignment to the Genbank viral/phage divisions (gbvrl and gbphage). In the Virmap.pl driver this is enabled with the --useBbnorm flag alongside --useMegahit, so normalized reads are what feed the BBMap nucleotide alignment (90% identity / 5% aligning-bases threshold), the DIAMOND translated alignment (20% identity / 8% aligning-bases threshold), and the per-genome naive pileup assembly. The khmer tool normalize-by-median.py is the alternative implementation referenced for digital normalization. Rationale: flattening coverage prevents highly redundant reads from dominating read assignment to viral genomes and stabilizes the iterative merge-and-rebuild improvement loop, where contigs are rebuilt by naive pileup assembly of the dereplicated, normalized reads.

## Related tools

- **VirMAP** (Pipeline driver whose pre-processing stage dereplicates and digitally normalizes input reads; normalization is toggled with --useBbnorm) — https://github.com/cmmr/VirMAP
- **BBMap (bbnorm)** (BBTools component performing read dereplication and digital normalization with default parameters)
- **khmer (normalize-by-median.py)** (Referenced implementation of digital normalization for metagenomic reads) — https://github.com/dib-lab/khmer

## Examples

```
Virmap.pl --threads $(nproc) --useMegahit --useBbnorm --sampleName SRR9875293 --readF SRR9875293_1.fastq
```

## Evaluation signals

- The normalized FASTQ output exists, parses as valid FASTQ, and contains fewer/less-redundant reads than the raw input.
- Downstream BBMap (90% identity / 5% aligning-bases) and DIAMOND (20% identity / 8% aligning-bases) alignments run on the normalized reads without coverage-skew errors.
- The VirmapRun output directory is populated with taxonomic classifications and abundance results, with diagnostics captured in VirmapRun.err.
- Per-genome read sets support naive pileup reconstruction, indicating normalization did not strip coverage needed for assembly.
- For the NetoVir validation path, the downstream run reproduces approximately 37.04% of 6,707,800 reads classified as viral (R² = 0.9972 vs Conceição-Neto proportions).

## Limitations

- Normalization is applied with bbnorm default parameters; the article notes parameters that tune the sensitivity/specificity tradeoff are documented only in Supplementary Data 14, which is not provided here.
- No software versions are stated for BBMap or khmer in the article, so reproducibility of the exact normalization behavior depends on tool version pinning done externally.
- The dynamically calculated thresholds used downstream (e.g. LCA radius) are not fully specified in the text, so normalization effects on classification cannot be fully decoupled.

## Evidence

- [discussion] Digital normalization is part of VirMAP pre-processing: "input reads are dereplicated and digitally normalized18 using default parameters"
- [methods] Normalization is enabled via the --useBbnorm flag in the Virmap.pl driver: "`Virmap.pl --threads $(nproc) --readF /home/$USER/VirmapTest/SRR9875293_1.fastq"
- [methods] khmer's normalize-by-median.py is the referenced normalization implementation: "normalize-by-median.py (https://github.com/dib-lab/khmer)"
- [discussion] Normalized reads feed the iterative rebuild stage of the improvement loop: [evidence span withheld — anchor status: not_found]
