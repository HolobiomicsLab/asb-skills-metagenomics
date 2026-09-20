---
name: ribonuclease-h-domain-recognition
description: Use when candidate IS nucleotide sequences (e.g. ITR-flanked contigs
  from Palidis) must be confirmed as encoding transposase-family proteins before inclusion
  in an insertion sequence catalogue.
when_to_use_negative:
- Discovery of insertion sequences lacking inverted terminal repeats — Palidis is
  only equipped with discovering ISs with ITRs
- Comprehensive transposase annotation of whole genomes rather than filtering candidate
  ISs flanked by ITRs in the 500-3000 bp proximity range
edam_operation: http://edamontology.org/operation_2489
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0749
tools:
- name: Prodigal
  role: Predicts protein-coding ORFs on candidate IS nucleotide sequences before family
    annotation
- name: InterProScan
  role: Annotates predicted proteins with InterPro and PANTHER protein-family signatures
    used for the Transposase/Integrase-like/Ribonuclease H filter
- name: Palidis
  role: Pipeline whose final getISInfo step writes the FASTA and tab-delimited IS
    table encoding family accessions and positions
  repo: https://github.com/blue-moon22/palidis4
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1099/mgen.0.000917
    title: 'Palidis: fast discovery of novel insertion sequences'
  source_content_hash: sha256:828f200fd11d640274a4480b52a4ec2e14a54ad7931159986f5536c96e8f13d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/blue-moon22/palidis4
  tools:
  - InterProScan
  - Palidis
  - Prodigal
  edam_operation: http://edamontology.org/operation_2489
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# Ribonuclease H domain recognition

## Summary

Annotate candidate insertion-sequence ORFs with InterProScan and retain those whose InterPro/PANTHER protein-family descriptions include Transposase, Integrase-like and/or Ribonuclease H, so only transposase-family ISs are reported.

## When to use

Use when candidate IS nucleotide sequences (e.g. ITR-flanked contigs from Palidis) must be confirmed as encoding transposase-family proteins before inclusion in an insertion sequence catalogue.

## When NOT to use

- Discovery of insertion sequences lacking inverted terminal repeats — Palidis is only equipped with discovering ISs with ITRs
- Comprehensive transposase annotation of whole genomes rather than filtering candidate ISs flanked by ITRs in the 500-3000 bp proximity range

## Inputs

- Candidate IS nucleotide sequences (FASTA, ITR-flanked, 500-3000 bp by Palidis defaults)
- Manifest with sample_id and contig metadata (tab-delimited: lane_id, read_directory, sample_id, contigs_path)

## Outputs

- FASTA file of insertion sequences
- Tab-delimited information table with columns IS_name, sample_id, contig, is_start, is_end, description (InterPro/PANTHER accessions, positions and family descriptions)

## How to apply

First predict protein-coding ORFs on each candidate IS nucleotide sequence using Prodigal. Annotate the predicted proteins with InterProScan in chunks (default chunk_size 80000 sequences) to obtain InterPro and PANTHER protein-family signatures. Filter the annotated candidates, retaining only insertion sequences whose protein-family descriptions include Transposase, Integrase-like and/or Ribonuclease H. Write the retained records to a tab-delimited table where each IS name encodes the IS length plus the InterPro or PANTHER accession(s) of detected transposases and their positions, alongside sample_id, contig, is_start and is_end. Correct application is judged by the presence of a description column mapping each accession (e.g. IPR002686:Transposase IS200-like) and by IS names like IS_length_655-IPR002686_154_418-...PTHR36966_133_625.

## Related tools

- **Prodigal** (Predicts protein-coding ORFs on candidate IS nucleotide sequences before family annotation)
- **InterProScan** (Annotates predicted proteins with InterPro and PANTHER protein-family signatures used for the Transposase/Integrase-like/Ribonuclease H filter)
- **Palidis** (Pipeline whose final getISInfo step writes the FASTA and tab-delimited IS table encoding family accessions and positions) — https://github.com/blue-moon22/palidis4

## Examples

```
nextflow palidis.nf --manifest manifest.tsv --batch_name out -c configs/conf/sanger.config
```

## Evaluation signals

- Every retained IS_name begins with IS_length_<n> and embeds at least one IPR or PTHR accession with start/end positions, e.g. IPR002686_154_418
- Each accession in IS_name has a matching family description in the description column (e.g. IPR002686:Transposase IS200-like)
- Descriptions of retained records contain Transposase, Integrase-like and/or Ribonuclease H family terms only
- Output table is tab-delimited with exactly the columns IS_name, sample_id, contig, is_start, is_end, description
- Reported IS lengths fall within the default min_is_len/max_is_len range of 500-3000 bp

## Limitations

- Filter retains only common DDE-type ISs with ITRs; ISs lacking ITRs are not discovered
- Candidates with ITRs shorter than the default 25 nt minimum are missed at default settings
- ISs lengthened by nested insertion events beyond the 500-3000 bp proximity range are missed

## Evidence

- [methods] Filter retaining only transposase-family ISs: [evidence span withheld — anchor status: not_found]
- [methods] InterProScan annotation step: [evidence span withheld — anchor status: not_found]
- [readme] IS name encoding: "Name assigned by PaliDIS which contains the length, interpro or PANTHER accessions of transposases and their positions"
- [discussion] ITR-only limitation: "Palidis is currently only equipped with discovering ISs with ITRs."
