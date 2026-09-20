---
name: protein-family-signature-annotation
description: Use when candidate IS nucleotide sequences (FASTA) need protein family
  annotation — e.g. to decide whether an IS encodes a transposase via InterPro (IPR)
  / PANTHER (PTHR) accessions before reporting it.
when_to_use_negative:
- The insertion sequences lack inverted terminal repeats — Palidis is currently only
  equipped with discovering ISs with ITRs, so no candidates will reach this annotation
  step
- Candidate ISs have ITRs shorter than the default minimum of 25 nt (ITRs can be as
  short as ten nucleotides but are missed at default settings)
- The goal is annotation of non-transposase protein families — the filter retains
  only Transposase, Integrase-like and/or Ribonuclease H family descriptions
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0621
tools:
- name: Prodigal
  role: Predicts protein-coding ORFs on candidate IS nucleotide sequences before family
    annotation
- name: InterProScan
  role: Annotates predicted proteins with InterPro (IPR) and PANTHER (PTHR) family
    signatures used for the transposase-family filter
- name: Palidis
  role: Nextflow pipeline whose runProdigal/runInterproscan/getISInfo processes wrap
    this annotation and filtering step
  repo: https://github.com/blue-moon22/palidis4
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1099/mgen.0.000917
    title: 'Palidis: fast discovery of novel insertion sequences'
  source_content_hash: sha256:fe594f4d741274d19eaf694f2495642990fcb27d9032a438e895c17194cfcd1a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/blue-moon22/palidis4
  tools:
  - InterProScan
  - Palidis
  - Prodigal
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# protein-family-signature-annotation

## Summary

Annotate predicted protein-coding ORFs from candidate insertion sequences with InterProScan to obtain InterPro and PANTHER protein family signatures, then filter candidates so only those encoding transposase-family proteins are retained as final Palidis IS records.

## When to use

Use when candidate IS nucleotide sequences (FASTA) need protein family annotation — e.g. to decide whether an IS encodes a transposase via InterPro (IPR) / PANTHER (PTHR) accessions before reporting it.

## When NOT to use

- The insertion sequences lack inverted terminal repeats — Palidis is currently only equipped with discovering ISs with ITRs, so no candidates will reach this annotation step
- Candidate ISs have ITRs shorter than the default minimum of 25 nt (ITRs can be as short as ten nucleotides but are missed at default settings)
- The goal is annotation of non-transposase protein families — the filter retains only Transposase, Integrase-like and/or Ribonuclease H family descriptions

## Inputs

- Candidate insertion sequence nucleotide sequences (FASTA) from earlier Palidis steps
- Manifest with sample_id and contig headers (tab-delimited)
- Contig FASTA files providing contig names and IS start/end coordinates

## Outputs

- Tab-delimited IS information table with columns IS_name, sample_id, contig, is_start, is_end, description
- IS_name strings encoding IS length and InterPro/PANTHER accessions with positions (e.g. IS_length_1455-IPR013762_1393_1918)
- Description column mapping each accession to family names (e.g. IPR002686:Transposase IS200-like; PTHR36966:REP-ASSOCIATED TYROSINE TRANSPOSASE)

## How to apply

First predict protein-coding ORFs on each candidate IS nucleotide sequence using Prodigal (pipeline steps runProdigal). Next run InterProScan on the predicted proteins to obtain InterPro (IPR) and PANTHER (PTHR) protein family signatures and their positions (installInterproscan, runInterproscan; chunk size default 80000 sequences per chunk, with optional LSF cluster mode). Filter annotated candidates, retaining only ISs whose protein family descriptions include Transposase, Integrase-like and/or Ribonuclease H. Finally, in getISInfo, write retained records to a tab-delimited table where the IS_name encodes the IS nucleotide length and every detected transposase accession with its start/end positions (e.g. IS_length_655-IPR002686_154_418-PTHR36966_133_625), alongside sample_id, contig, is_start, is_end and the family descriptions.

## Related tools

- **Prodigal** (Predicts protein-coding ORFs on candidate IS nucleotide sequences before family annotation)
- **InterProScan** (Annotates predicted proteins with InterPro (IPR) and PANTHER (PTHR) family signatures used for the transposase-family filter)
- **Palidis** (Nextflow pipeline whose runProdigal/runInterproscan/getISInfo processes wrap this annotation and filtering step) — https://github.com/blue-moon22/palidis4

## Examples

```
nextflow palidis.nf --manifest manifest.tsv --batch_name batch1 -c configs/conf/sanger.config --chunk_size 80000 --lsf
```

## Evaluation signals

- Every retained IS_name matches the pattern IS_length_<nt>-<ACCESSION>_<start>_<end> (repeated), e.g. IS_length_655-IPR002686_154_418-PTHR36966_133_625
- All retained ISs have at least one protein family description containing Transposase, Integrase-like or Ribonuclease H
- Tab-delimited table contains exactly the columns IS_name, sample_id, contig, is_start, is_end, description
- Each accession recorded in IS_name has a corresponding description entry (e.g. IPR013762:Integrase-like, catalytic domain superfamily)
- is_start/is_end coordinates lie on the named contig and their span equals the IS length encoded in IS_name (e.g. 74408–75062 for length 655)

## Limitations

- ISC v1.0.0 only contains common DDE types of ISs with ITRs; transposase-family annotation is only as complete as InterPro/PANTHER signature coverage
- Default chunking of 80000 protein sequences per InterProScan run means very large candidate sets are processed in batches; LSF cluster mode is optional
- The description column only records accessions kept in IS_name, so filtered-out family hits are not reported

## Evidence

- [readme] Get insertion sequences: "Get insertion sequences [`runProdigal`, `installInterproscan`, `runInterproscan`, `getISInfo`]"
- [full_text] Filter step: [evidence span withheld — anchor status: not_found]
- [readme] IS_name interpretation: "represents an IS of nucleotide length 655 with transposases detected including Interpro accession IPR002686 in positions 154-418"
- [full_text] Workflow rationale: [evidence span withheld — anchor status: not_found]
