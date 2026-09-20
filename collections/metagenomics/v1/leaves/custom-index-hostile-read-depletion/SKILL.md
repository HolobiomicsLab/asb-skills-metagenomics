---
name: custom-index-hostile-read-depletion
description: Use when Hostile-cleaned retained reads still show non-human representation
  (e.g. EBV in human read retention figures) and those contaminant reads must be subtracted
  to obtain adjusted retention metrics.
when_to_use_negative:
- When the goal is only human host read removal for privacy/legal reasons using the
  default human-t2t-hla index — a custom contaminant index is unnecessary
- When contaminant identity is unknown and de novo assembly/screening has not yet
  identified a reference to subtract against
- When samples contain no detectable contaminant reads (retention metrics will be
  unchanged and the extra pass is wasted)
edam_operation: http://edamontology.org/operation_3187
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: Hostile
  role: Runs the clean subcommand with a custom --index (Minimap2 mode) to subtract
    contaminant reads and emits JSON logs with reads_in/reads_out/reads_removed_proportion
  repo: https://github.com/bede/hostile
- name: MEGAHIT
  role: De novo assembles retained reads to reveal contaminant (EBV) representation
    before subtraction
- name: Minimap2
  role: Aligns assemblies and reads against EBV references for screening; also powers
    long-read/custom-index clean mode
- name: Bowtie2
  role: Short-read alignment backend used by Hostile for index-based read removal
- name: Bedtools
  role: Masks aligned regions (with hostile mask) when constructing custom indexes
    for target retention
- name: Samtools
  role: Supports alignment file handling during screening and index workflows
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad728
    title: 'Hostile: accurate decontamination of microbial host sequences'
  source_content_hash: sha256:89614f8e2badd6f4e27a5e457785073601f01dce45931eba77a668d324cc2d69
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/bede/hostile
  tools:
  - Bedtools
  - Bowtie2
  - Hostile
  - MEGAHIT
  - Minimap2
  - Samtools
  edam_operation: http://edamontology.org/operation_3187
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# custom-index-hostile-read-depletion

## Summary

Build a custom Hostile index from a specific contaminant reference (e.g. Epstein-Barr Virus NC_007605.1/NC_009334.1) and run hostile clean against it to subtract contaminant reads from Hostile-cleaned FASTQ, then recompute host read retention. This extends Hostile's human read removal to targeted decontamination of non-human contaminants discovered in retained reads.

## When to use

Use when Hostile-cleaned retained reads still show non-human representation (e.g. EBV in human read retention figures) and those contaminant reads must be subtracted to obtain adjusted retention metrics.

## When NOT to use

- When the goal is only human host read removal for privacy/legal reasons using the default human-t2t-hla index — a custom contaminant index is unnecessary
- When contaminant identity is unknown and de novo assembly/screening has not yet identified a reference to subtract against
- When samples contain no detectable contaminant reads (retention metrics will be unchanged and the extra pass is wasted)

## Inputs

- Paired Hostile-cleaned retained FASTQ files (*.clean_1.fastq.gz / *.clean_2.fastq.gz)
- Contaminant reference genome FASTA (e.g. EBV NC_007605.1 and NC_009334.1)
- Assembled contigs from de novo assembly of retained reads (e.g. MEGAHIT output)
- Hostile JSON clean logs with reads_in/reads_out/reads_removed_proportion fields

## Outputs

- EBV-depleted FASTQ pairs (hostile clean output)
- Custom Hostile index built from contaminant references
- Per-sample retention metrics table before and after contaminant subtraction
- Flagged samples with high contaminant proportion (e.g. ERR3242202 at 1.3% EBV)

## How to apply

First confirm the contaminant by de novo assembling the retained reads (e.g. MEGAHIT) and aligning contigs against candidate references such as EBV NC_007605.1 and NC_009334.1 with Minimap2/Bowtie2. Build a custom index from those FASTA references and run hostile clean with --index pointing at the custom genome file (Minimap2 mode) on the paired retained FASTQ (*.clean_1.fastq.gz / *.clean_2.fastq.gz). Hostile supports custom indexes alongside standard ones like human-t2t-hla, and masking via hostile mask can maximize retention of specific target organisms when building such indexes. Recompute retention per sample from the reads_in/reads_out/reads_removed_proportion fields in the Hostile JSON logs, subtracting the removed proportion from the original retention. In the Byrska-Bishop et al. 27-genome 2x150bp dataset, this shifted overall Hostile retention from 0.594% to 0.393% and NA12878 from 0.641% to 0.406%, with ERR3242202 flagged at 1.3% EBV contamination.

## Related tools

- **Hostile** (Runs the clean subcommand with a custom --index (Minimap2 mode) to subtract contaminant reads and emits JSON logs with reads_in/reads_out/reads_removed_proportion) — https://github.com/bede/hostile
- **MEGAHIT** (De novo assembles retained reads to reveal contaminant (EBV) representation before subtraction)
- **Minimap2** (Aligns assemblies and reads against EBV references for screening; also powers long-read/custom-index clean mode)
- **Bowtie2** (Short-read alignment backend used by Hostile for index-based read removal)
- **Bedtools** (Masks aligned regions (with hostile mask) when constructing custom indexes for target retention)
- **Samtools** (Supports alignment file handling during screening and index workflows)

## Examples

```
hostile clean --fastq1 ERR3242202.clean_1.fastq.gz --fastq2 ERR3242202.clean_2.fastq.gz --index custom-ebv-nc_007605-nc_009334 --threads 8
```

## Evaluation signals

- Overall human read retention across the 27 Byrska-Bishop et al. 2x150bp genomes drops from 0.594% to 0.393% after EBV subtraction
- NA12878 2x150bp retention drops from 0.641% to 0.406% for Hostile after EBV subtraction
- ERR3242202 is flagged with an EBV read proportion of approximately 1.3%
- Hostile JSON logs report consistent reads_in/reads_out/reads_removed_proportion values before and after the custom-index pass
- The custom index is used without corrupting output: FASTQ files remain valid even when few or no reads remain after decontamination (fixed issue #24)

## Limitations

- Earlier Hostile versions could corrupt empty fastq.gz files when no reads remain after decontamination (#24)
- Downloaded index checksums were not verified in earlier versions (verification added only in latest release, issue #20)
- Hostile requires ~4GB RAM for short reads (Bowtie2) and ~13GB for long reads (Minimap2); long-read clean mode auto-builds .mmi files

## Evidence

- [other] hostile clean with --index pointing at the custom genome file (Minimap2 mode): [evidence span withheld — anchor status: not_found]
- [methods] Reads aligning to an index removed from fastq: "Remove reads aligning to an index from fastq[.gz] input files or stdin."
- [methods] The mask subcommand makes it easy to create custom-masked indexes: "The `mask` subcommand makes it easy to create custom-masked indexes in order to achieve maximum retention of specific target organisms"
- [intro] Hostile removes host sequences from short and long read (meta)genomes: "Hostile removes host sequences from short and long read (meta)genomes, consuming single or paired FASTQ from files or stdin."
- [readme] Custom indexes are also supported (see below): "Custom indexes are also supported (see below)."
