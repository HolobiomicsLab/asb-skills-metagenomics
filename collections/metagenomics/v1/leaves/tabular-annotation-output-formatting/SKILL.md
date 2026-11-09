---
name: tabular-annotation-output-formatting
description: 'Use when writing the final output of an insertion-sequence discovery
  run: candidate IS nucleotide sequences have been ORF-predicted (Prodigal) and InterProScan-annotated,
  and each retained IS must be reported with its length, protein family accessions,
  positions, and genomic context in a.'
when_to_use_negative:
- Candidate ISs lack any protein family description containing Transposase, Integrase-like
  or Ribonuclease H — they should be filtered out, not formatted into the table
- Input is raw reads or unassembled contigs without prior ITR detection and ORF prediction
  — the annotation table requires candidate IS sequences as input
- You need structured per-domain GFF/GFF3 annotation rather than Palidis's encoded
  IS_name + semicolon-delimited description format
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0797
- http://edamontology.org/topic_0622
tools:
- name: Prodigal
  role: Predicts protein-coding ORFs on candidate IS nucleotide sequences prior to
    annotation
- name: InterProScan
  role: Annotates predicted proteins with InterPro and PANTHER family signatures used
    to build the IS_name accessions and positions
- name: Palidis
  role: Pipeline whose getISInfo step filters annotated candidates (Transposase/Integrase-like/Ribonuclease
    H) and writes the FASTA plus tab-delimited output table
  repo: https://github.com/blue-moon22/palidis4
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1099/mgen.0.000917
    title: 'Palidis: fast discovery of novel insertion sequences'
  source_content_hash: sha256:c88a02e1e8ce020940f4dbd5fa0a6cffb84f8095107bec4942b7c1ac08acebe2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/blue-moon22/palidis4
  tools:
  - InterProScan
  - Palidis
  - Prodigal
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0797
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# tabular-annotation-output-formatting

## Summary

Formats predicted-protein annotation results into a tab-delimited information table whose IS_name encodes IS length plus InterPro/PANTHER accessions with positions, used by Palidis to record which candidate insertion sequences encode transposase-family proteins.

## When to use

Use when writing the final output of an insertion-sequence discovery run: candidate IS nucleotide sequences have been ORF-predicted (Prodigal) and InterProScan-annotated, and each retained IS must be reported with its length, protein family accessions, positions, and genomic context in a machine-readable table.

## When NOT to use

- Candidate ISs lack any protein family description containing Transposase, Integrase-like or Ribonuclease H — they should be filtered out, not formatted into the table
- Input is raw reads or unassembled contigs without prior ITR detection and ORF prediction — the annotation table requires candidate IS sequences as input
- You need structured per-domain GFF/GFF3 annotation rather than Palidis's encoded IS_name + semicolon-delimited description format

## Inputs

- Candidate insertion sequence nucleotide sequences (FASTA) from ITR clustering
- Prodigal-predicted protein-coding ORFs on candidate ISs
- InterProScan annotations giving InterPro and PANTHER protein family signatures and match positions

## Outputs

- Tab-delimited information table with columns: IS_name, sample_id, contig, is_start, is_end, description
- FASTA file of named insertion sequences whose headers encode length and transposase accessions/positions

## How to apply

After Prodigal predicts ORFs on candidate IS nucleotide sequences and InterProScan assigns InterPro and PANTHER family signatures, filter candidates by keeping only those whose protein family descriptions contain 'Transposase', 'Integrase-like' and/or 'Ribonuclease H' (implemented in Palidis's getISInfo step). For each retained IS, build the IS_name as 'IS_length_<len>[-<accession>_<start>_<end>]+', e.g. IS_length_655-IPR002686_154_418-PTHR36966_133_625, encoding nucleotide length plus each detected transposase accession and its amino-acid positions. Write one row per IS to a tab-delimited table with columns IS_name, sample_id, contig, is_start, is_end, and description, where description lists each accession's family description separated by semicolons (e.g. IPR002686:Transposase IS200-like). Simultaneously emit the retained IS records as a FASTA file so sequence and annotation stay paired.

## Related tools

- **Prodigal** (Predicts protein-coding ORFs on candidate IS nucleotide sequences prior to annotation)
- **InterProScan** (Annotates predicted proteins with InterPro and PANTHER family signatures used to build the IS_name accessions and positions)
- **Palidis** (Pipeline whose getISInfo step filters annotated candidates (Transposase/Integrase-like/Ribonuclease H) and writes the FASTA plus tab-delimited output table) — https://github.com/blue-moon22/palidis4

## Examples

```
nextflow palidis.nf --manifest manifest.tsv --batch_name my_batch -c configs/conf/sanger.config
```

## Evaluation signals

- Output table parses as tab-delimited with exactly the columns IS_name, sample_id, contig, is_start, is_end, description
- Every IS_name matches the pattern IS_length_<nucleotide length> followed by hyphen-separated <IPRxxxxx or PTHRnnnnn>_<start>_<end> tokens
- Every accession appearing in IS_name has a corresponding '<accession>:<family description>' entry in the description column
- All retained rows have protein family descriptions containing Transposase, Integrase-like, or Ribonuclease H
- FASTA record count equals table row count, and each IS length in the name matches the FASTA sequence length

## Limitations

- Palidis only discovers ISs with ITRs, and the ISC catalogue contains ITRs of 25 or more nucleotides at default settings, so ISs with shorter ITRs never reach this annotation/formatting step
- ISs flanked by ITRs beyond the 500 to 3000 bp proximity range (e.g. lengthened by nested insertion events) are missed and will not appear in the table
- Lowering --min_itr_length or --kmer_length to catch more candidates makes the pipeline more computationally intensive

## Evidence

- [results] getISInfo output encodes accessions in IS name: [evidence span withheld — anchor status: not_found]
- [readme] README interpretation of IS_name: "represents an IS of nucleotide length 655 with transposases detected including Interpro accession IPR002686 in positions 154-418 and 148-565"
- [readme] README table schema: "IS_name | sample_id | contig | is_start | is_end | description"
- [methods] Filter criteria: [evidence span withheld — anchor status: not_found]
- [readme] ORF prediction and annotation step: "Get insertion sequences [`runProdigal`, `installInterproscan`, `runInterproscan`, `getISInfo`]"
