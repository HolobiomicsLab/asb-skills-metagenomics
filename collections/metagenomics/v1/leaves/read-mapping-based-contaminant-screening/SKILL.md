---
name: read-mapping-based-contaminant-screening
description: Use when retained reads after host decontamination show unexpectedly
  high human retention (e.g. NA12878 0.641% vs 0.406%) and a recurring contaminant
  such as EBV is suspected across a genome set.
when_to_use_negative:
- The input FASTQs have not yet passed host decontamination (raw reads rather than
  Hostile-retained *.clean_*.fastq.gz)
- The goal is removal of human host reads with the default human-t2t-hla index rather
  than screening for a specific non-human contaminant
- Long-read or single-end data where paired clean_1/clean_2 retained FASTQs are not
  available
edam_operation: http://edamontology.org/operation_3741
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: Hostile
  role: Removes reads aligning to a custom EBV index from the retained FASTQs (hostile
    clean with --index, Minimap2 mode) and reports reads_in/reads_out/reads_removed_proportion
    in JSON logs
  repo: https://github.com/bede/hostile
- name: MEGAHIT
  role: De novo assembly of retained reads per genome to generate contigs for contaminant
    detection
- name: Bowtie2
  role: Short-read alignment used to screen retained reads and assemblies against
    EBV references
- name: Minimap2
  role: Alignment of assemblies/reads against NC_007605.1 and NC_009334.1; used by
    Hostile in Minimap2 mode for the custom-index clean
- name: Samtools
  role: Downstream alignment/BAM handling during contaminant screening
- name: Bedtools
  role: Downstream interval operations during contaminant screening
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btad728
    title: 'Hostile: accurate decontamination of microbial host sequences'
  source_content_hash: sha256:77c4ccdecb00d76bc1f0f69930377e49eabee5c70efcec6cf23a71125a2e0b66
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
  edam_operation: http://edamontology.org/operation_3741
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# read-mapping-based-contaminant-screening

## Summary

Detects and subtracts contaminant sequences (e.g. Epstein-Barr Virus) from Hostile-cleaned retained FASTQs by de novo assembly with MEGAHIT, alignment to contaminant references (NC_007605.1/NC_009334.1) using Minimap2/Bowtie2, and a custom Hostile clean pass with a contaminant-only index, so host read retention metrics are corrected.

## When to use

Use when retained reads after host decontamination show unexpectedly high human retention (e.g. NA12878 0.641% vs 0.406%) and a recurring contaminant such as EBV is suspected across a genome set.

## When NOT to use

- The input FASTQs have not yet passed host decontamination (raw reads rather than Hostile-retained *.clean_*.fastq.gz)
- The goal is removal of human host reads with the default human-t2t-hla index rather than screening for a specific non-human contaminant
- Long-read or single-end data where paired clean_1/clean_2 retained FASTQs are not available

## Inputs

- Hostile-cleaned retained paired FASTQ files (*.clean_1.fastq.gz / *.clean_2.fastq.gz) from 27 Byrska-Bishop et al. Illumina 2x150bp genomes
- EBV reference genomes NC_007605.1 and NC_009334.1
- Hostile JSON logs (reads_in, reads_out, reads_removed_proportion) from upstream hostile clean runs

## Outputs

- MEGAHIT contig assemblies per sample
- Per-sample EBV contamination proportions (e.g. ERR3242202 at 1.3%)
- EBV-subtracted FASTQ files after hostile clean with a custom EBV index
- Metrics table of per-sample host retention before and after EBV subtraction and an adjusted retention figure

## How to apply

Take the Hostile-cleaned paired FASTQs (*.clean_1.fastq.gz / *.clean_2.fastq.gz) from the 27 Byrska-Bishop et al. 2x150bp genomes and de novo assemble them per sample with MEGAHIT. Screen the resulting contigs and retained reads for EBV representation by aligning against the EBV references NC_007605.1 and NC_009334.1 with Minimap2/Bowtie2. Build a custom Hostile index from those references and run hostile clean with --index in Minimap2 mode to subtract EBV-matching reads. Recompute retention from reads_in/reads_out/reads_removed_proportion in the Hostile JSON logs; corrected overall retention should drop from 0.594% to 0.393% and NA12878 from 0.641% to 0.406%, with ERR3242202 flagged at ~1.3% EBV.

## Related tools

- **Hostile** (Removes reads aligning to a custom EBV index from the retained FASTQs (hostile clean with --index, Minimap2 mode) and reports reads_in/reads_out/reads_removed_proportion in JSON logs) — https://github.com/bede/hostile
- **MEGAHIT** (De novo assembly of retained reads per genome to generate contigs for contaminant detection)
- **Bowtie2** (Short-read alignment used to screen retained reads and assemblies against EBV references)
- **Minimap2** (Alignment of assemblies/reads against NC_007605.1 and NC_009334.1; used by Hostile in Minimap2 mode for the custom-index clean)
- **Samtools** (Downstream alignment/BAM handling during contaminant screening)
- **Bedtools** (Downstream interval operations during contaminant screening)

## Examples

```
hostile clean --fastq1 sample.clean_1.fastq.gz --fastq2 sample.clean_2.fastq.gz --index custom-ebv-nc_007605-nc_009334 --alignment-mode minimap2
```

## Evaluation signals

- Adjusted overall human retention for the 27 2x150bp genomes reproduces 0.393% after EBV subtraction (from 0.594% before)
- NA12878 2x150bp retention reproduces 0.406% after EBV subtraction (from 0.641% before)
- ERR3242202 is flagged with an EBV read proportion of approximately 1.3%
- Hostile JSON logs after subtraction show reads_removed_proportion consistent with the EBV fraction and reads_out smaller than reads_in
- De novo contigs from every one of the 27 genomes show alignment to NC_007605.1/NC_009334.1 confirming broad EBV representation

## Limitations

- Retention figures before and after subtraction depend on the upstream Hostile JSON metrics (reads_in/reads_out/reads_removed_proportion); earlier Hostile versions could corrupt empty fastq.gz files when no reads remain after decontamination
- Hostile requires ~4GB RAM for short-read (Bowtie2) and ~13GB for long-read (Minimap2) decontamination, which constrains where the custom-index subtraction can run
- Detection depends on de novo assembly recovering contaminant sequence; low-abundance contamination below assembly representation may be missed

## Evidence

- [other] De novo assembly of the retained reads revealed Epstein-Barr Virus representation in all 27 studied Byrska-Bishop et al. genomes: [evidence span withheld — anchor status: not_found]
- [other] Build a custom EBV index and subtract EBV-matching reads: [evidence span withheld — anchor status: not_found]
- [intro] Hostile removes host sequences from short and long read (meta)genomes, consuming single or paired FASTQ from files or stdin.: "Hostile removes host sequences from short and long read (meta)genomes, consuming single or paired FASTQ from files or stdin."
- [methods] Remove reads aligning to an index from fastq[.gz] input files or stdin.: "Remove reads aligning to an index from fastq[.gz] input files or stdin."
- [readme] Custom indexes are also supported: "Custom indexes are also supported (see below)."
