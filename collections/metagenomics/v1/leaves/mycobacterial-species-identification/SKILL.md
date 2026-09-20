---
name: mycobacterial-species-identification
description: Use when raw FASTQ reads from suspected mycobacterial isolates (MTBC
  or NTM, e.g. MGIT culture positives or SRA downloads) need phylo_group/species/lineage
  calls, or when Mykrobe species calls must be compared against KvarQ lineage labels.
when_to_use_negative:
- Input samples are already known non-mycobacterial species — the panel targets Mycobacterium
  (and the parser explicitly drops NTM-positive samples from its output)
- The goal is antimicrobial resistance prediction rather than species/lineage identification
  — use the resistance-prediction skill with the tb probe set and mykrobe-parser report
  output instead
- Reads are nanopore or other long-read data where this validated depth/conf-thresholded
  Illumina-style workflow has not been demonstrated
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0625
tools:
- name: Mykrobe predictor
  role: 'Runs de Bruijn graph probe-based classification of phylo_group, species,
    and lineage from raw reads using the 201901/202010 panels with --format '
  repo: https://github.com/Mykrobe-tools/mykrobe
- name: Mykrobe Parser
  role: R script that parses Mykrobe JSON output (-d directory of . files, -p 2020
    panel) into output-data.csv with speciation columns
  repo: https://github.com/phac-nml/mykrobe-parser
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/ncomms10063
    title: Rapid antibiotic-resistance predictions from genome sequence data for Staphylococcus
      aureus and Mycobacterium tuberculosis
  source_content_hash: sha256:fd2a2964aa820133591468412d77c48e8b404c5bb48848567d7a3f886eea4ae1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mykrobe-tools/mykrobe
  tools:
  - Mykrobe Parser
  - Mykrobe predictor
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# mycobacterial-species-identification

## Summary

Identify mycobacterial phylo_group, species, and lineage directly from raw sequencing reads using Mykrobe predictor's species/lineage probe panel (e.g. 202010) with default QC thresholds, discriminating MTBC from non-tuberculous mycobacteria and assigning MTBC lineages concordant with KvarQ.

## When to use

Use when raw FASTQ reads from suspected mycobacterial isolates (MTBC or NTM, e.g. MGIT culture positives or SRA downloads) need phylo_group/species/lineage calls, or when Mykrobe species calls must be compared against KvarQ lineage labels.

## When NOT to use

- Input samples are already known non-mycobacterial species — the panel targets Mycobacterium (and the parser explicitly drops NTM-positive samples from its output)
- The goal is antimicrobial resistance prediction rather than species/lineage identification — use the resistance-prediction skill with the tb probe set and mykrobe-parser report output instead
- Reads are nanopore or other long-read data where this validated depth/conf-thresholded Illumina-style workflow has not been demonstrated

## Inputs

- Raw sequencing reads (FASTQ/FASTQ.gz) for mycobacterial isolates from ENA/NCBI SRA (e.g. MTBC_A2, Myco_Retro)
- Mykrobe species/lineage probe panel (201901 or 202010)
- Optional: precomputed KvarQ lineage calls for MTBC isolates as a concordance reference

## Outputs

- Per-sample phylo_group, species, and lineage calls with depths and percent coverage (JSON, or output-data.csv via mykrobe-parser)
- MTBC/NTM misclassification counts and species-level unresolved tallies
- Lineage concordance summary vs KvarQ (e.g. species_lineage_concordance.csv)

## How to apply

Download raw reads for the isolate sets (e.g. MTBC_A2, 1,157 MTBC isolates; Myco_Retro, 147 mycobacterial isolates) from ENA/NCBI SRA, then run Mykrobe predictor per sample with --format  and the 202010 species/lineage probe panel at default QC thresholds --min_depth 5 and --min_variant_conf 10. Parse the JSON outputs (directly, or with phac-nml/mykrobe-parser's R script, which reads the Mykrobe JSON directory with -d and emits output-data.csv) to extract each sample's phylo_group, species, and lineage calls with associated depths and percent coverage. Cross-tabulate species calls against expected labels (MTBC vs non-tuberculous mycobacteria) to count MTBC/NTM misclassifications, and compare MTBC lineage assignments to precomputed KvarQ calls for concordance. Note the parser removes samples in which Mykrobe detects non-tuberculous mycobacteria as a quality-control step, so downstream resistance tables exclude NTM by design.

## Related tools

- **Mykrobe predictor** (Runs de Bruijn graph probe-based classification of phylo_group, species, and lineage from raw reads using the 201901/202010 panels with --format ) — https://github.com/Mykrobe-tools/mykrobe
- **Mykrobe Parser** (R script that parses Mykrobe JSON output (-d directory of . files, -p 2020 panel) into output-data.csv with speciation columns) — https://github.com/phac-nml/mykrobe-parser

## Examples

```
mykrobe predict --sample my_sample_name --species tb --output out. --format  --seq reads.fq.gz
```

## Evaluation signals

- No samples are misclassified between MTBC and NTM across the MTBC_A2 + Myco_Retro union; only a handful remain unresolved to species level within MTBC
- Mykrobe lineage calls on MTBC_A2 are 100% concordant with precomputed KvarQ lineage calls
- Mykrobe was run with --format  and a compatible panel (201901 or 202010); min_depth 5 and min_variant_conf 10 are recorded
- Parsed output-data.csv contains phylo_group, species, and lineage columns with associated depths and percentages of coverage
- Samples Mykrobe flags as non-tuberculous mycobacteria are excluded from downstream QC-filtered outputs

## Limitations

- The mykrobe-parser removes samples in which Mykrobe detects non-tuberculous mycobacteria, so NTM-positive samples disappear from downstream resistance reporting rather than being flagged
- Some samples may be unresolved to species level within MTBC rather than misclassified
- Mykrobe Parser is version-locked: it is compatible with Mykrobe v0.7.0 and the 201901/202010 panels only
- Results are for M. tuberculosis and related mycobacteria; the predictor covers S. aureus, S. sonnei, S. typhi, and S. Paratyphi B for other taxa but species calls rely on the probe panel supplied

## Evidence

- [discussion] Mykrobe predictor, which can identify species, resistance profile: "We have developed a software application, extensible to many bacterial species, called Mykrobe predictor, which can identify species, resistance profile"
- [readme] Samples that Mykrobe detects non-tuberculous mycobacteria in are removed for quality control purposes.: "Samples that Mykrobe detects non-tuberculous mycobacteria in are removed for quality control purposes."
- [readme] Stores Mykrobe's speciation data such as "phylo_group", "species", and "lineage" and their associated depths and percentages of coverage.: "Stores Mykrobe's speciation data such as "phylo_group", "species", and "lineage" and their associated depths and percentages of coverage."
- [discussion] the validation set for mycobacteria included the set Myc_Retro: "the validation set for mycobacteria included the set Myc_Retro consisting of all mycobacterial (meaning positive MGIT culture) samples sent to the laboratories"
- [readme] --format  stores results as .: [evidence span withheld — anchor status: not_found]
