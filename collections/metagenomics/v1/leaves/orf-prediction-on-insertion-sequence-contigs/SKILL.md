---
name: orf-prediction-on-insertion-sequence-contigs
description: Use when candidate IS nucleotide sequences (e.g. FASTA of ISs from ITR-based
  discovery) must be annotated to decide which encode transposase-family proteins.
when_to_use_negative:
- The ISs of interest lack inverted terminal repeats — Palidis is 'currently only
  equipped with discovering ISs with ITRs', so no candidate IS FASTA exists to run
  Prodigal on
- Candidate regions fall outside the 500-3000 bp IS length range (e.g. lengthened
  by nested insertion events) and would be missed by the default --min_is_len/--max_is_len
  settings
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Prodigal
  role: Predicts protein-coding ORFs on the candidate IS nucleotide sequences before
    family annotation
- name: InterProScan
  role: Annotates the Prodigal-predicted proteins with InterPro and PANTHER protein
    family signatures used for the Transposase/Integrase-like/Ribonuclease H filter
- name: Palidis
  role: Pipeline (runProdigal, runInterproscan, getISInfo steps) that wraps the ORF
    prediction, annotation, and transposase-family filtering on candidate ISs
  repo: https://github.com/blue-moon22/palidis4
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1099/mgen.0.000917
    title: 'Palidis: fast discovery of novel insertion sequences'
  source_content_hash: sha256:3b077e7976b565b3401fe412faa4cf273ea7c0722f10f484a312ed8a11f5cec4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/blue-moon22/palidis4
  tools:
  - InterProScan
  - Palidis
  - Prodigal
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# orf-prediction-on-insertion-sequence-contigs

## Summary

Predicts protein-coding ORFs on candidate insertion-sequence (IS) nucleotide sequences using Prodigal and annotates the predicted proteins with InterProScan to obtain InterPro and PANTHER family signatures, so that only ISs encoding transposase-family proteins are retained in the Palidis output.

## When to use

Use when candidate IS nucleotide sequences (e.g. FASTA of ISs from ITR-based discovery) must be annotated to decide which encode transposase-family proteins.

## When NOT to use

- The ISs of interest lack inverted terminal repeats — Palidis is 'currently only equipped with discovering ISs with ITRs', so no candidate IS FASTA exists to run Prodigal on
- Candidate regions fall outside the 500-3000 bp IS length range (e.g. lengthened by nested insertion events) and would be missed by the default --min_is_len/--max_is_len settings

## Inputs

- FASTA file of candidate insertion sequence nucleotide sequences (output of Palidis ITR clustering steps)
- Contig names and sample IDs from the manifest for the IS coordinates
- Manifest (tab-delimited: lane_id, read_directory, sample_id, contigs_path)

## Outputs

- FASTA file of insertion sequences
- Tab-delimited information table with IS_name (length + InterPro/PANTHER accessions and positions), sample_id, contig, is_start, is_end, and protein family descriptions
- Predicted ORF protein sequences from Prodigal annotated with InterProScan family signatures

## How to apply

Take the candidate IS nucleotide FASTA produced by the Palidis pipeline (putative ITRs flanked by 500-3000 bp regions) and run Prodigal to predict protein-coding ORFs on each candidate IS. Annotate the predicted proteins with InterProScan, optionally chunked (default 80,000 protein sequences per chunk) and optionally run in LSF cluster mode, to obtain InterPro and PANTHER protein family signatures. Filter the annotated candidates, retaining only ISs whose protein family descriptions include Transposase, Integrase-like and/or Ribonuclease H. In the final getISInfo step, write retained IS records with InterPro/PANTHER accessions and family positions into a tab-delimited table; IS names encode the IS length and detected transposase accessions with positions (e.g. IS_length_655-IPR002686_154_418-PTHR36966_133_625), plus sample ID, contig, IS start/end, and protein family descriptions.

## Related tools

- **Prodigal** (Predicts protein-coding ORFs on the candidate IS nucleotide sequences before family annotation)
- **InterProScan** (Annotates the Prodigal-predicted proteins with InterPro and PANTHER protein family signatures used for the Transposase/Integrase-like/Ribonuclease H filter)
- **Palidis** (Pipeline (runProdigal, runInterproscan, getISInfo steps) that wraps the ORF prediction, annotation, and transposase-family filtering on candidate ISs) — https://github.com/blue-moon22/palidis4

## Evaluation signals

- Output table contains one row per retained IS with columns IS_name, sample_id, contig, is_start, is_end, description (tab-delimited)
- IS_name field parses as IS_length_<n>-<ACCESSION>_<start>_<end> repeats, e.g. IS_length_655-IPR002686_154_418-PTHR36966_133_625
- Retained ISs have protein family descriptions containing Transposase, Integrase-like and/or Ribonuclease H (e.g. IPR002686:Transposase IS200-like)
- IS coordinates in the table are consistent with IS length encoded in IS_name (e.g. is_end - is_start + 1 = 655 for IS_length_655)
- Regression test harness reports 'Test passed. All outputs expected.' for the pipeline outputs

## Limitations

- ISC v1.0.0 (and hence the annotation filter) only contains common DDE-type ISs with ITRs; ISs lacking ITRs cannot be discovered or annotated via this route
- Default minimum ITR length of 25 misses ITRs as short as ten nucleotides; lowering --min_itr_length and --kmer_length is more computationally intensive
- ISs flanked by ITRs beyond the 500-3000 bp proximity range (e.g. nested insertions) are missed and never reach the ORF prediction step

## Evidence

- [full_text] Predict protein-coding ORFs on the candidate IS nucleotide sequences using Prodigal.: [evidence span withheld — anchor status: not_found]
- [full_text] Filter annotated candidates for transposase families: [evidence span withheld — anchor status: not_found]
- [readme] Pipeline step list in README: "7. Get insertion sequences [`runProdigal`, `installInterproscan`, `runInterproscan`, `getISInfo`]"
- [readme] Chunk size for InterProScan runs: "--chunk_size        Number of protein sequences in each chunk for Interproscan. (Default: 80000)"
- [readme] Interpretation of IS_name field: "Name assigned by PaliDIS which contains the length, interpro or PANTHER accessions of transposases and their positions"
